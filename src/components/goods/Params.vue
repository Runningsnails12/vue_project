<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>参数列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <el-alert title="注意！只允许设置第三级分类" type="warning" :closable="false" show-icon></el-alert>
            <!-- 选择商品分类 -->
            <el-row class="cat_opt">
                <el-col>
                    <span>选择商品分类:</span>
                    <!-- 选择框 -->
                    <el-cascader
                        expand-trigger="hover"
                        :options="cateList"
                        :props="cateProps"
                        v-model="selectedCateKeys"
                        @change="handleChange"
                    ></el-cascader>
                </el-col>
            </el-row>
            <el-tabs v-model="activeName" @tab-click="handleTabClick">
                <el-tab-pane label="动态参数" name="many">
                    <el-button
                        @click="addDialogVisible=true"
                        type="primary"
                        size="mini"
                        :disabled="isBtnDisabled"
                    >添加参数</el-button>
                    <!-- 表格 -->
                    <el-table :data="manyTableData" style="width: 100%" border stripe>
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <el-tag
                                    v-for="(item, i) in scope.row.attr_vals"
                                    :key="i"
                                    closable
                                    @close="handleClose(i, scope.row)"
                                >{{item}}</el-tag>
                                <el-input
                                    class="input-new-tag"
                                    v-if="scope.row.inputVisible"
                                    v-model="scope.row.inputValue"
                                    ref="saveTagInput"
                                    size="small"
                                    @keyup.enter.native="handleInputConfirm(scope.row)"
                                    @blur="handleInputConfirm(scope.row)"
                                ></el-input>
                                <el-button
                                    v-else
                                    class="button-new-tag"
                                    size="small"
                                    @click="showInput(scope.row)"
                                >+ New Tag</el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index"></el-table-column>
                        <el-table-column prop="attr_name" label="参数名称" width="width"></el-table-column>
                        <el-table-column label="操作" width="width">
                            <template slot-scope="scope">
                                <el-button
                                    @click="showEditDialog(scope.row.attr_id)"
                                    size="mini"
                                    type="primary"
                                    icon="el-icon-edit"
                                >编辑</el-button>
                                <el-button
                                    @click="removeParams(scope.row.attr_id)"
                                    size="mini"
                                    type="danger"
                                    icon="el-icon-delete"
                                >删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
                <el-tab-pane label="静态属性" name="only">
                    <el-button
                        @click="addDialogVisible=true"
                        type="primary"
                        size="mini"
                        :disabled="isBtnDisabled"
                    >添加属性</el-button>
                    <!-- 表格 -->
                    <el-table :data="onlyTableData" style="width: 100%" border stripe>
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <el-tag
                                    v-for="(item, i) in scope.row.attr_vals"
                                    :key="i"
                                    closable
                                    @close="handleClose(i, scope.row)"
                                >{{item}}</el-tag>
                                <el-input
                                    class="input-new-tag"
                                    v-if="scope.row.inputVisible"
                                    v-model="scope.row.inputValue"
                                    ref="saveTagInput"
                                    size="small"
                                    @keyup.enter.native="handleInputConfirm(scope.row)"
                                    @blur="handleInputConfirm(scope.row)"
                                ></el-input>
                                <el-button
                                    v-else
                                    class="button-new-tag"
                                    size="small"
                                    @click="showInput(scope.row)"
                                >+ New Tag</el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index"></el-table-column>
                        <el-table-column prop="attr_name" label="属性名称" width="width"></el-table-column>
                        <el-table-column label="操作" width="width">
                            <template slot-scope="scope">
                                <el-button
                                    @click="showEditDialog(scope.row.attr_id)"
                                    size="mini"
                                    type="primary"
                                    icon="el-icon-edit"
                                >编辑</el-button>
                                <el-button
                                    @click="removeParams(scope.row.attr_id)"
                                    size="mini"
                                    type="danger"
                                    icon="el-icon-delete"
                                >删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
            </el-tabs>
        </el-card>

        <!-- 添加参数对话框 -->
        <el-dialog
            :title="'添加'+titleText"
            :visible.sync="addDialogVisible"
            width="50%"
            :before-close="dialogBeforeClose"
            @close="addDialogClosed"
        >
            <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="80px">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="addForm.attr_name"></el-input>
                </el-form-item>
            </el-form>

            <div></div>
            <div slot="footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addParams">确 定</el-button>
            </div>
        </el-dialog>
        <!-- 修改参数的对话框 -->
        <el-dialog
            :title="'修改' + titleText"
            :visible.sync="editDialogVisible"
            width="50%"
            @close="editDialogClosed"
        >
            <!-- 添加参数的对话框 -->
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="editForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editParams">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    data() {
        return {
            cateList: [],
            // 选择框配置
            cateProps: {
                value: "cat_id",
                label: "cat_name",
                children: "children"
            },
            // 选择框绑定数组
            selectedCateKeys: [],
            // 被激活页签
            activeName: "many",
            // 动态参数
            manyTableData: [],
            // 静态属性
            onlyTableData: [],
            addDialogVisible: false,
            addForm: {
                attr_name: ""
            },
            addFormRules: {
                attr_name: [{ required: true, message: '请输入参数名称', trigger: 'blur' }]
            },
            editDialogVisible: false,
            editForm: {},
            editFormRules: {
                attr_name: [
                    { required: true, message: '请输入参数名称', trigger: 'blur' }
                ]
            },

        }
    },
    created() {
        this.getCateList();
    },
    methods: {
        async getCateList() {
            const { data: res } = await this.$http.get("categories");
            if (res.meta.status != 200) {
                return this.$message, erroe("获取商品分类失败！");
            }
            this.cateList = res.data;

        },
        // 选择框选中项触发函数
        handleChange() {
            this.getParamsData();

        },
        // Tab页签点击函数
        handleTabClick() {
            this.getParamsData();
        },
        async getParamsData() {
            if (this.selectedCateKeys.length != 3) {
                this.selectedCateKeys = [];
                this.manyTableData = [];
                this.onlyTableData = [];
                return
            }


            const { data: res } = await this.$http.get(
                `categories/${this.cateId}/attributes`,
                {
                    params: { sel: this.activeName }
                }
            )
            if (res.meta.status != 200) {
                return this.$message.error("获取参数列表失败！");
            }
            res.data.forEach(item => {
                item.attr_vals = item.attr_vals ? item.attr_vals.split(" ") : [];
                // 控制文本框显示与隐藏
                item.inputVisible = false;
                item.inputValue = "";
            })
            // console.log(res.data);
            if (this.activeName == "many") {
                this.manyTableData = res.data;
            } else {
                this.onlyTableData = res.data;
            }
        },
        addDialogClosed() {
            this.$refs.addFormRef.resetFilelds();
        },
        addParams() {
            this.$refs.addFormRef.validate(async valid => {
                if (!valid) return
                const { data: res } = await this.$http.post(
                    `categories/${this.cateId}/attributes`,
                    {
                        attr_name: this.addForm.attr_name,
                        attr_sel: this.activeName
                    }
                )

                if (res.meta.status !== 201) {
                    return this.$message.error('添加参数失败！')
                }

                this.$message.success('添加参数成功！')
                this.addDialogVisible = false
                this.getParamsData()
            })
        },
        async showEditDialog(attr_id) {
            // 查询当前参数的信息
            const { data: res } = await this.$http.get(
                `categories/${this.cateId}/attributes/${attr_id}`,
                {
                    params: { attr_sel: this.activeName }
                }
            )

            if (res.meta.status !== 200) {
                return this.$message.error('获取参数信息失败！')
            }

            this.editForm = res.data
            this.editDialogVisible = true
        },
        editDialogClosed() {
            this.$refs.editFormRef.resetFields()
        },
        editParams() {
            this.$refs.editFormRef.validate(async valid => {
                if (!valid) return
                const { data: res } = await this.$http.put(
                    `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
                    { attr_name: this.editForm.attr_name, attr_sel: this.activeName }
                )

                if (res.meta.status !== 200) {
                    return this.$message.error('修改参数失败！')
                }

                this.$message.success('修改参数成功！')
                this.getParamsData()
                this.editDialogVisible = false
            })
        },
        // 根据Id删除对应的参数项
        async removeParams(attr_id) {
            const confirmResult = await this.$confirm(
                '此操作将永久删除该参数, 是否继续?',
                '提示',
                {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }
            ).catch(err => err)

            // 用户取消了删除的操作
            if (confirmResult !== 'confirm') {
                return this.$message.info('已取消删除！')
            }

            // 删除的业务逻辑
            const { data: res } = await this.$http.delete(
                `categories/${this.cateId}/attributes/${attr_id}`
            )

            if (res.meta.status !== 200) {
                return this.$message.error('删除参数失败！')
            }

            this.$message.success('删除参数成功！')
            this.getParamsData()
        },
        async handleInputConfirm(row) {
            if (row.inputValue.trim().length == 0) {
                row.inputValue = "";
                row.inputVisible = false;
                return;
            }
            row.attr_vals.push(row.inputValue.trim());
            row.inputValue = "";
            row.inputVisible = false;
            this.saveAttrVals(row);
        },
        async saveAttrVals(row) {
            const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
                attr_name: row.attr_name,
                attr_sel: row.attr_sel,
                attr_vals: row.attr_vals.join(" ")
            })
            if (res.meta.status != 200) {
                return this.$message.error("修改参数项失败！");
            }
            this.$message.success("修改参数项成功！");
        },
        showInput(row) {
            row.inputVisible = true;
            this.$nextTick(_ => {
                this.$refs.saveTagInput.$refs.input.focus();
            });
        },
        handleClose(i, row) {
            row.attr_vals.splice(i, 1);
            this.saveAttrVals(row);
        }
    },
    computed: {
        // 按钮判断
        isBtnDisabled() {
            if (this.selectedCateKeys.length != 3) {
                return true;
            }

            return false;
        },
        // 当前选中三级分类id
        cateId() {
            if (this.selectedCateKeys.length == 3) {
                return this.selectedCateKeys[2];
            }
            return null;
        },
        titleText() {
            if (this.activeName == "many") {
                return "动态参数";
            }
            return "静态属性";
        }
    }
}
</script>

<style lang="less" scoped>
.cat_opt {
    margin: 15px 0;
}
.el-tag {
    margin-right: 10px;
}
.input-new-tag {
    width: 120px;
}
</style>