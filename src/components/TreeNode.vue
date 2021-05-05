<template>
    <div v-if="node != null">
        <div>
            <span class="arrow" @click="expandTree"
                >{{ node.children.length > 0 ? (expanded ? '[-]' : '[+]') : '|||' }} </span
            ><span>{{ node.name }}</span>
        </div>
        <div v-if="expanded" class="children-container">
            <TreeNode
                v-for="child in node.children.filter((c) => c.isMesh)"
                :key="child.uuid"
                :node="child"
            />
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
