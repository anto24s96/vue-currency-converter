<script>
import CurrencySelector from './components/CurrencySelector.vue';

import axios from 'axios';
import { store } from './store';

export default {
    components: {
        CurrencySelector,
    },
    data() {
        return {
            store,
            currencies: []
        }
    },
    mounted() {
        this.getCurrencies();
    },
    methods: {
        async getCurrencies() {
            try {
                // Effettuo una richiesta GET per ottenere le valute dall'endpoint
                const response = await axios.get(`${this.store.endpoint}/currencies`);

                // Estraggo l'oggetto delle valute dalla risposta
                const currenciesObj = response.data;

                // Mappo le valute in un array di oggetti con chiave 'name' e 'id'
                this.currencies = Object.keys(currenciesObj).map((key) => ({
                    name: currenciesObj[key],
                    id: key
                }));
            } catch (error) {
                console.error('Errore durante il recupero delle valute:', error);
            }
        }

    },
}
</script>

<template lang="">
    <div class="my-container">
        <CurrencySelector :currencies="currencies" />
    </div>
</template>

<style lang="scss">
@use './styles/generals.scss' as *;
@use './styles/partials/variables' as *;
</style>