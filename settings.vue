<template>
    <Page actionBarHidden="true">
        <ScrollView>
            <StackLayout class="form">
            <Label class="h2" text="SNS 구독 계정 설정" />

            <GridLayout columns="2*, *" rows="*, *" class="input-field" marginBottom="25">
                <TextField row="0" col="0" class="input" hint="구독할 Twitter ID 입력" autocorrect="false" autocapitalizationType="none" v-model="setting.twitter" returnKeyType="next" fontSize="15" />
                <Label row="1" colSpan="2" class="header" :text="settedMessage(setted.twitter)"/>
                <Button row="0" col="1" text="설정" @tap="submit(0)" class="btn btn-primary" />
            </GridLayout>
            <GridLayout columns="2*, *" rows="*, *" class="input-field" marginBottom="25">
                <TextField row="0" col="0" class="input" hint="구독할 Instagram ID 입력" autocorrect="false" autocapitalizationType="none" v-model="setting.instagram" returnKeyType="next" fontSize="15" />
                <Label row="1" colSpan="2" class="header" :text="settedMessage(setted.instagram)"/>
                <Button row="0" col="1" text="설정" @tap="submit(1)" class="btn btn-primary" />
            </GridLayout>
            <GridLayout columns="2*, *" rows="*, *" class="input-field" marginBottom="25">
                <TextField row="0" col="0" class="input" hint="구독할 Youtube 채널 입력" autocorrect="false" autocapitalizationType="none" v-model="setting.youtube" returnKeyType="done" fontSize="15" />
                <Label row="1" colSpan="2" class="header" :text="settedMessage(setted.youtube)"/>
                <Button row="0" col="1" text="설정" @tap="submit(2)" class="btn btn-primary" />
            </GridLayout>
            </StackLayout>
        </ScrollView>
    </Page>
</template>

<script>
    const appSettings = require("tns-core-modules/application-settings");
    import axios from "axios";
    export default {
        data() {
            return {
                setting: {
                    twitter: '',
                    instagram: '',
                    youtube: '',
                },
                setted: {
                    twitter: '',
                    instagram: '',
                    youtube: '',
                },
                result: ''
            };
        },
        methods: {
            submit(idx) {
                if(idx == 0) {
                    var idstr = this.setting.twitter.replace(/ /g,"")
                } else if(idx == 1) {
                    var idstr = this.setting.instagram.replace(/ /g,"")
                } else if(idx == 2) {
                    var idstr = this.setting.youtube.replace(/ /g,"")
                }
                this.result = ''
                var arrayid = idstr.split(',');
                arrayid.forEach((id) => this.confirm(idx,id))
            },
            async confirm(idx,id) {
                if(idx == 0){
                    var feed = "https://api.rss2json.com/v1/api.json?rss_url=" + encodeURIComponent("https://rsshub.app/twitter/user/" + id);
                    var tweet = await fetch(feed).then(response => response.json())
                    if(tweet.status != 'ok') {
                        alert({
                            message: id +'를 확인하여 다시 시도해주세요.',
                            okButtonText: 'ok'
                        })
                        this.setting.twitter = ''
                        return 0
                    } else {
                        this.result = this.result + id + ',';
                        appSettings.setString('twitter', this.result);
                        this.setted.twitter = appSettings.getString("twitter");
                        if((this.setting.twitter.match(/,/g) != null ? this.setting.twitter.match(/,/g).length : 0) == this.setted.twitter.match(/,/g).length-1) {
                            var ids = appSettings.getString('twitter')
                            ids = ids.slice(0,ids.length-1)
                            alert({
                                message: ids + '가 설정되었습니다.',
                                okButtonText: "OK"
                            })
                        }
                    }
                } else if(idx == 1){
                    var feed = "https://www.instagram.com/" + id;
                    var insta = await fetch(feed)
                    if(insta.ok != false) {
                        this.result = this.result + id + ',';
                        appSettings.setString('instagram', this.result);
                        this.setted.instagram = appSettings.getString("instagram");
                        if((this.setting.instagram.match(/,/g) != null ? this.setting.instagram.match(/,/g).length : 0) == this.setted.instagram.match(/,/g).length-1){
                            var ids = appSettings.getString('instagram')
                            ids = ids.slice(0,ids.length-1)
                            alert({
                                message: ids + '가 설정되었습니다.',
                                okButtonText: "OK"
                            })
                        }
                    } else {
                        alert({
                            message: id +'를 확인하여 다시 시도해주세요.',
                            okButtonText: 'ok'
                        })
                        this.setting.instagram = ''
                        return 0
                    }
                } else if(idx == 2){
                    var feed = "https://api.rss2json.com/v1/api.json?rss_url=" + encodeURIComponent("https://www.youtube.com/feeds/videos.xml?channel_id=" + id);
                    var youtu = await fetch(feed).then(response => response.json())
                    if(youtu.status != 'ok') {
                        alert({
                            message: id +'를 확인하여 다시 시도해주세요.',
                            okButtonText: 'ok'
                        })
                        this.setting.youtube = ''
                        return 0
                    } else {
                        this.result = this.result + id + ',';
                        appSettings.setString('youtube', this.result);
                        this.setted.youtube = appSettings.getString("youtube");
                        if((this.setting.youtube.match(/,/g) != null ? this.setting.youtube.match(/,/g).length : 0) == this.setted.youtube.match(/,/g).length-1){
                            var ids = appSettings.getString('youtube')
                            ids = ids.slice(0,ids.length-1)
                            alert({
                                message: ids + '가 설정되었습니다.',
                                okButtonText: "OK"
                            })
                        }
                    }
                }
            },
            settedMessage: function(key) {
                if (key == undefined) {
                    return '구독 중인 계정이 없습니다.'
                } else {
                    key = key.slice(0,key.length-1)
                    return '현재 ' + key + '를(을) 구독하고 있습니다.'
                }
            },
        },
        created: function() {
            this.setted.twitter = appSettings.getString("twitter");
            this.setted.instagram = appSettings.getString("instagram");
            this.setted.youtube = appSettings.getString("youtube");
            console.log(this.setted.color);
        }
    };
</script>
<style scoped>
</style>