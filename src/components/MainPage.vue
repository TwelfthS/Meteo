<script setup lang="js">
import { ref, onMounted } from 'vue';
import axios from 'axios';

const data = ref([])
const opened = ref({})
const meteo = ref({})
const search = ref('')
onMounted(() => {

})

const onSubmit = async () => {
  console.log(search.value)
  const response = await axios.get('https://geocoding-api.open-meteo.com/v1/search?name=' + search.value + '&count=10&language=en&format=json')
  data.value = response.data.results
}

const open = async (event) => {
  const id = event.target.id
  const info = data.value.find(elem => elem.id == id)
  opened.value = info
  console.log(data.value)
  console.log(opened.value)
  const currentDate = new Date()
  console.log(currentDate.getHours())
  const currentDateFormatted = getDate(currentDate)
  const endDate = new Date()
  endDate.setDate(currentDate.getDate() + 7)
  const endDateFormatted = getDate(endDate)
  console.log(Intl.DateTimeFormat('en-US').resolvedOptions().timeZone)
  const timezone = Intl.DateTimeFormat('en-US').resolvedOptions().timeZone
  console.log('https://api.open-meteo.com/v1/forecast?latitude=' + opened.value.latitude + '&longitude=' + opened.value.longitude + 
        '&hourly=temperature_2m&start_date=' + currentDateFormatted + '&end_date=' + endDateFormatted + '&timezone=' + timezone)
  const response = await axios.get('https://api.open-meteo.com/v1/forecast?latitude=' + opened.value.latitude + '&longitude=' + opened.value.longitude + 
        '&hourly=temperature_2m&start_date=' + currentDateFormatted + '&end_date=' + endDateFormatted + '&timezone=' + timezone + '&current_weather=true')
  meteo.value = response.data
  console.log(meteo.value)
}

const getDate = (date) => {
  const year = date.getFullYear()
  const month = String(date.getMonth() + 1).padStart(2, '0')
  const day = String(date.getDate()).padStart(2, '0')
  const result = `${year}-${month}-${day}`
  console.log(result)
  return result
}

</script>

<template>
  <form @submit.prevent="onSubmit">
    <input id="search" name="search" v-model="search"></input>
    <label for="search">Enter city name</label>
    <button>search</button>
  </form>

  <div v-for="value in data">
    <button :id="value.id" @click="open">{{ value.name + ', ' + value.admin1 + ', ' + value.country }}</button>
  </div>
  <div v-if="meteo.current_weather">
    <h3>Current weather</h3>
    <p>{{ meteo.current_weather.temperature }}</p>
  </div>
  <div v-if="meteo.hourly">
    <h3>Weather forecast</h3>
    <ul v-for="dayIndex in 7">
      <li v-for="hourIndex in 24">
        {{ meteo.hourly.time[(hourIndex - 1) + (dayIndex - 1) * 24] }} - {{ meteo.hourly.temperature_2m[(hourIndex - 1) + (dayIndex - 1) * 24] }}
      </li>
    </ul>
  </div>
</template>
