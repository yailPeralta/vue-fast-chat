<template> 
    <div class="container-message-manager">
        <div v-if="sendAttachments" class="icon-send-message attachments-button" title="Adjuntar" @click="openAttachmentOptions">
            <PaperclipIcon :size="attachmentsIconSize" :fill-color="colors.attachmentIcon"/>
        </div>
        <div class="message-text-box">
            <div ref="userInput" class="message-input" :placeholder="placeholder"
                 tabIndex="0" contenteditable="true"
                 @input="handleType" @keyup.enter.exact="sendMessage">
            </div>
        </div>
        <div class="container-send-message icon-send-message" title="Enviar Mensaje" @click.prevent="sendMessage">
            <SendIcon :size="submitIconSize" :fill-color="colors.submitIcon"/>
        </div>
        <div v-if="showAttachmentOptions" title="Ubicación" class="container-send-message icon-send-message" @click="attachLocation">
            <MapMarkerRadiusIcon :size="mapAttachmentIconSize" :fill-color="colors.submitImageIcon"/>
        </div>
        <div v-if="showAttachmentOptions" title="Subir archivo" class="container-send-message icon-send-message" @click="pickFile">
            <input ref="inputFile" :accept="acceptedFileTypes.join(', ')" type="file" style="display: none;" @input="uploadFile">
            <FileUploadIcon :size="fileUploadIconSize" :fill-color="colors.submitImageIcon"/>
        </div>
        <div v-if="sendImages" title="Subir imagen" class="container-send-message icon-send-message" @click="pickImage">
            <input ref="inputImage" :accept="acceptImageTypes" type="file" style="display: none;" @input="handleImageChange">
            <ImageIcon :size="submitImageIconSize" :fill-color="colors.submitImageIcon"/>
        </div>
    </div>
</template>

<script>
    //import 'vue-material-design-icons/styles.css';
    import { mapMutations } from 'vuex'
    import { DateTime } from "luxon";
    import SendIcon from 'vue-material-design-icons/Send';
    import ImageIcon from 'vue-material-design-icons/Image';
    import MapMarkerRadiusIcon from 'vue-material-design-icons/MapMarkerRadius';
    import PaperclipIcon from 'vue-material-design-icons/Paperclip';
    import FileUploadIcon from 'vue-material-design-icons/FileUpload';

    export default {
        components: {
            SendIcon,
            ImageIcon,
            MapMarkerRadiusIcon,
            PaperclipIcon,      
            FileUploadIcon
        },
        props: {
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
            attachmentsIconSize: {
                type: Number,
                required: false,
                default: 18
            },
            fileUploadIconSize: {
                type: Number,
                required: false,
                default: 24
            },
            mapAttachmentIconSize: {
                type: Number,
                required: false,
                default: 24
            },
            /* onImageSelected: {
                type: Function,
                required: false,
                default: () => false
            }, */
            sendImages: {
                type: Boolean,
                required: false,
                default: true
            },
            acceptImageTypes: {
                type: String,
                required: true
            },
            sendAttachments: {
                type: Boolean,
                required: false,
                default: false
            }
        },
        data() {
            return {
                textInput: '',
                showAttachmentOptions: false,
                acceptedFileTypes: [
                    'image/jpeg',
                    'image/gif',
                    'image/bmp',
                    'image/png',
                    'application/pdf',
                    'x-pdf',
                    'application/msword',
                    'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
                    'application/vnd.oasis.opendocument.text', // odt 
                    'application/vnd.oasis.opendocument.text-template' // odt
                ],
                pdfLogo: require('@/assets/pdf_icon.png'),
                wordLogo: require('@/assets/word_icon.png')
            }
        },
        computed: {
            myself() {
                return this.$store.state.myself;
            },
            placeholder() {
                return this.$store.state.placeholder;
            }
        },
        methods: {
            ...mapMutations([
                'newMessage'
            ]),
            sendMessage(e) {
                this.textInput = this.$refs.userInput.textContent;
                this.$refs.userInput.textContent = '';

                if (this.textInput) {
                    let inputLen = this.textInput.length;
                    let inputText = this.textInput;
                    if (this.textInput[inputLen - 1] === '\n') {
                        inputText = inputText.slice(0, inputLen - 1)
                    }
                    let message = {
                        content: inputText,
                        // myself: true,
                        participantId: this.myself.id,
                        timestamp: DateTime.local(),
                        uploaded: false,
                        viewed: false,
                        type: 'text'
                    };
                    this.$emit("onmessagesubmit", message);
                    this.newMessage(message)
                }
            },
            handleType: function (e) {
                this.$emit("ontype", e);
            },
            pickImage: function() {
                this.$refs.inputImage.click()
            },
            pickFile: function() {
                this.$refs.inputFile.click()
            },
            openAttachmentOptions: function() {
                this.showAttachmentOptions = !this.showAttachmentOptions;
            },
            uploadFile: function(e) {
                const files = e.target.files;

                if (!!files) {
                    let mimeType = files[0].type;
                    let fileName = files[0].name;
                    let preview = '';
                    let type = mimeType.indexOf('image/') !== -1 
                        ? 'image' : 'file';

                    if (type === 'image') {
                        preview = URL.createObjectURL(files[0]);
                    } else {
                        preview = (mimeType.indexOf('/pdf') !== -1)
                            ? this.pdfLogo
                            : this.wordLogo;
                    }

                    let message = {
                        type,
                        preview,
                        src: '',
                        content: fileName,
                        participantId: this.myself.id,
                        timestamp: DateTime.local(),
                        uploaded: false,
                        viewed: false
                    };
     
                    this.$emit("onuploadfile", {
                        file: files[0], 
                        message, 
                        myselfId: this.myself.id 
                    });

                    this.newMessage(message);
                }
            },
            getCurrentLocation: function() {
      
                return new Promise((resolve, reject) => {

                    if(!("geolocation" in navigator)) {
                        reject(new Error('No se puede detectar su ubicación.'));
                    }

                    navigator.geolocation.getCurrentPosition(pos => {
                        resolve(pos);
                    }, err => {
                        reject(err);
                    });

                });
            },
            attachLocation: async function() {
                try {
                    let location = await this.getCurrentLocation();

                    let message = {
                        type: 'location',
                        preview: '',
                        src: '',
                        content: JSON.stringify({
                            lat: location.coords.latitude,
                            lng: location.coords.longitude
                        }),
                        participantId: this.myself.id,
                        timestamp: DateTime.local(),
                        uploaded: false,
                        viewed: false
                    };
                    this.$emit("onattachlocation", {
                        myselfId: this.myself.id,
                        message  
                    });
                    this.newMessage(message)
                } catch (e) {
                    alert(`There was en error getting the current location:  ${e.message}`);
                }
            },
            handleImageChange: async function(e) {
                const files = e.target.files
                let message = {
                    type: 'image',
                    preview: URL.createObjectURL(files[0]),
                    src: '',
                    content: 'image',
                    participantId: this.myself.id,
                    timestamp: DateTime.local(),
                    uploaded: false,
                    viewed: false
                };
                this.$emit("onimageselected", {file: files[0], message});
                //this.onImageSelected(files, message)
                this.newMessage(message)
            }
        }
    }
