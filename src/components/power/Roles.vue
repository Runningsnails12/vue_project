<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>

        <el-card>
            <!-- 添加角色按钮 -->
            <el-row :gutter="10">
                <el-col :span="6">
                    <el-button type="primary">添加角色</el-button>
                </el-col>
            </el-row>

            <el-table :data="rolelist" style="width: 100%" border stripe>
                <!-- 展开 -->
                <el-table-column type="expand">
                    <template slot-scope="scope">
                        <el-row
                            :class="['bdbottom',i1==0?'bdtop':'','vcenter']"
                            v-for="(item1, i1) in scope.row.children"
                            :key="item1.id"
                        >
                            <!-- 一级 -->
                            <el-col :span="5">
                                <el-tag
                                    closable
                                    @close="removeRightById(scope.row,item1.id)"
                                >{{item1.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <!-- 二三级 -->
                            <el-col :span="19">
                                <el-row
                                    :class="[i2==0?'':'bdtop','vcenter']"
                                    v-for="(item2, i2) in item1.children"
                                    :key="item2.id"
                                >
                                    <!-- 二级 -->
                                    <el-col :span="6">
                                        <el-tag
                                            type="success"
                                            closable
                                            @close="removeRightById(scope.row,item2.id)"
                                        >{{item2.authName}}</el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <el-col :span="18">
                                        <el-tag
                                            v-for="(item3) in item2.children"
                                            :key="item3.id"
                                            type="warning"
                                            closable
                                            @close="removeRightById(scope.row,item3.id)"
                                        >{{item3.authName}}</el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <!-- 索引 -->
                <el-table-column type="index"></el-table-column>
                <el-table-column label="角色名称" prop="roleName"></el-table-column>
                <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
                <el-table-column label="操作" width="300px">
                    <template slot-scope="scope">
                        <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
                        <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
                        <el-button
                            size="mini"
                            type="warning"
                            icon="el-icon-setting"
                            @click="showSetRightDiolog(scope.row)"
                        >分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>

        <!-- 分配权限对话框 -->
        <el-dialog
            title="分配"
            :visible.sync="setRightDialogVisible"
            width="50%"
            @close="setRightDialogClosed"
        >
            <!-- 树形控件 -->
            <el-tree
                :data="rightslist"
                :props="treeProps"
                show-checkbox
                node-key="id"
                default-expand-all
                :default-checked-keys="defkeys"
                ref="treeRef"
            ></el-tree>
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRightDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="allotRights">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>
<script>
export default {
    data() {
        return {
            // 角色列表数据
            rolelist: [],
            // 控制分配权限对话框
            setRightDialogVisible: false,
            rightslist: [],
            // 树形控件
            treeProps: {
                label: "authName",
                children: "children",
            },
            defkeys: [],
            roleId: "",
        }
    },
    created() {
        this.getRolesList();
    },
    methods: {
        async getRolesList() {
            const { data: res } = await this.$http.get("roles");
            if (res.meta.status != 200) {
                return this.$message.error("获取角色列表失败！");
            }
            this.rolelist = res.data;
        },
        // 根据id删除
        async removeRightById(role, rightId) {
            const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err);
            if (confirmResult != 'confirm') {
                return this.$message.info("取消了删除");
            }
            const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`);
            if (res.meta.status != 200) {
                return this.$message.error("删除权限失败!");
            }
            role.children = res.data;
        },
        async showSetRightDiolog(role) {
            this.roleId = role.id;
            const { data: res } = await this.$http.get("rights/tree")
            if (res.meta.status != 200) {
                return this.$message.error("获取权限数据失败！");
            }
            // 权限数据保存
            this.rightslist = res.data;
            this.getLeafKeys(role, this.defkeys);

            this.setRightDialogVisible = true;

        },
        getLeafKeys(node, arr) {
            if (!node.children) {
                return arr.push(node.id);
            }
            node.children.forEach(item => {
                this.getLeafKeys(item, arr);
            })
        },
        setRightDialogClosed() {
            this.defkeys = [];
        },
        async allotRights() {
            const keys = [
                ...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()
            ]
            const idStr = keys.join(',');
            console.log(idStr);

            const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
            if (res.meta.status != 200) {
                return this.$message.error("分配权限失败！");
            }
            this.$message.success("分配权限成功！");
            this.getRolesList();
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
    border-top: 1px solid #eee;
}
.bdbottom {
    border-bottom: 1px solid #eee;
}
.vcenter {
    display: flex;
    align-items: center;
}
</style>