<template>
    <div class="filters">
        <v-row>
            <v-col>
                <h2 class="subtitle-2">Filtrer par : </h2>
            </v-col>
        </v-row>
        <v-row>
            <v-col>
                <v-select
                        v-model="regionSelected"
                        :items="regionList"
                        label="Choisissez"
                        multiple
                        chips
                        hint="Dans quelle région cherchez vous ?"
                        persistent-hint
                        @change="getDeptByRegions(regionSelected)"
                ></v-select>
            </v-col>
        </v-row>
        <v-row>
            <v-col>
                <v-select
                        v-model="deptSelected"
                        :items="deptList"
                        label="Choisissez"
                        multiple
                        chips
                        hint="Dans quel departement cherchez vous ?"
                        persistent-hint
                        @change="changeDept"
                ></v-select>
            </v-col>
        </v-row>
    </div>
</template>

<script>
    import axios from 'axios'

    export default {
        name: "FiltersList",
        props: ['options'],
        data() {
            return {
                deptSelected: [],
                regionSelected: [],
                deptList: [],
                regionList: []
            }
        },
        mounted() {
            this.deptList = this.initRegions();
            this.getDept(this.deptList);
        },
        methods: {
            async initRegions () {
                await axios.get('https://geo.api.gouv.fr/regions')
                    .then((regions) => {
                        this.regionList = regions.data.map((data) => {
                            return {text: data.nom, value: data}
                        });
                        let regionsCode = this.regionList.map((region) => region.value.code);
                        return this.getDeptByRegions(regionsCode);
                    })
            },
            getDeptByRegions(regionsCode) {
                let deptList = [];
                 regionsCode.forEach((code) => {
                     deptList = axios.get('https://geo.api.gouv.fr/regions/' + code + '/departements')
                        .then((dept) => {
                            return deptList
                                .concat(
                                    dept.data.map((data) => {
                                        return {text: '(' + data.code + ') ' + data.nom, value: data}
                                    })
                                )
                                .sort((a, b) => {
                                    return a.value.code - b.value.code
                                });
                        })
                });
                return deptList;
            },
            getDept(deptList) {
                // eslint-disable-next-line no-console
                console.log(deptList);
                if(this.options.some(option => option.name === 'refine.nom_dept')) {
                    const index = this.options.findIndex(option => option.name === 'refine.nom_dept');
                    this.options[index].value = deptList.map(dept => dept.value);
                } else {
                    this.options.push({name: 'refine.nom_dept', value: deptList.map(dept => dept.value)});
                }
            },
            changeDept() {
                if(this.options.some(option => option.name === 'refine.nom_dept')) {
                    const index = this.options.findIndex(option => option.name === 'refine.nom_dept');
                    this.options[index].value = this.deptSelected;
                } else {
                    this.options.push({name: 'refine.nom_dept', value: this.deptSelected});
                }
                this.$emit('update:options', this.options);
            }
        }
    }
</script>

<style scoped>
    .filters {
        padding: 1rem;
    }
</style>