<template>
  <div class="px-4 sm:px-6 lg:px-8">
    <div class="sm:flex sm:items-center">
      <div class="sm:flex-auto">
        <h1 class="text-base font-semibold leading-6 text-gray-900">Taux de change pour le {{ selectedDateDisplayed }}</h1>
        <p class="mt-2 text-sm text-gray-700">
          <ul class="list-none inline-block">
        <li v-for="currency in currencies">

        </li>
          </ul>
        </p>
      </div>
      <div class="mt-4 sm:mt-0 sm:ml-16 sm:flex-none">
        <input type="date" v-model="selectedDate" @change="getExchangesRates">
      </div>
    </div>
    <div class="mt-8 flow-root">
      <div class="-my-2 -mx-4 overflow-x-auto sm:-mx-6 lg:-mx-8">
        <div class="bg-red-400 py-4 px-4 text-center my-2 sm:rounded-lg" v-if="hasError">
          <p class="text-white">{{ errorMessage }}</p>
        </div>
        <div class="inline-block min-w-full py-2 align-middle sm:px-6 lg:px-8">
          <div class="overflow-hidden shadow ring-1 ring-black ring-opacity-5 sm:rounded-lg">
            <table class="min-w-full divide-y divide-gray-300">
              <thead class="bg-gray-50">
                <tr>
                  <th scope="col" class="text-center py-3.5 pl-4 pr-3 text-left text-sm font-semibold text-gray-900 sm:pl-6">Date</th>
                  <template v-for="(currency, currencyKey) in currencies" :key="currency">
                    <th scope="col" class="text-center px-3 py-3.5 text-left text-sm font-semibold text-gray-900">{{ currency }}</th>
                  </template>
                </tr>
              </thead>
              <tbody class="divide-y divide-gray-200">
                <tr v-for="(rate, date) in rates" :key="rate">
                  <td class="whitespace-nowrap py-4 pl-4 pr-3 text-sm font-medium text-gray-900 sm:pl-6 text-center">{{ date }}</td>
                  <template v-for="(currency, currencyKey) in currencies">
                    <td class="whitespace-nowrap px-3 py-4 text-sm text-gray-500 text-center">{{ rate[currency] }}</td>
                  </template>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import config from '../config.ts'

export default {
  name: "ExchangeTableRate",
  data() {
    return {
      //  Date sélectionnée
      selectedDate: '',

      //  Date affichée
      selectedDateDisplayed: '',

      //  Tableau de données
      rates: [],
      currencies: [],

      //  Devise courante
      currencyBase: 'USD',

      //  Message d'erreur
      hasError: false,
      errorMessage: '',
    };
  },
  mounted() {
    this.getDate();
    this.getCurrencies();
  },
  methods: {
    getDate() {
      let date = this.selectedDate === '' ? new Date() : new Date(this.selectedDate);
      let today = date.getFullYear() + '-' + (date.getMonth() + 1 < 10 ? `0${date.getMonth() + 1}` : date.getMonth() + 1) + '-' + (date.getDate() < 10 ? `0${date.getDate()}` : date.getDate());
      this.selectedDate = today;
      this.selectedDateDisplayed = date.toLocaleDateString('fr');
    },
    reload() {

    },
    getExchangesRates() {
      let currencies = this.currencies.toString();
      axios.get(`http://optidigital.test/api/rates/date/${this.selectedDate}/${this.currencyBase}`)
          .then(response => {
            this.rates = response.data.rates;
          })
          .catch(error => {
            console.log(error);
            this.hasError = true;
            this.errorMessage = error.response.data.error;
          });
    },
    getCurrencies() {
      this.getDate();
      axios.get(`http://optidigital.test/api/currencies`)
          .then(response => {
            let apiCurrencies = response.data.data;
            apiCurrencies.forEach((rate) => {
              this.currencies.push(rate.code);
            });

            this.getExchangesRates();
          });
    }
  }
}
</script>

<style scoped>

</style>