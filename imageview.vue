<template>

    <Page class="page">
        <ActionBar title="상세정보" backgroundColor="white" color="grey">
            <NavigationButton text="Go Back" android.systemIcon="ic_menu_back"
                @tap="$navigateBack" />
        </ActionBar>
        <StackLayout>
            <Image :src="image" />
            <Button @tap="open(image)" text="이미지 저장 링크" />
            <Button @tap="open(link)" text="글 링크" />
        </StackLayout>
    </Page>

</template>

<script>
    const utilsModule = require("tns-core-modules/utils/utils");
    export default {
        props: ["image","link"],
        data() {
            return {
            };
        },
        methods: {
            open: function(url){
                utilsModule.openUrl(url)
            }
        },
        created() {
            console.log(this.image)
        }
    };
</script>

<style scope>
    Label {
        color: black;
		margin: 25px;
    }
</style>