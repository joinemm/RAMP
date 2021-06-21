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
                <input v-model="object.rotation.x" :disabled="object.showAxis!=undefined && object.showAxis[0]==false"/>
                <input v-model="object.rotation.y" :disabled="object.showAxis!=undefined && object.showAxis[1]==false"/>
                <input v-model="object.rotation.z" :disabled="object.showAxis!=undefined && object.showAxis[2]==false"/>
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
