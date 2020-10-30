<template>
    <div class="quick-chat-container"
         :style="{
             'border-bottom-left-radius': borderStyle.bottomLeft, 
             'border-bottom-right-radius': borderStyle.bottomRight, 
             'border-top-right-radius': borderStyle.topRight, 
             'border-top-left-radius': borderStyle.topLeft
         }
         ">
        <Header v-if="displayHeader"
                :colors="colors" 
                :border-style="borderStyle" 
                :hide-close-button="hideCloseButton" 
                :close-button-icon-size="closeButtonIconSize" 
                :search-messages="searchMessages"
                @ontogglesearchintput="onToggleSearchInput($event)" 
                @onClose="onClose()">
            <template #header>
                <slot name="header"></slot>
            </template>
        </Header>
        <div v-if="showSearchInput" class="input-search-container">
            <input type="text" class="input-search" placeholder="Buscar" v-model="search">
        </div>
        <MessageDisplay :colors="colors" :async-mode="asyncMode" :load-more-messages="loadMoreMessages"
                        :link-options="linkOptions"
                        :scroll-bottom="scrollBottom"
                        :profile-picture-config="profilePictureConfig"
                        :timestamp-config="timestampConfig"
                        :gmaps-api-key="gmapsApiKey"
                        :search="search"
                        @onimageclicked="onImageClicked"
                        @onfileclicked="onFileClicked"
                        @onmapclicked="onMapClicked"/>
        <MessageManager :colors="colors"
                        :border-style="borderStyle" :submit-icon-size="submitIconSize"
                        :submit-image-icon-size="submitImageIconSize"
                        :send-images="sendImages"
                        :accept-image-types="acceptImageTypes"
                        :send-attachments="sendAttachments"
                        @onimageselected="onImageSelected"
                        @onuploadfile="onUploadFile"
                        @onattachlocation="onAttachLocation"
                        @onmessagesubmit="onMessageSubmit"
                        @ontype="onType"/>
    </div>
</template>

<script>
    import Header from './Header.vue'
    import MessageDisplay from './MessageDisplay.vue'
    import MessageManager from './MessageManager.vue'
    import { mapMutations } from 'vuex'
    import store from '../store'
 
    export default {
        name: 'chat',
        components: {
            Header,
            MessageDisplay,
            MessageManager
        },
        props: {
            participants: {
                type: Array,
                required: true
            },
            messages: {
                type: Array,
                required: true
            },
            myself: {
                type: Object,
                required: true
            },
            chatTitle: {
                type: String,
                required: false,
                default: ''
            },
            placeholder: {
                type: String,
                required: false,
                default: 'type your message here'
            },
            colors: {
                type: Object,
                required: true
            },
            borderStyle: {
                type: Object,
                required: false,
                default: () => {
                    return {
                        topLeft: "10px",
                        topRight: "10px",
                        bottomLeft: "10px",
                        bottomRight: "10px",
                    }
                }
            },
            hideCloseButton: {
                type: Boolean,
                required: false,
                default: false
            },
            submitIconSize: {
                type: Number,
                required: false,
                default: 24
            },
            submitImageIconSize: {
                type: Number,
                required: false,
                default: 24
            },
            closeButtonIconSize: {
                type: String,
                required: false,
                default: "15px"
            },
            asyncMode: {
                type: Boolean,
                required: false,
                default: false
            },
            loadMoreMessages: {
                type: Function,
                required: false,
                default: null
            },
            scrollBottom:{
                type: Object,
                required: false,
                default: () => {
                    return {
                        messageSent: true,
                        messageReceived: false
                    }
                }
            },
            displayHeader: {
                type: Boolean,
                required: false,
                default: true
            },
            sendImages: {
                type: Boolean,
                required: false,
                default: true
            },
            searchMessages: {
                type: Boolean,
                required: false,
                default: true
            },
            profilePictureConfig: {
                type: Object,
                required: false,
                default: () => {
                    return {
                        others: true,
                        myself: false,
                        styles: {
                            width: '25px',
                            height: '25px',
                            borderRadius: '50%'
                        }
                    }
                }
            },
            timestampConfig: {
                type: Object,
                required: false,
                default: () => {
                    return {
                        format: 'HH:mm',
                        relative: false
                    }
                }
            },
            linkOptions: {
                type: Object,
                required: false,
                default: () => {
                    return {
                        myself: {},
                        others: {}
                    }
                }
            },
            acceptImageTypes: {
                type: String,
                required: false,
                default: "image/*"
            },
            gmapsApiKey: {
                type: String,
                required: false,
                default: ''
            },
            sendAttachments: {
                type: Boolean,
                required: false,
                default: false
            }
        },
        data() {
            return {
                showSearchInput: false,
                search: ''
            };
        },
        watch: {
            participants() {
                this.setParticipants(this.participants);
            },
            myself() {
                this.setMyself(this.myself);
            },
            messages() {
                this.setMessages(this.messages);
            },
            placeholder() {
                this.setPlaceholder(this.placeholder);
            },
            chatTitle() {
                this.setChatTitle(this.chatTitle);
            },
            search(value) { 
                this.$emit("onsearch", value);
            }
        },
        beforeCreate() {
            this.$store = store();
        },
        created() {
            this.setParticipants(this.participants);
            this.setMyself(this.myself);
            this.setMessages(this.messages);
            this.setPlaceholder(this.placeholder);
            this.setChatTitle(this.chatTitle);
        },
        methods: {
            ...mapMutations([
                'setParticipants',
                'setMyself',
                'setMessages',
                'setPlaceholder',
                'setChatTitle'
            ]),
            onClose: function(){
                this.$emit("onclose");
            },
            onType: function(e){
                this.$emit("ontype", e)
            },
            onMessageSubmit: function(message){
                this.$emit("onmessagesubmit", message)
            },
            onImageSelected: function(data){
                this.$emit("onimageselected", data)
            },
            onImageClicked: function(message){
                this.$emit("onimageclicked", message);
            },
            onUploadFile: function(data) {
                this.$emit("onuploadfile", data);
            },
            onAttachLocation: function(data) {
                this.$emit("onattachlocation", data);
            },
            onFileClicked: function(message) {
                this.$emit("onfileclicked", message);
            },
            onMapClicked: function(message) {
                this.$emit("onmapclicked", message);
            },
            onToggleSearchInput: function(data) {
                this.showSearchInput = data;
                if (!this.showSearchInput) {
                    this.search = '';
                }
                this.$emit("ontogglesearchintput", data);
            }
        },
    }
</script>

<style lang="less">
    .quick-chat-container {
        display: flex;
        width: 100%;
        height: 100%;
        background: #f0eeee;
        flex-direction: column;
        align-items: stretch;
        overflow: hidden;
    }
    .input-search-container {
        padding: 20px;
    }
    .input-search {
        white-space: pre-wrap;
        word-wrap: break-word;
        color: #565867;
        text-align: left;
        cursor: text;
    }

    input {
        border-radius: 3px;
        border: none;
        padding: 14px;
        color: white;
        background: #dcdbdb;
        width: 90%;
        font-size: 14px;
    }
    
    .fa-search {
        position: relative;
        left: -25px;
    }
</style>
