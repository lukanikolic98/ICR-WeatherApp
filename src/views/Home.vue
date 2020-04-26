<template>
    <div>

        <!-- dugmici -->
        <v-layout justify-center>
        <v-btn-toggle v-model="selectedDay"
        tile group
        v-for="day in days" :key="day.getTime()">
            <v-btn v-bind:value="day.getTime()"
                color="black">
                {{day.toString().slice(0, 10)}}
            </v-btn>
        </v-btn-toggle>
        </v-layout>

        <v-data-table
        v-model="selectedGraphData"
        :headers="weatherParams"
        :items="weatherData"
        item-key="id"
        show-select
        class="white">
        </v-data-table>

        <!-- Input -->
        <v-card class="mx-auto grey lighten-4">
        <v-container>
            <v-row justify="center">
                <v-col cols="1">
                    <v-btn 
                    outlined color="black"
                    @click="inputCities = true">
                    Izmeni gradove
                    </v-btn>
                </v-col>
            </v-row>
            <v-divider></v-divider>
            <v-row align="center" justify="center">
                <v-col cols="3">
                    <v-select
                    v-model="selectedGraphParam"
                    :items="graphParams"
                    label="Parametar grafa">
                    </v-select>
                </v-col>
                <v-col cols="1">
                    <v-btn
                    outlined color="black"
                    @click="drawGraph()">
                    Nacrtaj graf
                    </v-btn>
                </v-col>
            </v-row>
        </v-container>
        </v-card>


        <!-- Dijalog za dodavanje gradova -->
        <v-dialog v-model="inputCities"
        scrollable width="30%"
        min-width="200px">
            <v-card>
                <v-card-title 
                class="justify-center">
                    Izmena gradova
                </v-card-title>
                <v-divider></v-divider>
                <v-card-subtitle>
                <v-text-field
                class="centered-input"
                placeholder="Search Locations"
                v-model="searchTerm">
                    Search
                </v-text-field>
                </v-card-subtitle>
                <v-card-text>
                    <v-alert
                    outlined
                    border="left"
                    type="warning">
                    Prikazuje se samo prvih 100 rezultata!
                    </v-alert>
                    <v-list>
                    <v-list-item 
                    v-for="city in filteredCities" 
                    :key="city.id">
                        <v-checkbox 
                        min-width="100px"
                        v-model="city.show"
                        :label="city.name">
                        </v-checkbox>
                        <v-spacer></v-spacer>
                        <v-label>{{city.country}} 
                        </v-label>
                    </v-list-item>
                    </v-list>
                </v-card-text>
                <v-card-actions justify-center>
                    <v-btn @click="inputCities=false">
                        Zatvori
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
        
        <!-- Graf -->       
        <div id="chart" v-if="graphExists">
            <GChart
                :settings="{packages: ['line']}"    
                :data="graphData"
                :options="chartOptions"
                :createChart="(el, google) => new google.charts.Line(el)"
                @ready="onChartReady"
            />
        </div>
    </div>
</template>



<script>
const data = require("../assets/cities.json");
import { GChart } from 'vue-google-charts'

