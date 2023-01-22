<template>
  <div>
    <h2>{{ title }}</h2>
    <input type="text" v-model="filter" />
    <currency-selector
      :currencies="allCurrencies"
      :selectedCurrency="baseCurrency"
      @currencyChange="updateBaseCurrency"
    ></currency-selector>
    <select v-model="selectRoad">
      <option value="">Carte Routière</option>
      <option value="k">Satellite</option>
    </select>
    <table class="table">
      <thead>
        <tr>
          <th>Name</th>
          <th>Language</th>
          <th>Currencies</th>
          <th>Map</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="country in filteredCountries" v-bind:key="country.fifa">
          <td>
            <button class="button is-ghost" @click="openFlag(country)">
              {{ country.name.common }}
            </button>
          </td>
          <td>
            <ul>
              <li v-for="language in country.languages" :key="language">
                {{ language }}
              </li>
            </ul>
          </td>
          <td>
            <ul>
              <li v-for="(cur, key) in country.currencies" :key="key">
                <button
                  class="button is-ghost"
                  @click="openExchangeRate(country, key)"
                  >
                  {{ key }}
                </button>
              </li>
            </ul>
          </td>
          <td>
            <button class="button is-ghost" @click="openMap(country)">
              Map
            </button>
          </td>
        </tr>
      </tbody>
    </table>
    <div class="modal" :class="{ 'is-active': showOverlay }">
      <div class="modal-background"></div>
      <div class="modal-content">
        <component :is="targetComponent" v-bind="targetProperties" v-on="targetEventHandlers"></component>
      </div>
      <button
        class="modal-close is-large"
        aria-label="close"
        @click="closeOverlay"
      ></button>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import CountryFlag from "./components/CountryFlag.vue";
import CountryMap from "./components/CountryMap.vue";
import CountryExchangeRate from "./components/CountryExchangeRate.vue";
import CurrencySelector from "./components/CurrencySelector.vue";

export default {
  name: "App",
  components: {
    CountryFlag,
    CountryMap,
    CountryExchangeRate,
    CurrencySelector,
  },
  data() {
    return {
      title: "World Explorer",
      countries: [],
      filter: "",
      baseCurrency: "EUR",
      targetComponent: "",
      targetCountry: null,
      targetCurrency: "",
      roadType: "",
    };
  },
  methods: {
    fetchCountries: function () {
      axios
        .get("https://restcountries.com/v3.1/all")
        .then((response) => (this.countries = response.data))
        .catch((error) => console.error(error));
    },
    updateTargetCurrency(newCurrency) {
      this.targetCurrency = newCurrency;
    },
    updateBaseCurrency(newCurrency) {
      this.baseCurrency = newCurrency;
    },
    updateLocalRoadType(newRoad){
      this.roadType = newRoad;
    },
    openFlag: function (country) {
      this.targetCountry = country;
      this.targetComponent = "CountryFlag";
    },
    openMap: function (country) {
      this.targetCountry = country;
      this.targetComponent = "CountryMap";
    },
    openExchangeRate: function (country, currency) {
      this.targetCountry = country;
      this.targetCurrency = currency;
      this.targetComponent = "CountryExchangeRate";
    },
    closeOverlay: function () {
      this.targetComponent = "";
    },
  },
  computed: {
    sortedCountries: function () {
      return [...this.countries].sort((c1, c2) =>
        c1.name.common.localeCompare(c2.name.common)
      );
    },
    filteredCountries: function () {
      return this.sortedCountries.filter((c) =>
        c.name.common.toLowerCase().includes(this.filter.toLowerCase())
      );
    },
    targetProperties: function () {
      let props = {};

      switch (this.targetComponent) {
        case "CountryFlag":
          props.country = this.targetCountry;
          break;
        case "CountryMap":
          props.country = this.targetCountry;
          props.roadType = this.roadType;
          break;
        case "CountryExchangeRate":
          props.country = this.targetCountry;
          props.currencies = this.allCurrencies;
          props.currency = this.targetCurrency;
          props.baseCurrency = this.baseCurrency;
          break;
        default:
          props = null;
          break;
      }

      return props;
    },
    targetEventHandlers: function () {
      if (this.targetComponent === "CountryExchangeRate")
        return {
          currencyChange: this.updateTargetCurrency,
          baseCurrencyChange: this.updateBaseCurrency,
        }
        else if (this.targetComponent === "CountryMap") return {
          updateRoadType: this.updateLocalRoadType,
        };

      return {};
    },
    showOverlay: function () {
      return this.targetComponent !== "";
    },
    allCurrencies: function () {
      if (this.countries.length > 0) {
        const currencies = [];

        this.countries.forEach((country) => {
          if (country.currencies)
            currencies.push(...Object.keys(country.currencies));
        });
        return [...new Set(currencies)].sort();
      }

      return [];
    },
  },
  mounted() {
    this.fetchCountries();
  },
};
</script>

<style>
@import "~bulma/css/bulma.css";
</style>
