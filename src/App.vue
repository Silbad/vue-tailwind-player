<template>
    <div
        class="
            grid
            place-items-center
            h-screen
            bg-gradient-to-br
            from-slate-900
            to-indigo-600
        "
    >
        <div class="w-1/2">
            <!-- sinewave or frequencybars -->
            <VueTailwindPlayer
                source="Les Bernardo - Le Malheur De Sophie.mp3"
                autoplay
                download
                visualizer="sinewave"
            />
        </div>
    </div>
</template>

<script>
import VueTailwindPlayer from "@/components/VueTailwindPlayer.vue";

export default {
    name: "vue-tailwind-player-demo",
    components: {
        VueTailwindPlayer,
    },
    setup() {
        console.log(
            "%c %c %c Vue Tailwind Player ★ version " + __APP_VERSION__ + " ",
            "background: #35495e",
            "background: #42b883",
            "background: #fff"
        );
        return {};
    },
};
</script>
