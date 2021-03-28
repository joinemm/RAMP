<template>
    <div class="tab" v-show="isActive"><slot></slot></div>
</template>

<script>
export default {
    name: 'Tab',
    props: {
        name: { required: true },
        selected: { default: false },
    },
    data() {
        return {
            isActive: false,
        };
    },
    mounted() {
        this.isActive = this.selected;
    },
};
</script>

<style scoped>
.tab {
    background-color: #1a1737;
    border-bottom-left-radius: 10px;
    border-bottom-right-radius: 10px;
}
</style>
