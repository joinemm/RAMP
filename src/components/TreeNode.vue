<template>
    <div>
        <div>
            <span class="arrow" @click="expandTree"
                >{{ node.children ? (expanded ? '[-]' : '[+]') : '|||' }} </span
            ><span>{{ node.name }}</span>
        </div>
        <div v-if="expanded" class="children-container">
            <TreeNode v-for="child in node.children" :key="child.name" :node="child" />
        </div>
    </div>
</template>

<script>
export default {
    name: 'TreeNode',
    props: {
        node: Object,
    },
    data() {
        return {
            expanded: false,
        };
    },
    methods: {
        expandTree() {
            this.expanded = !this.expanded;
        },
    },
};
</script>

<style scoped>
.arrow {
    cursor: pointer;
}
.arrow:hover {
    font-weight: bold;
}
.children-container {
    border-left: solid 1px grey;
    padding-left: 20px;
}
</style>
