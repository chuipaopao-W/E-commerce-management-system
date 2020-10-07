<template>
<div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>用户管理</el-breadcrumb-item>
        <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card class="box-card">
        <div slot="header" class="clearfix">
            <span>用户列表</span>
            <el-button style="float: right; padding: 3px 0" type="text">操作按钮</el-button>
        </div>
        <el-row :gutter="40">
            <el-col :span="8">
                <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
                    <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
                </el-input>
            </el-col>
            <el-col :span="4">
                <el-button type="primary" @click="dialogVisible = true">添加用户</el-button>
            </el-col>
        </el-row>
        <el-table :data="userlist" style="width: 100%" border stripe>
            <el-table-column type="index"> </el-table-column>
            <el-table-column prop="username" label="姓名"> </el-table-column>
            <el-table-column prop="email" label="邮箱"> </el-table-column>
            <el-table-column prop="mobile" label="电话"> </el-table-column>
            <el-table-column prop="role_name" label="角色"> </el-table-column>
            <el-table-column label="状态">
                <template slot-scope="scope">
                    <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)">
                    </el-switch>
                </template>
            </el-table-column>
            <el-table-column label="操作">
                <template slot-scope="scope">
                    <el-button type="primary" icon="el-icon-edit" size="mini" circle @click="showEdit(scope.row.id)"></el-button>
                    <el-button type="danger" icon="el-icon-delete" size="mini" circle @click="removeUserById(scope.row.id)"></el-button>
                    <el-tooltip effect="dark" content="分配角色" placement="top">
                        <el-button type="warning" icon="el-icon-setting" size="mini" circle @click="setRole(scope.row)">
                        </el-button>
                    </el-tooltip>
                </template>
            </el-table-column>
        </el-table>
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum" :page-sizes="[1, 2, 5, 10]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
        </el-pagination>
    </el-card>

    <el-dialog title="添加用户" :visible.sync="dialogVisible" width="50%" @close="addClosed">
        <el-form :model="ruleForm" :rules="rules" ref="ruleFormRef" label-width="70px">
            <el-form-item label="用户名" prop="username">
                <el-input v-model="ruleForm.username"></el-input>
            </el-form-item>
            <el-form-item label="密码" prop="password">
                <el-input v-model="ruleForm.password"></el-input>
            </el-form-item>
            <el-form-item label="邮箱" prop="email">
                <el-input v-model="ruleForm.email"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="dialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addUser">确 定</el-button>
        </span>
    </el-dialog>
    <el-dialog title="修改用户信息" :visible.sync="editDialogVisible" width="50%" @close="editClosed">
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
            <el-form-item label="用户名">
                <el-input v-model="editForm.username" disabled></el-input>
            </el-form-item>
            <el-form-item label="邮箱" prop="email">
                <el-input v-model="editForm.email"></el-input>
            </el-form-item>
            <el-form-item label="手机" prop="mobile">
                <el-input v-model="editForm.mobile"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editUser">确 定</el-button>
        </span>
    </el-dialog>
    <el-dialog title="分配角色" :visible.sync="setRoleDialogVisible" width="50%" @close=setRoleDialogClosed>
        <div>
            <p>当前的用户：{{userInfo.username}}</p>
            <p>当前的角色：{{userInfo.role_name}}</p>
            <p>分配新角色：
                <el-select v-model="selectedRoleId" placeholder="请选择">
                    <el-option v-for="item in rolesList" :key="item.id" :label="item.roleName" :value="item.id">
                    </el-option>
                </el-select>
            </p>
        </div>
        <span slot="footer" class="dialog-footer">
            <el-button @click="setRoleDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
        </span>
    </el-dialog>
</div>
</template>

