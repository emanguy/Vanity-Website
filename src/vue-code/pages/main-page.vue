<template>
    <div class="flex flex--vertical flex--ai-center full-width">
        <decorated-picture id="decorated-picture" msg="Hello Vue 3 + TypeScript + Vite" />

        <h1><span class="introAnim" ref="greeting">Hi.</span> <span class="introAnim" ref="selfRef">I'm</span> <span class="introAnim" ref="name">Evan Rittenhouse.</span></h1>

        <div class="introAnim" ref="content">
            <p>This is my personal website. It's currently under construction, but be sure to check back soon.</p>
        </div>
    </div>
</template>

<script lang="ts">

import {defineComponent, onMounted, ref} from "vue";
import DecoratedPicture from '../components/decorated-picture.vue';

export default defineComponent({
    name: 'MainPage',
    components: { DecoratedPicture },
    setup() {
        const animatableRefs: {[key: string]: Ref<HTMLSpanElement>} = {
            greeting: ref<HTMLSpanElement>(null),
            selfRef: ref<HTMLSpanElement>(null),
            name: ref<HTMLSpanElement>(null),
            content: ref<HTMLSpanElement>(null),
        }

        onMounted(async () => {
            const isNotFirstVisit = window.localStorage.getItem('firstVisit') === 'true';
            const queryParams = new URLSearchParams(window.location.search);
            const forceAnimation = queryParams.get('force-intro') !== null;

            if (!isNotFirstVisit || forceAnimation) {
                const animation = [
                    {opacity: 0, transform: 'translateY(10px)'},
                    {opacity: 1, transform: 'translateY(0px)'},
                ];

                const greetingAnim = animatableRefs.greeting.value.animate(animation, {
                    duration: 250,
                }).finished.then(() => animatableRefs.greeting.value.classList.add('revealed'));
                const selfRefAnim = animatableRefs.selfRef.value.animate(animation, {
                    duration: 250,
                    delay: 1000,
                }).finished.then(() => animatableRefs.selfRef.value.classList.add('revealed'));
                const nameAnim = animatableRefs.name.value.animate(animation, {
                    duration: 250,
                    delay: 1100,
                }).finished.then(() => animatableRefs.name.value.classList.add('revealed'));

                await Promise.all([greetingAnim, selfRefAnim, nameAnim]);
                await animatableRefs.content.value.animate(animation, {
                    duration: 250,
                    delay: 500,
                }).finished;
                animatableRefs.content.value.classList.add('revealed');

                window.localStorage.setItem('firstVisit', 'true');
            } else {
                for (let refName of Object.keys(animatableRefs)) {
                    animatableRefs[refName].value.classList.add('revealed');
                }
            }
        });

        return {...animatableRefs};
    },
});

</script>

<style lang="scss">
    #decorated-picture {
        margin-top: 10%;
    }

    .introAnim {
        display: inline-block;
        opacity: 0;

        &.revealed {
            opacity: 1;
        }
    }

    h1 > * {
        font-weight: inherit;
    }

    p {
        text-align: center;
    }
</style>
