<script setup lang="js">
import { ref } from 'vue';
import axios from 'axios';

const data = ref([])
const opened = ref({})
const meteo = ref({})
const search = ref('')
const errorCity = ref('')
const errorMeteo = ref('')

const onSubmit = async () => {
  if (search.value) {
    data.value = []
    meteo.value = {}
    try {
      const response = await axios.get(`https://geocoding-api.open-meteo.com/v1/search?name=${search.value}&count=10&language=en&format=json`)
      if (response.data.results) {
        data.value = response.data.results
        errorCity.value = ''
      } else {
        errorCity.value = 'City not found'
      }
    } catch (err) {
      console.log(err)
      errorCity.value = 'Error fetching data'
    }
  }
}

const open = async (event) => {
  const id = event.target.id
  const info = data.value.find(elem => elem.id == id)
  opened.value = info
  const currentDate = new Date()
  const currentDateFormatted = getDate(currentDate)
  const endDate = new Date()
  endDate.setDate(currentDate.getDate() + 7)
  const endDateFormatted = getDate(endDate)
  const timezone = Intl.DateTimeFormat('en-US').resolvedOptions().timeZone
  try {
    const response = await axios.get(
      `https://api.open-meteo.com/v1/forecast?latitude=${opened.value.latitude}&longitude=${opened.value.longitude}&hourly=temperature_2m&start_date=${currentDateFormatted}&end_date=${endDateFormatted}&timezone=${timezone}&current_weather=true`)
    meteo.value = response.data
    errorMeteo.value = ''
  } catch (err) {
    console.log(err)
    errorMeteo.value = 'Error fetching data'
  }
}

const getDate = (date) => {
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  const result = `${year}-${month}-${day}`
  return result
}

</script>

<template>
  <form @submit.prevent="onSubmit" class="search-form">
    <input class="search-input" id="search" name="search" v-model="search" placeholder="Enter city name..."></input>

    <button class="search-btn">Search</button>
  </form>

  <p v-if="errorCity" class="error-text">{{ errorCity }}</p>

  <div v-if="data.length > 0" class="locations">
    <button v-for="value in data" class="location-btn" :id="value.id" @click="open">{{ value.name + ', ' + value.admin1 + ', ' + value.country }}</button>
  </div>

  <p v-if="errorMeteo" class="error-text">{{ errorMeteo }}</p>

  <div v-if="meteo.current_weather">
    <h3>Current weather in {{ opened.name + ', ' + opened.admin1 + ', ' + opened.country }}</h3>
    <p>{{ meteo.current_weather.temperature }} C</p>
  </div>
  <div v-if="meteo.hourly" class="forecast">
    <h3>Weather forecast for {{ opened.name + ', ' + opened.admin1 + ', ' + opened.country }}</h3>
    <div class="meteo">
      <ul v-for="dayIndex in 7" class="meteo-day">
        <h4>{{ meteo.hourly.time[(dayIndex - 1) * 24].slice(0, 10) }}</h4>
        <li v-for="hourIndex in 24">
          {{ meteo.hourly.time[(hourIndex - 1) + (dayIndex - 1) * 24].slice(-5) }} &nbsp — &nbsp {{ meteo.hourly.temperature_2m[(hourIndex - 1) + (dayIndex - 1) * 24] }} C
        </li>
      </ul>
    </div>
  </div>
</template>
