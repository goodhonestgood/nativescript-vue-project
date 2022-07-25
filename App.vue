<template>
    <Page>
        <ActionBar>
            <GridLayout width="100%" columns="auto, *">
                <Label class="title" :text="currentComponent"  col="0" width="80%"/>
                <Label text="MENU" @tap="$refs.drawer.nativeView.toggleDrawerState()" col="1"/>
            </GridLayout>
        </ActionBar>

        <RadSideDrawer ref="drawer">
            <StackLayout ~drawerContent backgroundColor="#ffffff">
                <!--<Label class="drawer-header" text="Menu"/>-->
                <Label text="News" @tap="onItemTap(0)" class="drawer-item" />
                <Label text="SNS" @tap="onItemTap(1)" class="drawer-item"/>
                <Label text="Setting" @tap="onItemTap(2)" class="drawer-item" />
            </StackLayout>

            <GridLayout ~mainContent columns="*" rows="*">
                <Frame row="0" col="0" id="frameid">
                    <component :is="currentComponent"/>
                </Frame>
            </GridLayout>
        </RadSideDrawer>
    </Page>
</template>

<script >
    import News from "./CatInfo";
    import SNS from "./sns";
    import Setting from "./settings";

  export default {
    data() {
      return {
        msg: 'jiyeon',
        currentComponent: "SNS",
        componentsArray: ["News", "SNS", "Setting"]
      }
    },
    methods: {
        onItemTap: function(index) {
            this.currentComponent = this.componentsArray[index];
            this.$refs.drawer.nativeView.closeDrawer();
        },
    },
    components: {
        News,
        SNS,
        Setting
    },
  }
</script>

<style scoped>
    .title {
        text-align: left;
        padding-left: 4;
        font-size: 20px;
    }

    .message {
        vertical-align: center;
        text-align: center;
        font-size: 20;
        color: #333333;
    }

    .drawer-header {
        padding: 16 16 16 16;
        margin-bottom: 16;
        background-color: #53ba82;
        color: #ffffff;
        font-size: 24;
    }

    .drawer-item {
        padding: 8 16;
        color: #333333;
        font-size: 16;
    }
</style>
