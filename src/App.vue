<template>
  <div id="app">
    <h1>{{city}}</h1>
    <ul id="app" >
        <div v-for="item in temperature " :key="item.time">
            <li v-if="item.value">
                {{item.value}} °C
                {{(new Date(item.time))}}
            </li> 
        </div>
    </ul>
  </div>
</template>

<script>

  import Metolib from '@fmidev/metolib';

  import {server_url} from '../config/env.var.js'
  import { interval } from '../config/env.var.js'
  import { stored_query_forecast } from '../config/env.var.js'
  //const STORED_QUERY_FORECAST = "fmi::forecast::hirlam::surface::point::multipointcoverage";

  export default {
    name: 'app',
    components: {
    },

    data() {
        return{
          city:'',
          temperature:[],
        }
    },

    methods:{
       forecastTemperature: function(url) {
        var connection = new Metolib.WfsConnection();
        if (connection.connect(url, stored_query_forecast)) {
            connection.getData({
                requestParameter : "temperature",
                begin : new Date(),
                end : new Date((new Date()).getTime() + 60 * 60 * 1000 * 24 *7),
                timestep : 1000 * 60 * 60 * 4,
                sites : ["Helsinki"],
                callback : (data, errors)=> {
                  if(data){
                    this.city=data['locations'][0]['info']['name'];
                    this.temperature=data['locations'][0]['data']['temperature']['timeValuePairs'];
                  }
                  if(errors){
                    console.error(errors);
                  }  
                  connection.disconnect();
                    }
                });
          }
        }    
    },
    created() {
     this.forecastTemperature(server_url) 
    },
    mounted(){
      setInterval(() => {
        this.forecastTemperature(server_url)
        }, interval)
      
    }
    
  }
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
ul {
  list-style-type: none;
}
</style>
