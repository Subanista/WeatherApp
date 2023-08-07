
<template>
  <div id="app" :class="weatherClass">
    <div class="background-image" :style="backgroundImageStyle"></div>
    <div class="overlay">
      <div class="weather-container">
        <div class="weather-header">
          <h1 class="app-title">Weather Application</h1>
          <WeatherComponent />
        </div>
        <div class="search-box">
          <div class="search-container">
            <input
              type="text"
              class="search-bar"
              v-model="query"
              @input="fetchSuggestions"
              @keydown.enter="handleEnter"
              @keypress="fetch_weather"
              placeholder="Search location"
            >
            <div class="search-icon">
              <i class="fa fa-search"></i>
            </div>
          </div>
          <ul class="suggestion-list" v-if="suggestions.length">
            <li v-for="suggestion in suggestions" :key="suggestion" @click="selectSuggestion(suggestion)">
              {{ suggestion }}
            </li>
          </ul>
        </div>
        <div class="weather-info" v-if="typeof weather.main != 'undefined'">
          <div class="location">{{ weather.name }}, {{ weather.sys.country }}</div>
          <div class="date">{{ dateBuilder() }}</div>
          <div class="temperature">{{ Math.round(weather.main.temp) }}°C</div>
          <div class="weather-description">{{ weather.weather[0].description }}</div>
          <div class="weather-image">
            <img id="wicon" alt="weather icon" :src="`https://openweathermap.org/img/w/${weather.weather[0].icon}.png`">
          </div>
        </div>
        <div class="clock">
          <div class="clock-face">
            <div class="hour-hand" :style="hourHandStyle"></div>
            <div class="minute-hand" :style="minuteHandStyle"></div>
            <div class="second-hand" :style="secondHandStyle"></div>
          </div>
        </div>
      </div>
    </div>
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
      backgroundImageUrl: '', // Initialize with a default or actual value
    hourRotation: 0, // Initialize with a default or actual value
    minuteRotation: 0, // Initialize with a default or actual value
    secondRotation: 0, // Initialize with a default or actual value
  
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

  computed: {
    backgroundImageStyle() {
      console.log("Calculating backgroundImageStyle...");
      return {
        backgroundImage: `url(${this.backgroundImageUrl})`
      };
    },
    weatherClass() {
      console.log("Calculating backgroundImageStyle...");
      return {
        'app.hot': typeof this.weather.main != 'undefined' && this.weather.main.temp > 5
      };
    },
    hourHandStyle() {
       console.log("Calculating backgroundImageStyle...");
      return {
        transform: `rotate(${this.hourRotation}deg)`
      };
    },
    minuteHandStyle() {
      console.log("Calculating backgroundImageStyle...");
      return {
        transform: `rotate(${this.minuteRotation}deg)`
      };
    },
    secondHandStyle() {
       console.log("Calculating backgroundImageStyle...");
      return {
        transform: `rotate(${this.secondRotation}deg)`
      };
    
  },
  },
 
created() {
  setInterval(() => {
    const now = new Date();
    this.secondRotation = (now.getSeconds() / 60) * 360;
    this.minuteRotation = ((now.getMinutes() + now.getSeconds() / 60) / 60) * 360;
    this.hourRotation = ((now.getHours() + now.getMinutes() / 60) / 12) * 360;
  }, 1000); // Update every second
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
//
      } else {
        //
      }
    })
    .catch(error => {
      console.error('Error fetching data:', error);
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

    selectSuggestion(suggestion){
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
    }
  

  }
    

}; 

</script>

<style scoped>
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



.background-image {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 600%;
  z-index: -1;
  /* Add appropriate background images for different weather conditions */
}

.overlay {
  background-color: rgba(0, 0, 0, 0.5);
  /* Add overlay for better visibility of text and UI elements */
}

.weather-container {
  padding: 20px;
  color: white;
  text-align: center;
  position: relative;
}

.weather-header {
  margin-bottom: 20px;
}

.search-bar {
  /* Customize search input styling */
}

.suggestion-list {
  list-style: none;
  padding: 0;
  margin: 10px 0;
}

.suggestion-list li {
  cursor: pointer;
  padding: 5px;
  background-color: rgba(255, 255, 255, 0.2);
  border-radius: 5px;
  margin-bottom: 5px;
}

.weather-info {
  margin-top: 20px;
}

.temperature {
  font-size: 48px;
  font-weight: bold;
  margin: 10px 0;
}

.weather-description {
  font-size: 20px;
  font-style: italic;
  margin-bottom: 20px;
}

.clock {
  margin-top: 40px;
  text-align: center;
}

.clock-face {
  width: 150px;
  height: 150px;
  border-radius: 50%;
  background-color: rgba(255, 255, 255, 0.2);
  position: relative;
  margin: 0 auto;
  display: flex;
  justify-content: center;
  align-items: center;
}

.hour-hand,
.minute-hand,
.second-hand {
  position: absolute;
  transform-origin: bottom center;
  transition: transform 0.5s cubic-bezier(0.4, 2.3, 0.5, 1);
}

.second-hand {
  width: 2px;
  height: 50px;
  background-color: black;
  top: 25%;
}

.minute-hand {
  width: 4px;
  height: 40px;
  background-color: black;
  top: 30%;
}

.hour-hand {
  width: 6px;
  height: 30px;
  background-color: black;
  top: 35%;
}



</style>


