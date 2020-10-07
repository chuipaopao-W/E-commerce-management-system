<template>
<div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
        <el-row>
            <el-col>
                <el-button type="primary" @click="dialogVisible = true">添加角色</el-button>
            </el-col>
        </el-row>
        <!--角色列表区域-->
        <el-table :data="roleList" border stripe>
            <!--展开列-->
            <el-table-column type="expand">
                <template slot-scope="scope">
                    <el-row :class="['bdbottom',i1===0?'bdtop':'','vcen']" v-for=" (item1, i1) in scope.row.children" :key="item1.id">
                        <!--渲染一级权限-->
                        <el-col :span="5">
                            <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                            <i class="el-icon-caret-right"></i>
                        </el-col>
                        <!--渲染二级和三级权限-->
                        <el-col :span="19">
                            <el-row :class="[i2===0?'':'bdtop','vcen']" v-for="(item2, i2) in item1.children" :key="item2.id">
                                <el-col :span="6">
                                    <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                                    <i class="el-icon-caret-right"></i>
                                </el-col>
                                <el-col :span="18">
                                    <el-tag type="warning" v-for="(item3, i3) in item2.children" :key="item3.id" closable @close="removeRightById(scope.row,item3.id)">{{item3.authName}}</el-tag>
                                </el-col>
                            </el-row>
                        </el-col>
                    </el-row>
                </template>
            </el-table-column>

            <!--索引列-->
            <el-table-column type="index"> </el-table-column>
            <el-table-column prop="roleName" label="角色名称"> </el-table-column>
            <el-table-column prop="roleDesc" label="角色描述"> </el-table-column>
            <el-table-column label="操作" width="300px">
                <template slot-scope="scope">
                    <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEdit(scope.row.id)">编辑</el-button>
                    <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)">删除</el-button>
                    <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRight(scope.row)">分配权限</el-button>
                </template>
            </el-table-column>
        </el-table>
    </el-card>
    <el-dialog title="添加角色" :visible.sync="dialogVisible" width="50%" @close="addClosed">
        <el-form :model="ruleForm" :rules="rules" ref="ruleFormRef" label-width="90px">
            <el-form-item label="角色名称" prop="roleName">
                <el-input v-model="ruleForm.roleName"></el-input>
            </el-form-item>
            <el-form-item label="角色描述" prop="roleDesc">
                <el-input v-model="ruleForm.roleDesc"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="dialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addUser">确 定</el-button>
        </span>
    </el-dialog>
    <el-dialog title="修改角色信息" :visible.sync="editDialogVisible" width="50%" @close="editClosed">
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="90px">
            <el-form-item label="角色名称" prop="roleName">
                <el-input v-model="editForm.roleName"></el-input>
            </el-form-item>
            <el-form-item label="角色描述" prop="roleDesc">
                <el-input v-model="editForm.roleDesc"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editUser">确 定</el-button>
        </span>
    </el-dialog>
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightClosed()">
        <!--树形控件-->
        <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defkeys" ref="treeRef"></el-tree>
        <span slot="footer" class="dialog-footer">
            <el-button @click="setRightDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="allotright">确 定</el-button>
        </span>
    </el-dialog>
</div>
</template>

