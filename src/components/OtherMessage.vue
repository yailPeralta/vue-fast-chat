<template>
    <div class="other-message-body">
        <div v-if="profilePictureConfig.others" class="thum-container">
            <img class="participant-thumb" :src="getParticipantById(message.participantId).profilePicture"
                 :style="{'width': profilePictureConfig.styles.width, 'height': profilePictureConfig.styles.height, 'border-radius': profilePictureConfig.styles.borderRadius}">
        </div>
        <div class="message-content">
            <template v-if="message.type == 'image'">
                <p class="message-username-image">{{getParticipantById(message.participantId).name}}</p>
                <div v-if="message.uploaded" class="message-image">
                    <img class="message-image-display" :src="message.src" alt="" @click="onImageClicked(message)">
                </div>
                <div v-else class="message-image">
                    <img class="message-image-display img-overlay" :src="message.preview" alt="">
                    <div class="img-loading"></div>
                </div>
            </template>
            <template v-else-if="message.type == 'location'">
                <p class="message-username-image">{{myself.name}}</p>
                <div class="message-image">
                    <a :href="getMapLink(message.content)" target="_blank">
                        <img class="message-image-display" :src="getMapThumbnail(message.content)" alt="" @click="onMapClicked(message)">
                    </a>
                </div>
            </template>
            <template v-else-if="message.type == 'file'">
                <p class="message-username-image">{{myself.name}}</p>
                <div v-if="message.uploaded" class="message-image file-wrapper">
                    <img class="message-image-display file-logo" :src="message.src" alt="" @click="onFileClicked(message)">
                </div>
                <div v-else class="message-image file-wrapper">
                    <img class="message-image-display img-overlay file-logo" :src="message.preview" alt="">
                    <div class="img-loading"></div>
                </div>
                <p ref="message-content" v-html="messageBody"></p>
            </template>
            <template v-else>
                <div class="message-text" :style="{background: colors.message.others.bg, color: colors.message.others.text}">
                    <p class="message-username">{{getParticipantById(message.participantId).name}}</p>
                    <p ref="message-content" v-html="messageBody"></p>
                </div>
            </template>
            <div class="message-timestamp" :style="{'justify-content': 'baseline'}">
                <template v-if="timestampConfig.relative">
                    {{message.timestamp.toRelative()}}
                </template>
                <template v-else> 
                    {{message.timestamp.toFormat(timestampConfig.format)}}
                </template>
                <CheckIcon v-if="asyncMode && message.uploaded && !message.viewed" :size="14" class="icon-sent"/>
                <CheckAll v-else-if="asyncMode && message.uploaded && message.viewed" :size="14" class="icon-sent viewed"/>
                <div v-else-if="asyncMode" class="message-loading"></div>
            </div>
        </div>
    </div>
</template>

<script>
    import CheckIcon from 'vue-material-design-icons/Check';
    import CheckAll from 'vue-material-design-icons/CheckAll';
    import {mapGetters, mapMutations} from 'vuex';
    import linkParse from '../mixins/linkParse';
    export default {
        components:{
            CheckIcon,
            CheckAll,
        },
        mixins: [linkParse],
        props:{
            message: {
                type: Object,
                required: true
            },
            asyncMode: {
                type: Boolean,
                required: false,
                default: false
            },
            colors: {
                type: Object,
                required: true
            },
            profilePictureConfig: {
                type: Object,
                required: true
            },
            timestampConfig: {
                type: Object,
                required: true
            },
            gmapsApiKey: {
                type: String,
                required: false,
                default: ''
            },
            search: {
                type: String,
                required: false,
                default: ''
            }
        },
        computed: {
            ...mapGetters([
                'getParticipantById',
                'messages',
                'myself'
            ]),
            messageBody: function() {
                if (!!this.search && !!this.message.search_match) {
                    let search = this.search.replace(/[|\\{}()[\]^$+*?.]/g, '\\$&')
		                .replace(/-/g, '\\x2d');
                        
                    let regex = new RegExp(search, 'gi');

                    return this.message.content.replace(
                        regex, 
                        `<span style="background: yellow !important; color: dimgray;">${this.search}</span>`
                    );
                }

                return this.message.content; 
            }
        },
        methods: {
            onImageClicked: function(message){
                this.$emit("onimageclicked", message)
            },
            onFileClicked: function(message){
                this.$emit("onfileclicked", message);
            },
            onMapClicked: function(message){
                this.$emit("onmapclicked", message)
            },
            getMapLink(content) {
                let cordsObj = JSON.parse(content);
                return  `https://maps.google.com/?q=${cordsObj.lat},${cordsObj.lng}`;
            },
            getMapThumbnail(content) {
                let cordsObj = JSON.parse(content);
                
                return `
                    https://maps.googleapis.com/maps/api/staticmap?zoom=15&size=600x300&maptype=roadmap%20
                    &markers=color:red%7Clabel:C%7C${cordsObj.lat},${cordsObj.lng}%20&key=${this.gmapsApiKey}
                `;
            }
        }
    }
</script>

<style lang="less">
    .container-message-display .other-message-body{
        display: flex;
        align-items: flex-end;
        padding-left: 10px;

        .message-content{
            display: flex;
            align-items: flex-start;
            justify-content: flex-start;
            flex-direction: column;
            flex-grow: 1;
        }

        .participant-thumb{
            /* width: 25px;
            height: 25px;
            border-radius: 50%; */
            margin-right: 10px;
        }

        .message-timestamp {
            padding: 2px 7px;
            border-radius: 15px;
            margin: 0;
            max-width: 50%;
            overflow-wrap: break-word;
            text-align: left;
            font-size: 10px;
            color: #bdb8b8;
            width: 100%;
            display: flex;
            align-items: center;
        }

        .message-text {
            background: #fff;
            padding: 6px 10px;
            line-height: 14px;
            border-radius: 15px;
            margin: 5px 0 5px 0;
            max-width: 70%;
            overflow-wrap: break-word;
            text-align: left;
            white-space: pre-wrap;
            border-bottom-left-radius: 0px;
            word-break: break-word;
        }

        .file-logo {
            width: 80px !important;
        }

        .file-wrapper {
            padding-right:  !important;
        }
    }
</style>