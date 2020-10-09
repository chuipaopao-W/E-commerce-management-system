<template>
<div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
        <el-row>
            <el-col>
                <el-button type="primary" @click="showDialogVisible">添加分类 </el-button>
            </el-col>
        </el-row>
        <tree-table class="treeTable" :data="catalist" :columns="columns" :expand-type="false" :selection-type="false" show-index index-text="#" border>
            <template slot="isok" slot-scope="scope">
                <i class="el-icon-success" v-if="scope.row.cat_deleted===false" style="color:lightgreen;"></i>
                <i class="el-icon-error" v-else style="color:red;"></i>
            </template> <template slot="order" slot-scope="scope">
                <el-tag size="mini" v-if="scope.row.cat_level===0">一级</el-tag>
                <el-tag type="success" size="mini" v-else-if="scope.row.cat_level===1">二级</el-tag>
                <el-tag type="warning" size="mini" v-else>三级</el-tag>
            </template>
            <template slot="opt" slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
            </template>
        </tree-table>
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum" :page-sizes="[1, 2, 5, 10]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
        </el-pagination>
    </el-card>
    <el-dialog title="添加分类" :visible.sync="dialogVisible" width="50%" @close="addCateClosed">
        <el-form :model="ruleForm" :rules="rules" ref="ruleFormRef" label-width="90px">
            <el-form-item label="分类名称" prop="cat_name">
                <el-input v-model="ruleForm.cat_name"></el-input>
            </el-form-item>
            <el-form-item label="父级分类">
                <el-cascader v-model=" selectKeys" :options="parentCateList" :props="cascaderProps" @change="handleChange" clearable></el-cascader>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="dialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addCate">确 定</el-button>
        </span>
    </el-dialog>
</div>
</template>

<script>
export default {
    data() {
        return {
            dialogVisible: false,
            ruleForm: {
                cat_pid: 0,
                cat_name: "",
                //默认添加的为一级分类
                cat_level: 0,
            },
            //父级分类的列表
            parentCateList: [],
            //选中的父级分类的id数组
            selectKeys: [],
            //指定级联选择器的配置对象
            cascaderProps: {
                checkStrictly: true,
                expandTrigger: 'hover',
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            rules: {
                roleName: [{
                        required: true,
                        message: "请输入分类名称",
                        trigger: "blur",
                    },
                    {
                        message: "长度最好在 3 到 10 个字符",
                        trigger: "blur",
                    },
                ],
                roleDesc: [{
                        required: true,
                        message: "请输入父级描述",
                        trigger: "blur",
                    },
                    {
                        message: "最好描述的详细一点",
                        trigger: "blur",
                    },
                ],
            },
            //查询参数
            queryInfo: {
                type: 3,
                pagenum: 1,
                pagesize: 5
            },
            //商品分类的数据列表
            catalist: [],
            //总数据条数
            total: 0,
            //为table指定列的定义分类名称 
            columns: [{
                    label: '分类名称 ',
                    prop: 'cat_name'
                }, {
                    label: '是否有效',

                    //表示将当前列定义为模板列
                    type: 'template',
                    //表示当前这一列使用的模板名称
                    template: 'isok'
                },
                {
                    label: '排序',

                    //表示将当前列定义为模板列
                    type: 'template',
                    //表示当前这一列使用的模板名称
                    template: 'order'
                },
                {
                    label: '操作',

                    //表示将当前列定义为模板列
                    type: 'template',
                    //表示当前这一列使用的模板名称
                    template: 'opt'
                },
            ]
        }
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
            await this.$http.get('categories', {
                params: this.queryInfo
            })
            if (res.meta.status !== 200) {
                return this.$message.error('获取商品信息列表失败！')

            }
            //把数据列表赋值给catalist
            this.catalist = res.data.result
            //为总数据条数赋值
            this.total = res.data.total
        },
        addCate() {
            this.$refs.ruleFormRef.validate(async (valid) => {
                if (!valid) return;
                //可以发起添加分类的网络请求
                const {
                    data: res
                } = await this.$http.post("categories", this.ruleForm);
                if (res.meta.status !== 201) {
                    this.$message.error("添加分类失败！");
                }
                this.$message.success("添加分类成功");
                //隐藏添加用户的对话框
                this.dialogVisible = false;
                //重新获取用户数据
                this.getCateList();
            });
        },
        showDialogVisible() {
            //先获取父级分类的数据列表 再展示出对话框
            this.getParentCateList()
            this.dialogVisible = true;
        },
        async getParentCateList() {
            const {
                data: res
            } = await this.$http.get("categories", {
                params: [{
                    type: 2
                }]
            });
            if (res.meta.status !== 200) {
                this.$message.error("获取父级分类失败！");
            }
            // this.$message.success("获取父级分类成功");
            this.parentCateList = res.data
        },
        handleSizeChange(newSize) {
            this.queryInfo.pagesize = newSize;
            this.getCateList();
        },
        handleCurrentChange(newPage) {
            this.queryInfo.pagenum = newPage;
            this.getCateList();
        },
        //选择项发生变化触发这个函数
        handleChange() {
            if (this.selectKeys.length > 0) {
                this.ruleForm.cat_pid = this.selectKeys[this.selectKeys.length - 1]
                this.ruleForm.cat_level = this.selectKeys.length
                return
            } else {
                this.ruleForm.cat_pid = 0
                this.ruleForm.cat_level = 0
            }
        },
        addCateClosed() {
            this.$refs.ruleFormRef.resetFields()
            this.selectKeys = []
            this.ruleForm.cat_level = 0
            this.ruleForm.cat_pid = 0
        }
    },

}
</script>

<style lang="less" scoped>
.treeTable {
    margin-top: 15px;
}

.el-cascader {
    width: 100%;
}
</style>
