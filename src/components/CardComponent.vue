<template>
  <div class="card">
    <h3 class="card__title">{{ city }}</h3>
    <span class="card__text"
      >Температура воздуха {{ wellDegs(temp) }}&nbsp;&#8451;</span
    >
    <span class="card__text"
      >Ощущается, как {{ wellDegs(feels) }}&nbsp;&#8451;</span
    >
    <div class="card__cloudy card__text">
      <img :src="pictSrc" alt="" class="card__pict center" /><span class="center first-letter">{{ clouds }}</span>
    </div>
    <div  class="card__text">
      <span class="center">Ветер {{ wind }}&nbsp;м/с</span>
      <span class="center card__arrow" :style="windDirection">&#8593;</span>
    </div>

    <span  class="card__text">Восход - {{ wellTime(sunrise) }}&nbsp;AM</span>
    <span  class="card__text">Закат - {{ wellTime(sunset) }}&nbsp;PM</span>
  </div>
</template>
<script setup>
import { onMounted, ref, reactive } from "vue";
const props = defineProps(["lat", "lon", "city"]);
const temp = ref(null);
const feels = ref(null);
const wind = ref(null);
const clouds = ref(null);
const sunrise = ref(null);
const sunset = ref(null);
const pictSrc = ref("");
const windDirection = ref("");
const API_KEY = 'c8311f146b5e9f1e12593772274f7a18'
const wellTime = (time) =>
  new Intl.DateTimeFormat("ru", { timeStyle: "medium" }).format(time);
const wellDegs = (deg) => (deg - 273.15).toFixed(1);
const fetchWeatherInTheCity = (lat, lon) => {
  fetch(
    `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&exclude=current&appid=${API_KEY}&lang=ru`
  )
    .then((res) => res.json())
    .then((data) => {
      temp.value = data.main.temp;
      feels.value = data.main.feels_like;
      wind.value = data.wind.speed;
      clouds.value = data.weather[0].description;
      sunrise.value = data.sys.sunrise;
      sunset.value = data.sys.sunset;
      pictSrc.value =
        "http://openweathermap.org/img/w/" + data.weather[0].icon + ".png";
      windDirection.value = "transform: rotate(" + data.wind.deg + "deg)";
    });
};
onMounted(() => fetchWeatherInTheCity(props.lat, props.lon));
</script>
<style lang="scss" scoped>
.first-letter {
    &::first-letter {
        text-transform: uppercase;
    }
}
.center {
    margin: auto 0;
}
.card__cloudy {
    display: flex;
    flex-direction: row;
}
.card__text {
  display: flex;
  font-family: "Roboto";
  font-size: 14px;
  line-height: 20px;
}
.card__title {
  display: flex;
  margin: 0 auto 10px;
  padding: 0;
  font-family: "Roboto Serif";
  width: 100%;
  text-align: center;
  text-transform: uppercase;
  justify-content: center;
}
.card__arrow {
  display: flex;
  width: 10px;
  height: 40px;
  font-size: 30px;
  margin-left: 20px;
}
.card__pict {
  display: flex;
  width: 70px;
  margin-right: 20px;
}
.card {
  display: flex;
  flex-direction: column;
  width: 250px;
  padding: 16px;
  box-sizing: border-box;
  border: solid 1px #ccc;
  border-radius: 6px;
}
* {
    box-sizing: border-box;
}
</style>
