<template>
    <div class="myself-message-body">
        <div class="message-content">
            <div class="message-text" :style="{background: colors.message.myself.bg, color: colors.message.myself.text}">
                <p class="message-username">{{myself.name}}</p>
                <p>{{message.content}}</p>
            </div>
            <div class="message-timestamp" :style="{'justify-content': 'flex-end'}">
                {{message.timestamp.toFormat('HH:mm')}}
                <CheckIcon v-if="asyncMode && message.uploaded && !message.viewed" :size="14" class="icon-sent"/>
                <CheckAll v-else-if="asyncMode && message.uploaded && message.viewed" :size="14" class="icon-sent"/>
                <div v-else-if="asyncMode" class="message-loading"></div>
            </div>
        </div>
        <div class="thum-container">
            <img class="participant-thumb" src="https://lh3.googleusercontent.com/-G1d4-a7d_TY/AAAAAAAAAAI/AAAAAAAAAAA/AAKWJJPez_wX5UCJztzEUeCxOd7HBK7-jA.CMID/s83-c/photo.jpg" alt="">
        </div>
    </div>
</template>

<script>
    import CheckIcon from 'vue-material-design-icons/Check';
    import CheckAll from 'vue-material-design-icons/CheckAll';
    import {mapGetters, mapMutations} from 'vuex';
    export default {
        components:{
            CheckIcon,
            CheckAll,
        },
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
        },
        computed: {
            ...mapGetters([
                'getParticipantById',
                'messages',
                'myself'
            ]),
        }
    }
</script>

<style lang="less">
    .container-message-display .myself-message-body{
        display: flex;
        align-items: flex-end;
        padding-right: 10px;
        justify-content: flex-end;


        .message-content{
            display: flex;
            align-items: flex-end;
            justify-content: flex-start;
            flex-direction: column;
        }

        .participant-thumb{
            width: 25px;
            height: 25px;
            border-radius: 50%;
            margin-left: 10px;
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
            border-bottom-right-radius: 0px;
            word-break: break-word;
        }
    }
</style>