<template>
    <div>
        <el-tree
            :data="menus"
            :props="defaultProps"
            node-key="catId"
            show-checkbox
            :default-expanded-keys="expandedKeys"
            :expand-on-click-node="false"
        >
            <span
                class="custom-tree-node"
                slot-scope="{ node, data }"
            >
                <span>{{ node.label }}</span>
                <span>
                    <el-button
                        v-if="node.level <= 2"
                        type="text"
                        size="mini"
                        @click="() => append(data)"
                    >
                        添加
                    </el-button>
                    <el-button
                        type="text"
                        size="mini"
                        @click="() => edit(data)"
                    >
                        修改
                    </el-button>
                    <el-button
                        v-if="node.childNodes.length === 0"
                        type="text"
                        size="mini"
                        @click="() => remove(node, data)"
                    >
                        删除
                    </el-button>
                </span>
            </span>
        </el-tree>

        <el-dialog
            :title="title"
            :visible.sync="dialogVisible"
            width="30%"
        >
            <el-form :model="category">
                <el-form-item label="分类名称">
                    <el-input v-model="category.name"></el-input>
                </el-form-item>
                <el-form-item label="图标">
                    <el-input v-model="category.icon"></el-input>
                </el-form-item>
                <el-form-item label="计量单位">
                    <el-input v-model="category.productUnit"></el-input>
                </el-form-item>
            </el-form>
            <span
                slot="footer"
                class="dialog-footer"
            >
                <el-button @click="dialogVisible = false">取 消</el-button>
                <el-button
                    type="primary"
                    @click="submitData"
                >确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    data() {
        return {
            menus: [],
            expandedKeys: [],
            defaultProps: {
                children: 'children',
                label: 'name',
            },
            category: {
                name: '',
                parentCid: 0,
                catLevel: 0,
                showStatus: 1,
                sort: 0,
                productUnit: '',
                icon: '',
                catId: null,
            },
            dialogType: '', // edit,add
            title: '',
            dialogVisible: false,
        }
    },
    methods: {
        getMenus() {
            this.$http({
                url: this.$http.adornUrl('/product/category/list/tree'),
                method: 'get',
            }).then(({ data }) => {
                this.menus = data.data
            })
        },

        append(data) {
            this.title = '添加分类'
            this.dialogType = 'add'
            this.dialogVisible = true
            this.category.parentCid = data.catId
            this.category.catLevel = data.catLevel + 1
        },

        update(data) {
            this.title = '修改分类'
            this.dialogType = 'edit'
            this.dialogVisible = true
            this.category = data
        },

        remove(node, data) {
            this.$confirm(
                `此操作将永久删除菜单【${data.name}】，是否继续?`,
                '提示',
                {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning',
                }
            )
                .then(() => {
                    var ids = [data.catId]
                    this.$http({
                        url: this.$http.adornUrl('/product/category/delete'),
                        method: 'post',
                        data: this.$http.adornData(ids, false),
                    }).then(({ data }) => {
                        this.$message({
                            type: 'success',
                            message: '删除成功!',
                        })
                        // 刷新菜单
                        this.getMenus()
                        // 设置需要默认展开的菜单
                        this.expandedKeys = [node.parent.data.catId]
                    })
                })
                .catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除',
                    })
                })
        },

        submitData() {
            if (this.dialogType == 'add') {
                this.$http({
                    url: this.$http.adornUrl('/product/category/save'),
                    method: 'post',
                    data: this.$http.adornData(this.category, false),
                }).then(({ data }) => {
                    this.$message({
                        type: 'success',
                        message: '添加成功!',
                    })
                    // 刷新菜单
                    this.getMenus()
                    // 设置需要默认展开的菜单
                    this.expandedKeys = [node.parent.data.catId]
                })
            } else {
                this.$http({
                    url: this.$http.adornUrl('/product/category/update'),
                    method: 'post',
                    data: this.$http.adornData(this.category, false),
                }).then(({ data }) => {
                    this.$message({
                        type: 'success',
                        message: '添加成功!',
                    })
                    // 刷新菜单
                    this.getMenus()
                    // 设置需要默认展开的菜单
                    this.expandedKeys = [node.parent.data.catId]
                })
            }
        },
    },
    created() {
        this.getMenus()
    },
}
</script>

<style>
.custom-tree-node {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 14px;
    padding-right: 8px;
}
</style>
