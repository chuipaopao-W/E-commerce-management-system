<template>
<div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
        <el-alert show-icon title="注意！只允许为第三级分类设置相关参数" type="warning" :closable="false">
        </el-alert>
        <el-row class="cat_opt">
            <el-col>
                <span>选择商品分类：</span>
                <el-cascader v-model="selectKeys" :options="catalist" :props="cascaderProps" @change="handleChange" clearable></el-cascader>
            </el-col>
        </el-row>
        <el-tabs v-model="activeName" @tab-click="handleTabClick">
            <el-tab-pane label="动态参数" name="many">
                <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="dialogVisible=true">添加参数</el-button>
                <el-table :data="manyTableData" border stripe>
                    <el-table-column type="expand">
                        <template slot-scope="scope">
                            <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i,scope.row)">{{item}}</el-tag>
                            <el-input class="input-new-tag" v-if="scope.row.inputVisible" v-model="scope.row.inputValue" ref="saveTagInput" size="small" @keyup.enter.native="handleInputConfirm(scope.row)" @blur="handleInputConfirm(scope.row)"></el-input>
                            <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ new tag</el-button>
                        </template>
                    </el-table-column>
                    <el-table-column type=" index">
                    </el-table-column>
                    <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                    <el-table-column label="操作">
                        <template slot-scope="scope">
                            <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                            <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </el-tab-pane>
            <el-tab-pane label="静态属性" name="only">
                <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="dialogVisible=true">添加属性</el-button>
                <el-table :data="onlyTableData" border stripe>
                    <el-table-column type="expand"></el-table-column>
                    <el-table-column type="index"></el-table-column>
                    <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                    <el-table-column label="操作">
                        <template slot-scope="scope">
                            <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                            <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </el-tab-pane>
        </el-tabs>
    </el-card>
    <el-dialog :title="'添加'+ titleText" :visible.sync="dialogVisible" width="50%" @close="closed">
        <el-form :model="ruleForm" :rules="rules" ref="ruleFormRef" label-width="90px">
            <el-form-item :label="titleText" prop="attr_name">
                <el-input v-model="ruleForm.attr_name"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="dialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addParams">确 定</el-button>
        </span>
    </el-dialog>
</div>
</template>

<script>
export default {
    data() {
        return {
            inputVisible: false,
            inputValue: '',
            dialogVisible: false,
            ruleForm: {
                attr_name: "",
            },
            rules: {
                cat_name: [{
                    required: true,
                    message: "请输入分类名称",
                    trigger: "blur",
                }],
            },
            catalist: [],
            cascaderProps: {
                checkStrictly: true,
                expandTrigger: 'hover',
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            selectKeys: [],
            activeName: 'many',
            manyTableData: [],
            onlyTableData: []
        }
        j
    },
    created() {
        this.getCateList()
    },
    methods: {
        //获取商品分类列表
        async getCateList() {
            const {
                data: res
            } =
            await this.$http.get('categories')
            if (res.meta.status !== 200) {
                return this.$message.error('获取商品信息列表失败！')

            }
            //把数据列表赋值给catalist
            this.catalist = res.data
        },
        handleChange() {
            this.getParamsData()
        },
        handleTabClick() {
            this.getParamsData()
        },
        async getParamsData() {
            //当选中的不是三级分类
            if (this.selectKeys.length !== 3) {
                this.selectKeys = []
                return
            } else {
                //根据所选的分类的Id和当前所处的面板，获取对应的参数
                const {
                    data: res
                } =
                await this.$http.get(`categories/${this.cateId}/attributes`, {
                    params: {
                        sel: this.activeName
                    }
                })
                if (res.meta.status !== 200) {
                    return this.$message.error('获取参数列表失败！')
                }
                res.data.forEach(item => {
                    item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
                    //添加一个布尔值控制文本框的显示与隐藏
                    item.inputVisible = false
                    item.inputValue = ''
                })
                console.log(res.data)
                if (this.activeName === 'many') {
                    this.manyTableData = res.data
                } else {
                    this.onlyTableData = res.data
                }
            }

        },
        closed() {
            this.$refs.ruleFormRef.resetFields()
        },
        addParams() {
            this.$refs.ruleFormRef.validate(async (valid) => {
                if (!valid) return;
                //可以发起添加分类的网络请求
                const {
                    data: res
                } = await this.$http.post(`categories/${this.cateId}/attributes`, {
                    attr_name: this.ruleForm.attr_name,
                    attr_sel: this.activeName
                });
                if (res.meta.status !== 201) {
                    this.$message.error("添加分类失败！");
                }
                this.$message.success("添加参数成功");
                //隐藏添加用户的对话框
                this.dialogVisible = false;
                //重新获取用户数据
                this.getParamsData()
            });
        },
        async handleInputConfirm(row) {
            // row.inputVisible = false
            if (row.inputValue.trim().length === 0) {
                row.inputValue = ''
                row.inputVisible = false
                return
            }
            //如果没有返回 则证明输入了内容 需要做添加操作
            row.attr_vals.push(row.inputValue.trim())
            row.inputValue = ''
            row.inputVisible = false
            //接下来进行数据库操作  发起put请求
            const {
                data: res
            } = await this.$http.put(
                `categories/${this.cateId}/attributes/${row.attr_id}`, {
                    attr_name: row.attr_name,
                    attr_sel: row.attr_sel,
                    attr_vals: row.attr_vals.join(' ')
                }
            );
            if (res.meta.status !== 200) {
                return this.$message.error("添加参数失败！");
            }
            this.$message.success("修改成功！")
        },

        showInput(row) {
            row.inputVisible = true
            this.$nextTick(_ => {
                this.$refs.saveTagInput.$refs.input.focus();
            });
        },
        async handleClose(i, row) {
            row.attr_vals.splice(i, 1)
            const {
                data: res
            } = await this.$http.put(
                `categories/${this.cateId}/attributes/${row.attr_id}`, {
                    attr_name: row.attr_name,
                    attr_sel: row.attr_sel,
                    attr_vals: row.attr_vals.join(' ')
                }
            );
            if (res.meta.status !== 200) {
                return this.$message.error("删除参数失败！");
            }
            this.$message.success("删除成功！")
        }
    },
    computed: {
        // 如果按钮需要被禁用 返回true  
        isBtnDisabled() {
            if (this.selectKeys.length !== 3) {
                return true
            }
            return false
        },
        cateId() {
            if (this.selectKeys.length === 3) {
                return this.selectKeys[2]
            }
            return null
        },
        titleText() {
            if (this.activeName === 'many') {
                return '动态参数'
            }
            return '静态属性'
        }

    }
}
</script>

<style lang="less" scoped>
.cat_opt {
    margin: 15px 0;
}

.el-tag {
    margin: 10px;
}

.input-new-tag {
    width: 120px;
}
</style>
