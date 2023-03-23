<template>
  <div class="header">
    <h3 class="font-medium">Where in the world ?</h3>
    <div class="toggle-theme" @click="toggleTheme">
      <img v-if="themeColor === 'theme-dark'" src="./assets/img/dark_mode.svg" alt="moon icon dark mode">
      <img v-else src="./assets/img/light_mode.svg" alt="sun icon light mode">
      <p>{{ themeColor === 'theme-dark' ? "Dark Mode" : "Light Mode" }}</p>
    </div>
  </div>
  <div class="main-content">
    <Transition name="slide-fade" mode="out-in">
      <div v-if="!showDetails" class="country-list">
        <div class="filters">
          <div class="search-bar">
            <img v-if="themeColor === 'theme-dark'" src="./assets/img/search_light.svg" alt="search icon"
                 class="search-icon">
            <img v-else src="./assets/img/search.svg" alt="search icon" class="search-icon">
            <input class="font-xs" type="text" placeholder="Search for a country ..."
                   @change="fetchData($event)" v-model="lastSearch">
          </div>

          <div class="select-wrap">
            <select v-model="regionFilter" required>
              <option value="" hidden>Filter by region</option>
              <option value="">All</option>
              <option value="Africa">Africa</option>
              <option value="Americas">America</option>
              <option value="Asia">Asia</option>
              <option value="Europe">Europe</option>
              <option value="Oceania">Oceania</option>
            </select>
          </div>
        </div>

        <div class="grid">
          <template v-for="country in dataCountries" class="card">
            <div v-show="!regionFilter || regionFilter === country.region" class="card"
                  @click="setCurrent(country)">
              <div class="card__header">
                <img :src="country.flags.svg" :alt="country.name.common + '\'s flag'">
              </div>
              <div class="card__footer">
                <h4 class="font-small">{{ country.name.common }}</h4>
                <div class="country-stats">
                  <p class="font-xs">
                    <span class="category">Region : </span>
                    {{ country.region }}
                  </p>
                  <p class="font-xs">
                    <span class="category">Capital : </span>
                    {{ country.capital ? country.capital[0] : "" }}
                  </p>
                  <p class="font-xs">
                    <span class="category">Currency : </span>
                    {{ country.currencies ? Object.values(country.currencies)[0].name : "" }}
                  </p>
                </div>
              </div>
            </div>
          </template>
        </div>
      </div>

      <div v-else class="country-details">
        <button class="custom-button font-xs" @click="showDetails = false">
          <span>←</span>Back
        </button>
        <div class="country-details__content">
          <div class="flag-wrap">
            <img :src="currentCountry.flags.svg"
                 :alt="currentCountry.name.common + '\'s flag'">
          </div>
          <div class="country-info">
            <h2 class="country-info__title">{{ currentCountry.name.common }}</h2>
            <div class="country-info__content">
              <div class="country-info__firstCol">
                <p><span class="category">Native Name : </span>
                  {{ currentCountry.nativeName ? Object.values(currentCountry.nativeName)[0].common : ""}}
                </p>
                <p><span class="category">Population : </span>{{ currentCountry.population.toLocaleString() }}</p>
                <p><span class="category">Region : </span>{{ currentCountry.region }}</p>
                <p><span class="category">Sub Region : </span>{{ currentCountry.subRegion }}</p>
                <p><span class="category">Capital : </span>{{ currentCountry.capital ? currentCountry.capital[0] : "" }}</p>
              </div>
              <div class="country-info__secCol">
                <p><span class="category">Top Level Domain : </span>{{ currentCountry.region }}</p>
                <p>
                  <span class="category">Currencies : </span>
                  <template v-if="currentCountry.currencies">
                    <template v-for="(currency,index) in currentCountry.currencies">
                      {{ currency.name }}<span v-if="index < Object.values(currentCountry.currencies).length - 1">, </span>
                    </template>
                  </template>
                </p>
                <p>
                  <span class="category">Languages : </span>
                  <template v-if="currentCountry.languages">
                    <template v-for="(language,key,index) in currentCountry.languages">
                      {{ language }}<span v-if="index < Object.values(currentCountry.languages).length - 1">, </span>
                    </template>
                  </template>
                </p>
              </div>
              <div class="country-info__borders">
                <span class="category">Border Countries : </span>
                <button v-for="border in currentBorders" class="custom-button"
                        @click="setCurrent(border)">
                  {{ border.name.common }}
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script>
export default {
  data() {
    return {
      themeColor: "",
      lastSearch: "",
      dataCountries: [],
      baseURL: "https://restcountries.com/v3.1/",
      regionFilter: "",
      showDetails: false,
      currentCountry: {},
      currentBorders: [],
    }
  },
  beforeMount() {
    this.fetchData();
  },
  mounted() {
    this.themeColor = localStorage.getItem("theme") ?? "theme-dark";
    document.querySelector("body").setAttribute("data-theme", this.themeColor);
  },
  methods: {
    toggleTheme() {
      this.themeColor = this.themeColor === 'theme-dark' ? 'theme-light' : 'theme-dark';
      document.querySelector("body").setAttribute("data-theme", this.themeColor);
    },

    fetchData(event) {
      let addToURL = (event && event.target.value) ? "name/" + event.target.value : "all";
      fetch(this.baseURL
          + addToURL)
          .then((response) => response.json())
          .then((data) => this.dataCountries = data);
    },
    setCurrent(country) {
      this.currentCountry = country;
      this.showDetails = true;
      this.setBorders(country.borders);
    },
    setBorders(borders) {
      let addToURL = "alpha?codes=" + borders.join(',');
      fetch(this.baseURL
          + addToURL)
          .then((response) => response.json())
          .then((data) => this.currentBorders = data);
    }
  },
}
</script>





