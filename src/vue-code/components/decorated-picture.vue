<template>
    <div :style="{width: borderSize, height: borderSize}">
        <img src="/img/headshot-square.jpeg" alt="Picture of me" :style="{width: imageSize, height: imageSize, top: imageOffset, left: imageOffset}">
        <svg class="border border--clockwise" width="228" height="228" :class="{animationStop: animationStopped}">
            <defs>
                <radialGradient :id="`greenGradient-${svg1ID}`">
                    <stop offset="50%" stop-color="#4D6958FF"></stop>
                    <stop offset="80%" stop-color="#4D6958CC"></stop>
                    <stop offset="100%" stop-color="#61846F88"></stop>
                </radialGradient>
            </defs>
            <path :d="pulsingSpiralPathFaster" :fill="`url(#greenGradient-${svg1ID})`" stroke="none" transform="translate(114,114)"></path>
        </svg>
        <svg class="border border--counterclockwise" width="228" height="228" :class="{animationStop: animationStopped}">
            <defs>
                <radialGradient :id="`greenGradient-${svg2ID}`">
                    <stop offset="50%" stop-color="#4D6958FF"></stop>
                    <stop offset="80%" stop-color="#4D6958CC"></stop>
                    <stop offset="100%" stop-color="#61846F88"></stop>
                </radialGradient>
            </defs>
            <path :d="pulsingSpiralPathSlower" :fill="`url(#greenGradient-${svg2ID})`" stroke="none" transform="translate(114,114)"></path>
        </svg>
        <div class="staticBackground" :class="{'staticBackground--expanded': borderUp}" :style="{width: borderSize, height: borderSize}"></div>
    </div>
</template>

<script lang="ts">
import {computed, defineComponent, ref, watch, toRef} from 'vue'
import * as d3 from 'd3';

export default defineComponent({
    name: 'DecoratedPicture',
    props: {
        imageSizePx: {
            type: Number,
            default: 200,
        },
        animated: {
            type: Boolean,
            default: true,
        },
    },
    setup: (props) => {
        // Border animation
        const step = Math.PI/64;

        const svg1ID = ref(Math.random().toString(16).substring(0, 5));
        const svg2ID = ref(Math.random().toString(16).substring(0, 5));

        const radialSineAmplitudeSlower = ref(7);
        const radialSineAmplitudeFaster = ref(7);
        const irregularity = Array.from({length: 130}, () => (Math.random() + 0.5) * -7);
        const generatedSpiralSlower = computed(() => Array.from({ length: 130 }, (_, i) => [step * i, 107 + (radialSineAmplitudeSlower.value * Math.sin(Math.PI/4 * i)) + irregularity[i]]));
        const generatedSpiralFaster = computed(() => Array.from({ length: 130 }, (_, i) => [step * i, 107 + (radialSineAmplitudeFaster.value * Math.sin(Math.PI/4 * i)) + irregularity[i]]));

        // @ts-ignore
        const pulsingSpiralPathSlower = computed(() => d3.lineRadial().curve(d3.curveCardinalOpen)(generatedSpiralSlower.value));
        // @ts-ignore
        const pulsingSpiralPathFaster = computed(() => d3.lineRadial().curve(d3.curveCardinalOpen)(generatedSpiralFaster.value));

        let intervalNum: number|null = null;

        const startAnimation = () => {
            if (intervalNum !== null) {
                console.error("Illegal: cannot start animation twice");
                return;
            }

            const framesInOneSecond = 60;
            let frameNum = 0;
            let fasterFrameNum = 1.5 * framesInOneSecond;

            intervalNum = window.setInterval(() => {
                frameNum = (frameNum % (5 * framesInOneSecond)) + 1;
                fasterFrameNum = (fasterFrameNum % (3 * framesInOneSecond)) + 1;

                // We want to go from 7 to 0 and back again in a linear, absolute value fashion
                let animationCompletionPercentage = frameNum / (5 * framesInOneSecond);
                let fasterAnimationCompletionPercentage = fasterFrameNum / (3 * framesInOneSecond);

                radialSineAmplitudeSlower.value = Math.abs(animationCompletionPercentage * 2 - 1) * 7;
                radialSineAmplitudeFaster.value = Math.abs(fasterAnimationCompletionPercentage * 2 - 1) * 7;
            }, 1000/framesInOneSecond);
        };

        const stopAnimation = () => {
            if (intervalNum === null) {
                console.warn("Attempted to stop animation twice");
                return;
            }
            window.clearInterval(intervalNum);
            intervalNum = null;
        };

        if (props.animated) {
            startAnimation();
        }

        // Border control
        const animationStopped = ref(!props.animated);
        const borderUp = ref(!props.animated);

        const switchToStaticBorder = async () => {
            stopAnimation();
            borderUp.value = true;
            return new Promise<void>((resolve) => {
                window.setTimeout(() => {
                    animationStopped.value = true;
                    resolve();
                }, 200);
            });
        }

        const switchToAnimatedBorder = () => {
            startAnimation();
            animationStopped.value = false;
            borderUp.value = false;
        }

        watch(toRef(props, 'animated'), (isAnimated, oldValue) => {
            if (isAnimated === oldValue) {
                return;
            }

            if (isAnimated) {
                switchToAnimatedBorder();
            } else {
                switchToStaticBorder();
            }
        });

        // Additional style stuff

        const imageSize = computed(() => `${props.imageSizePx}px`);
        const borderSize = computed(() => `${props.imageSizePx * 1.15}px`);
        const imageOffset = computed(() => `${props.imageSizePx * .15 / 2}px`);

        return {
            svg1ID,
            svg2ID,

            animationStopped,
            borderUp,
            pulsingSpiralPathFaster,
            pulsingSpiralPathSlower,

            imageSize,
            borderSize,
            imageOffset,

            switchToStaticBorder,
            switchToAnimatedBorder,
        };
    }
})
</script>

<style scoped lang="scss">
$transitionDurations: 200ms;

div {
    position: relative;
    width: 228px;
    height: 228px;
}
img {
    position: absolute;
    border-radius: 100px;
    transition: width $transitionDurations, height $transitionDurations;
    mask-image: radial-gradient(rgba(0, 0, 0, 1.0) 50%, transparent 70%);
}

svg {
    position: absolute;
    top: 0;
    left: 0;
    z-index: -1;
}

@keyframes spin-clockwise {
    from {
        transform: rotate(0deg);
    }

    to {
        transform: rotate(359deg);
    }
}

@keyframes spin-counterclockwise {
    from {
        transform: rotate(359deg);
    }

    to {
        transform: rotate(0deg);
    }
}

.border {
    animation-duration: 60s;
    animation-iteration-count: infinite;
    animation-timing-function: linear;

    &.animationStop {
        animation-name: none;
        display: none;
    }
}

.border--clockwise {
    animation-name: spin-clockwise;
}

.border--counterclockwise {
    animation-name: spin-counterclockwise;
}

.staticBackground {
    background-color: #38483e;
    border-radius: 114px;
    width: 228px;
    height: 228px;
    position: absolute;
    top: 0;
    left: 0;
    z-index: -1;
    transform: scale(0.5);

    transition: transform $transitionDurations, width $transitionDurations, height $transitionDurations;

    &.staticBackground--expanded {
        transform: scale(1);
    }
}
</style>
