<template>
    <div class="create-board-modal">
        <form>
            <div class="create-board-modal-header">
                <div class="header-text-create-boards-modal">
                    <h1>Create board</h1>
                </div>
                <div class="header-close-create-boards-modal" @click="closeModal">
                    <span class="icon-sm icon-closed"></span>
                </div>
            </div>

            <div class="create-board-background-preview">
                <div
                    class="create-board-img-preview"
                    :style="setColor ? { backgroundColor: setColor } : { 'background-image': `url(${setPhoto})`, 'background-size': 'cover', 'background-repeat': 'no-repeat' }"
                >
                    <img
                        src="https://a.trellocdn.com/prgb/dist/images/board-preview-skeleton.14cda5dc635d1f13bc48.svg"
                        alt
                    />
                </div>
            </div>
            <label class="background-label">Background</label>
            <div class="create-background-photo">
                <div class="photos" v-for="photo in photos" :key="photo">
                    <div
                        @click="setBoardPhoto(photo.urls)"
                        class="photo-pref"
                        :style="setPhoto === photo.urls.thumb ? { 'background-image': `linear-gradient(rgba(0, 0, 0, 0.527),rgba(0, 0, 0, 0.5)) , url(${photo.urls.thumb})`, 'background-size': 'cover' } : { 'background-image': `url(${photo.urls.thumb})`, 'background-size': 'cover' }"
                        alt
                    >
                        <span
                            v-if="setPhoto === photo.urls.thumb"
                            class="checked"
                            :style="setPhoto === photo.urls.thumb ? { 'padding-top': '5px' } : null"
                        ></span>
                    </div>
                </div>
            </div>
            <div class="create-board-background-color">
                <div @click="setBoardColor('#0079BF')" class="blue color-pref">
                    <span v-if="setColor === '#0079BF'" class="checked"></span>
                </div>
                <div @click="setBoardColor('#D29034')" class="yellow color-pref">
                    <span v-if="setColor === '#D29034'" class="checked"></span>
                </div>
                <div @click="setBoardColor('#519839')" class="green color-pref">
                    <span v-if="setColor === '#519839'" class="checked"></span>
                </div>
                <div @click="setBoardColor('#B04632')" class="red color-pref">
                    <span v-if="setColor === '#B04632'" class="checked"></span>
                </div>
                <div @click="setBoardColor('#89609E')" class="purple color-pref">
                    <span v-if="setColor === '#89609E'" class="checked"></span>
                </div>
                <div @click="openUnsplash" class="more color-pref">
                    <svg
                        width="16"
                        height="16"
                        role="presentation"
                        focusable="false"
                        viewBox="0 0 24 24"
                        xmlns="http://www.w3.org/2000/svg"
                    >
                        <path
                            fill-rule="evenodd"
                            clip-rule="evenodd"
                            d="M5 14C6.10457 14 7 13.1046 7 12C7 10.8954 6.10457 10 5 10C3.89543 10 3 10.8954 3 12C3 13.1046 3.89543 14 5 14ZM12 14C13.1046 14 14 13.1046 14 12C14 10.8954 13.1046 10 12 10C10.8954 10 10 10.8954 10 12C10 13.1046 10.8954 14 12 14ZM21 12C21 13.1046 20.1046 14 19 14C17.8954 14 17 13.1046 17 12C17 10.8954 17.8954 10 19 10C20.1046 10 21 10.8954 21 12Z"
                            fill="currentColor"
                        />
                    </svg>
                </div>
            </div>
            <unsplash-modal v-if="unsplashOpen" @close="closeUnsplash" v-clickOutside="closeUnsplash"
             :style="{ 'left': '960' + 'px', 'bottom': '0' }"
             @changeBgc="setBoardPhoto"
            ></unsplash-modal>
            <label for>Board title</label>
            <div class="create-board-title-add-btns">
                <input v-focus type="text" v-model="newBoard.title" />
                <button
                    :style="!newBoard.title ? {
                        'cursor': 'no-drop',
                        'background-color': '#F5F6F8',
                        'color': '#BFC4CE',
                    } : null"
                    @click.stop.prevent="saveNewBoard"
                >Create</button>
            </div>
        </form>
    </div>
</template>      

<script>
import { uploadImg } from '../services/unsplash-service.js';
import unsplashModal from './unsplash-modal.vue';
export default {

    name: 'board-app',
    props: {
        newBoard: {
            type: Object,
            required: true,
        },
    },
    data() {
        return {
            isEdit: false,
            unsplashOpen: false,
            setColor: '#0079BF',
            setPhoto: '',
            photos: null,
            num: 4,
        }
    },
    created() {
        this.onUploadImg()
        // this.setRightPos()
    },
    computed: {
        boards() {
            return this.$store.getters.boards
        },
    },
    methods: {
        closeModal() {
            this.$emit("close");
        },
        saveNewBoard() {
            if (!this.newBoard.title) return
            if (!this.newBoard.background && !this.newBoard.backgroundPhoto) {
                this.newBoard.background = '#0079BF'
            }
            this.$emit("add", this.newBoard)
            this.setColor = '#0079BF'
            this.setPhoto = ''
        },
        setBoardColor(color = '#0079BF') {
            this.setColor = color
            this.setPhoto = ''
            this.newBoard.backgroundPhoto = ''
            this.newBoard.background = color
        },
        async onUploadImg() {
            const res = await uploadImg('abstract', this.num);
            this.photos = res.results
        },
        setBoardPhoto(photo) {
            this.setPhoto = photo.thumb
            this.setColor = ''
            this.newBoard.background = ''
            this.newBoard.backgroundPhoto = photo.full
            this.newBoard.backgroundThumb = photo.thumb
        },
        openUnsplash() {
            this.unsplashOpen = true
        },
        closeUnsplash() {
            this.unsplashOpen = false
        },

    },
    components: {
        unsplashModal,
    },
}
</script>