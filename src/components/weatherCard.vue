<template>
<el-card v-if="!loading" class="mww-weatherCard-container">

    <div class="mww-weatherCard-header">
        
        <label class="mww-weatherCard-header-label">
            <el-icon :size="20"><LocationFilled /></el-icon>
            {{weatherInfo.cityName}}
        </label>
        <el-button 
            text
            @click="context.emit('gearButton')"
        >
            <el-icon :size="24"><Tools /></el-icon>
        </el-button>
    </div>

    <div  class="mww-weatherCard-icon">
        <el-image  :src="weatherInfo.icon"/>
        <label class="mww-weatherCard-label-degree">
            {{weatherInfo.temp}}<sup>o</sup> C
        </label>
    </div>

    <div>
        <label class="mww-weatherCard-description-label">
            Ощущается как - {{weatherInfo.feels_like}}<sup>o</sup> C, {{weatherInfo.description}}.
        </label>   
    </div>

    <div class="mww-weatherCard-addInfo">
        <label class="mww-weatherCard-addInfo-label">
            <el-icon :size="20" v-if="windDirection == 'N'"><Bottom/></el-icon>
            <el-icon :size="20" v-if="windDirection == 'NNE' || windDirection == 'NE' || windDirection == 'ENE'"><BottomLeft/></el-icon>

            <el-icon :size="20" v-if="windDirection == 'E'"><Back/></el-icon>
            <el-icon :size="20" v-if="windDirection == 'ESE' || windDirection == 'SE' || windDirection == 'SSE'"><TopLeft/></el-icon>

            <el-icon :size="20" v-if="windDirection == 'S'"><Top/></el-icon>
            <el-icon :size="20" v-if="windDirection == 'SSW' || windDirection == 'SW' || windDirection == 'WSW'"><TopRight/></el-icon>
            
            <el-icon :size="20" v-if="windDirection == 'W'"><Right/></el-icon>
            <el-icon :size="20" v-if="windDirection == 'WNW' || windDirection == 'NW' || windDirection == 'NNW'"><BottomRight/></el-icon>

            &nbsp;{{weatherInfo.wind_speed}}м/с {{weatherInfo.wind_speed ? ', ' : '' }} {{windDirection}}
        </label>

        <label class="mww-weatherCard-addInfo-label">
           <el-icon :size="20"><Stopwatch /></el-icon>&nbsp;{{weatherInfo.pressure}}hPA
        </label>
    </div>

    <div class="mww-weatherCard-addInfo">
        <label class="mww-weatherCard-addInfo-label">
            Влажность: {{weatherInfo.humidity}} %
        </label>

        <label class="mww-weatherCard-addInfo-label">
           Точка росы: н/д
        </label>
    </div>

    <div class="mww-weatherCard-addInfo">
        <label class="mww-weatherCard-addInfo-label">
            Видимость: {{weatherInfo.visibility}}км
        </label>
    </div>
    
</el-card>
</template>
<script>
import {reactive, inject, computed} from 'vue'
 export default {
    name  : "weatherCard",
    props : ['coords'],
    emits : ['gearButton'],
    setup(props, context){

        const getWeather     = inject('getWeather');
        const loading        = inject('loading');
        const countryOptions = inject('countryOptions');
        const apiKey         = inject('apiKey');

        const cityCoords     = props.coords;
        const weatherInfo    = reactive({});

        async function getData(){
            loading.value = true;
            let result =  await getWeather('https://api.openweathermap.org/data/2.5/weather?', '', cityCoords.lat, cityCoords.lon, countryOptions.lang, countryOptions.units, apiKey);
            //Ниже сообираем объект с нужными данными
            weatherInfo.cityName    = `${result.name}, ${result.sys.country}`;
            weatherInfo.temp        = Math.floor(result.main.temp);
            weatherInfo.feels_like  = Math.floor(result.main.feels_like);
            weatherInfo.humidity    = result.main.humidity;
            weatherInfo.pressure    = result.main.pressure;
            weatherInfo.visibility  = result.visibility;
            weatherInfo.description = result.weather[0].description;
            weatherInfo.icon        = `http://openweathermap.org/img/wn/${result.weather[0].icon}@2x.png`;
            weatherInfo.wind_speed  = Math.floor(result.wind.speed);
            weatherInfo.wind_gust   = Math.floor(result.wind.gust);
            weatherInfo.wind_deg    = result.wind.deg;
            loading.value = false;
        };
        getData();

        const windDirection = computed(() => {
            let deg = weatherInfo.wind_deg;
            let directions = [
                {min:0,   max:21, dir:'N'},
                {min:21,  max:44, dir:'NNE'},
                {min:44,  max:66, dir:'NE'},
                {min:66,  max:89, dir:'ENE'},
                {min:89,  max:111, dir:'E'},
                {min:111, max:134, dir:'ESE'},
                {min:134, max:156, dir:'SE'},
                {min:156, max:179, dir:'SSE'},
                {min:179, max:201, dir:'S'},
                {min:201, max:224, dir:'SSW'},
                {min:224, max:246, dir:'SW'},
                {min:246, max:269, dir:'WSW'},
                {min:269, max:291, dir:'W'},
                {min:291, max:314, dir:'WNW'},
                {min:314, max:336, dir:'NW'},
                {min:336, max:359, dir:'NNW'},
            ]
            return weatherInfo.wind_deg ? directions.find(el => {if (deg > el.min && deg <= el.max) return true }).dir : '';

        })

        return {
            context, weatherInfo, loading,
            windDirection, 
        }
    }
}
</script>
<style>
</style>