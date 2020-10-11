<template>
<div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>订单管理</el-breadcrumb-item>
        <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
        <el-row>
            <el-col :span="8">
                <el-input placeholder="请输入内容" v-model="queryinfo.query" clearable @clear="getOrdersList">
                    <el-button slot="append" icon="el-icon-search" @click="getOrdersList"></el-button>
                </el-input>
            </el-col>
        </el-row>
        <el-table :data="orderslist" border stripe>
            <el-table-column type="index">
            </el-table-column>
            <el-table-column label="订单编号" prop="order_number"></el-table-column>
            <el-table-column label="订单价格" prop="order_price"></el-table-column>
            <el-table-column label="是否付款" prop="order_pay">
                <template slot-scope="scope">
                    <el-tag type="success" v-if="scope.row.pay_status==='1'">已付款</el-tag>
                    <el-tag type="danger" v-else>未付款</el-tag>
                </template>
            </el-table-column>
            <el-table-column label="是否发货" prop="is_send">
            </el-table-column>
            <el-table-column label="下单时间" prop="create_time">
                <template slot-scope="scope">
                    {{scope.row.create_time | dateFormat}}
                </template>
            </el-table-column>
            <el-table-column label="操作">
                <template slot-scope="scope">
                    <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
                    <el-button type="danger" icon="el-icon-location" size="mini" @click="showProgressBox"></el-button>
                </template>
            </el-table-column>
        </el-table>
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryinfo.pagenum" :page-sizes="[5, 10,15,20]" :page-size="queryinfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total" background>
        </el-pagination>
    </el-card>
    <el-dialog title="物流进度" :visible.sync="progressVisible" width="50%">
        <el-timeline>
            <el-timeline-item v-for="(activity, index) in progressinfo" :key="index" :timestamp="activity.time">
                {{activity.context}}
            </el-timeline-item>
        </el-timeline>
    </el-dialog>
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
            orderslist: [],
            total: 0,
            progressinfo: [],
            progressVisible: false,
        }
    },
    created() {
        this.getOrdersList()
    },
    methods: {
        async getOrdersList() {
            const {
                data: res
            } = await this.$http.get('orders', {
                params: this.queryinfo
            })
            if (res.meta.status !== 200) {
                return this.$message.error('获取订单列表失败！')
            }
            this.$message.success('获取订单列表成功')
            this.orderslist = res.data.goods
            this.total = res.data.total

        },
        handleSizeChange(newSize) {
            this.queryinfo.pagesize = newSize
            this.getOrdersList()
        },
        handleCurrentChange(newPage) {
            this.queryinfo.pagenum = newPage
            this.getOrdersList()
        },
        async showProgressBox() {
            const {
                data: res
            } = await this.$http.get('/kuaidi/1106975712662')
            if (res.meta.status !== 200) {
                return this.$message.error('获取订单列表失败！')
            }
            this.progressinfo = res.data
            this.progressVisible = true
        }

    }
}
</script>

<style lang="less" scoped>

</style>
