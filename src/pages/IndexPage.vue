<template>
  <q-page class="flex column" :class="bgClass">    <!-- so everything on the page is stacked -->
    <div class="col" q-pt-lg q-px-md>   <!-- padding top large and padding left/right medium-->
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        placeholder="search"
        light
        borderless
      > <!-- v-model="text" is the model for the input -->
        <template v-slot:before>    <!-- this is the icon before the input -->
          <q-icon
            @click="getLocation"
            name="my_location" 
            />
        </template>

        <template v-slot:append>
          <q-btn @click="getWeatherBySearch" round dense flat icon="search" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{weatherData.name}}
        </div>
        <div class="text-h6 text-weight-light">
        {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
        <span>{{Math.round(weatherData.main.temp)}}</span>
        <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>

    <div class="col text-center">
      <img :src="`https://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`">
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
          flat>
      <q-icon left size="3em" name="my_location" />
      <div>find my location</div>
    </q-btn>
      </div>
    </template>

    <div class="col skyline">

    </div>
  </q-page>
</template>

<script>

import { defineComponent } from 'vue'

export default defineComponent({
  name: 'IndexPage',
  data(){
    return {
      search: '',
      weatherData: null,
      lat: null,
      long: null,
      apiUrl: 'https://api.openweathermap.org/data/2.5/weather',
      apiKey: process.env.VUE_APP_API_KEY,
      apiKey2: process.env.VUE_APP_API_KEY2
    }
  
  },
  computed:{
    bgClass(){
      if(this.weatherData){
        if(this.weatherData.weather[0].icon.endsWith('n')){
          return 'bg-night'
        }
        else{
          return 'bg-day'
        }
      }
    }
  },
  methods: {
    getLocation(){
      this.$q.loading.show()

      if(this.$q.platform.is.android || this.$q.platform.is.electron){
          this.$axios.get(`https://apiip.net/api/check?accessKey=${this.apiKey2}`).then(response =>{
            console.log(response)
            this.lat = response.data.latitude
            this.long = response.data.longitude
            this.getWeatherByCoords()
          })
      }else{
        navigator.geolocation.getCurrentPosition(position=>{
        console.log('position', position)
        this.lat = position.coords.latitude
        this.long = position.coords.longitude
        this.getWeatherByCoords()
      })
      }
    },
    getWeatherByCoords(){
      this.$q.loading.show()
      this.$axios(`${this.apiUrl}?lat=${this.lat}&lon=${this.long}&appid=${this.apiKey}&units=metric`).then(response =>{
        this.weatherData = response.data
        this.$q.loading.hide()
      })
    },
    getWeatherBySearch(){
      this.$q.loading.show()
      this.$axios(`${this.apiUrl}?q=${this.search}&appid=${this.apiKey}&units=metric`).then(response =>{
        this.weatherData = response.data
        this.$q.loading.hide()
      })
    }
  }
})
</script>

<style lang="sass">
.q-page
  background: linear-gradient(to bottom, #f0f2f0, #000c40) !important
  &.bg-night
    background: linear-gradient(to bottom, #203A43, #000c40) !important
  &.bg-day
    background: linear-gradient(to bottom, #74ebd5, #000c40) !important
.degree
  top: -44px 

.skyline
  flex: 0 0 100px
  background: url(../assets/skyline.png)
  background-size: contain
  background-position: center bottom
</style>