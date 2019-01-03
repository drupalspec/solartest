<template>
  <div v-if="notEmptyObject(currentWather) > 0">
    <div class="card">
      <div class="city">
        <span v-for="(item, index) in icon" :key="index">
          <img :src="item">
        </span>
        <strong>{{currentWather.name}}, {{currentWather.sys.country}}</strong>
      </div>
      <p>Температура: {{currentWather.main.temp | round}}℃</p>
      <p>Скорость ветра: {{currentWather.wind.speed}} <small>м/сек.</small></p>
    </div>
    <div>
      <h2>Найти город</h2>
      <input type="text" v-model="city" @click="city = ''" placeholder="Ведите город" required>
      <button :disabled="isDisabled" @click="getDataByCity">Найти</button>
    </div>
    <div class="citues-list">
        <ol>
          <li v-for="(city, index) in cities" :key="index"><a href="#" @click="selectCity($event)">{{city}}</a></li>
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
      latitude: undefined,
      longitude: undefined,
      currentWather: {},
      icon: undefined,
      city: "",
      cities: []
    }
  },
  created() {
    this.getLocation();
  },
  computed: {
    isDisabled() {
      return this.city == "";
    }
  },
  methods: {
    recordPosition(position) {
      this.latitude = position.coords.latitude;
      this.longitude = position.coords.longitude;
      this.getDataFromApi();
    },
    getLocation() {
      if (navigator.geolocation) {
        return navigator.geolocation.getCurrentPosition(this.recordPosition, error => this.errorHendler(error));
      }
    },
    getDataFromApi() {
      this.request(`${API_URL}?lat=${this.latitude}&lon=${this.longitude}&appid=${APP_ID}&units=metric&lang=ru`);
    },
    getDataByCity(event) {
      this.request(`${API_URL}?q=${this.city}&appid=${APP_ID}&units=metric&lang=ru`, event);
    },
    addCityToList(item) {
      if(this.cities.indexOf(item) == -1) {
        return this.cities.push(item);
      }
    },
    selectCity(event) {
      this.city = event.target.innerText;
      this.getDataByCity(event);
    },
    notEmptyObject(someObject){
      return Object.keys(someObject).length
    },
    errorHendler(error) {
      if(error.PERMISSION_DENIED) {
        this.city = DEFAULT_CITY;
        this.getDataByCity();
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
                this.addCityToList(this.city);
              }
              this.currentWather = data;
              this.icon = data.weather.map(
                x => `http://openweathermap.org/img/w/${x.icon}.png`
              );
            });
          }
        )
        .catch((err) => {
          alert('Server Error', err.message);
        });
    },
  },
  filters: {
    round: function (text) {
      return Math.round(text);
    },
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
  .citues-list {
    margin-top: 40px;
  }
  .citues-list li {
    margin-bottom: 5px;
  }
  .overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(255,255,255,.9);
    z-index:10000;
  }

  .spinner{
    width: 40px;
    height: 40px;
    border: 2px solid #007bff;
    border-top:3px solid #007bff;
    border-radius: 100%;
    position: absolute;
    top:0;
    bottom:0;
    left:0;
    right: 0;
    margin: auto;
    animation: spin 1s infinite linear;
  }

  @keyframes spin {
    from{
      transform: rotate(0deg);
    }to{
       transform: rotate(360deg);
     }
  }

</style>
