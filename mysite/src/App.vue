<template>
  <div id="appTitle">
    <h1 class="app-title">Weather Application</h1>
    <WeatherComponent />
  </div>
  <div id="App" :class="typeof weather.main !='undefined' &&weather.main.temp>5? 'app.hot': ''  ">
    <main>
      
      <div class="search-box">
  <div class="search-container">
    <input
      name=""
      id=""
      type="text"
      class="search-bar"
      v-model="query"
      @input="fetchSuggestions"
         @keydown.enter="handleEnter"
      @keypress="fetch_weather"
      placeholder="Choose the location"
    >
    <div class="search-icon">
      <i class="fa fa-search"></i> <!-- Font Awesome search icon -->
    </div>
  </div>
  <div class="suggestions-box" v-if="suggestions.length">
    <ul class="suggestion-list">
      <li v-for="suggestion in suggestions" :key="suggestion" @click="selectSuggestion(suggestion)">
        {{ suggestion }}
      </li>
    </ul>
  </div>
</div>


      <div class="weather-wrap" v-if="typeof weather.main !='undefined'">
         <div class="location-box">
          <div class="location">{{weather.name}},{{weather.sys.country}}</div>
          <div class="date">{{dateBuilder()}}</div>
         </div>
         <div class="weather-box">
          <div class="temp">{{Math.round(weather.main.temp)}}'c </div>
          <div class="weather-image"><img id="wicon" alt="weather icon" v-bind:src="`https://openweathermap.org/img/w/${weather.weather[0].icon}.png`"></div>
          <div class="weather">{{weather.weather[0].main}}</div>
          <div class="weather-description">{{weather.weather[0].description}}</div>
         </div>
      </div>
      <div class="clock">
  <span class="clock-time">{{ currentTime }}</span>
</div>
</main>
</div>


</template>

<script>
import stringSimilarity from 'string-similarity'; // Import the string similarity library
import Fuse from 'fuse.js'; // Import the fuse.js library
import 'font-awesome/css/font-awesome.min.css';

export default {
  name: 'App',
  data(){
    return{
      currentDateTime: '',
      suggestions: ['New York, USA',
        'Paris, France',
        'Tokyo, Japan','jaffna','kandy','malabe'],
      fuse: null,
      locations: ['New York, USA',
        'Paris, France',
        'Tokyo, Japan','jaffna','kandy','malabe'],
      apiKey: 'AIzaSyAfOXNs2EEcOjwLqkjxJtDWE3hiUDdyxzk',
      manualSuggestions: [
      'New York, USA',
        'Paris, France',
        'Tokyo, Japan','jaffna','kandy','malabe'
        // Add more manual suggestions here
      ],
      api_key:'b250277ba17c175a86bd32040ef2c63a',
      url_base:'https://api.openweathermap.org/data/2.5/',
      query:'',
      weather:{}
    }
  },
  created() {
    // Fetch locations and initialize fuse.js
    this.fetchLocations().then(() => {
      this.fuse = new Fuse(this.locations, { keys: ['formatted_address'] });
    });
    this.updateCurrentTime();
  setInterval(() => {
    this.updateCurrentTime();
  }, 1000); // Update every second
  },
  methods:{ 
    updateCurrentTime() {
    const now = new Date();
    const options = { hour: '2-digit', minute: '2-digit', second: '2-digit', hour12: false };
    this.currentTime = now.toLocaleTimeString(undefined, options);
  },
    handleEnter() {
      if (this.suggestions.length === 1) {
        this.query = this.suggestions[0]; // Set the query to the selected suggestion
        this.suggestions = []; // Clear suggestions
        this.fetchWeather(); // Fetch weather data
      }
    },

    fetchLocations() {
      return fetch(`https://maps.googleapis.com/maps/api/geocode/json?address=&key=${this.apiKey}`)
        .then(response => response.json())
        .then(data => {
          if (data.status === 'OK') {
            this.locations = data.results;
          }
        });
    },

    fetchSuggestions() {
      if (this.query) {
        fetch(`https://maps.googleapis.com/maps/api/geocode/json?address=${this.query}&key=${this.apiKey}`)
          .then(response => response.json())
          .then(data => {
            if (data.status === 'OK') {
              // Use string similarity to suggest corrections
              const correctedSuggestions = this.correctQuery(this.query, data.results);
              this.suggestions = correctedSuggestions;
            }
          });
      } else {
        this.suggestions = [];
      }
    },

    selectSuggestion(suggestion) {
  console.log('Selected suggestion:', suggestion);
  this.query = suggestion;
  this.suggestions = [];
  this.fetchWeather();
},



    fetch_weather(e){
  if(e.key==="Enter"){
    fetch(`${this.url_base}weather?q=${this.query}&units=metric&APPID=${this.api_key}`)
    .then(res =>{
      return res.json();
    }).then(this.setResults);
  }
},
setResults(results){
  this.weather = results;
},
dateBuilder(){
  let d= new Date();
  let months= ["January", "February","March", "April","May","June","July","August","September","October","November","December"];
  let days=["Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"];
  let day= days[d.getDay()];
  let date = d.getDate();
  let month=months[d.getMonth()];
  let year= d.getFullYear();
  return `${day} ${date} ${month} ${year}`;

},

    correctQuery(query, results) {
      const locations = results.map(result => result.formatted_address.toLowerCase());
      const matches = stringSimilarity.findBestMatch(query.toLowerCase(), locations);
      const correctedSuggestions = matches.ratings
        .filter(rating => rating.rating > 0.5) // Adjust the threshold as needed
        .map(match => results[match.index].formatted_address);

      return correctedSuggestions;
    },
  

    }
    

  }