<script>
export default {
    data() {
        let checkEmail = (rule, value, cb) => {
            const regEmail = /^[A-Za-z0-9]+([_\.][A-Za-z0-9]+)*@([A-Za-z0-9\-]+\.)+[A-Za-z]{2,6}$/;
            if (regEmail.test(value)) {
                return cb();
            }
            cb(new Error("请输入合法的邮箱！"));
        };
        let checkMobile = (rule, value, cb) => {
            const regMobile = /^[1]([3-9])[0-9]{9}$/;
            if (regMobile.test(value)) {
                return cb();
            }
            cb(new Error("请输入合法的手机号！"));
        };
        return {
            queryInfo: {
                query: "",
                pagesize: 1,
                pagenum: 1,
            },
            userlist: [],
            //所有角色的数据列表
            rolesList: [],
            //已选中的角色
            selectedRoleId: '',
            total: 0,
            dialogVisible: false,
            editDialogVisible: false,
            setRoleDialogVisible: false,
            //需要被分配角色的用户信息
            userInfo: {},
            ruleForm: {
                username: "",
                password: "",
                email: "",
                mobile: "",
            },
            editForm: {},
            rules: {
                username: [{
                        required: true,
                        message: "请输入用户名",
                        trigger: "blur",
                    },
                    {
                        min: 3,
                        max: 10,
                        message: "长度在 3 到 10 个字符",
                        trigger: "blur",
                    },
                ],
                password: [{
                        required: true,
                        message: "请输入密码",
                        trigger: "blur",
                    },
                    {
                        min: 6,
                        max: 10,
                        message: "长度在 6 到 10 个字符",
                        trigger: "blur",
                    },
                ],
                email: [{
                        required: true,
                        message: "请输入邮箱",
                        trigger: "blur",
                    },
                    {
                        validator: checkEmail,
                        trigger: "blur",
                    },
                ],
                mobile: [{
                        required: true,
                        message: "请输入手机号",
                        trigger: "blur",
                    },
                    {
                        validator: checkMobile,
                        trigger: "blur",
                    },
                ],
            },
            editFormRules: {
                email: [{
                        required: true,
                        message: "请输入邮箱",
                        trigger: "blur",
                    },
                    {
                        validator: checkEmail,
                        trigger: "blur",
                    },
                ],
                mobile: [{
                        required: true,
                        message: "请输入手机号",
                        trigger: "blur",
                    },
                    {
                        validator: checkMobile,
                        trigger: "blur",
                    },
                ],
            },
        };
    },
    created() {
        this.getUserList();
    },
    methods: {
        async getUserList() {
            const {
                data: res
            } = await this.$http.get("users", {
                params: this.queryInfo,
            });
            if (res.meta.status !== 200) {
                return this.$message.error("获取用户列表失败！");
            }
            this.userlist = res.data.users;
            this.total = res.data.total;
        },
        handleSizeChange(newSize) {
            this.queryInfo.pagesize = newSize;
            this.getUserList();
        },
        handleCurrentChange(newPage) {
            this.queryInfo.pagenum = newPage;
            this.getUserList();
        },
        async userStateChanged(userinfo) {
            const {
                data: res
            } = await this.$http.put(
                `users/${userinfo.id}/state/${userinfo.mg_state}`
            );
            if (res.meta.status !== 200) {
                userinfo.mg_state = !userinfo.mg_state;
                return this.$message.error("更新用户状态失败！");
            }
            this.$message.success("更新用户状态成功！");
        },
        //监听对话框关闭事件
        addClosed() {
            this.$refs.ruleFormRef.resetFields();
        },
        editClosed() {
            this.$refs.editFormRef.resetFields();
        },
        //点击按钮 添加新用户  预校验
        addUser() {
            this.$refs.ruleFormRef.validate(async (valid) => {
                if (!valid) return;
                //可以发起添加用户的网络请求
                const {
                    data: res
                } = await this.$http.post("users", this.ruleForm);
                if (res.meta.status !== 201) {
                    this.$message.error("添加用户失败！");
                }
                this.$message.success("添加用户成功");
                //隐藏添加用户的对话框
                this.dialogVisible = false;
                //重新获取用户数据
                this.getUserList();
            });
        },
        //展示修改用户的对话框
        async showEdit(id) {
            const {
                data: res
            } = await this.$http.get("users/" + id);
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
                    "users/" + this.editForm.id, {
                        email: this.editForm.email,
                        mobile: this.editForm.mobile,
                    }
                );
                if (res.meta.status !== 200) {
                    return this.$message.error("更新用户信息失败！");
                }
                //隐藏修改用户的对话框
                this.editDialogVisible = false;
                //重新获取用户数据
                this.getUserList();
                //提示修改成功
                this.$message.success("更新用户信息成功");
            });
        },
        //根据id删除对应的用户
        async removeUserById(id) {
            //弹框询问是否删除
            const result = await this.$confirm(
                "此操作将永久删除该用户, 是否继续?",
                "提示", {
                    confirmButtonText: "确定",
                    cancelButtonText: "取消",
                    type: "warning",
                }
            ).catch((err) => err); //用catch来捕获取消操作 返回值为cancel
            //如果用户确认删除则返回值为confirm
            if (result !== 'confirm') {
                return this.$message.info('已经取消了删除')
            }
            const {
                data: res
            } =
            await this.$http.delete('users/' + id)
            if (res.meta.status !== 200) {
                return this.$message.error("删除用户失败！");
            }
            //重新获取用户数据
            this.getUserList();
            //提示修改成功
            this.$message.success("删除用户成功");
        },
        //展示分配角色的对话框
        async setRole(userInfo) {
            this.userInfo = userInfo
            //展示对话框之前， 获取所有的角色列表
            const {
                data: res
            } = await this.$http.get('roles');
            if (res.meta.status !== 200) {
                return this.$message.error("获取角色列表失败！");
            }
            this.rolesList = res.data
            this.setRoleDialogVisible = true
        },
        async saveRoleInfo() {
            if (!this.selectedRoleId) {
                return this.$message.error('请选择要分配的角色！')
            }
            const {
                data: res
            } = await this.$http.put(`users/${this.userInfo.id}/role`, {
                rid: this.selectedRoleId
            })
            console.log(res.data)
            console.log(this.userInfo.id)
            console.log(this.selectedRoleId)
            if (res.meta.status !== 200) {
                return this.$message.error("更新角色失败！");
            }
            this.$message.success('更新角色成功！')
            this.getUserList()
            this.setRoleDialogVisible = false
        },
        setRoleDialogClosed() {
            this.selectedRoleId = ''
            this.userInfo = {}
        }
    },
};
</script>

<style lang='less' scoped>
</style>
