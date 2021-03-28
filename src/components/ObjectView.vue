<template>
    <div class="wrapper">
        <p v-if="!object">Nothing selected</p>
        <div v-if="object">
            <h3>{{ object.name }}</h3>
            <p>{{ object.position }}</p>
            <p>{{ object.rotation }}</p>
            <button @click="goup">go up</button>
            <input v-model="object.position.y" />
        </div>
    </div>
</template>

<script>
export default {
    name: 'ObjectView',
    props: {
        object: Object,
    },
    methods: {
        goup() {
            this.object.position.y += 1;
        },
    },
};
</script>

<style scoped>
.wrapper {
    padding: 10px;
}
</style>
