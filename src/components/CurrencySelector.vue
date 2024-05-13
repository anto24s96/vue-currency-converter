<script>
import VueApexCharts from "vue3-apexcharts";

import axios from 'axios';
import { store } from '../store';

export default {
    name: 'CurrencySelector',
    components: {
        apexchart: VueApexCharts,
    },
    props: {
        currencies: Array,
    },
    data() {
        return {
            store,
            currencyOne: 'EUR',
            currencyTwo: 'USD',
            amountOne: 1,
            amountTwo: 1,
            series: [{
                name: "Frankfurt Exchange",
                data: []
            }],
            chartOptions: {
                chart: {
                    type: 'area',
                    height: 350,
                    zoom: {
                        enabled: true
                    }
                },
                dataLabels: {
                    enabled: false
                },
                stroke: {
                    curve: 'straight'
                },
                title: {
                    text: 'Rapporto tra le valute',
                    align: 'left',
                    style: {
                        color: '#ffffff'
                    }
                },
                subtitle: {
                    text: 'Price Movements',
                    align: 'left',
                    style: {
                        color: '#ffffff'
                    }
                },
                xaxis: {
                    type: 'datetime',
                    labels: {
                        style: {
                            colors: '#ffffff'
                        }
                    }
                },
                yaxis: {
                    opposite: false,
                    labels: {
                        style: {
                            colors: '#ffffff'
                        }
                    }
                },
                legend: {
                    horizontalAlign: 'left'
                },
                fill: {
                    colors: ['#FCA311']
                },
            },

        }
    },
    mounted() {
        this.getValues('amountOne');
        this.getChart(this.currencyOne, this.currencyTwo, this.amountOne);
    },
    methods: {
        async getValues(from) {
            try {
                if (!this.amountOne || !this.amountTwo) return;

                const amount = from === 'amountOne' ? this.amountOne : this.amountTwo;
                const fromCurrency = from === 'amountOne' ? this.currencyOne : this.currencyTwo;
                const toCurrency = from === 'amountOne' ? this.currencyTwo : this.currencyOne;

                const response = await axios.get(`${this.store.endpoint}/latest?amount=${amount}&from=${fromCurrency}&to=${toCurrency}`);

                const rates = Object.values(response.data.rates);

                if (from === 'amountOne') {
                    this.amountTwo = rates;
                    this.store.fromChange = `${this.amountOne} ${this.currencyOne}`;
                    this.store.toChange = `${this.amountTwo} ${this.currencyTwo}`;
                } else if (from === 'amountTwo') {
                    this.amountOne = rates;
                    this.store.fromChange = `${this.amountOne} ${this.currencyOne}`;
                    this.store.toChange = `${this.amountTwo} ${this.currencyTwo}`;
                }

                await this.getChart(fromCurrency, toCurrency, amount);

            } catch (error) {
                console.error(error);
            }
        },
        async updateChart() {
            const toCurrency = this.currencyTwo;
            const fromCurrency = this.currencyOne;
            const amount = this.amountTwo;
            await this.getChart(fromCurrency, toCurrency, amount);
        },
        async getChart(fromCurrency, toCurrency, amount) {
            try {
                const date = '2024-01-01..';

                const response = await axios.get(`${this.store.endpoint}/${date}?amount=${amount}&from=${fromCurrency}&to=${toCurrency}`);
                const rates = response.data.rates;

                const data = Object.keys(rates).map(date => ({
                    x: new Date(date),
                    y: rates[date][toCurrency]
                }));
                this.series = [{ data }];

            } catch (error) {
                console.error(error);
            }
        },
    },

    computed: {
        displayAmountOne() {
            return `${this.amountOne} ${this.currencyOne}`;
        },
        displayAmountTwo() {
            return `${this.amountTwo} ${this.currencyTwo}`;
        },
    }
}
</script>
<template lang="">
    <div class="row">
        <h1 class="text-center text-uppercase fw-bolder my-title">Currency boolverter</h1>
        <div class="col my-3">
            <div class="fs-5 text-white"><span class="fst-italic">{{ displayAmountOne }}</span> è uguale a</div>
            <div class="fs-1 text-white fst-italic">{{ displayAmountTwo }}</div>
    </div>
    <div class="row justify-content-center text-white">
        </div>

        <div class="col-12">
            <div class="input-group mb-3">
                <input type="text" class="form-control text-center my-select" aria-label="Nome" v-model="amountOne" @keyup="getValues('amountOne')">

                <select class="form-select text-center py-3 my-select" id="selectOption" v-model="currencyOne" @change="getValues('amountOne')">
                    <option v-for="(currency, index) in currencies" :key="index" :value="currency.id" :disabled="currency.id === currencyTwo">{{ currency.name }}</option>
                </select>
            </div>
        </div>
        <div class="col-12">
            <div class="input-group mb-3">
                <input type="text" class="form-control text-center py-3 my-select" aria-label="Nome" v-model="amountTwo" @keyup="getValues('amountTwo')">

                <select class="form-select text-center my-select" id="selectOption" v-model="currencyTwo" @change="getValues('amountOne')">
                    <option v-for="(currency, index) in currencies" :key="index" :value="currency.id" :disabled="currency.id === currencyOne">{{ currency.name }}</option>
                </select>
            </div>
        </div>
    </div>

    <div class="my-3">
        <apexchart type="area" height="350" :options="chartOptions" :series="series"></apexchart>
    </div>
    
</template>

<style lang="scss" scoped></style>