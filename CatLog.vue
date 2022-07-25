<template>

    <Page class="page">
        <ActionBar title="Details" backgroundColor="white" color="grey">
            <NavigationButton text="Go Back" android.systemIcon="ic_menu_back"
                @tap="$navigateBack" />
        </ActionBar>
        <StackLayout>
            <WebView height="100%" :src="items.link.indexOf('https') != -1 ? items.link : 'https'+items.link.substr(4,items.link.length)" />
        </StackLayout>
    </Page>

</template>

<script>
    export default {
        props: ["items"],
        data() {
            return {};
        },
        methods: {
            hours(pubDate) {
                let date = new Date(pubDate);
                date.setHours(date.getHours() + 9);

                return date.toLocaleDateString();
            }
        }
    };
</script>

<style scope>
    Label {
        color: black;
		margin: 25px;
    }
</style>