</script>

<style lang="less">
    .quick-chat-container .container-message-manager {
        height: 65px;
        background: #fff;
        display: flex;
        align-items: center;
        padding: 0 20px 0 20px;
        -webkit-box-shadow: 0px -2px 40px 0px rgba(186, 186, 186, 0.67);
        box-shadow: 0px -2px 40px 0px rgba(186, 186, 186, 0.67);

        .option-list {
            list-style: none !important;
            list-style-type: none !important;
        }

        .attachements-options {
            position: relative;
            top: -80px;
            left: 0;
            background: white;
            width: 350px;
            display: block;
            margin-left: 0;
            padding-left: 0;
        }

        .message-text-box {
            padding: 0 10px 0 10px;
            flex: 1;
            overflow: hidden;
        }

        .message-input {
            width: 100%;
            resize: none;
            border: none;
            outline: none;
            box-sizing: border-box;
            font-size: 15px;
            font-weight: 400;
            line-height: 1.33;
            white-space: pre-wrap;
            word-wrap: break-word;
            color: #565867;
            -webkit-font-smoothing: antialiased;
            max-height: 40px;
            bottom: 0;
            overflow: scroll;
            overflow-x: hidden;
            overflow-y: auto;
            text-align: left;
            cursor: text;
            display: inline-block;
        }

        .message-input:empty:before {
            content: attr(placeholder);
            display: block; /* For Firefox */
            filter: contrast(15%);
            outline: none;
        }

        .message-input:focus {
            outline: none;
        }

        .container-send-message {
            margin-left: 10px;
        }

        .container-send-message svg {
            -webkit-box-sizing: content-box;
            box-sizing: content-box;
        }

        .icon-send-message {
            cursor: pointer;
            opacity: 0.7;
            transition: 0.3s;
            padding: 8px;
            cursor: pointer;
            border-radius: 11px !important;
            min-width: 20px;
            margin: 0;
            text-align: center;
        }

        .icon-send-message:hover {
            opacity: 1;
            background: #eee;
        }

        .attachments-button {
            padding-left: 0 !important;
        }
    }
</style>
