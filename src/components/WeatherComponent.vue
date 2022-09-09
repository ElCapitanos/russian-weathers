<template>
  <div class="weather">
    <img
      src="@/assets/img/settings.png"
      alt="Настроить"
      class="weather__setts"
      @click="togglePopup()"
    />
    <CardComponent
      v-for="(city, index) in currentLocalStorage.value"
      :key="index"
      :lat="city[0]"
      :lon="city[1]"
      :city="city[2]"
      class="center"
    />
    <div class="overlay" v-if="popupFlag" @click="togglePopup()"></div>
    <div class="weather__popup" v-if="popupFlag">
      <VueDraggableNext id="dnd">
        <div
          v-for="(city, index) in currentLocalStorage.value"
          :key="index"
          class="weather__popup-item"
          :id="city[2]"
          ref="cityInList"
        >
          <span
            class="weather__popup-cross"
            title="Удалить город из списка"
            @click="delCity(index)"
            >&times;</span
          >
          <span
            class="weather__popup-burger"
            title="Перемещение города по списку"
          >
            &#9776;</span
          >
          {{ city[2] }}
        </div>
      </VueDraggableNext>

      <p>***</p>
      <div class="weather__popup-add">
        <input
          ref="choiceCity"
          type="text"
          placeholder="Добавить город"
          class="weather__popup-input"
          @focus="toggleCitiesFlag"
          @input="citiesFilter()"
        />
        <button class="weather__popup-btn" @click="addNewCity()">&#43;</button>
      </div>
      <div class="weather__popup-cities" v-if="!citiesFlag">
        <span
          v-for="(item, index) in allCitiesArrFiltered.value"
          :key="index"
          @click="choiceTheCity(item)"
          class="weather__popup-city"
        >
          {{ item }}
        </span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, reactive, ref } from "vue";
import CardComponent from "@/components/CardComponent.vue";
import { useCitiesStore } from "@/stores/index.js";
import { VueDraggableNext } from "vue-draggable-next";