</script>

<style>
.clock {
  font-size: 28px;
  color: #333; /* Change to your desired text color */
  text-align: center; /* Align the clock to the center */
  margin-top: 10px; /* Add some spacing above the clock */
}

.clock-time {
  font-weight: bold;
}
#appTitle {
  text-align: center;
  padding: 20px;
}

.app-title {
  color: white;
  margin-top: 0; /* Remove default margin from h1 */
}
*{
  margin:0;
  padding: 0;
  box-sizing: border-box;
}
body{
  font-family: 'montserrat', sans-serif;
}
#app{
  background-image:url("assets/dk.jpg");
  background-size: cover;
  background-position: bottom;
  transition: 0.4s;
}
#app.hot{
  background-image:url("assets/h.jpeg");
}
main{
  min-height: 100vh;
  padding: 25px;
  
}
.search-box{
  width:100%;
  margin-bottom: 30px;
}
.search-box .search-bar{
display:block;
width:100%;
padding:15px;
color: #313131;
font-size: 20px;
box-shadow: 0px 0px 16px rgba(0,0,0,0.25);

appearance: none;
border: none;
background: none;
outline: none;
background-color: rgba(255,255,255,0.5);
border-radius: 10px 10px 10px 10px;
transition: 0.4s;

}



.location-box .location {
  color: #FFFFFF;
  font-size: 32px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0,0,0,0.25);

}
.location-box .date{
  color: #FFFFFF;
  font-size: 20px;
  font-weight: 300;
  font-style: italic;
  text-align: center;

}
.weather-box{
  text-align: center;
}

.weather-box .temp{
  display: inline-block;
  padding: 10px 25px;
  color: #FFFFFF;
  font-size: 102px;
  font-weight: 900;

  text-shadow: 3px 6px rgba(0,0,0,0.25);
  background-color: rgba(255,255,255,0.25);
  border-radius: 16px;
  margin:30px 0px;

  box-shadow: 3px 6px rgba(0,0,0,0.25);
}
.weather-box .weather{
  color: #FFFFFF;
  font-size: 35px;
  font-weight: 300;
  font-style: italic;
  text-shadow:3px 6px rgba(0,0,0,0.25) ;
}
.weather-box .weather-description{
  color: #FFFFFF;
  font-size: 10px;
  font-weight: 300;
  font-style: italic;
}
.weather-box .weather-image{
  position: center;
}
.suggestions-box {
  position: absolute;
  width: 50%;
  max-height: 200px;
  overflow-y: auto;
  border: 1px solid #ccc;
  background-color: white;
  z-index: 1;
}

.suggestion-list {
  list-style-type: none;
  padding: 0;
  margin: 0;
}

.suggestion-list li {
  padding: 5px 10px;
  cursor: pointer;
}

.suggestion-list li:hover {
  background-color: #f2f2f2;
}



.search-container {
  position: relative;
}

.search-icon {
  position: absolute;
  top: 50%;
  right: 20px;
  transform: translateY(-50%);
  color: #ccc;
}
.current-date-time {
  font-size: 18px;
  color: #333; /* Change to your desired text color */
  text-align: center; /* Align the date and time to the center */
  margin-top: 10px; /* Add some spacing above the date and time */
}
</style>
