<template>
    <audio
        ref="audioElement"
        :controls="native ? '' : null"
        :src="source"
        :autoplay="autoplay"
        :loop="isLoop"
        :muted="isMuted"
    ></audio>
    <div
        :class="`
            w-full
            flex flex-wrap
            rounded-md
            bg-white
            p-4 xl:p-8
            shadow
            shadow-${tailwindSettings.theme.primary}-800
            ${debug ? 'debug' : ''}
            `"
        v-if="!native"
    >
        <!-- CURRENT TIME -->
        <div class="w-1/2 text-left text-indigo-900 order-3 xl:order-2 my-2 select-none">
            {{ timeCurrent }}
        </div>

        <!-- TIME RANGE -->
        <div class="w-full order-2 xl:order-1 mt-4 xl:mt-0">
        <input
            class="
                w-full
                appearance-none
                h-2
                bg-indigo-100
                rounded-full
                cursor-pointer
            "
            type="range"
            :value="audioElement ? audioElement.currentTime : '0'"
            min="0"
            :max="audioElement ? audioElement.duration : '0'"
            step="1"
            @focus="play(false, stayPaused)"
            @input="setCurrentTime($event)"
            @change="$event.target.blur()"
            @blur="play(true, stayPaused)"
        />
        </div>

        <!-- REMAINING TIME -->
        <div class="w-1/2 text-right text-indigo-900 order-4 xl:order-3 my-2 select-none">
            {{ timeRemaining }}
        </div>

        <!-- GROUP PLAYER ACTIONS -->
        <div class="w-full xl:w-1/3 flex flex-wrap xl:flex-nowrap justify-start items-center order-5 xl:order-4">

            <!-- Button Play / Pause -->
            <div class="flex text-slate-900 w-full xl:w-auto justify-center xl:justify-start">
                <button @click="play(!isPlaying, null)">
                    <svg
                        class="w-16 h-16 fill-slate-900"
                        stroke-width="1.5"
                        viewBox="0 0 24 24"
                        fill="none"
                        xmlns="http://www.w3.org/2000/svg"
                        v-if="!isPlaying"
                    >
                        <path
                            d="M6.90588 4.53682C6.50592 4.2998 6 4.58808 6 5.05299V18.947C6 19.4119 6.50592 19.7002 6.90588 19.4632L18.629 12.5162C19.0211 12.2838 19.0211 11.7162 18.629 11.4838L6.90588 4.53682Z"
                            stroke="currentColor"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                        />
                    </svg>
                    <svg
                        class="w-16 h-16 fill-slate-900"
                        stroke-width="1.5"
                        viewBox="0 0 24 24"
                        fill="none"
                        xmlns="http://www.w3.org/2000/svg"
                        v-if="isPlaying"
                    >
                        <path
                            d="M6 18.4V5.6C6 5.26863 6.26863 5 6.6 5H9.4C9.73137 5 10 5.26863 10 5.6V18.4C10 18.7314 9.73137 19 9.4 19H6.6C6.26863 19 6 18.7314 6 18.4Z"
                            stroke="currentColor"
                            stroke-width="1.5"
                        />
                        <path
                            d="M14 18.4V5.6C14 5.26863 14.2686 5 14.6 5H17.4C17.7314 5 18 5.26863 18 5.6V18.4C18 18.7314 17.7314 19 17.4 19H14.6C14.2686 19 14 18.7314 14 18.4Z"
                            stroke="currentColor"
                            stroke-width="1.5"
                        />
                    </svg>
                </button>
            </div>

            <!-- Button Loop -->
            <div class="flex ml-4 text-indigo-900 transition hover:scale-110">
                <button
                    @click="toggleLoop()"
                    :class="!isLoop ? 'text-indigo-100' : null"
                >
                    <svg
                        class="w-10 h-10"
                        stroke-width="1.5"
                        viewBox="0 0 24 24"
                        fill="none"
                        xmlns="http://www.w3.org/2000/svg"
                    >
                        <path
                            d="M17 17H8C6.33333 17 3 16 3 12C3 8 6.33333 7 8 7H16C17.6667 7 21 8 21 12C21 13.4943 20.5348 14.57 19.865 15.3312"
                            stroke="currentColor"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                        />
                        <path
                            d="M14.5 14.5L17 17L14.5 19.5"
                            stroke="currentColor"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                        />
                    </svg>
                </button>
            </div>

            <!-- Button Download -->
            <div class="flex ml-4 text-indigo-900 transition hover:scale-110"
                v-if="download"
            >
                <a :href="source" download>
                    <svg
                        class="w-10 h-10"
                        stroke-width="1.5"
                        viewBox="0 0 24 24"
                        fill="none"
                        xmlns="http://www.w3.org/2000/svg"
                    >
                        <path
                            d="M6 20L18 20"
                            stroke="currentColor"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                        />
                        <path
                            d="M12 4V16M12 16L15.5 12.5M12 16L8.5 12.5"
                            stroke="currentColor"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                        />
                    </svg>
                </a>
            </div>
        </div>

        <!-- VISUALIZER -->
        <div class="w-full xl:w-1/3 flex justify-center items-center order-1 xl:order-5"
            v-if="visualizer"
        >
            <canvas ref="canvasElement" class="w-full h-16 rounded-md"></canvas>
        </div>

        <!-- GROUP VOLUME -->
        <div class="w-full xl:w-1/3 flex text-indigo-900 justify-end items-center order-6 xl:order-6"
        >
            <button @click="toggleMuted()">
                <svg
                    class="w-10 h-10 transition hover:scale-110"
                    stroke-width="1.5"
                    viewBox="0 0 24 24"
                    fill="none"
                    xmlns="http://www.w3.org/2000/svg"
                    v-if="volumeCurrent > 0.59 && !isMuted"
                >
                    <path
                        d="m 0.75,14 v -4 c 0,-0.55228 0.44772,-1 1,-1 H 4.44722 C 4.64465,9 4.83766,8.94156 5.00192,8.83205 L 9.1953,6.03647 C 9.8599,5.59343 10.75,6.06982 10.75,6.86852 V 17.1315 c 0,0.7987 -0.8901,1.2751 -1.5547,0.832 L 5.00192,15.1679 C 4.83766,15.0584 4.64465,15 4.44722,15 H 1.75 c -0.55228,0 -1,-0.4477 -1,-1 z"
                        stroke="currentColor"
                        stroke-width="1.5"
                    />
                    <path
                        d="m 15.25,7.5 c 0,0 1.5,1.5 1.5,4 0,2.5 -1.5,4 -1.5,4"
                        stroke="currentColor"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                    />
                    <path
                        d="m 18.25,4.5 c 0,0 2.5,2.5 2.5,7 0,4.5 -2.5,7 -2.5,7"
                        stroke="currentColor"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                    />
                </svg>
                <svg
                    :class="twButtonVolumeSVG"
                    stroke-width="1.5"
                    viewBox="0 0 24 24"
                    fill="none"
                    xmlns="http://www.w3.org/2000/svg"
                    v-if="
                        volumeCurrent < 0.6 && volumeCurrent > 0.09 && !isMuted
                    "
                >
                    <path
                        d="m 15.25,7.5 c 0,0 1.5,1.5 1.5,4 0,2.5 -1.5,4 -1.5,4"
                        stroke="currentColor"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                    />
                    <path
                        d="m 0.75,14 v -4 c 0,-0.55228 0.44772,-1 1,-1 H 4.44722 C 4.64465,9 4.83766,8.94156 5.00192,8.83205 L 9.1953,6.03647 C 9.8599,5.59343 10.75,6.06982 10.75,6.86852 V 17.1315 c 0,0.7987 -0.8901,1.2751 -1.5547,0.832 L 5.00192,15.1679 C 4.83766,15.0584 4.64465,15 4.44722,15 H 1.75 c -0.55228,0 -1,-0.4477 -1,-1 z"
                        stroke="currentColor"
                        stroke-width="1.5"
                    />
                </svg>
                <svg
                    :class="twButtonVolumeSVG"
                    stroke-width="1.5"
                    viewBox="0 0 24 24"
                    fill="none"
                    xmlns="http://www.w3.org/2000/svg"
                    v-if="volumeCurrent < 0.1 && volumeCurrent > 0 && !isMuted"
                >
                    <path
                        d="m 0.75,14 v -4 c 0,-0.55228 0.44772,-1 1,-1 H 4.4472 C 4.6446,9 4.8377,8.94156 5.0019,8.83205 L 9.1953,6.03647 C 9.8599,5.59343 10.75,6.06982 10.75,6.86852 V 17.1315 c 0,0.7987 -0.8901,1.2751 -1.5547,0.832 L 5.0019,15.1679 C 4.8377,15.0584 4.6446,15 4.4472,15 H 1.75 c -0.55228,0 -1,-0.4477 -1,-1 z"
                        stroke="currentColor"
                        stroke-width="1.5"
                    />
                </svg>
                <svg
                    :class="twButtonVolumeSVG"
                    stroke-width="1.5"
                    viewBox="0 0 24 24"
                    fill="none"
                    xmlns="http://www.w3.org/2000/svg"
                    v-if="isMuted || volumeCurrent === 0"
                >
                    <path
                        d="m 14.75,14 2.0005,-2 m 1.9995,-2 -1.9995,2 m 0,0 -2.0005,-2 m 2.0005,2 1.9995,2"
                        stroke="currentColor"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                    />
                    <path
                        d="m 0.75,14 v -4 c 0,-0.55228 0.44772,-1 1,-1 H 4.44722 C 4.64465,9 4.83766,8.94156 5.00192,8.83205 L 9.1953,6.03647 C 9.8599,5.59343 10.75,6.06982 10.75,6.86852 V 17.1315 c 0,0.7987 -0.8901,1.2751 -1.5547,0.832 L 5.00192,15.1679 C 4.83766,15.0584 4.64465,15 4.44722,15 H 1.75 c -0.55228,0 -1,-0.4477 -1,-1 z"
                        stroke="currentColor"
                        stroke-width="1.5"
                    />
                </svg>
            </button>
            <input
                class="
                    appearance-none
                    h-2
                    ml-2
                    bg-indigo-100
                    rounded-full
                    cursor-pointer
                    order-0
                "
                type="range"
                :value="audioElement ? audioElement.volume : 1"
                min="0"
                max="1"
                step="0.01"
                @input="setCurrentVolume($event)"
            />
        </div>
    </div>
</template>

<script>
import { ref, onMounted, watch } from "vue";

export default {
    name: "vue-tailwind-player",
    props: {
        autoplay: {
            type: Boolean,
            default: false,
        },
        native: {
            type: Boolean,
            default: false,
        },
        loop: {
            type: Boolean,
            default: false,
        },
        muted: {
            type: Boolean,
            default: false,
        },
        source: {
            type: String,
            default: "",
        },
        download: {
            type: Boolean,
            default: false,
        },
        showTimeline: {
            type: Boolean,
            default: false,
        },
        visualizer: {
            type: String,
            default: null,
        },
        twVisualizerCanvasColors: {
            type: Object,
            default: {
                theme: {
                    primary: "rgb(224, 231, 255)",
                    secondary: "rgb(15, 23, 42)",
                },
            },
        },
        tailwindSettings: {
            type: Object,
            default: {
                theme: {
                    primary: "slate",
                    secondary: "indigo",
                },
                shadow: "shadow",
                rounded: "rounded",
            },
        },
        debug: {
            type: Boolean,
            default: false,
        },
    },
    setup(props) {
        let audioContext = new AudioContext();
        let analyser = audioContext.createAnalyser();
        let audioSource = null;

        const audioElement = ref(null);
        const canvasElement = ref(HTMLCanvasElement);
        const isPlaying = ref(props.autoplay);
        const isLoop = ref(props.loop);
        const isMuted = ref(props.muted);
        const timeCurrent = ref("0:00");
        const timeRemaining = ref("0:00");
        const stayPaused = ref(!props.autoplay);
        const volumeCurrent = ref(1);
        const volumePrevious = ref(1);

        onMounted(async () => {
            audioElement.value.onpause = function () {
                isPlaying.value = false;
            };

            audioElement.value.onplay = function () {
                isPlaying.value = true;
            };

            audioElement.value.ontimeupdate = function () {
                timeCurrent.value = formatDuration(
                    audioElement.value.currentTime
                );
            };

            audioElement.value.oncanplay = function () {
                timeRemaining.value = formatDuration(
                    audioElement.value.duration
                );
            };

            audioElement.value.onvolumechange = function () {
                volumeCurrent.value = audioElement.value.volume;
                isMuted.value = audioElement.value.muted;
            };

            // Visualizer
            audioSource = audioContext.createMediaElementSource(
                audioElement.value
            );
            audioSource.connect(analyser);
            visualize();
        });

        const visualize = () => {
            let canvasContext = canvasElement.value.getContext("2d");
            let width = canvasElement.value.width;
            let height = canvasElement.value.height;
            analyser.fftSize = 256;
            const bufferLength = analyser.frequencyBinCount;
            const dataArray = new Float32Array(bufferLength);
            canvasContext.clearRect(0, 0, width, height);

            function draw() {
                if (props.visualizer === "frequencybars") {
                    requestAnimationFrame(draw);

                    analyser.getFloatFrequencyData(dataArray);

                    canvasContext.fillStyle =
                        props.twVisualizerCanvasColors.theme.primary;
                    canvasContext.fillRect(0, 0, width, height);

                    var barWidth = (width / bufferLength) * 2.5;
                    var barHeight;
                    var x = 0;

                    for (var i = 0; i < bufferLength; i++) {
                        barHeight = (dataArray[i] + 140) * 2;

                        const colorString =
                            props.twVisualizerCanvasColors.theme.secondary;
                        const arrayRGB = colorString
                            .substring(
                                colorString.indexOf("(") + 1,
                                colorString.lastIndexOf(")")
                            )
                            .split(/,\s*/);

                        let alpha = (barHeight * 100) / height;
                        alpha = alpha < 0 ? 0 : alpha / 100;

                        canvasContext.fillStyle =
                            "rgba(" +
                            arrayRGB[0] +
                            ", " +
                            arrayRGB[1] +
                            ", " +
                            arrayRGB[2] +
                            ", " +
                            alpha +
                            ")";

                        canvasContext.fillRect(
                            x,
                            height - barHeight / 2,
                            barWidth,
                            barHeight / 2
                        );

                        x += barWidth + 1;
                    }
                } else if (props.visualizer === "sinewave") {
                    requestAnimationFrame(draw);

                    analyser.getFloatTimeDomainData(dataArray);

                    canvasContext.fillStyle =
                        props.twVisualizerCanvasColors.theme.primary;
                    canvasContext.fillRect(0, 0, width, height);

                    canvasContext.lineWidth = 2;
                    canvasContext.strokeStyle =
                        props.twVisualizerCanvasColors.theme.secondary;

                    canvasContext.beginPath();

                    var sliceWidth = (width * 1.0) / bufferLength;
                    var x = 0;

                    for (var i = 0; i < bufferLength; i++) {
                        var v = dataArray[i] * 200.0;
                        var y = height / 2 + v;

                        if (i === 0) {
                            canvasContext.moveTo(x, y);
                        } else {
                            canvasContext.lineTo(x, y);
                        }

                        x += sliceWidth;
                    }

                    canvasContext.lineTo(width, height / 2);
                    canvasContext.stroke();
                }
            }
            audioSource.connect(audioContext.destination);
            draw();
        };

        const formatDuration = (seconds) => {
            const h = Math.floor(seconds / 3600);
            const m = Math.floor((seconds % 3600) / 60);
            const s = Math.round(seconds % 60);
            return [h, m > 9 ? m : h ? "0" + m : m || "0", s > 9 ? s : "0" + s]
                .filter((a) => a)
                .join(":");
        };

        const play = (state, pause) => {
            if (state) {
                if (!pause) {
                    audioElement.value.play();
                }
            } else {
                audioElement.value.pause();
            }
            if (pause === null) {
                stayPaused.value = !state;
            }
        };

        const setCurrentTime = (event) => {
            audioElement.value.currentTime = event.target.value;
        };

        const setCurrentVolume = (event) => {
            audioElement.value.volume = event.target.value;
        };

        const toggleMuted = () => {
            isMuted.value = !isMuted.value;
        };

        const toggleLoop = () => {
            isLoop.value = !isLoop.value;
        };

        watch(
            () => isMuted.value,
            (newMuted) => {
                if (newMuted) {
                    volumePrevious.value = audioElement.value.volume;
                    audioElement.value.volume = 0;
                } else {
                    audioElement.value.volume =
                        volumePrevious.value > 0 ? volumePrevious.value : 0.5;
                }
            }
        );

        return {
            audioElement,
            canvasElement,
            isPlaying,
            isLoop,
            isMuted,
            play,
            timeCurrent,
            timeRemaining,
            setCurrentTime,
            setCurrentVolume,
            stayPaused,
            volumeCurrent,
            volumePrevious,
            toggleMuted,
            toggleLoop,
        };
    },
};
</script>

<style scoped>
input[type="range"]::-moz-range-progress,
input[type="range"]::-webkit-range-value {
    @apply h-2 rounded-full bg-indigo-600;
}

input[type="range"]::-moz-range-thumb,
input[type="range"]::-webkit-slider-thumb {
    @apply appearance-none w-5 h-5 border-0 rounded-full bg-indigo-600 shadow;
}

.debug div {
    border: 1px solid red !important;
}
.debug button,
.debug input {
    border: 1px solid blue !important;
}
.debug canvas {
    border: 1px solid orange !important;
}
</style>