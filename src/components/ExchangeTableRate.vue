<template>
  <div class="px-4 sm:px-6 lg:px-8">
    <div class="sm:flex sm:items-center">
      <div class="sm:flex-auto">
        <h1 class="text-base font-semibold leading-6 text-gray-900">Taux de change pour le {{ (new Date(selectedDate)).toLocaleDateString('fr') }} depuis {{ currencyBase }}</h1>
        <ul class="list-none inline-block mt-2 text-sm text-gray-700" v-if="loaded && !hasError && rates[selectedDate]">
          <li class="inline px-4" v-for="currency in currencies">
            <strong>{{ currency }}</strong> : {{ rates[selectedDate].hasOwnProperty(currency) ? rates[selectedDate][currency] : 'N.C' }}
          </li>
        </ul>
      </div>
      <div class="mt-4 sm:mt-0 sm:ml-16 sm:flex-none flex">
        <div class="mr-2">
          <label for="date" class="sr-only">Date</label>
          <input type="date" name="date" id="date" v-model.lazy="selectedDate" @change="changeDate" class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm">
        </div>
        <div>
          <label for="base" class="sr-only">Conversion depuis</label>
          <select id="base" name="base" v-model="currencyBase" @change="changeCurrencyBase" class="block w-full rounded-md border-gray-300 py-2 pl-3 pr-10 text-base focus:border-indigo-500 focus:outline-none focus:ring-indigo-500 sm:text-sm">
            <template v-for="currency in currencies">
              <option>{{ currency }}</option>
            </template>
          </select>
        </div>
        <div>
          <button type="button" @click.prevent="reload" class="ml-3 inline-flex items-center rounded-md border border-transparent bg-indigo-500 px-4 py-2 text-sm font-medium text-white shadow-sm hover:bg-indigo-600 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2 focus:ring-offset-gray-800">Recharger</button>
        </div>
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
              <tbody class="bg-white divide-y divide-gray-200">
                <tr v-for="(rate, date) in rates" :key="rate">
                  <td class="whitespace-nowrap py-4 pl-4 pr-3 text-sm font-medium text-gray-900 sm:pl-6 text-center">{{ (new Date(date)).toLocaleDateString('fr') }}</td>
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

      //  Tableau de données
      rates: [],
      currencies: [],

      //  Devise courante
      currencyBase: 'USD',

      //  Message d'erreur
      hasError: false,
      errorMessage: '',

      loaded: false,
    };
  },
  mounted() {
    this.getDate();
    this.getCurrencies();
  },
  methods: {
    changeDate() {
        this.loaded = false;
        this.getExchangesRates();
    },
    getDate() {
      let date = this.selectedDate === '' ? new Date() : new Date(this.selectedDate);
      let today = date.getFullYear() + '-' + (date.getMonth() + 1 < 10 ? `0${date.getMonth() + 1}` : date.getMonth() + 1) + '-' + (date.getDate() < 10 ? `0${date.getDate()}` : date.getDate());
      this.selectedDate = today;
    },
    reload() {
      this.resetError();
      this.selectedDate = '';
      this.currencyBase = 'USD';
      this.currencies = [];
      this.getCurrencies();
    },
    getExchangesRates() {
      this.resetError();
      let currencies = this.currencies.toString();
      axios.get(`http://optidigital.test/api/rates/date/${this.selectedDate}/${this.currencyBase}`)
          .then(response => {
            this.rates = response.data.rates;
            console.log(this.rates);
            this.loaded = true;
          })
          .catch(error => {
            console.log(error);
            this.hasError = true;
            this.errorMessage = error.response.data.error;
          });
    },
    getCurrencies() {
      this.getDate();
      this.resetError();
      axios.get(`http://optidigital.test/api/currencies`)
          .then(response => {
            let apiCurrencies = response.data.data;
            apiCurrencies.forEach((rate) => {
              this.currencies.push(rate.code);
            });

            this.getExchangesRates();
          })
          .catch(error => {
            console.log(error);
            this.hasError = true;
            this.errorMessage = error.response.data.error;
          });
    },
    changeCurrencyBase() {
      this.resetError();
      this.getExchangesRates();
    },
    resetError() {
      this.hasError = false;
      this.errorMessage = '';
    }
  }
}
</script>

<style scoped>

</style>