export default {
    components: {
        GChart
    },
    data: () => ({
        weatherData: [],
        selectedGraphData: [],
        graphParams: [
            'Vlaznost',
            'Pritisak',
            'Temperatura',
        ],
        weatherParams: [
        {
            text: 'Ime grada',
            align: 'start',
            sortable: false,
            value: 'name',
        },
        {text: 'Vreme merenja', value: 'time',},
        {text: 'Vlaznost (%)', value: 'humidity'},
        {text: 'Pritisak (mb)', value: 'pressure'},
        {text: 'Min_temp', value: 'min_temp'},
        {text: 'Max_temp', value: 'max_temp'},
        {text: 'Trenutna temperatura', value: 'temperature'},
        {text: 'Vidljivost', value: 'visibility'},
        ],
        supportedCities: [],
        inputCities: false,
        searchTerm: "",
        days: [],
        selectedDay: 0,
        graphExists: false,
        chartsLib: null,
        selectedGraphParam: "Vlaznost",
        graphData: [
            ['Vlaznost']
        ],
        graphParameterEquivalents: {
            Vlaznost: 'humidity', 
            Pritisak: 'pressure', 
            Temperatura: 'temp'
        }
    }),
    created(){
        this.supportedCities = data
        this.supportedCities.forEach(element => {
            element["show"] = false;
        });

        // intialize days
        for(let index = 0; index < 8; index++){
            const newDay = new Date()
            newDay.setDate(newDay.getDate() + index)
            this.days.push(newDay)
        }
    },
    computed: {
        chartOptions () {
            if (!this.chartsLib) return null
            return this.chartsLib.charts.Line.convertOptions({
                chart: {},
                hAxis: {
                    format: 'decimal',
                },
                height: 400,
                axes: {
                    x: {
                        0: {side: 'top'}
                    }
                }
            })
        },
        filteredCities(){
            const result =  this.supportedCities.filter((city) =>{
                return city.name.includes(this.searchTerm)
            })
            return result.length > 100 ? result.slice(0, 100) : result
        },
    },
    methods: {
        async updateWeatherData(cities) {
            let link = "https://api.openweathermap.org/data/2.5/onecall?"
            for(let index = 0; index < cities.length; index++){
                let city = cities[index];
                let cityId = `lat=${city.coord.lat}&lon=${city.coord.lon}`
                let appid = `&appid=f6a1f32e27a46c01b7e71768e9901f77`
                let response = await fetch(link + cityId + appid);
                let data = await response.json();
                console.log(data);
                // formatiraj da uzmes ono sto treba
                this.weatherData.push(this.formatWeatherData(data, city.name, city.id))
            }
        },
        formatWeatherData(rawData, cityName, cityId){
            let {current, daily, hourly} = rawData
            const result = {current, daily, hourly}
            result.humidity = current.humidity
            result.pressure = current.pressure
            result.temperature = (current.temp - 273.15).toFixed(2)
            let time = new Date(current.dt*1000)
            result.time = time.toString().slice(0, 21)
            result.min_temp = (daily[0].temp.min - 273.15).toFixed(2)
            result.max_temp = (daily[0].temp.max - 273.15).toFixed(2)
            result.visibility = current.visibility ? current.visibility : 'N/A'
            result.visibility = current.visibility
            result.name = cityName
            result.id = cityId
            return result
        },
        newCities() {
            const result = []
            this.supportedCities.forEach(element =>{
                if(element.show && (this.weatherData.filter(el =>
                    element.id == el.id
                ).length == 0)){
                    result.push(element)
                }
            })
            return result
        },
        onChartReady (chart, google) {
            this.chartsLib = google
        },
        drawGraph () {
            if(this.selectedGraphData.length == 0) {
                alert("Niste izabrali gradove!")
                return
            }
            this.graphData = [
                ["Vlaznost"]
            ]
            this.graphExists = true
            // formatiraj podatke za crtanje na grafu
            this.selectedGraphData.forEach(el =>{
                this.graphData[0].push(el.name)
                for(let i = 0, j = 1; i < 48; i+=3, j++){
                    if(Array.isArray(this.graphData[j])){
                        if(this.selectedGraphParam == "Temperatura") {
                            let argument = "temp"
                            this.graphData[j].push(Number((el.hourly[i][argument]- 273.15).toFixed(2)))
                        } else{
                            let argument = this.graphParameterEquivalents[this.selectedGraphParam]
                            this.graphData[j].push(el.hourly[i][argument])
                        }

                    }else{
                        let date = new Date(el.hourly[i].dt*1000)
                        let dateString = (date.toDateString()).slice(4, 10)
                        let timeString = (date.toTimeString()).slice(0, 2)
                        this.graphData[j] = [dateString+'. '+timeString+'h']
                        if(this.selectedGraphParam == "Temperatura") {
                            let argument = "temp"
                            this.graphData[j].push(Number((el.hourly[i][argument]- 273.15).toFixed(2)))
                        } else{
                            let argument = this.graphParameterEquivalents[this.selectedGraphParam]
                            this.graphData[j].push(el.hourly[i][argument])
                        }
                    }
                }
                console.log(this.graphData)
            })
        }
    },
    watch: {
        inputCities(value) {
            if(value == false){
                this.updateWeatherData(this.newCities())
            }
        },
        selectedDay(timestamp) {
            //update weatherData for specific day
            for(let value = 0; value < 8; value++){
                if(this.days[value].getTime()==timestamp){

                    this.weatherData.forEach(el =>{
                        if(value == 0){
                            el.humidity = el.current.humidity
                            el.pressure = el.current.pressure
                            el.min_temp = (el.daily[0].temp.min - 273.15).toFixed(2)
                            el.max_temp = (el.daily[0].temp.max - 273.15).toFixed(2)
                            el.visibility = el.current.visibility
                            el.temperature = (el.current.temp - 273.15).toFixed(2)
                        } else{
                            el.humidity = el.daily[value].humidity
                            el.pressure = el.daily[value].pressure
                            el.min_temp = (el.daily[value].temp.min - 273.15).toFixed(2)
                            el.max_temp = (el.daily[value].temp.max - 273.15).toFixed(2)
                            el.visibility = "N/A"
                            el.temperature = "N/A"
                        }
                    })
                }
            }
        },
        selectedGraphParam(value){
            this.graphData[0][0] = value
        }
    }
}
</script>

<style scoped>
</style>