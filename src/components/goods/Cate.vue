<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>

        <el-card>
            <el-row :gutter="10">
                <el-col :span="6">
                    <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
                </el-col>
            </el-row>

            <!-- 表格 -->
            <tree-table
                :data="catelist"
                :columns="columns"
                :selection-type="false"
                :expand-type="false"
                show-index
                index-text="#"
                border
                :show-row-hover="false"
                class="tree-table"
            >
                <!-- 是否有效 -->
                <template slot="isok" slot-scope="scope">
                    <i
                        class="el-icon-success"
                        v-if="scope.row.cat_deleted==false"
                        style="color:lightgreen"
                    ></i>
                    <i class="el-icon-error" v-else style="color:red"></i>
                </template>
                <!-- 排序 -->
                <template slot="order" slot-scope="scope">
                    <el-tag size="mini" v-if="scope.row.cat_level==0">一级</el-tag>
                    <el-tag size="mini" v-else-if="scope.row.cat_level==1" type="success">二级</el-tag>
                    <el-tag size="mini" v-else type="warning">三级</el-tag>
                </template>
                <template slot="opt">
                    <el-button size="mini" type="primary" icon="el-icon-edit"></el-button>
                    <el-button size="mini" type="dinger" icon="el-icon-delete"></el-button>
                </template>
            </tree-table>

            <!-- 分页 -->
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="querInfo.pagenum"
                :page-sizes="[3,5,10]"
                :page-size="querInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total"
            ></el-pagination>
        </el-card>

        <!-- 添加分类 -->
        <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%">
            <!-- 添加分类表单 -->
            <el-form
                ref="addCateFormRef"
                rules="addCateFormRules"
                :model="addCateForm"
                label-width="80px"
                @close="addCateDialogClosed"
            >
                <el-form-item label="分类名称" prop="cat_name">
                    <el-input v-model="addCateForm.cat_name"></el-input>
                </el-form-item>
                <el-form-item label="父级分类">
                    <!-- option定义数据源 -->
                    <!-- props指定配置对象 -->
                    <el-cascader
                        expand-trigger="hover"
                        :props="cascaderProps"
                        :options="parentCateList"
                        v-model="selectedKeys"
                        @change="parentCateChanged"
                        clearable
                        change-on-select
                    ></el-cascader>
                </el-form-item>
            </el-form>

            <div slot="footer">
                <el-button @click="dialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addCate">确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
export default {
    data() {
        return {
            querInfo: {
                type: 3,
                pagenum: 1,
                pagesize: 5
            },
            // 商品分类
            catelist: [],
            // 总数据条数
            total: 0,
            columns: [{
                label: "分类名称",
                prop: "cat_name"
            }, {
                label: "是否有效",
                type: "template",
                template: "isok"
            }, {
                label: "排序",
                type: "template",
                template: "order"
            }, {
                label: "操作",
                type: "template",
                template: "opt"
            }],
            addCateDialogVisible: false,
            // 要添加的数据
            addCateForm: {
                cat_name: "",
                cat_pid: 0,
                cat_level: 0
            },
            // 验证规则
            addCateFormRules: {
                cat_name: [
                    { required: true, message: '请输入分类名称', trigger: 'blur' }
                ]
            },
            parentCateList: [],
            // 指定级联选择器配置对象
            cascaderProps: {
                value: "cat_id",
                label: "cat_name",
                children: "children"
            },
            // 选中的分类
            selectedKeys: [],
        }
    },
    created() {
        this.getCateList()
    },
    methods: {
        async getCateList() {
            const { data: res } = await this.$http.get("categories", { params: this.querInfo })
            if (res.meta.status != 200) {
                return this.$message.error("获取商品数据失败！");
            }
            this.catelist = res.data.result;
            this.total = res.data.total;
        },
        // 监听pagesize
        handleSizeChange(newsize) {
            this.querInfo.pagesize = newsize;
            this.getCateList();
        },
        // 监听pagenum
        handleCurrentChange(newnum) {
            this.querInfo.pagenum = newnum;
            this.getCateList();

        },
        showAddCateDialog() {
            this.getParentCateList();
            this.addCateDialogVisible = true;
        },
        // 获取父级分类列表

        async getParentCateList() {
            const { data: res } = await this.$http.get("categories", { params: { type: 2 } })
            if (res.meta.status != 200) {
                return this.$message.error("获取数据失败！");
            }
            this.parentCateList = res.data;
        },
        parentCateChanged() {
            if (this.selectedKeys.length > 0) {
                this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1];
                this.addCateForm.cat_level = this.selectedKeys.length;
                return
            } else {
                this.addCateForm.cat_pid = 0;
                this.addCateForm.cat_level = 0;
            }
        },
        addCate() {
            this.$refs.addCateFormRef.validate(async valid => {
                if (!valid) return
                const { data: res } = await this.$http.post("categories", this.addCateForm);
                if (res.meta.status != 201) {
                    return this.$message.error("添加分类失败！");
                }
                this.$message.success("添加分类成功！");
                this.getCateList();
                this.addCateDialogVisible = false;

            })
        },
        addCateDialogClosed() {
            this.$refs.addCateFormRef.resetFields();
            this.selectedKeys = [];
            this.addCateForm.cat_level = 0;
            this.addCateForm.cat_pid = 0;
        }
    }
}
</script>
<style lang="less" scoped>
.tree-table {
    margin-top: 15px;
}
.el-cascader {
    width: 100%;
}
</style>