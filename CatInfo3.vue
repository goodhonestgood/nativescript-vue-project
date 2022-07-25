<template>
    <Page class="page" actionBarHidden="true">
        <GridLayout rows="auto, auto, auto">
            <StackLayout row="0" class="home-panel" :class="{disabled : isBusy}">
                <Label text="오늘" class="ifLabel"/>
                <ListView height="600px" v-if="items.length > 0" for="(item,idx) in items">
                    <v-template>
                        <ScrollView orientation="horizontal" scrollBarIndicatorVisible="false">
                            <StackLayout orientation="horizontal">
                                <Image v-for="slide in item.slides" :key="slide" :src=slide @tap="downloading(slide,idx,0)" width="145" class="img-rounded m-5" />
                                <Label textWrap="true" class="h5" width="145" height="145">
                                    <FormattedString>
                                        <Span :text=item.author fontWeight="bold" />
                                        <Span text=" " />
                                        <Span :text=item.title />
                                    </FormattedString>
                                </Label>
                                <!-- <button text="다운로드" @tap="downloadImage()" /> -->
                            </StackLayout>
                        </ScrollView>
                    </v-template>
                </ListView>
                <Label height="600px" v-else text="피드 없음" class="elseLabel"/>
                <Label text="1일 전" class="ifLabel"/>
                <ListView height="600px" v-if="items2.length > 0" for="(item,idx) in items2">
                    <v-template>
                        <ScrollView orientation="horizontal" scrollBarIndicatorVisible="false">
                            <StackLayout orientation="horizontal">
                                <Image v-for="slide in item.slides" :key="slide" :src=slide @tap="downloading(slide,idx,1)" width="145" class="img-rounded m-5" />
                                <Label textWrap="true" class="h5" width="145" height="145">
                                    <FormattedString>
                                        <Span :text=item.author fontWeight="bold" />
                                        <Span text=" " />
                                        <Span text=" " />
                                        <Span :text=item.title />
                                    </FormattedString>
                                </Label>
                                <!-- <button text="다운로드" @tap="downloadImage()" /> -->
                            </StackLayout>
                        </ScrollView>
                    </v-template>
                </ListView>
                <Label height="600px" v-else text="피드 없음" class="elseLabel"/>   
            </StackLayout>
            <ActivityIndicator rowSpan="1" :busy="isBusy" color="aqua"/>
        </GridLayout>
    </Page>
</template>

<script>
    import axios from "axios";
    import CatLog from "./imageview";
    const appSettings = require("tns-core-modules/application-settings");

    export default {
        data() {
            return {
                items: [],
                items2: [],
                isBusy: true,
            };
        },
        methods: {
            downloading: function(img,idx,n) {
                this.$navigateTo(CatLog, {
                    props: {
                        image: img,
                        link: n == 0 ? this.items[idx].link : this.items2[idx].link
                    }
                });
            },
            dates: function(pubDate, diff) {
                var pubDates = new Date(pubDate);
                pubDates.setHours(pubDates.getHours() + 9);
                var date = new Date();
                if (pubDates.getDate() == (date.getDate() - diff)) {
                    return true;
                } else return false;
            },
            fetchData: function() {
                this.items = [];
                var instaid = appSettings.getString('instagram');
                instaid = instaid.replace(/ /g,"");
                if(instaid[instaid.length-1] == ','){
                    instaid = instaid.slice(0,instaid.length-1)
                }
                const arrayid = instaid.split(',');
                arrayid.forEach(id => {
                    var feed = "https://www.instagram.com/" + id;
                    
                    fetch(feed)
                    .then(a => {
                        return a.text();
                    }).then(a => {
                        const media = JSON.parse(a.slice(a.indexOf("edge_owner_to_timeline_media") + 30, a.indexOf("edge_saved_media") - 2));
                        media.edges.forEach(m => {
                            var date = new Date(m.node.taken_at_timestamp * 1000);
                            var arr = m.node.edge_sidecar_to_children;
                            var slide = []
                            if ( arr != undefined){
                                arr.edges.forEach(a => {slide.push(a.node.display_url)});
                            } else {
                                slide.push(m.node.display_url)
                            }
                            if (date.getYear() == new Date().getYear()) {
                                if (date.getMonth() == new Date().getMonth()) {
                                    if (date.getDate() == new Date().getDate()){
                                        this.items.push({
                                            link: "https://www.instagram.com/p/" + m.node.shortcode,
                                            title: m.node.edge_media_to_caption.edges[0].node.text,
                                            author: m.node.owner.username,
                                            pubDate: date,
                                            slides: slide
                                        })
                                    } else if (date.getDate() == (new Date().getDate() - 1)){
                                        this.items2.push({
                                            link: "https://www.instagram.com/p/" + m.node.shortcode,
                                            title: m.node.edge_media_to_caption.edges[0].node.text,
                                            author: m.node.owner.username,
                                            pubDate: date,
                                            slides: slide
                                        })
                                    }
                                }
                            }
                        });
                        this.isBusy = false;
                    })
                    .catch(error => {
                            console.log("error");
                            console.log(error);
                            this.isBusy = false;
                    });
                });
            },
        },
        created: function() {
            if (appSettings.getString("instagram") != undefined) {
                this.fetchData();
            } else {
                this.isBusy = false;
            }
        }
    };
</script>

<style scope>
    .home-panel {
        font-size: 20;
        margin: 15;
    }

    .description-label {
        margin-bottom: 15;
    }

    .ifLabel {
        margin: 10px;
    }
    .elseLabel {
        font-size: 13px;
    }
    .disabled {
        opacity: 0.5;
    }
    Label {
        vertical-align: center;
    }
</style>