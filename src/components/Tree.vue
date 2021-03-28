<template>
    <div class="wrapper">
        <h3>forest</h3>
        <TreeNode :node="treeData" />
    </div>
</template>

<script>
import TreeNode from './TreeNode';
export default {
    name: 'Tree',
    components: { TreeNode },
    data() {
        return {
            treeData: {
                name: 'Scene',
                children: [
                    { name: 'Cube' },
                    { name: 'Chair' },
                    { name: 'Thingy', children: [{ name: 'subthingy' }] },
                    {
                        name: 'Another thingy',
                        children: [{ name: 'subthingy2' }, { name: 'subthingy3' }],
                    },
                ],
            },
        };
    },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.wrapper {
    padding: 10px;
}
</style>
