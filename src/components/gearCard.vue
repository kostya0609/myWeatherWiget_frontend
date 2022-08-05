<template>
    <el-card v-if="!loading" class="mww-gearCard-container">

        <div class="mww-gearCard-header">
            <label class="mww-gearCard-header-label">Настройки</label>
            <el-button
                text    
                @click="context.emit('gearButton')"
            >
                <el-icon :size="24"><CloseBold /></el-icon>
            </el-button>
        </div>

<!--        <gearCity-->
<!--            v-if="myCitiesList.length > 0"-->
<!--            v-model:myCitiesList="myCitiesList"-->
<!--            @deleteCity="deleteCity"-->
<!--        />-->
      <draggable
          v-if="myCitiesList.length > 0"
          :list="myCitiesList"
          :disabled="!dragEnabled"
          item-key="name"
          ghost-class="ghost"
          @start="dragging = true"
          @end="dragging = false"
      >
<!--     :move="checkMove"-->
        <template #item="{ element }">
            <gearCity
              :city="element"
              @enableDrag="dragEnabled = true"
              @disableDrag="dragEnabled = false"
              @deleteCity="deleteCity"
            >
            </gearCity>
        </template>
      </draggable>

        <div v-else>
            <label class="mww-gearCard-cityInfo-label-empty">добавьте город</label>
         </div>

         <div class="mww-gearCard-newCity">
            <label class="mww-gearCard-newCity-add-label">Добавить город</label>
            <span class="mww-gearCard-newCity-add">
                <el-input
                    v-model="cityName"
                    placeholder="Введите название"
                    @change="addCity"
                />
                
                <el-button
                    class="mww-gearCard-newCity-add-button"
                    text  
                    @click="addCity"  
                >
                    <el-icon :size="20"><Plus /></el-icon>
                </el-button>
            </span>
         </div>

    </el-card>
</template>

<script>

import {ref, inject, reactive, computed} from 'vue';
import {ElMessageBox }  from 'element-plus';
import draggable from 'vuedraggable'
import gearCity from '../components/gearCity'
let id = 1;
    export default {
        name  : "gearCard",
        components : {draggable, gearCity},
        emits : ['gearButton'],
        setup(props, context){
            
            const getWeather     = inject('getWeather');
            const loading        = inject('loading');
            const apiKey         = inject('apiKey');
            const myCitiesList   = inject('myCitiesList');   
            const notify         = inject('notify');

            const cityName  = ref('');
            const myStorage = window.localStorage;

            let dragging = ref(false);
            let dragEnabled = ref(false);

            // полезно при проверке и надо выше вернуть в компонент   :move="checkMove"
            // function checkMove(e) {
            //     window.console.log("Future index: " + e.draggedContext.futureIndex);
            // }

            function findCity(name){ 
                return myCitiesList.find(el => name == el.name ? true : false); 
            };

            async function addCity(){
                loading.value = true;
                let result = await getWeather('http://api.openweathermap.org/geo/1.0/direct?limit=1', cityName.value,'', '', '', '', apiKey); 
                
                if (Array.isArray(result) && result.length > 0) {
                    if (!findCity(result[0].name)){
                        let newId = myCitiesList.length;
                        myCitiesList.push({id : newId, country : result[0].country ? result[0].country : '', name : result[0].name, localName : result[0].local_names ? result[0].local_names.ru : '', coords : {lat : result[0].lat, lon : result[0].lon}})
                        cityName.value = '';
                        updateMyStorage();
                        notify('Добавление города', 'Успешно.', 'success');
                    } else  notify('Ошибка добавления города', 'Указанный город уже есть в списке', 'error');   
                    
                } else {
                    notify('Ошибка добавления города', 'Возможно ошибка в названии города, или для указанного города нет информации о погоде', 'error');
                }
              
                loading.value = false;
            };
            
            function deleteCity(city){
                ElMessageBox.confirm(`Вы уверены, что хотите удалить город ${city.localName ? city.localName : city.name}?`)
                .then(async () => {
                    let idx = await myCitiesList.findIndex(el => el.name == city.name);
                    await myCitiesList.splice(idx,1);
                    await myCitiesList.forEach((el, idx) => {el.id = idx})
                    updateMyStorage();
                })
                .catch(() => {});
            };
            
            function updateMyStorage(){
                myStorage.removeItem('mww_citiesList');
                myStorage.setItem('mww_citiesList', JSON.stringify(myCitiesList));
            };
            return { 
                context, myCitiesList, loading, cityName,
                addCity, deleteCity,
                dragging, dragEnabled,
                //checkMove
            }
        }
    }
 </script>   
 <style>
 </style> 