<template>
  <q-page class="flex column" :class="bgClass"
  >
    <div class="col q-pt-lg q-px-md">
      <q-input
        @keyup.enter="getWeatherBySearch"
        v-model="search"
        placeholder="Search"
        dark
        borderless
      >
        <template v-slot:before>
          <q-icon
            @click="getLocation"
            name="my_location"
          />
        </template>

        <template v-slot:append>
          <q-btn
            @click="getWeatherBySearch"
            round
            dense
            flat
            icon="search"
          />
        </template>
      </q-input>
    </div>
    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{ weatherData.name }}
        </div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text=h4">&deg;C</span>
        </div>
      </div>
      <div class="col text-center">
        <q-avatar size="100px">
          <img :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`" :alt="weatherData.weather[0].description">
        </q-avatar>
      </div>
    </template>

    <template v-else>
      <div v-if="isLoading" class="col fixed-center">
        <q-spinner-puff color="purple" size="5.5em"/>
      </div>
      <div v-else class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">
          Quasar<br>Weather
        </div>
        <q-btn
          @click="getLocation"
          class="col"
          flat
        >
          <q-icon left size="3em" name="my_location" />
          <div>Find my location</div>
        </q-btn>
      </div>
    </template>
    <div class="col bridge"></div>
  </q-page>
</template>

<script>
const OPEN_WEATHER_API_URL = 'http://api.openweathermap.org/data/2.5/weather';

export default {
  name: 'PageIndex',
  data() {
    return {
      search: '',
      weatherData: null,
      isLoading: false,
    }
  },
  computed: {
    bgClass() {
      debugger
      const icon = this.weatherData ? this.weatherData.weather[0].icon : 'd';
      return icon.endsWith('d') ? 'bg-day' : 'bg-night'
    },
  },
  methods: {
    getLocation() {
      navigator.geolocation.getCurrentPosition(position => {
        this.getWeatherByCoords(position.coords)
      })
    },
    async getWeatherByCoords(coords) {
      // const data = this.$q.localStorage.getItem('weatherData')
      // if (!data) {
      this.isLoading = true;
      try {
        const response = await this.$axios.get(`${OPEN_WEATHER_API_URL}?lat=${coords.latitude}&lon=${coords.longitude}&appid=${process.env.OPEN_WEATHER_API_KEY}&units=metric`)
        // this.$q.localStorage.set('weatherData', response.data)
        this.weatherData = response.data
        this.isLoading = false;
      } catch (e) {
        console.log('Ups! Please try again')
        this.isLoading = false;
      }

      // } else {
      //   this.weatherData = data
      // }
    },
    async getWeatherBySearch() {
      this.isLoading = true;
      try {
        const response = await this.$axios.get(`${OPEN_WEATHER_API_URL}?q=${this.search}&appid=${process.env.OPEN_WEATHER_API_KEY}&units=metric`)
        // this.$q.localStorage.set('weatherData', response.data)
        this.weatherData = response.data
        this.search = ''
        this.isLoading = false;
      } catch (e) {
        console.log('Ups! Please try again')
        this.isLoading = false;
      }
    }
  },
}
</script>

<style lang="sass">
  .q-page
    &.bg-day
      background: linear-gradient(to top, #B06AB3, #4568DC)
    &.bg-night
      background: linear-gradient(to top, #42275a, #734b6d)
  .bridge
    flex: 0 0 250px
    background: url(~assets/city-bridge.png) center bottom
    background-size: cover
</style>
