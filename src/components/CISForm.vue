<template>
    <div>
        <div class="row">
            <div class="col">
                <form class="bg-yayellow b-radius-small p-5 m-0">
                    <div class="row mb-3">
                        <div class="col-6"><h3 class="fw-normal">Найти автомобиль</h3></div>
                        <div class="col-6 text-end">
                            <ul class="list-inline text-minus pt-2">
                                <li class="list-inline-item ms-3">
                                    <a 
                                        href="#" 
                                        class="text-uppercase c-yablack c-h-yablue text-decoration-none letter-spacing-plus fw-bold"
                                        :class="{'c-yablue': link =='new' }"
                                        @click.prevent="getBrands('new')"
                                        >Новые</a>
                                </li>
                                <li class="list-inline-item ms-3">
                                    <a 
                                        href="#" 
                                        class="text-uppercase c-yablack c-h-yablue text-decoration-none letter-spacing-plus fw-bold"
                                        :class="{'c-yablue': link =='used' }"
                                        @click.prevent="getBrands('used')"
                                        >С пробегом</a>
                                </li>
                            </ul>
                        </div>
                    </div>
                    <div class="row">
                        <div class="col">
                            <select 
                                class="form-select form-select-lg b-radius-small"
                                v-if="response"
                                v-model="brandIndx"
                                >
                                <option selected value="null">Все бренды</option>
                                <option
                                    v-for="(item, indx) in response.filter.brands"
                                    :key="item.id"
                                    :value="indx"
                                    >{{ item.name }}</option>
                            </select>
                            <select 
                                class="form-select form-select-lg b-radius-small"
                                v-else
                                >
                                <option selected disabled value="null">Все бренды</option>
                            </select>
                        </div>
                        <div class="col">
                            <select 
                                class="form-select form-select-lg b-radius-small"
                                v-if="response.filter.brands[brandIndx]"
                                v-model="modelIndx"
                                >
                                <option selected value="null">Все модели</option>
                                <option
                                    v-for="(item, indx) in response.filter.brands[brandIndx].models"
                                    :key="item.id"
                                    :value="indx"
                                    >{{ item.name }}</option>
                            </select>
                            <select 
                                class="form-select form-select-lg b-radius-small"
                                v-else>
                                <option selected disabled value="null">Все модели</option>
                            </select>
                        </div>
                        <div class="col">
                            <div 
                                class="bg-yawhite b-yagray b-radius-small position-relative"
                                v-if="response">
                                <div class="row px-3 pt-1 mb-2">
                                    <div class="col-6 text-start">
                                        <div class="text-minus-minus c-yagray" style="line-height: 1;">От</div>
                                        {{ Format(rangeValue[0]) }} ₽
                                    </div>
                                    <div class="col-6 text-end">
                                        <div class="text-minus-minus c-yagray" style="line-height: 1;">До</div>
                                        {{ Format(rangeValue[1]) }} ₽
                                    </div>
                                </div>
                                <section class="cis-filter-on-main-range-slider">
                                    <vue-slider 
                                        v-model="rangeValue"
                                        :min="rangeMin"
                                        :max="rangeMax"
                                        :interval="rangeInterval"
                                        tooltip="none"
                                        ></vue-slider>
                                </section>
                            </div>
                            <div 
                                class="bg-yawhite b-yagray b-radius-small py-4"
                                v-else></div>
                        </div>
                        <div class="col">
                            <a 
                                :href="buttonLink" 
                                class="d-block text-center c-yawhite c-h-yawhite bg-yablue bg-h-yadarkblue text-decoration-none b-radius-small but-lg"
                                >Показать {{ Format(totalCount) }} авто</a>
                        </div>
                    </div>
                </form>
            </div>
        </div>

    </div>
</template>

<script>
import VueSlider from 'vue-slider-component'
import 'vue-slider-component/theme/default.css'

export default {
    name: 'CISForm',
    components: {
        VueSlider
    },
    data: function() {
        return {
            rangeValue: [0, 0],
            rangeMin: 0,
            rangeMax: 100000000,
            brandIndx: null,
            modelIndx: null,
            totalCount: 0,
            rangeInterval: 1,
            buttonLink: '/cars/new'
        }
    },
    computed: {

        link() {return this.$root.link},
        response() {return this.$root.response},
    },
    watch: {
        brandIndx: function(newValue) {

            let total = 0
            if ( this.brandIndx == null || this.brandIndx == 'null' ) {
                this.$root.response.filter.brands.forEach(function(item) { total += Number(item.vehicles) })
            } else {
                total = Number(this.$root.response.filter.brands[newValue].vehicles)
            }
            this.totalCount = total

            this.buildLink()
        }
    },

    mounted: function() {

        setTimeout(() => {
            this.buildRange()
            this.buildTotal()
            this.buildLink()

            console.log(this.response.filter)
        }, 500);
    },

    methods: {

        buildLink() {

            this.buttonLink = '/cars/'+this.link
            if ( this.brandIndx !== null || this.brandIndx != 'null' ) this.buttonLink += '/'+this.response.filter.brands[this.brandIndx].alias
            this.buttonLink += '?minprice='+this.rangeValue[0]+'&maxprice='+this.rangeValue[1]
        },
        buildRange() {
            this.rangeMin = Number( this.$root.response.filter.minPrice )
            this.rangeMax = Number( this.$root.response.filter.maxPrice )
            this.rangeValue = [this.rangeMin, this.rangeMax]
        },
        buildTotal() {
            let total = 0
            this.$root.response.filter.brands.forEach(function(item) { total += Number(item.vehicles) })
            this.totalCount = total
        },


        Format(q) {
			
            var Price = new Intl.NumberFormat('ru', { currency: 'RUR' });
            return Price.format(q);	
        }
    }
}
</script>

<style scoped>
.cis-filter-on-main-range-slider {
    position: absolute;
	bottom: -25px;
    width: 100%;
    height: 35px;
    text-align: center;
}
</style>
