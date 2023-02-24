<template>
  <div
    id="app"
    v-loading = "loading"
    element-loading-text="Загрузка данных..."
    :element-loading-spinner="svg"
    element-loading-svg-view-box="-10, -10, 50, 50"
  >
    <gearCard
        v-if="showGearCard"
        @gearButton="showGearCard = !showGearCard"
    />
    <weatherCard
      v-else
      v-for="(city, idx) in myCitiesList"
      :coords="city.coords"
      :key="idx"
      @gearButton="showGearCard = !showGearCard"
    />
  </div>
</template>

<script>

import {ElNotification}  from 'element-plus';
import {reactive, ref, provide, watch} from 'vue'
import weatherCard from "./components/weatherCard.vue"
import gearCard from "./components/gearCard.vue"

export default {
    name: 'App',
    components : { weatherCard, gearCard},
     setup() {
        const loading        = ref(false);
        const svg            = `
        <path class="path" d="
          M 30 15
          L 28 17
          M 25.61 25.61
          A 15 15, 0, 0, 1, 15 30
          A 15 15, 0, 1, 1, 27.99 7.5
          L 15 15
        " style="stroke-width: 4px; fill: rgba(0, 0, 0, 0)"/>
      `;
        const apiKey         = 'f91bbe3c21ae7202db1054f2f222735a';
        const countryOptions = {
          units : 'metric',
          lang  : 'ru',
        };
        const notify         = (title, message, type) => {
          ElNotification({
            title    : title,
            message  : message,
            type     : type,
            duration : 4000,
          })
        };
        const myStorage      = window.localStorage;
        const myCitiesList   = reactive([]);
        const showGearCard   = ref(false);
            
        function initApp(){
          loading.value = true;
          let citiesList = [];
          myStorage.getItem('mww_citiesList') ? citiesList = JSON.parse(myStorage.getItem('mww_citiesList')) : '';

          if(citiesList.length > 0) {
            myCitiesList.length = 0;
            citiesList.forEach(el => {myCitiesList.push(el)});
          } else {
            let currentCity = new Promise((resolve, reject) => {
              function success(position) {
                let coords = {lat : null, lon : null};
                coords.lat = position.coords.latitude;
                coords.lon = position.coords.longitude;
                resolve(coords);  
              };  
              navigator.geolocation.getCurrentPosition(success);
            });
            currentCity.then(async coords => {
              
              let result = await getWeather(
                  'http://api.openweathermap.org/geo/1.0/reverse?limit=1',
                  '',
                  coords.lat,
                  coords.lon,
                  '',
                  '',
                  apiKey
              );
              if (Array.isArray(result) && result.length > 0)
                myCitiesList.push({
                  id        : 0,
                  country   : result[0].country,
                  name      : result[0].name,
                  localName : result[0].local_names.ru,
                  coords
                });
              else
                myCitiesList.push({
                  id        : 0,
                  country   : 'нет данных',
                  name      : 'нет данных' ,
                  localName : 'нет данных',
                  coords
                });
            })
          }
          loading.value = false;
        };
        initApp();

        async function getWeather(url, q, lat, lon, lang, units, apiKey) {
          return await fetch(`${url}&q=${q}&lat=${lat}&lon=${lon}&lang=${lang}&units=${units}&appid=${apiKey}`)
          .then(response => response.json())
          .catch(error => {return {status : 'error',message : 'Некорректный ответ сервера',system : error.message} });
        };

        provide('countryOptions', countryOptions);
        provide('apiKey', apiKey);
        provide('getWeather', getWeather);
        provide('loading', loading);
        provide('myCitiesList', myCitiesList);
        provide('notify', notify);

        watch(showGearCard, () => {
          if(!showGearCard.value) myCitiesList.length == 0 ? initApp() : ''
        })

        return {loading, svg, myCitiesList, showGearCard}
    },
}
</script>

<style>

#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
}
.mww-weatherCard-container{
  padding: 15px;
  margin-bottom: 5px;
  background: #e9e9e9;
  width: 20%;
}
.mww-weatherCard-header{
  display: flex;
  justify-content: space-between;
}
.mww-weatherCard-header-label{
  display: flex;
  font-weight: bold;
  align-items: center;
}
.mww-weatherCard-icon {
  display: flex;
  align-items: center;
  justify-content: center;
}
.mww-weatherCard-label-degree{
  font-weight: bold;
  font-size: 30px;
}
.mww-weatherCard-description-label{
  text-align: left;
}
.mww-weatherCard-addInfo{
  display: flex;
  justify-content:space-between;
  margin-top: 10px;
}
.mww-weatherCard-addInfo-label{
  display: flex;
  align-items: center;
  font-size: 14px;
  font-weight:600;
}
.mww-gearCard-container{
  width: 20%;
}
.mww-gearCard-header{
  display: flex;
  justify-content: space-between;
  margin-bottom: 15px;
}
.mww-gearCard-header-label{
  display: flex;
  font-weight: bold;
  align-items: center;
}
.mww-gearCard-cityInfo{
  display: flex;
  justify-content: space-between;
  margin-bottom: 15px;
  background-color: #ebe9e8;
}
.mww-gearCard-cityInfo-label{
  display: inline-flex;
  font-weight: bold;
  align-items: center;
}
.mww-gearCard-cityInfo-label-empty{
  font-size: 14px;
  align-items: center;
}
.mww-gearCard-newCity{
  margin-top : 40px;
}
.mww-gearCard-newCity-add{
  display: flex;
  justify-content: space-between;
  margin-top: 5px;
}
.mww-gearCard-newCity-add-label{
  font-size: 14px;
  font-weight: 600;
}
.mww-gearCard-newCity-add-button{
  margin-left: 5px;
}
</style>
