<template>
    <div v-if="video && isEmbed" class="absolute top-0 left-0 h-full w-full bg-black z-50">
        <VideoPlayer
            ref="videoPlayer"
            :video="video"
            :sponsors="sponsors"
            :playlist="playlist"
            :index="index"
            :selected-auto-play="false"
            :selected-auto-loop="selectedAutoLoop"
            :is-embed="isEmbed"
        />
    </div>

    <div v-if="video && !isEmbed" class="w-full">
        <ErrorHandler v-if="video && video.error" :message="video.message" :error="video.error" />

        <div v-show="!video.error">
            <div
                :class="[
                    isMobile ? 'flex-col' : 'flex',
                    showThea ? 'w-screen relative right-[10vw] max-h-[90vh]' : 'max-h-[75vh]',
                ]"
            >
                <VideoPlayer
                    ref="videoPlayer"
                    :video="video"
                    :sponsors="sponsors"
                    :playlist="playlist"
                    :index="index"
                    :selected-auto-play="selectedAutoPlay"
                    :selected-auto-loop="selectedAutoLoop"
                    @timeupdate="onTimeUpdate"
                />
                <ChaptersBar
                    :mobileLayout="isMobile"
                    v-if="video?.chapters?.length > 0 && showChapters"
                    :chapters="video.chapters"
                    :player-position="currentTime"
                    :theater="showThea"
                    @seek="navigate"
                />
            </div>
            <!-- video title -->
            <div class="pp-video-title font-bold mt-2 text-2xl break-words" v-text="video.title" />
            <div class="pp-bellow-video flex flex-wrap mt-3 mb-3">
                <!-- views / date -->
                <div class="flex flex-auto children:ml-2">
                    <span v-t="{ path: 'video.views', args: { views: addCommas(video.views) } }" />
                    <span> • </span>
                    <span v-text="uploadDate" />
                </div>
                <!-- Likes/dilikes -->
                <div class="pp-likes flex children:mr-2">
                    <template v-if="video.likes >= 0">
                        <div class="flex">
                            <div class="i-fa-solid:thumbs-up" />
                            <strong class="ml-1" v-text="addCommas(video.likes)" />
                        </div>
                        <div class="flex">
                            <div class="i-fa-solid:thumbs-down" />
                            <strong class="ml-1" v-text="video.dislikes >= 0 ? addCommas(video.dislikes) : '?'" />
                        </div>
                    </template>
                    <template v-if="video.likes < 0">
                        <div>
                            <strong v-t="'video.ratings_disabled'" />
                        </div>
                    </template>
                </div>
            </div>
            <!-- Channel info & options flex container -->
            <div class="pp-watch-bellow-options flex">
                <!-- Channel Image & Info -->
                <div class="flex">
                    <router-link v-if="video.uploaderUrl" class="link ml-1.5" :to="video.uploaderUrl">
                        <img :src="video.uploaderAvatar" alt="" loading="lazy" class="w-auto" />
                    </router-link>
                    <div>
                        <router-link v-if="video.uploaderUrl" class="link ml-1.5" :to="video.uploaderUrl">{{
                            video.uploader
                        }}</router-link>
                        <!-- Verified Badge -->
                        <font-awesome-icon class="ml-1" v-if="video.uploaderVerified" icon="check" />
                        <p class="ml-1.5">{{ numberFormat(video.uploaderSubscriberCount) }} Subscribers</p>
                    </div>
                </div>

                <div class="pp-watch-buttons">
                    <!-- Subscribe button -->
                    <button
                        class="btn"
                        @click="subscribeHandler"
                        v-t="{
                            path: `actions.${subscribed ? 'unsubscribe' : 'subscribe'}`,
                        }"
                    />
                    <!-- RSS Feed button -->
                    <a
                        aria-label="RSS feed"
                        title="RSS feed"
                        role="button"
                        v-if="video.uploaderUrl"
                        :href="`${apiUrl()}/feed/unauthenticated/rss?channels=${video.uploaderUrl.split('/')[2]}`"
                        target="_blank"
                        class="btn flex-col"
                    >
                        <font-awesome-icon icon="rss" />
                    </a>
                    <!-- watch on youtube button -->
                    <button class="btn" @click="showShareModal = !showShareModal">
                        <i18n-t class="lt-lg:hidden" keypath="actions.share" tag="strong"></i18n-t>
                        <font-awesome-icon class="mx-1.5 ml-1" icon="fa-share" />
                    </button>
                    <!-- LBRY -->
                    <a v-if="video.lbryId" :href="'https://odysee.com/' + video.lbryId" class="btn">
                        <i18n-t keypath="player.watch_on" tag="strong">LBRY</i18n-t>
                    </a>
                    <!-- listen / watch toggle -->
                    <router-link
                        :to="toggleListenUrl"
                        :aria-label="(isListening ? 'Watch ' : 'Listen to ') + video.title"
                        :title="(isListening ? 'Watch ' : 'Listen to ') + video.title"
                        class="btn flex-col"
                    >
                        <font-awesome-icon :icon="isListening ? 'tv' : 'headphones'" />
                    </router-link>
                    <!-- Playlist Add button -->
                    <button class="btn" v-if="authenticated" @click="showModal = !showModal">
                        {{ $t("actions.add_to_playlist") }}<font-awesome-icon class="ml-1" icon="circle-plus" />
                    </button>
                    <PlaylistAddModal v-if="showModal" :video-id="getVideoId()" @close="showModal = !showModal" />
                    <ShareModal
                        v-if="showShareModal"
                        :video-id="getVideoId()"
                        :current-time="currentTime"
                        @close="showShareModal = !showShareModal"
                    />
                </div>
            </div>

            <hr />

            <div efy_select>
                <input v-if="!isMobile" id="showThea" type="checkbox" v-model="showThea" @change="onChange($event)" />
                <label v-if="!isMobile" for="showThea" v-t="'Theater'" />
                <input id="showDesc" type="checkbox" v-model="showDesc" @change="onChange($event)" />
                <label for="showDesc" v-t="'actions.show_description'" />
                <input id="showRecs" type="checkbox" v-model="showRecs" @change="onChange($event)" />
                <label for="showRecs" v-t="'actions.show_recommendations'" />
                <input id="chkAutoLoop" v-model="selectedAutoLoop" type="checkbox" @change="onChange($event)" />
                <label for="chkAutoLoop" v-text="`${$t('actions.loop_this_video')}`" />
                <input id="chkAutoPlay" v-model="selectedAutoPlay" type="checkbox" @change="onChange($event)" />
                <label for="chkAutoPlay" v-text="`${$t('actions.auto_play_next_video')}`" />
                <span v-show="video?.chapters?.length > 0">
                    <input id="showChapters" type="checkbox" v-model="showChapters" />
                    <label class="ml-2" for="showChapters" v-t="'actions.show_chapters'" />
                </span>
            </div>

            <div v-show="showDesc" class="break-words mb-2" v-html="purifyHTML(video.description)" />
            <div
                v-if="showDesc && sponsors && sponsors.segments"
                v-text="`${$t('video.sponsor_segments')}: ${sponsors.segments.length}`"
            />
            <hr />
        </div>

        <div class="grid pp-rec-vids">
            <div v-if="!commentsEnabled" class="">
                <p class="text-center mt-8" v-t="'comment.user_disabled'"></p>
            </div>
            <div v-else-if="!comments" class="">
                <p class="text-center mt-8" v-t="'comment.loading'"></p>
            </div>
            <div v-else-if="comments.disabled" class="">
                <p class="text-center mt-8" v-t="'comment.disabled'"></p>
            </div>
            <div v-else ref="comments" class="">
                <CommentItem
                    v-for="comment in comments.comments"
                    :key="comment.commentId"
                    :comment="comment"
                    :uploader="video.uploader"
                    :video-id="getVideoId()"
                />
            </div>

            <div v-if="video" class="order-first sm:order-last">
                <PlaylistVideos
                    v-if="playlist"
                    :playlist-id="playlistId"
                    :playlist="playlist"
                    :selected-index="index"
                />
                <div v-show="showRecs" class="pp-show-recs">
                    <VideoItem
                        v-for="related in video.relatedStreams"
                        :key="related.url"
                        :video="related"
                        height="94"
                        width="168"
                    />
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import VideoPlayer from "./VideoPlayer.vue";
import VideoItem from "./VideoItem.vue";
import ErrorHandler from "./ErrorHandler.vue";
import CommentItem from "./CommentItem.vue";
import ChaptersBar from "./ChaptersBar.vue";
import PlaylistAddModal from "./PlaylistAddModal.vue";
import ShareModal from "./ShareModal.vue";
import PlaylistVideos from "./PlaylistVideos.vue";

