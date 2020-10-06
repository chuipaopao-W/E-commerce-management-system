<template>
<el-container class="home-container">
    <el-header>
        <div>
            <img src="../assets/heima.png" alt="">
            <span>电商后台管理系统</span>
        </div>
        <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <el-container>
        <el-aside :width="iscollapse?'64px':'200px'">
            <div class=" zhankaiBtn" @click="zhankai">
                |||</div>
            <el-menu background-color="#333744" text-color="#fff" active-text-color="#409eff" :unique-opened="true" :collapse='iscollapse' :collapse-transition="false" :router="true" :default-active="activePath">
                <el-submenu :index="item.id.toString()" v-for="item in menulist" :key="item.id">
                    <template slot="title">
                        <i :class="obj[item.id]"></i>
                        <span>{{item.authName}}</span>
                    </template>
                    <el-menu-item :index="'/'+ subitem.path" v-for="subitem in item.children" :key="subitem.id" @click="saveActive('/'+ subitem.path)">
                        <template slot="title">
                            <i class="el-icon-menu"></i>
                            <span>{{subitem.authName}}</span>
                        </template>
                    </el-menu-item>
                </el-submenu>
            </el-menu>
        </el-aside>
        <el-main>
            <router-view></router-view>
        </el-main>
    </el-container>
</el-container>
</template>

<script>
export default {
    data() {
        return {
            menulist: [],
            obj: {
                '125': "iconfont icon-users",
                '103': "iconfont icon-tijikongjian",
                '101': "iconfont icon-shangpin",
                '102': "iconfont icon-danju",
                '145': "iconfont icon-baobiao"
            },
            iscollapse: false,
            activePath: ''

        }

    },

    created() {
        this.getMenulist()
        this.activePath = window.sessionStorage.getItem('activePath')
    },
    methods: {
        logout() {
            window.sessionStorage.clear();
            this.$router.push("/login")
        },
        async getMenulist() {
            const {
                data: res
            } =
            await this.$http.get('menus')
            if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
            this.menulist = res.data
        },
        zhankai() {
            this.iscollapse = !this.iscollapse
        },
        saveActive(activePath) {
            window.sessionStorage.setItem('activePath', activePath)
            this.activePath = activePath
        }
    }
};
</script>

<style lang="less" scoped>
.home-container {
    height: 100%;
}

.el-header {
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    padding-left: 0;
    align-items: center;
    color: #fff;
    font-size: 20px;

    >div {
        display: flex;
        align-items: center;

        span {
            margin-left: 15px;
        }
    }
}

.el-aside {
    background-color: #333744;

    .el-menu {
        border-right: 0;
    }
}

.el-main {
    background-color: #eaedf1;
}

.zhankaiBtn {
    background-color: #4A5064;
    font-size: 10px;
    line-height: 24px;
    color: #fff;
    text-align: center;
    letter-spacing: 0.5px;
    cursor: pointer;
}
</style>
