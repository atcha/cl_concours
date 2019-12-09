<template>
    <v-app>
        <v-app-bar
                app
                color="primary"
                dark
                elevation="0"
        >
            <div class="d-flex align-center">
                « Monuments historiques » de 1851 à 1914
            </div>
        </v-app-bar>

        <v-content>
            <FiltersList :options.sync="options"/>
            <PicturesList :options="options"/>
        </v-content>
    </v-app>
</template>

<script>
    import PicturesList from './components/PicturesList';
    import FiltersList from "@/components/FiltersList";

    export default {
        name: 'App',

        components: {
            PicturesList,
            FiltersList
        },

        data: () => ({
            options: [
                {name: 'rows', value: '80'}
            ]
        }),
    };
</script>

<style>
    .v-content__wrap {
        display: grid;
        grid-template-columns: 300px 1fr;
    }
</style>
