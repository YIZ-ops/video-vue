<template>
    <div>
        <el-switch
            v-model="draggable"
            active-text="开启拖拽"
            inactive-text="关闭拖拽"
        ></el-switch>
        <el-button
            v-if="draggable"
            @click="batchSave"
            type="success"
            size="small"
        >批量保存</el-button>
        <el-button
            type="danger"
            @click="batchDelete"
            size="small"
        >批量删除</el-button>
        <el-tree
            :data="menus"
            :props="defaultProps"
            node-key="catId"
            show-checkbox
            :default-expanded-keys="expandedKeys"
            :expand-on-click-node="false"
            :draggable="draggable"
            :allow-drop="allowDrop"
            @node-drop="handleDrop"
            ref="menuTree"
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
            <el-form
                :model="category"
                label-width="70px"
            >
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

            expandedKeys: [],
            updateNodes: [], // 拖拽更新节点
            draggable: false,
            maxLevel: 0,
            pCid: [],
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
            this.category.catId = null
            this.category.name = ''
            this.category.icon = ''
            this.category.productUnit = ''
        },

        edit(data) {
            this.title = '修改分类'
            this.dialogType = 'edit'
            this.dialogVisible = true
            // 发送请求获取当前节点的最新数据
            this.$http({
                url: this.$http.adornUrl(
                    `/product/category/info/${data.catId}`
                ),
                method: 'get',
            }).then(({ data }) => {
                this.category.catId = data.data.catId
                this.category.name = data.data.name
                this.category.icon = data.data.icon
                this.category.productUnit = data.data.productUnit
                this.category.parentCid = data.data.parentCid
            })
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
                this.addCategory()
            } else if (this.dialogType == 'edit') {
                this.editCategory()
            }
        },

        addCategory() {
            this.$http({
                url: this.$http.adornUrl('/product/category/save'),
                method: 'post',
                data: this.$http.adornData(this.category, false),
            }).then(({ data }) => {
                this.$message({
                    type: 'success',
                    message: '菜单保存成功!',
                })
                this.dialogVisible = false
                this.getMenus()
                this.expandedKeys = [this.category.parentCid]
            })
        },

        editCategory() {
            var { catId, name, icon, productUnit } = this.category
            var data = { catId, name, icon, productUnit }
            this.$http({
                url: this.$http.adornUrl('/product/category/update'),
                method: 'post',
                data: this.$http.adornData(data, false),
            }).then(({ data }) => {
                this.$message({
                    type: 'success',
                    message: '菜单修改成功!',
                })
                this.dialogVisible = false
                this.getMenus()
                this.expandedKeys = [this.category.parentCid]
            })
        },

        allowDrop(draggingNode, dropNode, type) {
            // 被拖拽节点的深度 + 结束拖拽时最后进入的节点的层数 <= 3
            this.countNodeLevel(draggingNode)
            // 被拖拽节点的深度 = 被拖拽节点的最大层数 - 被拖拽节点的层数
            let deep = Math.abs(this.maxLevel - draggingNode.level) + 1
            if (type == 'inner') {
                return deep + dropNode.level <= 3
            } else {
                return deep + dropNode.parent.level <= 3
            }
        },

        countNodeLevel(node) {
            // 找到所有子节点，求出最大深度
            if (node.childNodes != null && node.childNodes.length > 0) {
                node.childNodes.forEach((item) => {
                    if (item.level > this.maxLevel) {
                        this.maxLevel = item.level
                    }
                    this.countNodeLevel(item)
                })
            }
        },

        batchSave() {
            this.$http({
                url: this.$http.adornUrl('/product/category/update/sort'),
                method: 'post',
                data: this.$http.adornData(this.updateNodes, false),
            }).then(({ data }) => {
                this.$message({
                    message: '菜单顺序等修改成功',
                    type: 'success',
                })
                this.getMenus()
                this.expandedKeys = this.pCid
                this.updateNodes = []
                this.maxLevel = 0
                this.pCid = []
            })
        },

        handleDrop(draggingNode, dropNode, dropType, ev) {
            // 修改 parentCid (draggingNode) sort (draggingNode siblingNodes) catLevel (draggingNode childNodes)
            let pCid = 0
            let siblings = null // 兄弟节点
            if (dropType === 'inner') {
                pCid = dropNode.data.catId
                siblings = dropNode.childNodes
            } else {
                pCid =
                    dropNode.parent.data.catId == undefined
                        ? 0
                        : dropNode.parent.data.catId
                siblings = dropNode.parent.childNodes
            }
            this.pCid.push(pCid)
            siblings.forEach((item, index) => {
                // 如果遍历的是当前正在拖拽的节点
                if (item.data.catId === draggingNode.data.catId) {
                    // 当前节点的层级发生变化
                    let catLevel = draggingNode.level
                    if (item.level != draggingNode.level) {
                        catLevel = item.level
                        // 修改当前节点子节点的层级
                        this.updateChildNodeLevel(item)
                    }
                    this.updateNodes.push({
                        catId: item.data.catId,
                        parentCid: pCid,
                        sort: index,
                        catLevel: catLevel,
                    })
                } else {
                    // 兄弟节点
                    this.updateNodes.push({
                        catId: item.data.catId,
                        sort: index,
                    })
                }
            })
            // console.log('updateNodes', this.updateNodes)
        },
        

        updateChildNodeLevel(node) {
            if (node.childNodes.length > 0) {
                node.childNodes.forEach((item) => {
                    this.updateNodes.push({
                        catId: item.data.catId,
                        catLevel: item.level,
                    })
                    this.updateChildNodeLevel(item)
                })
            }
        },

        batchDelete() {
            // 被选中的元素
            let catIds = []
            let catNames = []
            this.$refs.menuTree.getCheckedNodes().forEach((item) => {
                catIds.push(item.catId)
                catNames.push(item.name)
            })
            this.$confirm(
                `此操作将永久删除菜单【${catNames}】，是否继续?`,
                '提示',
                {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning',
                }
            )
                .then(() => {
                    this.$http({
                        url: this.$http.adornUrl('/product/category/delete'),
                        method: 'post',
                        data: this.$http.adornData(catIds, false),
                    }).then(({ data }) => {
                        this.$message({
                            type: 'success',
                            message: '菜单批量删除成功!',
                        })
                        this.getMenus()
                    })
                })
                .catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除',
                    })
                })
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
