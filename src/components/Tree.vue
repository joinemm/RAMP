<template>
    <div class="wrapper">
        <h3>Objects</h3>
        <TreeNode :node="scene" />
    </div>
</template>

<script>
import TreeNode from './TreeNode';
export default {
    name: 'Tree',
    components: { TreeNode },
    props: { scene: Object },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.wrapper {
    padding: 10px;
}
</style>
