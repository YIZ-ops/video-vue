<template>
    <div>
        <el-input
            placeholder="输入关键字进行过滤"
            v-model="filterText"
        ></el-input>
        <el-tree
            :data="menus"
            :props="defaultProps"
            node-key="catId"
            ref="menuTree"
            @node-click="nodeclick"
            :filter-node-method="filterNode"
            :highlight-current="true"
        ></el-tree>
    </div>
</template>

<script>
export default {
    components: {},
    props: {},
    data() {
        return {
            filterText: '',
            menus: [],
            expandedKey: [],
            defaultProps: {
                children: 'children',
                label: 'name',
            },
        }
    },

    computed: {},

    watch: {
        filterText(val) {
            this.$refs.menuTree.filter(val)
        },
    },

    methods: {
        //树节点过滤
        filterNode(value, data) {
            if (!value) return true
            return data.name.indexOf(value) !== -1
        },
        getMenus() {
            this.$http({
                url: this.$http.adornUrl('/product/category/list/tree'),
                method: 'get',
            }).then(({ data }) => {
                this.menus = data.data
            })
        },
        nodeclick(data, node, component) {
            console.log('子组件category的节点被点击', data, node, component)
            //向父组件发送事件
            this.$emit('tree-node-click', data, node, component)
        },
    },

    created() {
        this.getMenus()
    },
}
</script>
<style scoped>
</style>