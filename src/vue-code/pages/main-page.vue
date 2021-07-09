<template>
    <!-- noinspection -->
    <button ref="navMenu" class="material-icons-outlined icon-button introAnim" :class="{revealed: navRevealed}" v-if="isMobile" @click="navExpanded = true">menu</button>
    <nav ref="nav" class="flex flex--ai-center introAnim" :class="{expanded: navExpanded, revealed: navRevealed}" :aria-hidden="isMobile && !navExpanded">
        <button class="material-icons-outlined icon-button" v-if="isMobile" @click="navExpanded = false">arrow_back</button>

        <decorated-picture class="nav-picture" :animated="false" :image-size-px="75" v-if="isMobile"/>
        <div id="decoratedPictureTarget" ref="pictureTarget" v-else style="margin-right: 12px"></div>

        <a ref="firstLink" href="/" class="link--active-page">Home</a>
        <a href="/">Personal Projects</a>
        <a href="/">Links & Social</a>
    </nav>
    <main class="flex flex--vertical flex--ai-center full-width">
        <span id="decoratedPictureOrigin" ref="decoratedPictureOrigin" v-if="determiningAnimationTarget"></span>
        <teleport to="#decoratedPictureTarget" :disabled="isMobile || !pictureInNav">
            <decorated-picture ref="personalPicture" id="decorated-picture" :image-size-px="personalPictureSizePx" @click="movePictureToNav()"/>
        </teleport>

        <h1><span class="introAnim" ref="greeting" :class="{revealed: greetingRevealed}">Hi.</span> <span class="introAnim" ref="selfRef" :class="{revealed: selfRefRevealed}">I'm</span> <span class="introAnim" ref="name" :class="{revealed: nameRevealed}">Evan Rittenhouse.</span></h1>

        <div class="introAnim" ref="content" :class="{revealed: contentRevealed}">
            <p>This is my personal website. It's currently under construction, but be sure to check back soon.</p>
        </div>
    </main>
</template>

<script lang="ts">

import {defineComponent, onMounted, ref, Ref} from "vue";
import DecoratedPicture from '../components/decorated-picture.vue';
import {wait} from "../../ts/async";

