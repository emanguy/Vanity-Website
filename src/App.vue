<template>
    <main-page></main-page>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import MainPage from './vue-code/pages/main-page.vue';

export default defineComponent({
    name: 'App',
    components: {
        MainPage
    }
})
</script>

<style lang="scss">
@import './assets/styles/layout.scss';
@import './assets/styles/typography.scss';

#app {
    //width: 100%;
}
</style>
