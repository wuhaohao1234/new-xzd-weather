<script setup>
import { onMounted, ref } from "vue";
import WeatherContent from "./components/WeatherContent.vue";
import WeatherHeader from "./components/WeatherHeader.vue";
const key = '6806a666fdfcd4bb5cac88647eb9dc34'
const adcode = ref('')
const cityInfo = ref(null);
const newCityInfo = ref(null)
const casts = ref([])
const cityList = ref([])
fetch(`https://restapi.amap.com/v3/ip?key=${key}`)
  .then((res) => res.json())
  .then((data) => {
    fetch(
      `https://restapi.amap.com/v3/weather/weatherInfo?key=${key}&city=${data.adcode}`
    )
      .then((res) => res.json())
      .then((result) => {
        adcode.value = data.adcode
        cityInfo.value = {
          city: result.lives[0].city,
          weather: result.lives[0].weather,
          temperature: result.lives[0].temperature,
          winddirection: result.lives[0].winddirection,
          windpower: result.lives[0].windpower
        }
        fetch(`https://restapi.amap.com/v3/weather/weatherInfo?key=${key}&city=${adcode.value}&extensions=all`).then((res) => res.json()).then((result) => {
          casts.value = result.forecasts[0].casts
          console.log(casts.value);
        })
      });
  });
onMounted(() => {
  cityList.value = JSON.parse(localStorage.getItem('cityList')) || []
  console.log(cityList.value);
})
const handleSave = () => {
  console.log(newCityInfo.value);
  localStorage.setItem('cityList', JSON.stringify([...cityList.value, newCityInfo.value]));
}
const changeNewCityInfo = (info) => {
  newCityInfo.value = info
}
</script>

<template>
  <div class="bg-weather-primary">
    <WeatherHeader :cityInfo="cityInfo" @handleSave="handleSave" />
    <WeatherContent :casts="casts" @changeNewCityInfo="changeNewCityInfo" />
  </div>
</template>

<style scoped>
.bg-weather-primary {
  background-color: rgb(0 102 138);
  min-height: 100vh;
}
</style>
