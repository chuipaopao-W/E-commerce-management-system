<template>
<div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
        <el-row :gutter="20">
            <el-col :span="8">
                <el-input placeholder="请输入内容" v-model="queryinfo.query" clearable @clear="getGoodsList">
                    <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
                </el-input>
            </el-col>
            <el-col :span="4">
                <el-button type="primary"> 添加商品</el-button>
            </el-col>
        </el-row>
        <el-table :data="goodslist" border stripe>
            <el-table-column type="index">
            </el-table-column>
            <el-table-column label="商品名称" prop="goods_name"></el-table-column>
            <el-table-column label="商品价格" prop="goods_price"></el-table-column>
            <el-table-column label="商品重量" prop="goods_weight"></el-table-column>
            <el-table-column label="创建时间" prop="add_time">
                <template slot-scope="scope">
                    {{scope.row.add_time | dateFormat}}
                </template>
            </el-table-column>
            <el-table-column label="操作">
                <template slot-scope="scope">
                    <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
                    <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
                </template>
            </el-table-column>
        </el-table>
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryinfo.pagenum" :page-sizes="[5, 10,15,20]" :page-size="queryinfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total" background>
        </el-pagination>
    </el-card>
</div>
</template>

<script>
export default {
    data() {
        return {
            queryinfo: {
                query: '',
                pagenum: 1,
                pagesize: 10
            },
            goodslist: [],
            total: 0
        }
    },
    created() {
        this.getGoodsList()
    },
    methods: {
        async getGoodsList() {
            const {
                data: res
            } = await this.$http.get('goods', {
                params: this.queryinfo
            })
            if (res.meta.status !== 200) {
                return this.$message.error('获取商品列表失败！')
            }
            this.$message.success('获取商品列表成功')
            this.goodslist = res.data.goods
            this.total = res.data.total
        },
        handleSizeChange(newSize) {
            this.queryinfo.pagesize = newSize
            this.getGoodsList()
        },
        handleCurrentChange(newPage) {
            this.queryinfo.pagenum = newPage
            this.getGoodsList()
        }
    }
}
</script>

<style lang="less" scoped>

</style>
