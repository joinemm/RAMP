<template>
    <div>
        <div class="tabs">
            <ul>
                <li
                    @click="selectTab(tab)"
                    v-for="tab in tabs"
                    :key="tab.name"
                    :class="{ 'is-active': tab.isActive }"
                >
                    <a>{{ tab.name }}</a>
                </li>
            </ul>
        </div>

        <div class="tabs-details">
            <slot></slot>
        </div>
    </div>
</template>

<script>
export default {
    name: 'Tabs',
    data() {
        return {
            tabs: [],
        };
    },
    created() {
        this.tabs = this.$children;
    },
    methods: {
        selectTab(selectedTab) {
            this.tabs.forEach((tab) => {
                tab.isActive = tab.name == selectedTab.name;
            });
        },
    },
};
</script>

<style scoped>
.tabs ul {
    display: flex;
    flex-direction: row;
    list-style: none;
    justify-content: space-evenly;
    padding: 0;
    margin: 0;
}
.tabs ul li {
    user-select: none;
    text-align: center;
    flex-grow: 1;
    padding: 5px;
    background-color: #595677;
    cursor: pointer;
    border-top-left-radius: 10px;
    border-top-right-radius: 10px;
}
.tabs ul li:hover:not(.is-active) {
    background-color: #797697;
}
.tabs ul .is-active {
    background-color: #1a1737;
}
</style>
