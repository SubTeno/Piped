<template>
    <div class="flex flex-col min-h-[calc(100vh_-_32rem)] justify-between">
        <div>
            <NavBar />
            <div class="pp-base reset">
                <router-view v-slot="{ Component }">
                    <keep-alive :max="5">
                        <component :is="Component" :key="$route.fullPath" />
                    </keep-alive>
                </router-view>
            </div>
        </div>
        <FooterComponent />
    </div>
</template>

<style>
/*Global*/
:root {
    --efy_color1_var: 239, 68, 68;
    --efy_color2_var: 220, 38, 38;
    --efy_radius: 12rem;
    --efy_sidebar_button: right_middle;
    --efy_font_family: "nunito", sans-serif, -apple-system, BlinkMacSystemFont, Segoe UI, Roboto, Helvetica Neue, Arial,
        Noto Sans, Apple Color Emoji, Segoe UI Emoji, Segoe UI Symbol, Noto Color Emoji;
    --efy_body_width: unset;
    --efy_audio_path: ./audio;
    --efy_modules: efy_quick, efy_mode, efy_filters, efy_backup, efy_accessibility, efy_audio;

    --efy_module_quick: on;
    --efy_module_mode: on;
    --efy_module_filters: on;
    --efy_module_backup: on;
    --efy_module_accessibility: on;
    --efy_module_audio: on;
}

body {
    @apply max-w-100vw p-16rem;
}

/*Default Mode*/
[efy_mode="default_mode"] {
    color-scheme: dark;
    --efy_bg_var: 255, 255, 255;
    --efy_bg: rgb(15, 15, 15);
    --efy_text: rgb(220, 220, 220);
}

/*Radius*/
input,
.btn,
button,
.shaka-video-container,
.shaka-video-container video,
.video-grid div,
.pp-show-recs div,
.grid .comment,
.shaka-scrim-container,
.suggestion-selected,
.pp-mobile-nav a,
.shaka-text-container span > span > span {
    border-radius: var(--efy_radius);
}

/*Radius 0*/
.video-grid img,
.thumbnail-overlay,
.thumbnail-left,
.thumbnail-right {
    border-radius: var(--efy_radius0);
}

/*Radius 2*/
.suggestions-container,
.modal-container {
    border-radius: var(--efy_radius2);
}
</style>

<script>
import NavBar from "./components/NavBar.vue";
import FooterComponent from "./components/FooterComponent.vue";
export default {
    components: {
        NavBar,
        FooterComponent,
    },
    mounted() {
        if (this.getPreferenceBoolean("watchHistory", false))
            if ("indexedDB" in window) {
                const request = indexedDB.open("piped-db", 1);
                request.onupgradeneeded = function () {
                    const db = request.result;
                    console.log("Upgrading object store.");
                    if (!db.objectStoreNames.contains("watch_history")) {
                        const store = db.createObjectStore("watch_history", { keyPath: "videoId" });
                        store.createIndex("video_id_idx", "videoId", { unique: true });
                        store.createIndex("id_idx", "id", { unique: true, autoIncrement: true });
                    }
                };
                request.onsuccess = e => {
                    window.db = e.target.result;
                };
            } else console.log("This browser doesn't support IndexedDB");

        const App = this;

        (async function () {
            const defaultLang = await App.defaultLangage;
            const locale = App.getPreferenceString("hl", defaultLang);
            if (locale !== App.TimeAgoConfig.locale) {
                const localeTime = await import(
                    "./../node_modules/javascript-time-ago/locale/" + locale + ".json"
                ).then(module => module.default);
                App.TimeAgo.addLocale(localeTime);
                App.TimeAgoConfig.locale = locale;
            }
            if (window.i18n.global.locale.value !== locale) {
                if (!window.i18n.global.availableLocales.includes(locale)) {
                    const messages = await import(`./locales/${locale}.json`).then(module => module.default);
                    window.i18n.global.setLocaleMessage(locale, messages);
                }
                window.i18n.global.locale.value = locale;
            }
        })();
    },
};
</script>
