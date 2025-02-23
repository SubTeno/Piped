<template>
    <!-- desktop view -->
    <div
        v-if="!mobileLayout"
        :class="
            theater
                ? 'pp-chapters flex-col overflow-y-scroll max-h-90vh min-h-64 lt-lg:hidden'
                : 'pp-chapters flex-col overflow-y-scroll max-h-75vh min-h-64 lt-lg:hidden'
        "
    >
        <h6 aria-label="chapters" title="chapters">{{ $t("video.chapters") }} - {{ chapters.length }}</h6>
        <div
            :key="chapter.start"
            v-for="(chapter, index) in chapters"
            @click="$emit('seek', chapter.start)"
            class="chapter efy_anim_pulse"
            :class="{ 'pp-chapter-active': isCurrentChapter(index) }"
        >
            <div class="flex">
                <img :src="chapter.image" :alt="chapter.title" />
                <div class="flex flex-col m-2">
                    <span :title="chapter.title" v-text="index + 1 + '. ' + chapter.title" class="font-bold" />
                    <span class="font-bold" v-text="timeFormat(chapter.start)" />
                </div>
            </div>
        </div>
    </div>
    <!-- mobile view -->
    <div v-else class="pp-chapters pp-mobile flex overflow-x-auto">
        <div
            :key="chapter.start"
            v-for="(chapter, index) in chapters"
            @click="$emit('seek', chapter.start)"
            class="chapter efy_anim_pulse"
            :class="{ 'pp-chapter-active': isCurrentChapter(index) }"
        >
            <img :src="chapter.image" :alt="chapter.title" />
            <div class="m-1 flex">
                <span class="text-truncate font-bold" :title="chapter.title" v-text="chapter.title" />
                <span class="px-1 font-bold" v-text="timeFormat(chapter.start)" />
            </div>
        </div>
    </div>
</template>

<style>
.chapter {
    @apply cursor-pointer self-center p-2.5;
}
.pp-mobile .chapter img {
    @apply w-full h-full;
}
.chapter img {
    @apply w-3/10 h-3/10;
}
.text-truncate {
    @apply truncate overflow-hidden inline-block w-10em;
}
</style>

<script setup>
import { defineProps, defineEmits } from "vue";

const props = defineProps({
    chapters: Object,
    theater: {
        type: Boolean,
        default: () => false,
    },
    mobileLayout: {
        type: Boolean,
        default: () => true,
    },
    playerPosition: {
        type: Number,
        default: () => 0,
    },
});

const isCurrentChapter = index => {
    return (
        props.playerPosition >= props.chapters[index].start &&
        props.playerPosition < (props.chapters[index + 1]?.start ?? Infinity)
    );
};

defineEmits(["seek"]);
</script>