export default {
    name: "App",
    components: {
        VideoPlayer,
        VideoItem,
        ErrorHandler,
        CommentItem,
        ChaptersBar,
        PlaylistAddModal,
        ShareModal,
        PlaylistVideos,
    },
    data() {
        const smallViewQuery = window.matchMedia("(max-width: 640px)");
        return {
            video: {
                title: "Loading...",
            },
            playlistId: null,
            playlist: null,
            index: null,
            sponsors: null,
            selectedAutoLoop: false,
            selectedAutoPlay: null,
            showDesc: true,
            showThea: false,
            showRecs: true,
            showChapters: true,
            comments: null,
            subscribed: false,
            channelId: null,
            active: true,
            smallViewQuery: smallViewQuery,
            smallView: smallViewQuery.matches,
            showModal: false,
            showShareModal: false,
            isMobile: true,
            currentTime: 0,
        };
    },
    computed: {
        isListening(_this) {
            return _this.getPreferenceBoolean("listen", false);
        },
        toggleListenUrl(_this) {
            const url = new URL(window.location.href);
            url.searchParams.set("listen", _this.isListening ? "0" : "1");
            return url.pathname + url.search;
        },
        isEmbed(_this) {
            return String(_this.$route.path).indexOf("/embed/") == 0;
        },
        uploadDate(_this) {
            return new Date(_this.video.uploadDate).toLocaleString(undefined, {
                month: "short",
                day: "numeric",
                year: "numeric",
            });
        },
        commentsEnabled() {
            return this.getPreferenceBoolean("comments", true);
        },
    },
    mounted() {
        if (this.isEmbed) {
            document.querySelectorAll("[efy_sidebar_btn_open]")[0].style.display = "none";
        }
        // check screen size
        if (window.innerWidth >= 1024) {
            this.isMobile = false;
        }
        // add an event listener to watch for screen size changes
        window.addEventListener("resize", () => {
            if (window.innerWidth >= 1024) {
                this.isMobile = false;
            } else {
                this.isMobile = true;
            }
        });
        this.getVideoData().then(() => {
            (async () => {
                const videoId = this.getVideoId();
                const instance = this;
                if (window.db && !this.video.error) {
                    var tx = window.db.transaction("watch_history", "readwrite");
                    var store = tx.objectStore("watch_history");
                    var request = store.get(videoId);
                    request.onsuccess = function (event) {
                        var video = event.target.result;
                        if (video) {
                            video.watchedAt = Date.now();
                        } else {
                            video = {
                                videoId: videoId,
                                title: instance.video.title,
                                duration: instance.video.duration,
                                thumbnail: instance.video.thumbnailUrl,
                                uploaderUrl: instance.video.uploaderUrl,
                                uploaderName: instance.video.uploader,
                                watchedAt: Date.now(),
                            };
                        }
                        store.put(video);
                    };
                }
            })();
            if (this.active) this.$refs.videoPlayer.loadVideo();
        });
        this.playlistId = this.$route.query.list;
        this.index = Number(this.$route.query.index);
        this.getPlaylistData();
        this.getSponsors();
        if (!this.isEmbed && this.commentsEnabled) this.getComments();
        window.addEventListener("resize", () => {
            this.smallView = this.smallViewQuery.matches;
        });
    },
    activated() {
        this.active = true;
        this.selectedAutoPlay = this.getPreferenceBoolean("autoplay", false);
        this.selectedAutoLoop = this.getPreferenceBoolean("loop", false);
        this.showThea = this.getPreferenceBoolean("theater", false);
        this.showDesc = !this.getPreferenceBoolean("minimizeDescription", false);
        this.showRecs = !this.getPreferenceBoolean("minimizeRecommendations", false);
        if (this.video.duration) {
            document.title = this.video.title + " - Piped";
            this.$refs.videoPlayer.loadVideo();
        }
        window.addEventListener("scroll", this.handleScroll);
    },
    deactivated() {
        this.active = false;
        window.removeEventListener("scroll", this.handleScroll);
    },
    unmounted() {
        window.removeEventListener("scroll", this.handleScroll);
    },
    methods: {
        fetchVideo() {
            return this.fetchJson(this.apiUrl() + "/streams/" + this.getVideoId());
        },
        async fetchSponsors() {
            return await this.fetchJson(this.apiUrl() + "/sponsors/" + this.getVideoId(), {
                category:
                    '["' +
                    this.getPreferenceString("selectedSkip", "sponsor,interaction,selfpromo,music_offtopic").replaceAll(
                        ",",
                        '","',
                    ) +
                    '"]',
            });
        },
        fetchComments() {
            return this.fetchJson(this.apiUrl() + "/comments/" + this.getVideoId());
        },
        onChange(event) {
            if (this.testLocalStorage) {
                switch (event.target.id) {
                    case "showThea":
                        this.setPreference("theater", this.showThea);
                        scroll(0, 0);
                        break;
                    case "showDesc":
                        this.setPreference("minimizeDescription", this.showDesc);
                        break;
                    case "showRecs":
                        this.setPreference("minimizeRecommendations", this.showRecs);
                        break;
                    case "chkAutoplay":
                        this.setPreference("autoplay", this.selectedAutoPlay);
                        break;
                    case "chkLoop":
                        this.setPreference("loop", this.selectedAutoLoop);
                        break;

                    default:
                        break;
                }
            }
        },
        async getVideoData() {
            await this.fetchVideo()
                .then(data => {
                    this.video = data;
                    this.video.id = this.getVideoId();
                })
                .then(() => {
                    if (!this.video.error) {
                        document.title = this.video.title + " - Piped";
                        this.channelId = this.video.uploaderUrl.split("/")[2];
                        if (!this.isEmbed) this.fetchSubscribedStatus();

                        const parser = new DOMParser();
                        const xmlDoc = parser.parseFromString(this.video.description, "text/html");
                        xmlDoc.querySelectorAll("a").forEach(elem => {
                            if (!elem.innerText.match(/(?:[\d]{1,2}:)?(?:[\d]{1,2}):(?:[\d]{1,2})/))
                                elem.outerHTML = elem.getAttribute("href");
                        });
                        xmlDoc.querySelectorAll("br").forEach(elem => (elem.outerHTML = "\n"));
                        this.video.description = this.urlify(xmlDoc.querySelector("body").innerHTML)
                            .replaceAll(/(?:http(?:s)?:\/\/)?(?:www\.)?youtube\.com(\/[/a-zA-Z0-9_?=&-]*)/gm, "$1")
                            .replaceAll(
                                /(?:http(?:s)?:\/\/)?(?:www\.)?youtu\.be\/(?:watch\?v=)?([/a-zA-Z0-9_?=&-]*)/gm,
                                "/watch?v=$1",
                            )
                            .replaceAll("\n", "<br>");
                    }
                });
        },
        async getPlaylistData() {
            if (this.playlistId) {
                await this.fetchJson(this.apiUrl() + "/playlists/" + this.playlistId).then(data => {
                    this.playlist = data;
                });
                await this.fetchPlaylistPages().then(() => {
                    if (!(this.index >= 0)) {
                        for (let i = 0; i < this.playlist.relatedStreams.length; i++)
                            if (this.playlist.relatedStreams[i].url.substr(-11) == this.getVideoId()) {
                                this.index = i + 1;
                                this.$router.replace({
                                    query: { ...this.$route.query, index: this.index },
                                });
                                break;
                            }
                    }
                });
            }
        },
        async fetchPlaylistPages() {
            if (this.playlist.nextpage) {
                await this.fetchJson(this.apiUrl() + "/nextpage/playlists/" + this.playlistId, {
                    nextpage: this.playlist.nextpage,
                }).then(json => {
                    this.playlist.relatedStreams = this.playlist.relatedStreams.concat(json.relatedStreams);
                    this.playlist.nextpage = json.nextpage;
                });
                await this.fetchPlaylistPages();
            }
        },
        async getSponsors() {
            if (this.getPreferenceBoolean("sponsorblock", true))
                this.fetchSponsors().then(data => (this.sponsors = data));
        },
        async getComments() {
            this.fetchComments().then(data => (this.comments = data));
        },
        async fetchSubscribedStatus() {
            if (!this.channelId) return;
            if (!this.authenticated) {
                this.subscribed = this.isSubscribedLocally(this.channelId);
                return;
            }

            this.fetchJson(
                this.authApiUrl() + "/subscribed",
                {
                    channelId: this.channelId,
                },
                {
                    headers: {
                        Authorization: this.getAuthToken(),
                    },
                },
            ).then(json => {
                this.subscribed = json.subscribed;
            });
        },
        subscribeHandler() {
            if (this.authenticated) {
                this.fetchJson(this.authApiUrl() + (this.subscribed ? "/unsubscribe" : "/subscribe"), null, {
                    method: "POST",
                    body: JSON.stringify({
                        channelId: this.channelId,
                    }),
                    headers: {
                        Authorization: this.getAuthToken(),
                        "Content-Type": "application/json",
                    },
                });
            } else {
                this.handleLocalSubscriptions(this.channelId);
            }
            this.subscribed = !this.subscribed;
        },
        handleScroll() {
            if (this.loading || !this.comments || !this.comments.nextpage) return;
            if (window.innerHeight + window.scrollY >= this.$refs.comments.offsetHeight - window.innerHeight) {
                this.loading = true;
                this.fetchJson(this.apiUrl() + "/nextpage/comments/" + this.getVideoId(), {
                    nextpage: this.comments.nextpage,
                }).then(json => {
                    this.comments.nextpage = json.nextpage;
                    this.loading = false;
                    json.comments.map(comment => this.comments.comments.push(comment));
                });
            }
        },
        getVideoId() {
            return this.$route.query.v || this.$route.params.v;
        },
        navigate(time) {
            this.$refs.videoPlayer.seek(time);
        },
        onTimeUpdate(time) {
            this.currentTime = time;
        },
    },
};
</script>