const citiesStore = reactive(useCitiesStore());
let choiceCity = ref(null);
let currentLocalStorage = reactive([]);
let popupFlag = ref(false);
let citiesFlag = ref(true);
let citiesArr = reactive([]);
let allCitiesArr = reactive([]);
let cityInList = ref(null);
const changeIndexInArr = () => {
  let arr = document.querySelector("#dnd");
  let items = arr.querySelectorAll(".weather__popup-item");
  let newArr = [];
  let sortedArr = [];
  items.forEach((item) => newArr.push(item.id));
  for (let i in newArr) {
    currentLocalStorage.value.forEach((sortedItem) => {
      sortedItem[2] == newArr[i] ? sortedArr.push(sortedItem) : null;
    });
  }

  localStorage.setItem("cities", JSON.stringify(sortedArr));
  getLocalStorage();
};
const addNewCity = () => {
  if (!choiceCity.value.value) {
    alert("Вбейте название города (кирилицей)");
  } else {
    choiceCity.value.value =
      choiceCity.value.value[0].toUpperCase() +
      choiceCity.value.value.slice(1).toLowerCase();
    findTheCity(choiceCity.value.value);
  }
  choiceCity.value.value = "";
};
let allCitiesArrFiltered = reactive([]);
const citiesFilter = () => {
  allCitiesArrFiltered.value = allCitiesArr;
  allCitiesArrFiltered.value = allCitiesArr.filter((item) =>
    item.toLowerCase().includes(choiceCity.value.value.toLowerCase())
  );
};
const toggleCitiesFlag = () => {
  citiesFlag.value = !citiesFlag.value;
};
const choiceTheCity = (item) => {
  choiceCity.value.value = item;
  toggleCitiesFlag();
};
const togglePopup = () => {
  popupFlag.value ? changeIndexInArr() : null
  popupFlag.value = !popupFlag.value;
};
const addCity = (cityInfo) => {
  let isUnique = true;
  for (let item of citiesArr) {
    item[2] == cityInfo[2] ? (isUnique = false) : null;
  }
  isUnique ? citiesArr.push(cityInfo) : alert("Такой город уже есть в списке");
  localStorage.setItem("cities", JSON.stringify(citiesArr));
  getLocalStorage();
};
const getLocalStorage = () => {
  currentLocalStorage.value = JSON.parse(localStorage.getItem("cities"));
};
const getLocation = () => {
  fetch('https://ipwho.is/&lang=ru')
    .then((res) => res.json())
    .then((data) => {
      findTheCity(data?.city);
      getLocalStorage();
    });
};
const findTheCity = (city) => {
  // находим координаты города
  let isCity = false;
  for (let i in citiesStore) {
    if (city.toLowerCase() == citiesStore[i].name.toLowerCase()) {
      addCity([citiesStore[i].coords.lat, citiesStore[i].coords.lon, city]);
      isCity = true;
      popupFlag.value = false;
    }
  }
  !isCity
    ? alert(
        "Город не найден. Используйте правильное написание кириличесскими символами"
      )
    : null;
};
const getCities = () => {
  for (let i in citiesStore) {
    allCitiesArr.push(citiesStore[i].name);
  }
  allCitiesArr.length = 1132;
  allCitiesArrFiltered.value = allCitiesArr;
};
const delCity = (index) => {
  citiesArr.splice(index, 1);
  localStorage.setItem("cities", JSON.stringify(citiesArr));
  getLocalStorage();
};
onMounted(() => {
  getCities(), getLocation();
});
</script>
<style scoped lang="scss">
.weather__popup-burger {
  display: flex;
  position: absolute;
  font-size: 20px;
  left: 10px;
  top: 50%;
  transform: translateY(-50%);
  cursor: move;
}
.weather__popup-cross {
  font-size: 20px;
  position: absolute;
  display: flex;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  transition: 0.4s linear;
  cursor: pointer;
  &:hover {
    color: red;
  }
}
.weather__popup-add {
  display: flex;
  flex-direction: row;
  width: 100%;
  justify-content: space-between;
  height: 40px;
}
.weather__popup-btn {
  display: flex;
  cursor: pointer;
  justify-content: center;
  width: 19%;
  height: 40px;
  line-height: 36px;
  font-family: "Roboto Serif";
  font-size: 40px;
  text-transform: uppercase;
}
.weather__popup-city {
  font-size: 12px;
  line-height: 18px;
  cursor: pointer;
  &:hover {
    background: #ccc;
  }
}
.weather__popup-cities {
  display: flex;
  flex-direction: column;
  max-width: 400px;
  overflow-y: auto;
  border: solid 1px #ccc;
}
.weather__popup-input {
  display: flex;
  padding-left: 10px;
  outline: none;
  line-height: 22px;
  width: 79%;
  height: 40px;
}
.weather__popup-item {
  position: relative;
  font-size: 12px;
  width: 90%;
  display: flex;
  background: #ebebeb;
  border: solid 1px #ccc;
  line-height: 30px;
  border-radius: 4px;
  margin: 0 auto 10px;
  justify-content: center;
  text-align: center;
}
.weather__popup {
  display: flex;
  flex-direction: column;
  background: #fff;
  border: solid 1px #ccc;
  border-radius: 6px;
  width: 310px;
  max-height: 400px;
  padding: 30px 10px 10px;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 20;
}
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.6);
  z-index: 10;
}
.weather__setts {
  width: 32px;
  display: block;
  transition: 0.6s linear;
  margin: 0 10px 0 auto;
  cursor: pointer;
  &:hover {
    transform: rotate(180deg);
  }
}
.center {
  margin: 20px auto 0;
}
.weather {
  display: flex;
  flex-direction: column;
  width: 300px;
  margin: 50px auto;
  border: solid 1px #ccc;
  border-radius: 6px;
  padding: 20px 0;
}
* {
  box-sizing: border-box;
  font-family: "Roboto";
}
</style>
