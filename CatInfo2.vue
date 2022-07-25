<template>
    <Page class="page" actionBarHidden="true">
        <GridLayout rows="auto, auto, auto">
            <StackLayout row="0" class="home-panel" :class="{disabled : isBusy}">
                <Label text="오늘" class="ifLabel"/>
                <ListView height="600px" v-if="items.length > 0" for="(item,idx) in items">
                    <v-template>
                        <ScrollView orientation="horizontal" scrollBarIndicatorVisible="false">
                            <StackLayout orientation="horizontal">
                                <Image v-for="img in item.image" :key="img" :src="img" width="150" height="150" stretch="aspectFit" @tap="downloading(img,idx,0)" class="img-rounded m-5" />
                                <Image v-for="img in item.video" :key="img" :src="img" width="150" height="150" stretch="aspectFit" class="img-rounded m-5" />
                                <Label textWrap="true" class="h5" width="145" height="145" >
                                    <FormattedString>
                                        <Span :text=item.author fontWeight="bold" />
                                        <Span text=" " />
                                        <Span :text=item.title />
                                    </FormattedString>
                                </Label>
                                <StackLayout orientation="vertical">    
                                    <Button v-for="(alink,index) in item.alink" :key="index" textWrap="true" :text="'링크 '+(index+1)" width="145" @tap="openalink(alink)"/>
                                </StackLayout>
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
                                <Image v-for="img in item.image" :key="img" :src="img" width="150" height="150" stretch="aspectFit" @tap="downloading(img,idx,1)" class="img-rounded m-5" />
                                <Image v-for="img in item.video" :key="img" :src="img" width="150" height="150" stretch="aspectFit" class="img-rounded m-5" />
                                <Label textWrap="true" class="h5" width="145" height="145" >
                                    <FormattedString>
                                        <Span :text=item.author fontWeight="bold" />
                                        <Span text=" " />
                                        <Span :text=item.title />
                                    </FormattedString>
                                </Label>
                                <StackLayout orientation="vertical">    
                                    <Button v-for="(alink,index) in item.alink" :key="index" textWrap="true" :text="'링크 '+(index+1)" width="145" @tap="openalink(alink)"/>
                                </StackLayout>
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
            allimages: function(content) {
                var str = content
                var a = ''
                var c = []
                if(str.match('<img') != undefined){
                    a = str.slice(str.indexOf('<img'),str.length)
                    var alen = a.match(/https/g).length
                    for(var i = 0; i<alen; i++) {
                        var img = a.slice(a.indexOf('https'),a.indexOf('orig')+4)
                        c.push(img.replace('amp;',''))
                        a = a.replace(img,'')
                    }
                }
               return c
            },
            allvideos: function(content) {
                var str = content
                var c = []
                if(str.match('<video') != undefined){
                    var v = str.slice(str.indexOf('poster'),str.lastIndexOf('</video>'))
                    c.push(v.slice(v.indexOf('https'),v.indexOf('.jpg')+4))
                }
                return c
            },
            alllinks: function(content) {
                var str = content
                var c = []
                if(str.match('<a') != undefined){
                    var b = str.slice(str.indexOf('<a'),str.lastIndexOf('</a>')+4)
                    var blen = b.match(/href/g).length
                    for(var i=0; i <blen; i++){
                        var href = b.slice(b.indexOf('http'),b.indexOf('target')-2)
                        c.push(href)
                        b = b.replace(href,'')
                        b = b.replace(href,'')
                        b = b.replace('target','')
                    }
                }
                return c
            },
            downloading: function(img,idx,n) {
                this.$navigateTo(CatLog, {
                    props: {
                        image: img,
                        link: n == 0 ? this.items[idx].link : this.items2[idx].link
                    }
                });
            },
            openalink: function(url) {
                utilsModule.openUrl(url)
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
                var twitterid = appSettings.getString("twitter");
                twitterid = twitterid.replace(/ /g,"");
                if(twitterid[twitterid.length-1] == ','){
                    twitterid = twitterid.slice(0,twitterid.length-1)
                }
                const arrayid = twitterid.split(',');
                arrayid.forEach(id => {
                    var feed = "https://api.rss2json.com/v1/api.json?rss_url=" + encodeURIComponent("https://rsshub.app/twitter/user/" + id);
                    axios 
                        .get(feed)
                        .then(response => {
                            const items = response.data.items;
                            for (let i = 0; i < items.length; i += 1) {
                                if (this.dates(items[i].pubDate, 0)) {
                                    if(items[i].title[0]+items[i].title[1] != "RT") {
                                        if(items[i].title[0]+items[i].title[1] != "Re") {
                                            this.items.push({
                                                title: items[i].title,
                                                author: items[i].author,
                                                image: this.allimages(items[i].content),
                                                video: this.allvideos(items[i].content),
                                                alink: this.alllinks(items[i].content),
                                                link: items[i].link
                                            });
                                        }
                                        
                                    }
                                    
                                } else if (this.dates(items[i].pubDate, 1)) {
                                    if(items[i].title[0]+items[i].title[1] != "RT") {
                                        if(items[i].title[0]+items[i].title[1] != "Re") {
                                            this.items2.push({
                                                title: items[i].title,
                                                author: items[i].author,
                                                image: this.allimages(items[i].content),
                                                video: this.allvideos(items[i].content),
                                                alink: this.alllinks(items[i].content),
                                                link: items[i].link
                                            });
                                        }
                                    }
                                }
                            }
                            this.isBusy = false;
                            console.log(this.items.length,' ',this.items2.length)
                        })
                        .catch(error => {
                            console.log(error);
                            this.isBusy = false;
                        });
                });
                
            },
        },
        created: function() {
            if (appSettings.getString("twitter") != undefined) {
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
        border-bottom-width: 1;
        border-bottom-color: rgb(151, 151, 151);
    }
    .elseLabel {
        font-size: 13px;
    }
    .borderstyle {
        border-bottom-width: 1;
        border-bottom-color: rgb(151, 151, 151);
    }
    .disabled {
        opacity: 0.5;
    }
</style>

var str = '태민 2020 A-Awards 솔로 아티스트 부문 수상! 👏🏻👏🏻👏🏻<br><br>#태민 #TAEMIN #샤이니 #SHINee #태민만의_아이덴티티_이데아 <br>#NeverGonnaDanceAgain <br>#Act2 <br>#IDEA #이데아 #ARENAKOREA #AAwards2020<br><img src="https://pbs.twimg.com/media/EnqsUbdVEAEx_k-?format=jpg&amp;name=orig" referrerpolicy="no-referrer">'
var a = ''
var c = []
// var d = []
if(str.match('<img') != undefined){
    console.log('1-1')
    a = str.slice(str.indexOf('<img'),str.length)
    alen = a.match(/https/g).length
    for(var i = 0; i<alen; i++) {
        console.log('3')
        var img = a.slice(a.indexOf('https'),a.indexOf('orig')+4)
        c.push(img)
        a = a.replaceAll(img,'')
    }
} else if(str.match('<video') != undefined){
    console.log('2-1')
    v = str.slice(str.indexOf('poster'),str.lastIndexOf('</video>'))
    c.push(v.slice(v.indexOf('https'),v.indexOf('.jpg')+4))
}
/** 
if(str.match('<a') != undefined){
    console.log('2-2')
    b = str.slice(str.indexOf('<a'),str.lastIndexOf('</a>')+4)
    for(var i=0; i <b.match(/href/g).length; i++){
        console.log('4')
        var href = b.slice(b.indexOf('https'),b.indexOf('target')-2)
        d.push(href)
        b = b.replaceAll(href,'')
        b = b.replace('target','')
    }
}
*/
