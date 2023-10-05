<template>
    <el-tree
        :data="menus"
        :props="defaultProps"
        node-key="catId"
        default-expand-all
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
                // console.log('data:', data.data)
                this.menus = data.data
            })
        },

        append(data) {
            console.log('data', data)
        },

        remove(node, data) {
            console.log(node, data)
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
