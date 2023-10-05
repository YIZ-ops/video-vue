<template>
    <el-tree
        :data="menus"
        :props="defaultProps"
        @node-click="handleNodeClick"
    >
        <span
            class="custom-tree-node"
            slot-scope="{ node, data }"
        >
            <span>{{ node.label }}</span>
            <span>
                <el-button
                    type="text"
                    size="mini"
                    @click="() => append(data)"
                >
                    Append
                </el-button>
                <el-button
                    type="text"
                    size="mini"
                    @click="() => remove(node, data)"
                >
                    Delete
                </el-button>
            </span>
        </span>
    </el-tree>
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
        }
    },
    methods: {
        getMenus() {
            this.$http({
                url: this.$http.adornUrl('/product/category/list/tree'),
                method: 'get',
            }).then(({ data }) => {
                console.log('data:', data.data)
                this.menus = data.data
            })
        },

        append(data) {
            const newChild = { id: id++, label: 'testtest', children: [] }
            if (!data.children) {
                this.$set(data, 'children', [])
            }
            data.children.push(newChild)
        },

        remove(node, data) {
            const parent = node.parent
            const children = parent.data.children || parent.data
            const index = children.findIndex((d) => d.id === data.id)
            children.splice(index, 1)
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
