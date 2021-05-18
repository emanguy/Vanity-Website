<template>
    <div>
        <img src="/img/headshot-square.jpeg" alt="Picture of me">
        <svg id="border1" width="228" height="228">
            <radialGradient id="greenGradient">
                <stop offset="50%" stop-color="#4D6958FF"></stop>
                <stop offset="80%" stop-color="#4D6958CC"></stop>
                <stop offset="100%" stop-color="#61846F88"></stop>
            </radialGradient>
            <path :d="pulsingSpiralPathFaster" fill="url(#greenGradient)" stroke="none" transform="translate(114,114)"></path>
        </svg>
        <svg id="border2" width="228" height="228">
            <radialGradient id="greenGradient">
                <stop offset="50%" stop-color="#4D6958FF"></stop>
                <stop offset="80%" stop-color="#4D6958CC"></stop>
                <stop offset="100%" stop-color="#61846F88"></stop>
            </radialGradient>
            <path :d="pulsingSpiralPathSlower" fill="url(#greenGradient)" stroke="none" transform="translate(114,114)"></path>
        </svg>
    </div>
</template>

<script lang="ts">
import {computed, defineComponent, ref} from 'vue'
import * as d3 from 'd3';

export default defineComponent({
    name: 'DecoratedPicture',
    props: {
        msg: {
            type: String,
            required: true
        }
    },
    setup: () => {
        const step = Math.PI/64;

        const radialSineAmplitudeSlower = ref(7);
        const radialSineAmplitudeFaster = ref(7);
        const irregularity = Array.from({length: 130}, () => (Math.random() + 0.5) * -7);
        const generatedSpiralSlower = computed(() => Array.from({ length: 130 }, (_, i) => [step * i, 107 + (radialSineAmplitudeSlower.value * Math.sin(Math.PI/4 * i)) + irregularity[i]]));
        const generatedSpiralFaster = computed(() => Array.from({ length: 130 }, (_, i) => [step * i, 107 + (radialSineAmplitudeFaster.value * Math.sin(Math.PI/4 * i)) + irregularity[i]]));

        // @ts-ignore
        const pulsingSpiralPathSlower = computed(() => d3.lineRadial().curve(d3.curveCardinalOpen)(generatedSpiralSlower.value));
        // @ts-ignore
        const pulsingSpiralPathFaster = computed(() => d3.lineRadial().curve(d3.curveCardinalOpen)(generatedSpiralFaster.value));

        const framesInOneSecond = 60;
        let frameNum = 0;
        let fasterFrameNum = 1.5 * framesInOneSecond;

        window.setInterval(() => {
            frameNum = (frameNum % (5 * framesInOneSecond)) + 1;
            fasterFrameNum = (fasterFrameNum % (3 * framesInOneSecond)) + 1;

            // We want to go from 7 to 0 and back again in a linear, absolute value fashion
            let animationCompletionPercentage = frameNum / (5 * framesInOneSecond);
            let fasterAnimationCompletionPercentage = fasterFrameNum / (3 * framesInOneSecond);

            radialSineAmplitudeSlower.value = Math.abs(animationCompletionPercentage * 2 - 1) * 7;
            radialSineAmplitudeFaster.value = Math.abs(fasterAnimationCompletionPercentage * 2 - 1) * 7;
        }, 1000/framesInOneSecond);

        return { pulsingSpiralPathFaster, pulsingSpiralPathSlower }
    }
})
</script>

<style scoped lang="scss">
@import '../../assets/styles/vendor-util';

div {
    position: relative;
    width: 228px;
    height: 228px;
}
img {
    position: absolute;
    top: 14px;
    left: 14px;
    border-radius: 100px;
    width: 200px;
    height: 200px;
    @include vendor-prefix('mask-image', 'radial-gradient(rgba(0, 0, 0, 1.0) 50%, transparent 70%)');
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

#border1 {
    animation-duration: 60s;
    animation-iteration-count: infinite;
    animation-timing-function: linear;
    animation-name: spin-clockwise;
}

#border2 {
    animation-duration: 40s;
    animation-iteration-count: infinite;
    animation-timing-function: linear;
    animation-name: spin-counterclockwise;
}
</style>
