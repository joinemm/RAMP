<template>
    <div class="wrapper">
        <p v-if="!object">Nothing selected</p>
        <div v-if="object">
            <h3>{{ object.name }}</h3>
            <h4>Position</h4>
            <div class="inline-inputs">
                <input v-model="object.position.x" />
                <input v-model="object.position.y" />
                <input v-model="object.position.z" />
            </div>
            <h4>Rotation</h4>
            <div class="inline-inputs">
                <input v-model="object.rotation.x" />
                <input v-model="object.rotation.y" />
                <input v-model="object.rotation.z" />
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'ObjectView',
    props: {
        object: Object,
    },
    methods: {},
};
</script>

<style scoped>
.wrapper {
    padding: 10px;
}
.inline-inputs {
    display: flex;
}
.inline-inputs input {
    width: 33%;
    padding: 0;
}
</style>
