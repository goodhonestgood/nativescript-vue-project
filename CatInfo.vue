<template>
    <Page class="page" actionBarHidden="true">
        <StackLayout>
            <TextField v-model="searchPhrase"
            hint="Enter text..." style="font-size: 20px;"
            returnKeyType="done"/>
            <Button class="btn btn-primary" text="search" @tap="fetchData()" />
            <GridLayout columns="*" rows="1*,7*" class="home-panel">
                <Label row="0" col="0" v-if="lastbuilddate != ''" :text="'검색 시간 '+new Date(lastbuilddate).toLocaleTimeString()" class="borderdash"/>
                <ListView row="1" col="0" v-if="items.length > 0" for="item in items" @itemTap="onItemTap"
                    style="height:1250px">
                    <v-template>
                        <FlexboxLayout flexDirection="row">
                            <Label :text="subtitle(item.title)" class="t-12"/>
                        </FlexboxLayout>
                    </v-template>
                </ListView>
                <Label row="0" col="0" v-else text="검색 결과 없음" style="fontSize:13px"/>
            </GridLayout>
        </StackLayout>
    </Page>

</template>

<script>
    var client_id = 'q5LSbXPY_eSqd1axrPG_';
    var client_secret = '4xH68Gzbuk';
    import axios from "axios";
    const utilsModule = require("tns-core-modules/utils/utils");
    export default {
        data() {
            return {
                searchPhrase: '',
                items: [],
                description: '',
                lastbuilddate: ''
            };
        },
        methods: {
            subtitle: function(str) {
                var str = str.replace('<b>','');
                str = str.replace('</b>','')
                return str
            },
            onItemTap: function(index) {
                console.log(this.items[index.index].link);
                utilsModule.openUrl(this.items[index.index].link)
            },
            fetchData: function() {
                this.items = []
                this.description = ''
                this.lastbuilddate = ''
                var api_url = "https://openapi.naver.com/v1/search/news.json"
                var options = {
                    params:{ query: this.searchPhrase, display: 30, sort: "date" },
                    headers: { 'X-Naver-Client-Id': client_id, 'X-Naver-Client-Secret': client_secret }
                };
                axios.get(api_url,options)
                    .then(response => {
                        this.items = response.data.items;
                        this.lastbuilddate = response.data.lastBuildDate;
                        console.log(this.items)
                    })
                    .catch(error => {
                        // eslint-disable-next-line
                        console.log("error");
                        console.log(error);
                    });
            },
        },
        created: function() {
        }
    };
</script>

<style scope>
    .home-panel {
        font-size: 20;
        margin: 15;
    }
    .borderdash {
        border:1px solid black;
    }
    .description-label {
        margin-bottom: 15;
    }
</style>