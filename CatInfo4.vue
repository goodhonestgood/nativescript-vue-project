<template>
    <Page class="page" actionBarHidden="true">
        <GridLayout rows="auto, auto, auto">
            <StackLayout row="0" class="home-panel" :class="{disabled : isBusy}">
                <Label text="오늘" class="ifLabel"/>
                <ListView height="600px" v-if="items.length > 0" for="item in items" @itemTap="onItemTap">
                    <v-template>
                        <StackLayout orientation="horizontal">
                            <Image :src="item.thumbnail" width="150" class="img-rounded m-5" backgroundColor="black" />
                            <Label textWrap="true" width="150" :text=item.title class="h5"/>
                        </StackLayout>
                    </v-template>
                </ListView>
                <Label height="600px" v-else text="피드 없음" class="elseLabel"/>
                <Label text="1일 전" class="ifLabel"/>
                <ListView height="600px" v-if="items2.length > 0" for="item in items2" @itemTap="onItemTap1">
                    <v-template>
                        <StackLayout orientation="horizontal">
                            <Image :src="item.thumbnail" width="150" class="img-rounded m-5" backgroundColor="black" />
                            <Label textWrap="true" width="150" :text=item.title class="h5"/>
                        </StackLayout>
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
    const appSettings = require("tns-core-modules/application-settings");
    const utilsModule = require("tns-core-modules/utils/utils");

    export default {
        data() {
            return {
                items: [],
                items2: [],
                isBusy: true,
            };
        },
        methods: {
            onItemTap: function(index) {
                console.log(index.index);
                utilsModule.openUrl(this.items[index.index].link)
            },
            onItemTap1: function(index) {
                console.log(index.index);
                utilsModule.openUrl(this.items2[index.index].link)
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
                var youtubeid = appSettings.getString('youtube');
                youtubeid = youtubeid.replace(/ /g,"");
                if(youtubeid[youtubeid.length-1] == ','){
                    youtubeid = youtubeid.slice(0,youtubeid.length-1)
                }
                const arrayid = youtubeid.split(',');
                arrayid.forEach(id => {
                    var feed = "https://api.rss2json.com/v1/api.json?rss_url=" + encodeURIComponent("https://www.youtube.com/feeds/videos.xml?channel_id=" + id);
                    axios 
                        .get(feed)
                        .then(response => {
                            const items = response.data.items;
                            for (let i = 0; i < items.length; i += 1) {
                                if (this.dates(items[i].pubDate, 0)) {
                                    this.items.push(items[i]);
                                } else if (this.dates(items[i].pubDate, 1)) {
                                    this.items2.push(items[i]);
                                }
                            }
                            this.isBusy = false;
                        })
                        .catch(error => {
                            // eslint-disable-next-line
                            console.log(error);
                            this.isBusy = false;
                        });
                });
                
            },
        },
        created: function() {
            if (appSettings.getString("youtube") != undefined) {
                this.fetchData();
            } else {
                this.isBusy = false;
            }
        }
    };
</script>
<style scoped>
    .disabled {
        opacity: 0.5;
    }
</style>