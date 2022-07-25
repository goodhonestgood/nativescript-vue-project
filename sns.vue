<template>
    <Page class="page" actionBarHidden="true">
        <ScrollView>    
            <GridLayout rows="auto, *, auto" columns="*, *, *">
                <Button text="Twitter" @tap="onItemTap(0)" row="0" col="0" :class="{btnClick : isActive[0]}"/>
                <Button text="Instagram" @tap="onItemTap(1)" row="0" col="1" :class="{btnClick : isActive[1]}"/>
                <Button text="Youtube" @tap="onItemTap(2)" row="0" col="2" :class="{btnClick : isActive[2]}"/>
                <Frame  row="1" col="0" colSpan="3" >
                    <component :is="currentComponent"/>
                </Frame>
            </GridLayout>
        </ScrollView>
    </Page>
</template>

<script>
    import CatInfo2 from "./CatInfo2";
    import CatInfo3 from "./CatInfo3";
    import CatInfo4 from "./CatInfo4";

    export default {
        data() {
            return {
                currentComponent: "CatInfo2",
                componentsArray: ["CatInfo2", "CatInfo3","CatInfo4"],
                isActive: [true, false, false],
            }
        },
        methods: {
            onItemTap: function(index) {
                this.currentComponent = this.componentsArray[index];
                this.isActive = [false, false, false]
                this.isActive[index] = true;
            },
        },
        components: {
            CatInfo2,
            CatInfo3,
            CatInfo4
        }
    };
</script>
<style scoped>
    .btnClick {
        background: rgb(201, 201, 201);
    }
    Button {
        background: white;
        margin: 2px;
    }
</style>