export default defineComponent({
    name: 'MainPage',
    components: { DecoratedPicture },
    setup() {
        // Basic page state data

        const navExpanded = ref(false);
        const isMobile = ref(window.matchMedia('screen and (max-width: 600px)').matches);
        const determiningAnimationTarget = ref(false);

        // Fancy intro animation values

        // @ts-ignore
        const decoratedPictureOrigin = ref<HTMLSpanElement>(null);
        const animatableRefs: {[key: string]: Ref<HTMLElement>} = {
            // @ts-ignore
            greeting: ref<HTMLSpanElement>(null),
            // @ts-ignore
            selfRef: ref<HTMLSpanElement>(null),
            // @ts-ignore
            name: ref<HTMLSpanElement>(null),
            // @ts-ignore
            content: ref<HTMLDivElement>(null),
            // @ts-ignore
            nav: ref<HTMLElement>(null),
            // @ts-ignore
            navMenu: ref<HTMLButtonElement>(null),
        };
        const elementsRevealed: {[key: string]: Ref<boolean>} = {
            greetingRevealed: ref(false),
            selfRefRevealed: ref(false),
            nameRevealed: ref(false),
            contentRevealed: ref(false),
            navRevealed: ref(false),
        };

        // Navigation toward and away from main page

        const pictureInNav = ref(false);
        const personalPictureSizePx = ref(200);
        // @ts-ignore
        const pictureTarget = ref<HTMLDivElement>(null);
        // @ts-ignore
        const firstLink = ref<HTMLAnchorElement>(null);
        // @ts-ignore
        const personalPicture = ref<DecoratedPicture>(null);

        const animateContent = async (contentAnimation: Array<Keyframe>) => {
            const animations = [];

            const duration = 250;
            animations.push(animatableRefs.content.value.animate(contentAnimation, duration).finished
                .then(() => elementsRevealed.contentRevealed.value = false));
            animations.push(animatableRefs.greeting.value.animate(contentAnimation, duration).finished
                .then(() => elementsRevealed.greetingRevealed.value = false));
            animations.push(animatableRefs.selfRef.value.animate(contentAnimation, duration).finished
                .then(() => elementsRevealed.selfRefRevealed.value = false));
            animations.push(animatableRefs.name.value.animate(contentAnimation, duration).finished
                .then(() => elementsRevealed.nameRevealed.value = false));

            return Promise.all(animations);
        };

        const animateContentOut = async () => {
            const fadeOutAnim = [
                {transform: 'translateY(0)', opacity: 1},
                {transform: 'translateY(10px)', opacity: 0},
            ];

            await animateContent(fadeOutAnim);
        };

        const animateContentIn = async () => {
            const fadeInAnim = [
                {transform: 'translateY(10px)', opacity: 0},
                {transform: 'translateY(0)', opacity: 1},
            ];

            await animateContent(fadeInAnim);
        }

        const movePictureToNav = async () => {
            await animateContentOut();
            await personalPicture.value.switchToStaticBorder();
            personalPictureSizePx.value = 50;
            await wait(250);

            const pictureBoundingRect: DOMRect = personalPicture.value.$el.getBoundingClientRect();
            const linkBoundingRect = pictureTarget.value.getBoundingClientRect();

            const xTranslation = linkBoundingRect.x - pictureBoundingRect.x;
            const yTranslation = linkBoundingRect.y - pictureBoundingRect.y - 29;

            const animations = [];

            // Shove the links to the side to make room for my picture
            animations.push(firstLink.value.animate([
                {margin: '0 8px 0 8px'},
                {margin: '0 8px 0 66px'},
            ], {
                duration: 750,
                easing: 'ease-in-out',
            }).finished);

            // Run X and Y translations separately but composited together to get a nice sideways curve up into the intended destination
            animations.push(personalPicture.value.$el.animate([
                {transform: 'translateX(0)'},
                {transform: `translateX(${xTranslation * 0.5}px)`, offset: 0.25},
                {transform: `translateX(${xTranslation * 0.9}px)`, offset: 0.5},
                {transform: `translateX(${xTranslation}px)`},
            ], {
                duration: 750,
                easing: 'ease-in-out',
            }).finished);
            animations.push(personalPicture.value.$el.animate([
                {transform: 'translateY(0)'},
                {transform: `translateY(${yTranslation * 0.5}px)`, offset: 0.75},
                {transform: `translateY(${yTranslation * 0.75}px)`, offset: 0.9},
                {transform: `translateY(${yTranslation}px)`},
            ], {
                duration: 750,
                offset: 500,
                easing: 'ease-in-out',
                composite: 'accumulate',
            }).finished);

            // Wait for those animations to run and then teleport my image to the nav section
            await Promise.all(animations);
            pictureInNav.value = true;
        };

        const movePictureFromNav = async () => {
            // First, estimate where the image is going to end up in the DOM
            determiningAnimationTarget.value = true;
            // Let the DOM update
            await new Promise(resolve => requestAnimationFrame(resolve));

            // Determine how far from the image's current location it'll need to move to get the animation to line up
            const {x: sourceX, y: sourceY} = personalPicture.value.$el.getBoundingClientRect();
            const {x: destinationX, y: destinationY} = decoratedPictureOrigin.value.getBoundingClientRect();

            const xTranslation = destinationX - sourceX;
            const yTranslation = destinationY - sourceY;

            // Now we don't need the placeholder element anymore
            determiningAnimationTarget.value = false;

            // Run the animation
            const animations = [];

            // Shove the links back into their original position
            animations.push(firstLink.value.animate([
                {margin: '0 8px 0 66px'},
                {margin: '0 8px 0 8px'},
            ], {
                duration: 750,
                easing: 'ease-in-out',
            }).finished);

            // Run X and Y translations separately but composited together to get a nice sideways curve up into the intended destination
            animations.push(personalPicture.value.$el.animate([
                {transform: 'translateX(0)'},
                {transform: `translateX(${xTranslation * 0.5}px)`, offset: 0.25},
                {transform: `translateX(${xTranslation * 0.9}px)`, offset: 0.5},
                {transform: `translateX(${xTranslation}px)`},
            ], {
                duration: 750,
                easing: 'ease-in-out',
            }).finished);
            animations.push(personalPicture.value.$el.animate([
                {transform: 'translateY(0)'},
                {transform: `translateY(${yTranslation * 0.5}px)`, offset: 0.75},
                {transform: `translateY(${yTranslation * 0.75}px)`, offset: 0.9},
                {transform: `translateY(${yTranslation}px)`},
            ], {
                duration: 750,
                offset: 500,
                easing: 'ease-in-out',
                composite: 'accumulate',
            }).finished);

            // Wait for those animations to run and then teleport my image out of the nav section
            await Promise.all(animations);
            pictureInNav.value = false;

            // Size the image back to normal, then animate in the content
            personalPictureSizePx.value = 200;
            await wait(250);
            await personalPicture.value.switchToAnimatedBorder();
            await animateContentIn();
        };

        // Hooks

        onMounted(async () => {
            const isNotFirstVisit = window.localStorage.getItem('firstVisit') === 'true';
            const queryParams = new URLSearchParams(window.location.search);
            const forceAnimation = queryParams.get('force-intro') !== null;

            if (!isNotFirstVisit || forceAnimation) {
                const animation = [
                    {opacity: 0, transform: 'translateY(10px)'},
                    {opacity: 1, transform: 'translateY(0px)'},
                ];
                const topAnimation = [
                    {opacity: 0, transform: 'translateY(-10px)'},
                    {opacity: 1, transform: 'translateY(0px)'},
                ];

                // "Hi, I'm Evan Rittenhouse"
                const greetingAnim = animatableRefs.greeting.value.animate(animation, {
                    duration: 250,
                }).finished.then(() => elementsRevealed.greetingRevealed.value = true);
                const selfRefAnim = animatableRefs.selfRef.value.animate(animation, {
                    duration: 250,
                    delay: 1000,
                }).finished.then(() => elementsRevealed.selfRefRevealed.value = true);
                const nameAnim = animatableRefs.name.value.animate(animation, {
                    duration: 250,
                    delay: 1100,
                }).finished.then(() => elementsRevealed.nameRevealed.value = true);

                await Promise.all([greetingAnim, selfRefAnim, nameAnim]);

                // Rest of the page content flies in
                const bodyAnim = animatableRefs.content.value.animate(animation, {
                    duration: 250,
                    delay: 500,
                }).finished.then(() => elementsRevealed.contentRevealed.value = true);
                const navAnim = animatableRefs.nav.value.animate(topAnimation, {
                    duration: 250,
                    delay: 500,
                }).finished.then(() => elementsRevealed.navRevealed.value = true);

                const finalAnimations: Promise<any>[] = [bodyAnim, navAnim];

                // Animate the menu button in on mobile
                if (isMobile.value) {
                    finalAnimations.push(animatableRefs.navMenu.value.animate(topAnimation, {
                        duration: 250,
                        delay: 500,
                    }).finished);
                }

                await Promise.all(finalAnimations);

                // Stash that we already did the fancy animation once
                window.localStorage.setItem('firstVisit', 'true');
            } else {
                for (let refName of Object.keys(elementsRevealed)) {
                    elementsRevealed[refName].value = true;
                }
            }
        });

        return {
            isMobile,
            navExpanded,
            determiningAnimationTarget,

            pictureInNav,
            firstLink,
            personalPicture,
            personalPictureSizePx,
            pictureTarget,
            movePictureToNav,
            movePictureFromNav,

            decoratedPictureOrigin,
            ...animatableRefs,
            ...elementsRevealed,
        };
    },
});

</script>

<style lang="scss">
@media screen and (max-width: 600px) {
    nav {
        flex-direction: column !important;
        align-items: start !important;
        background-color: #cecabf;
        position: fixed;
        width: 80%;
        height: 100% !important;
        top: 0;
        left: -80%;
        z-index: 1;
        padding: 16px;
        box-sizing: border-box;

        transition: left 250ms;

        .nav-picture {
            align-self: center;
            margin: 8px;
        }

        :nth-child(3) {
            margin-top: 32px;
        }

        &.expanded {
            left: 0;
            box-shadow: 8px 0 12px 4px rgba(39, 39, 39, 0.15);
        }
    }
}

.icon-button {
    background-color: transparent;
    border: none;
    cursor: pointer;
    margin-top: 8px;
}

nav {
    flex-direction: row;
    height: 64px;
}

#decoratedPictureOrigin {
    margin-top: 10%;
    width: 230px;
}

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
