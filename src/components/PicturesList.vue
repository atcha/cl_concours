<template>
    <div class="container" v-if="pictures.length > 0">
        <div class="card" v-for="picture in pictures" :key="picture.recordid">
            <v-img :src="picture.fields.video_v" :alt="picture.fields.edif">
                <template v-slot:placeholder>
                    <v-row
                            class="fill-height ma-0"
                            align="center"
                            justify="center"
                    >
                        <v-progress-circular indeterminate color="grey lighten-5"></v-progress-circular>
                    </v-row>
                </template>
            </v-img>
        </div>
    </div>
</template>

<script>
    import axios from 'axios'

    export default {
        name: "PicturesList",
        props: ['options'],
        data() {
            return {
                start: '0',
                pictures: [],
                pictureUrl: 'https://data.opendatasoft.com/explore/dataset/photographies-serie-monuments-historiques-de-1851-a-1914@culture/files/'
            }
        },
        mounted() {

            this.getPictures(this.options);
            this.scroll();
        },
        methods: {
            getPictures(options) {
                let query = '';
                options.forEach((option) => {
                    if(option.name.includes('refine')) {
                        option.value.forEach((opt) => {
                            query += '&' + option.name + '=' + opt.nom.toUpperCase();
                        });
                        query += '&disjunctive.' + option.name.split('.')[1] + '=true'
                    } else {
                        query += '&' + option.name + '=' + option.value;
                    }
                });
                query += '&start=' + this.start;
                axios.get('https://data.opendatasoft.com/api/records/1.0/search/?dataset=photographies-serie-monuments-historiques-de-1851-a-1914%40culture&facet=pays&facet=nom_reg&facet=nom_dept&refine.pays=France'+query)
                    .then((data) => {
                        if(this.start !== '0') {
                            this.pictures = this.pictures.concat(data.data.records);
                        } else {
                            this.pictures = data.data.records;
                        }
                    })
            },
            getFullPicture(picture) {
                return {
                    pictureUrl: this.pictureUrl + picture.video_p.id + '/300',
                    pictureWidth: picture.video_p.width,
                    pictureHeight: picture.video_p.height
                }
            },
            scroll() {
                window.onscroll = () => {
                    let bottomOfWindow = document.documentElement.scrollTop + window.innerHeight === document.documentElement.offsetHeight;

                    if (bottomOfWindow) {
                        const indexrows = this.options.findIndex(option => option.name === 'rows');
                        this.start = parseInt(this.start) + parseInt(this.options[indexrows].value);
                        this.getPictures(this.options);
                    }
                }
            }
        },
        watch: {
            options: {
                handler (newOptions) {
                    this.getPictures(newOptions);
                },
                deep: true
            }
        }
    }
</script>

<style scoped>
    .container {
        display: flex;
        flex-wrap: wrap;
        align-content: flex-start;
    }
    .card {
        border-radius: 0.3rem;
        margin: .3rem;
    }
    .card header {
        text-align: left;
    }
</style>