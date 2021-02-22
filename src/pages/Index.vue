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
          <img :src="`${imgUrl}/${weatherData.weather[0].icon}@2x.png`" :alt="weatherData.weather[0].description">
        </q-avatar>
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
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
const OPEN_WEATHER_API_URL = 'https://api.openweathermap.org/data/2.5/weather';
const OPEN_WEATHER_IMG_URL = 'https://openweathermap.org/img/wn';
const GEO_IP_URL = 'https://freegeoip.app/json/'

export default {
  name: 'PageIndex',
  data() {
    return {
      search: '',
      weatherData: null,
      imgUrl: OPEN_WEATHER_IMG_URL
    }
  },
  computed: {
    bgClass() {
      const icon = this.weatherData ? this.weatherData.weather[0].icon : 'd';
      return icon.endsWith('d') ? 'bg-day' : 'bg-night'
    },
  },
  methods: {
    async getLocation() {
      this.$q.loading.show()
      if (this.$q.platform.is.electron) {
        try {
          const myLocation = await this.$axios.get(GEO_IP_URL)
          await this.startLocation({
            lat: myLocation.data.latitude,
            long: myLocation.data.longitude
          })
        } catch (err) {
          this.$q.loading.hide()
          this.$q.dialog({
            title: 'Error',
            message: `${err}. Unable to identify your location.`
          })
        }

      } else {
        let success = async (position) => {
          await this.startLocation({
            lat: position.coords.latitude,
            long: position.coords.longitude
          })
        }
        let error = (err) => {
          this.$q.loading.hide()
          this.$q.dialog({
            title: 'Error',
            message: `${err}. Unable to identify your location.`
          })
        }

        let options = {
          timeout: 30000,
          enableHighAccuracy: true
        }

        // @todo: Could be replaced by 'https://freegeoip.app/json/' if not consistent cross platform
        navigator.geolocation.getCurrentPosition(success, error, options)
      }
    },
    async startLocation(options) {
      try {
        const response = await this.$axios.get(`${OPEN_WEATHER_API_URL}?lat=${options.lat}&lon=${options.long}&appid=${process.env.OPEN_WEATHER_API_KEY}&units=metric`)
        this.weatherData = response.data
        this.$q.loading.hide()
      } catch (err) {
        this.$q.dialog({
          title: 'Error',
          message: `${err}. Unable to identify your location.`
        })
        this.$q.loading.hide()
      }
    },
    async getWeatherBySearch() {
      try {
        this.$q.loading.show()
        const response = await this.$axios.get(`${OPEN_WEATHER_API_URL}?q=${this.search}&appid=${process.env.OPEN_WEATHER_API_KEY}&units=metric`)
        this.weatherData = response.data
        this.search = ''
        this.$q.loading.hide()
      } catch (err) {
        this.$q.dialog({
          title: 'Error',
          message: `${err}. Unable to identify your location.`
        })
        this.search = ''
        this.$q.loading.hide()
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
