<template>
  <div v-if="isWeatherData">
    <div class="card">
      <div class="city">
        <span v-for="(icon, index) in localData.icon" :key="`icon-${index}`">
          <img :src="icon">
        </span>
        <strong>{{localData.currentWeather.name}}, {{localData.currentWeather.sys.country}}</strong>
      </div>
      <p>Температура: {{localData.currentWeather.main.temp | round}}℃</p>
      <p>Скорость ветра: {{localData.currentWeather.wind.speed}} <small>м/сек.</small></p>
    </div>
    <div>
      <h2>Найти город</h2>
      <input type="text" v-model="localData.city" @click="localData.city = ''" placeholder="Введите город" required>
      <button :disabled="isButtonDisabled" @click="getWeatherByCity">Найти</button>
    </div>
    <div class="cities-list">
        <ol>
          <li v-for="(city, index) in localData.cities" :key="`city-${index}`">
            <a href="#" @click.prevent="selectCity(city)"> {{city}}</a>
            <a href="#" @click.prevent="removeCity(city)" class="close">×</a>
          </li>
        </ol>
    </div>
  </div>
</template>

<script>
const API_URL = 'https://api.openweathermap.org/data/2.5/weather';
const APP_ID = '56e16970e1331ae7ae3273cd28e53488';
const DEFAULT_CITY = 'Kiev';

export default {
  data() {
    return {
      localData: {
        latitude: undefined,
        longitude: undefined,
        currentWeather: {},
        icon: undefined,
        city: "",
        cities: []
      }
    }
  },
  created() {
    this.initMethod();
  },
  computed: {
    isButtonDisabled() {
      return this.localData.city == "";
    },
    isWeatherData() {
      return Object.keys(this.localData.currentWeather).length > 0;
    }
  },
  methods: {
    initMethod() {
      if(localStorage.getItem('weatherData')) {
        this.localData = JSON.parse(localStorage.getItem('weatherData'));
      } else if (navigator.geolocation) {
        return navigator.geolocation.getCurrentPosition(this.recordUserPosition, this.errorHandler);
      } else {
        alert("Your browser doesn't support geolocation.");
      }
    },
    recordUserPosition(position) {
      this.localData.latitude = position.coords.latitude;
      this.localData.longitude = position.coords.longitude;
      this.getCurrentWeather();
    },
    getCurrentWeather() {
      this.request(`${API_URL}?lat=${this.localData.latitude}&lon=${this.localData.longitude}&appid=${APP_ID}&units=metric&lang=ru`);
    },
    getWeatherByCity(event) {
      this.request(`${API_URL}?q=${this.localData.city}&appid=${APP_ID}&units=metric&lang=ru`, event);
    },
    addCityToList(item) {
      if(this.localData.cities.indexOf(item) == -1) {
        return this.localData.cities.push(item);
      }
    },
    selectCity(city) {
      this.localData.city = city;
      this.getWeatherByCity(city);
    },
    removeCity(city) {
      return this.localData.cities = this.localData.cities.filter(x => x !== city);
    },
    errorHandler(error) {
      if(error.PERMISSION_DENIED) {
        this.localData.city = DEFAULT_CITY;
        this.getWeatherByCity();
      }
    },
    request(url, event) {
      return fetch(url)
        .then((response) => {
            if (response.status == 404) {
              alert(response.statusText);
              return;
            }
            response.json().then((data) => {
              if(event !== undefined) {
                this.addCityToList(this.localData.city);
              }
              this.localData.currentWeather = data;
              this.localData.icon = data.weather.map(
                x => `http://openweathermap.org/img/w/${x.icon}.png`
              );
            });
          }
        )
        .catch((err) => {
          alert('Server Error', err.message);
        });
    }
  },
  filters: {
    round: function (text) {
      return Math.round(text);
    }
  },
  watch: {
    localData: {
      handler() {
        localStorage.setItem('weatherData', JSON.stringify(this.localData));
      },
      deep: true
    }
  }
}
</script>

<style>
  .city {
    font-size: 1.5em;
    display: flex;
    align-items: center;
  }
  .city img {
    position: relative;
    top:5px;
    left:-10px;
    margin-right:-5px;
  }
  .card {
    padding:20px;
    box-sizing: border-box;
    border: 1px solid #ccc;
    background-color: #f1f1f1;
    border-radius: 10px;
    margin-bottom: 40px;
  }
  input[type="text"] {
    border: 1px solid #ccc;
    padding:7px;
    font-size: 14px;
    width:80%;
    box-sizing: border-box;
  }
  button {
    padding:7px;
    background-color: #777;
    font-size: 14px;
    width: 18%;
    color: #fff;
    font-weight: bold;
    box-sizing: border-box;
    cursor: pointer;
  }
  button:disabled {
    background: #dddddd;
  }
  .cities-list {
    margin-top: 40px;
    margin-left: 0;
    counter-reset: section;
  }
  .cities-list li {
    margin-bottom: 5px;
    display: flex;
  }
  .cities-list li:before {
    counter-increment: section;
    content: counter(section) ".";
    position: absolute;
    margin-left: -20px;
  }
  .close {
    text-decoration: none;
    color:#000;
    padding-left: 2px;
    font-size: 18px;
    position: relative;
    top:1px;
    margin-left:auto;
  }

</style>