<script>
export default {
    data() {
        return {
            roleList: [],
            rightsList: [],
            defkeys: [],
            //当前即将分配权限的角色id
            roleId: '',
            treeProps: {
                label: 'authName',
                children: 'children'
            },
            dialogVisible: false,
            editDialogVisible: false,
            setRightDialogVisible: false,
            ruleForm: {
                roleName: "",
                roleDesc: "",
            },
            editForm: {},
            rules: {
                roleName: [{
                        required: true,
                        message: "请输入角色名称",
                        trigger: "blur",
                    },
                    {
                        message: "长度最好在 3 到 10 个字符",
                        trigger: "blur",
                    },
                ],
                roleDesc: [{
                        required: true,
                        message: "请输入角色描述",
                        trigger: "blur",
                    },
                    {
                        message: "最好描述的详细一点",
                        trigger: "blur",
                    },
                ],
            },
            editFormRules: {
                roleName: [{
                        required: true,
                        message: "请输入角色名称",
                        trigger: "blur",
                    },
                    {
                        message: "长度最好在 3 到 10 个字符",
                        trigger: "blur",
                    },
                ],
                roleDesc: [{
                        required: true,
                        message: "请输入角色描述",
                        trigger: "blur",
                    },
                    {
                        message: "最好描述的详细一点",
                        trigger: "blur",
                    },
                ],
            }
        }
    },
    created() {
        this.getRolesList()
    },
    methods: {
        async getRolesList() {
            const {
                data: res
            } =
            await this.$http.get('roles')
            if (res.meta.status !== 200) {
                return this.$message.error('获取角色列表失败！')
            }
            this.roleList = res.data
        },
        //监听对话框关闭事件
        addClosed() {
            this.$refs.ruleFormRef.resetFields();
        },
        editClosed() {
            this.$refs.editFormRef.resetFields();
        },
        //点击按钮 添加新角色  预校验
        addUser() {
            this.$refs.ruleFormRef.validate(async (valid) => {
                if (!valid) return;
                //可以发起添加用户的网络请求
                const {
                    data: res
                } = await this.$http.post("roles", this.ruleForm);
                if (res.meta.status !== 201) {
                    this.$message.error("添加角色失败！");
                }
                this.$message.success("添加角色成功");
                //隐藏添加用户的对话框
                this.dialogVisible = false;
                //重新获取用户数据
                this.getRolesList();
            });
        },
        //展示修改用户的对话框
        async showEdit(id) {
            const {
                data: res
            } = await this.$http.get("roles/" + id);
            if (res.meta.status !== 200) {
                return this.$message.error("查询用户失败！");
            }
            this.editForm = res.data;
            this.editDialogVisible = true;
        },
        editUser() {
            this.$refs.editFormRef.validate(async (valid) => {
                if (!valid) return;
                //可以发起添加用户的网络请求
                const {
                    data: res
                } = await this.$http.put(
                    "roles/" + this.editForm.roleId, {
                        roleName: this.editForm.roleName,
                        roleDesc: this.editForm.roleDesc,
                    }
                );
                console.log(res)
                if (res.meta.status !== 200) {
                    return this.$message.error("更新角色信息失败！");
                }
                //隐藏修改用户的对话框
                this.editDialogVisible = false;
                //重新获取用户数据
                this.getRolesList();
                //提示修改成功
                this.$message.success("更新角色信息成功");
            });
        },
        //根据id删除对应的角色
        async removeUserById(id) {
            //弹框询问是否删除
            const result = await this.$confirm(
                "此操作将永久删除该角色, 是否继续?",
                "提示", {
                    confirmButtonText: "确定",
                    cancelButtonText: "取消",
                    type: "warning",
                }
            ).catch((err) => err); //用catch来捕获取消操作 返回值为cancel
            //如果角色确认删除则返回值为confirm
            if (result !== 'confirm') {
                return this.$message.info('已经取消了删除')
            }
            const {
                data: res
            } =
            await this.$http.delete('roles/' + id)
            if (res.meta.status !== 200) {
                return this.$message.error("删除角色失败！");
            }
            //重新获取角色数据
            this.getRolesList();
            //提示修改成功
            this.$message.success("删除角色成功");
        },
        //根据id删除对应权限
        async removeRightById(role, rightId) {
            //弹框提示是否删除对应权限
            const result = await this.$confirm(
                "此操作将永久删除该角色, 是否继续?",
                "提示", {
                    confirmButtonText: "确定",
                    cancelButtonText: "取消",
                    type: "warning",
                }
            ).catch((err) => err); //用catch来捕获取消操作 返回值为cancel
            //如果角色确认删除则返回值为confirm
            if (result !== 'confirm') {
                return this.$message.info('已经取消了删除')
            }
            const {
                data: res
            } =
            await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
            if (res.meta.status !== 200) {
                return this.$message.error("删除权限失败！");
            }
            //重新获取角色数据
            // this.getRolesList();
            //上面的方法不太合理 直接重新付给role 这样不用重新渲染权限列表 提高了用户体验
            role.children = res.data
            //提示修改成功
            this.$message.success("删除权限成功");
        },
        //展示分配权限对话框
        async showSetRight(role) {
            this.roleId = role.id
            //获取所有权限数据
            const {
                data: res
            } =
            await this.$http.get('rights/tree')
            if (res.meta.status !== 200) {
                return this.$message.error('获取权限数据失败！')
            }
            this.rightsList = res.data
            this.getLeafKeys(role, this.defkeys)
            this.setRightDialogVisible = true
        },
        //通过递归的形式获取角色下所有三级权限的id，并保存到defkeys数组中
        getLeafKeys(node, arr) {
            if (!node.children) {
                return arr.push(node.id)
            }
            node.children.forEach(item => this.getLeafKeys(item, arr))
        },
        //监听分配权限对话框关闭事件
        setRightClosed() {
            this.defkeys = []
        },
        //点击为角色添加权限
        async allotright() {
            const keys = [
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()
            ]
            const idStr = keys.join(',')
            const {
                data: res
            } =
            await this.$http.post(`roles/${this.roleId}/rights`, {
                rids: idStr
            })
            if (res.meta.status !== 200) {
                return this.$message.error("分配权限失败！")

            }
            //console.log(res.data) //无响应数据
            this.$message.success('分配权限成功！')
            this.getRolesList()
            this.setRightDialogVisible = false;
        }
    }
}
</script>

<style lang="less" scoped>
.el-tag {
    margin: 7px;
}

.bdtop {
    border-top: 1px solid #eeeeee
}

.bdbottom {
    border-bottom: 1px solid #eee;
}

.vcen {
    display: flex;
    align-items: center;
}
</style>
