<template>
  <div id="app">
    <div class="container">
      <h1>Vue Weather App</h1>
      <inputautocomplete :cityGeoApiName="filteredList" @input="handleInput" @getStartGeoApi="getGeoApi" @getRequestDataInput="getCurrentApi"/>
      <div class="apiResponse">
        <span v-if="weatherInfo == Array[0]">Select city</span>
        <div v-else>
          <span>Current Temperatura:</span>
        
          <span>{{ weatherInfo.main.temp }} °</span>
          <img :src="icon">
          <span>Weather conditions: {{ weatherInfo.weather[0].main }}</span>
          <span>Feels Like (temp) {{ weatherInfo.main.feels_like }} °</span>
          <span>Winds speed: {{ weatherInfo.wind.speed }} m/sec</span>
        </div>
        <canvas id="myChart"></canvas>
        
      </div>
    </div>
  </div>
  
</template>

<script>
import inputautocomplete from './components/input-autocomplete.vue';
import Chart from 'chart.js/auto'

export default {
  name: 'App',
  components: {
      inputautocomplete
  },
  computed: {
    filteredList() {
      if (this.filterInput) {      
          return this.cityGeoApiName.filter(e => e.cityNameOriginal.toLowerCase().indexOf(this.filterInput.toLowerCase()) !== -1)
      } else {      
          return this.cityGeoApiName
      }
    }
  },
  data(){
    return {
      cityGeoApiName: [],
      inputValue: '',
      weatherInfo: null,
      icon: null,
      diagramInfo: null,
      myChart: null,
      filterInput: null,
      //relevantCity: []
    }
  },
  methods: {
    handleInput(e) {
        this.filterInput = e
    },
    getGeoApi(city){
      if (city.currentInputValue === '') return; //fix emtry bad request

      const baseApiUrl = 'http://api.openweathermap.org/geo/1.0/direct?'
      const APIkey = '9721642d6b12fa6e9bc14e2b8d4b88c8'
      let currentCityName = city.currentInputValue
      const limitListCity = '15'
      let finalApiUrl = baseApiUrl + 'q=' + currentCityName + '&limit=' + limitListCity + '&appid=' + APIkey
      console.log('urlrequest: ' + finalApiUrl)
      const myRequest = new Request(finalApiUrl);
      
      let requestGeoApi = fetch(myRequest)
        .then((response) => response.json())
        .then((data) => {
          let citylist = []
          for (let citycard of data) {
            citylist.push({
              cityNameOriginal: citycard.name,
              countryName: citycard.country ,
              lat: citycard.lat,
              lon: citycard.lon
            })
          }
          return Promise.all(citylist)
        })
        .then(function (citylist){
          return citylist
        })
        .catch(console.error);

        
        const cityNames = () => {
          requestGeoApi.then((a) =>{
            this.cityGeoApiName = a
          }
          )
        }
        cityNames()
      },
      getCurrentApi(data){
        const baseUrl = 'https://api.openweathermap.org/data/2.5/weather?'
        const APIkey = '9721642d6b12fa6e9bc14e2b8d4b88c8'
        let finalUrl = baseUrl + 'lat=' + data.currentLat + '&lon=' + data.currentLon + '&appid=' + APIkey + '&units=metric'
        //example: https://api.openweathermap.org/data/2.5/weather?lat=44.34&lon=10.99&appid={API key}

        const myRequest = new Request(finalUrl);

        let requestCurrentApi = fetch(myRequest)
          .then((response) => response.json())
          .catch(console.error);

        const cityWeatherInfo = () => {
          requestCurrentApi.then((a) =>{
            this.weatherInfo = a
            this.icon = 'http://openweathermap.org/img/wn/' + this.weatherInfo.weather[0].icon + '@2x.png' 
          }
          )
        }
        cityWeatherInfo()

        this.getHourlyApi(data)
      },
      getHourlyApi(data){
        const baseUrl = 'https://api.openweathermap.org/data/3.0/onecall?'
        const exclude = 'current,minutely,daily,alerts'
        const APIkey = '9721642d6b12fa6e9bc14e2b8d4b88c8'
        let finalUrl = baseUrl + 'lat=' + data.currentLat + '&lon=' + data.currentLon + '&exclude=' + exclude + '&appid=' + APIkey + '&units=metric'
        //example: https://api.openweathermap.org/data/3.0/onecall?lat={lat}&lon={lon}&exclude={part}&appid={API key}
        
        const myRequest = new Request(finalUrl);

        let requestHourApi = fetch(myRequest)
          .then((response) => response.json())
          .catch(console.error);

        const getDiagramInfo = () => {
            requestHourApi.then((a) =>{
              this.diagramInfo = a.hourly
              this.createDiagram(this.diagramInfo)
            }
          )
        }
        getDiagramInfo()
      },
      createDiagram(data){
        let point = []
        let pointFeelsLike = []


        for (let elArr of data){
            point.push(elArr.temp)
            pointFeelsLike.push(elArr.feels_like)
        }
        
        if(this.myChart !== null){
          this.myChart.destroy();
        }
        
        const ctx = document.getElementById('myChart');

        this.myChart = new Chart(ctx, {
          type: 'line',
          data: {
            labels: ['Current', '+1hour', '+2hour', '+3hour', '+4', '+5', '+6', '+7', '+8', '+9', '+10', '+11', '+12'],
            datasets: [{
              label: 'Current temp',
              data: point,
              //borderWidth: 1
            },
            {
              label: 'Feels like temp',
              data: pointFeelsLike,
              //borderWidth: 1
            }]

          },
          options: {
            scales: {
              y: {
                beginAtZero: true
              }
            }
          }
        });
      },
      /*
      caseInsensitiveSearch(what ='', where =''){
        return where.toLowerCase().search(what.toLowerCase())
      },
      addAutocomplete(selector, data){
        console.log('welcome')
        this.getGeoApi('dnipro')
        let input = document.querySelector(selector); //new
        console.log('input ' + input)
        let listCheck = document.getElementsByClassName('autocompleteList')
        if (listCheck.length > 1){
          console.log('listCheck: ')
          console.log(listCheck)
          listCheck[1].parentNode.removeChild(listCheck[1]);
        }
          input.classList.add('autocompleteInput')
          let wrap = document.createElement('div');
          wrap.className = 'autocompleteWrap'
          input.parentNode.insertBefore(wrap, input) //for parrent input add wrap element before input
          wrap.appendChild(input); //add wrap

          let list = document.createElement('div')
          console.log('list 1: ' + list)
          list.className = 'autocompleteList'
          wrap.appendChild(list)

          console.log(data)
          input.addEventListener('input', () =>{

            list.innerHTML = '' //clear list under before request
            console.log('list 2: ' + list)
            data.forEach((dataItem, index) => {
              let search = this.caseInsensitiveSearch(this.inputValue, dataItem)
              if(search === -1) return false;
              this.relevantCity.push(index);
              
              let item = document.createElement('div')
              item.className = 'autocompleteItem'
              item.innerText = dataItem
              list.appendChild(item)
            })
        })

          console.log('relevantCity: ')
          console.log(this.relevantCity)

  //      })
      },
    */
    },
  created(){
    /*PLACE FOR CURRENT API USER*/

    /*//
    const baseApiUrl = 'http://api.openweathermap.org/geo/1.0/direct?'
    const APIkey = '9721642d6b12fa6e9bc14e2b8d4b88c8'
    let currentCityName = 'Lon'
    const limitListCity = '15'
    let finalApiUrl = baseApiUrl + 'q=' + currentCityName + '&limit=' + limitListCity + '&appid=' + APIkey
    console.log('urlrequest: ' + finalApiUrl)
    const myRequest = new Request(finalApiUrl);
    

    
    let requestGeoApi = fetch(myRequest)
      .then((response) => response.json())
      .then((data) => {
        console.log(data)
        console.log('typeof data' + typeof data)
        let citylist = []
        for (let citycard of data) {
          citylist.push(citycard.name)
        }
        return Promise.all(citylist)
      })
      .then(function (citylist){
        console.log('citylist 3 then:')
        console.log(citylist)
        return citylist
      })
      .catch(console.error);

      
      const cityNames = () => {
        requestGeoApi.then((a) =>{
          console.log('a: ')
          console.log(a)
          this.cityGeoApiName = a
          //return a
        }
        )
      }

      //this.cityGeoApiName == cityNames()
      console.log('BOOM this.cityGeoApiName' + this.cityGeoApiName)
      console.log(this.cityGeoApiName)
      console.log(typeof this.cityGeoApiName)
      console.log('BOOM cityNames()' + cityNames())
      console.log(cityNames())
      console.log(typeof cityNames())

   //   cityNames()
/*
      this.cityGeoApiName = requestGeoApi;

      console.log('this.cityGeoApiName: ');
      console.log(this.cityGeoApiName);
      console.log('requestGeoApi: ');
      console.log(requestGeoApi);
*/      
      
/*///
    console.log('typeof requestGeoApi: ' + typeof requestGeoApi)
    //console.log('test:' + requestGeoApi.json())
    console.log('requestGeoApi: ' + requestGeoApi)
    
    console.log(JSON.stringify(requestGeoApi))    */
  },
  mounted() {
  //  this.$el.addEventListener('click', this.addAutocomplete());
  }
  
  
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

body {
  background: rgb(238,174,202);
  background: radial-gradient(circle, rgba(238,174,202,1) 0%, rgba(148,187,233,1) 100%);
}

.container {
  width: 550px;
  margin-left: auto;
  margin-right: auto;
}

.container span {
  display: block;
  margin: 15px 0 15px 0;
  font-size: 1.5em;
}
</style>
