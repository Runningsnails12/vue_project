<template>
    <el-container class="home-container">
        <el-header>
            <div>
                <span>我的后台</span>
            </div>
            <el-button type="info" @click="logout">退出</el-button>
        </el-header>
        <el-container>
            <!-- 侧边栏 -->
            <el-aside :width="isCollapse?'64px':'200px'">
                <div class="toggle-button" @click="toggleChange">|||</div>
                <el-menu
                    background-color="rgb(245, 245, 246)"
                    text-color="black"
                    active-text-color="#2461be"
                    unique-opened="true"
                    :collapse="isCollapse"
                    :collapse-transition="false"
                    router
                    :default-active="activePath"
                >
                    <!-- 一级菜单 -->
                    <el-submenu :index="'/'+item.path" v-for="item in menulist" :key="item.id">
                        <template slot="title">
                            <i :class="icons[item.id]"></i>
                            <span>{{item.authName}}</span>
                        </template>
                        <!-- 二级菜单 -->
                        <el-menu-item
                            :index="'/'+subItem.path"
                            v-for="subItem in item.children"
                            :key="subItem.id"
                            @click="saveNavState('/'+subItem.path)"
                        >
                            <template slot="title">
                                <i class="el-icon-menu"></i>
                                <span>{{subItem.authName}}</span>
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
            icons: {
                "125": "iconfont icon-user",
                "103": "iconfont icon-tijikongjian",
                "101": "iconfont icon-shangpin",
                "102": "iconfont icon-danju",
                "145": "iconfont icon-baobiao",
            },
            isCollapse: false,
            activePath: "",
        }
    },
    created() {
        this.getMenuList();
        this.activePath = window.sessionStorage.getItem("activePath");
    },
    methods: {
        logout() {
            window.sessionStorage.clear();
            this.$router.push("/login");
        },
        async getMenuList() {
            const { data: res } = await this.$http.get("menus");
            if (res.meta.status != 200) return this.$message.error(res.meta.msg);
            this.menulist = res.data;
        },
        // 菜单折叠
        toggleChange() {
            this.isCollapse = !this.isCollapse;
        },
        saveNavState(path) {
            window.sessionStorage.setItem('activePath', path);
            this.activePath = path;
        },
    }
}
</script>


<style lang="less" scoped>
.home-container {
    height: 100%;
}
.el-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #2461be;
    color: #fff;
    font-size: 24px;
    box-shadow: 0 1px 5px 0 rgba(57, 66, 60, 0.2);

    div {
        display: flex;
        align-items: center;
        span {
            padding-left: 40px;
        }
    }
}
.el-aside {
    background-color: rgb(245, 245, 246);
    .el-menu {
        border-right: 0;
    }
}
.el-main {
    background-color: #fff;
}
.iconfont {
    margin-right: 8px;
}
.toggle-button {
    background-color: #eee;
    font-size: 10px;
    line-height: 24px;
    text-align: center;
    letter-spacing: 0.4em;
    cursor: pointer;
}
</style>