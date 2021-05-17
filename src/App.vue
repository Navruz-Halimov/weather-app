<template>
  <div id="app" class="weather">
    <!-- :class="
      typeof weather.main != 'undefined' && weather.main.temp > 16 ? 'warm' : ''
    " -->
    <div class="weather__inner">
      <div class="weather__sidebar">
        <div class="weather__sidebar-searchbox">
          <form @submit.prevent="fetchWeather()">
            <input
              type="text"
              class="search-bar"
              placeholder="Enter the city name"
              v-model="cityName"
            />
            <input
              type="text"
              v-model.number="days"
              placeholder="Enter days"
              :class="['search-bar', {'Eroranim':!(days >= 1 && days < 16 && typeof(days)=='number') && days!==null  && (days!=='')}]"
            />
            <p v-if="!(days >= 1 && days < 16 && typeof(days)=='number') && days!==null  && (days!=='')" :class="['error',{'shake':!(days >= 1 && days < 16 && typeof(days)=='number') && days!==null  && (days!=='')}]">Oops, Enter valid number between 1 and 16</p>
            <button
              type="submit"
              :disabled="!cityName && !(days >= 1 && days < 16 && typeof(days)=='number')"
              class="search-bar btn"
            >
              Submit
            </button>
          </form>
        </div>
        <div
          class="weather__sidebar-weather"
          v-for="weatherDefault of weatherByLocation"
          :key="weatherDefault.temp"
        >
          <div class="weather__sidebar-weather-city">
            {{ weatherDefault.city_name }}
          </div>
          <div class="weather__sidebar-weather-timezone">
            {{ weatherDefault.timezone }}
          </div>
          <img
            :src="
              `https://www.weatherbit.io/static/img/icons/${weatherDefault.weather.icon}.png`
            "
            :alt="weatherDefault.weather.description"
          />
          <div class="weather__sidebar-weather-temp">
            {{ weatherDefault.temp.toFixed(0) }}<sup>°</sup>
          </div>
          <div class="weather__sidebar-weather-description">
            {{ weatherDefault.weather.description }}
          </div>
        </div>
      </div>
      <loader v-if="loading" />
      <div class="weather__info" v-else>
        <div
          class="weather__info-item"
          v-for="weathers in weather"
          :key="weathers.temp"
        >
          <p class="weather__info-item-city">{{ city }}</p>
          <p class="weather__info-item-date">{{ weathers.datetime }}</p>
          <img
            :src="
              `https://www.weatherbit.io/static/img/icons/${weathers.weather.icon}.png`
            "
            :alt="weathers.weather.icon"
          />
          <p class="weather__info-item-temp">
            {{ weathers.temp.toFixed(0) }}<sup>°</sup>
          </p>
          <p class="weather__info-item-description">
            {{ weathers.weather.description }}
          </p>
          <div class="weather__info-item-roughly">
            <div class="weather__info-item-roughly-up">
              <i class="fa fa-caret-down" aria-hidden="true"></i>
              <p>{{ weathers.min_temp.toFixed(0) }}</p>
              <span class="min">Min</span>
            </div>
            <div class="weather__info-item-roughly-down">
              <i class="fa fa-caret-up" aria-hidden="true"></i>
              <p>{{ weathers.max_temp.toFixed(0) }}</p>
              <span class="up">Max</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  <Snowf
  v-if="loading"
  :amount="50"
  :size="5"
  :speed="1.5"
  :wind="0"
  :opacity="0.8"
  :swing="1"
  :image="null"
  :zIndex="null"
  :resize="true"
  color="#fff"
/>
  </div>
</template>
<script>
import Snowf from 'vue-snowf';
import loader from "./components/loader";
import axios from "axios";
export default {
  components: {
    loader,
    Snowf
  },
  name: "app",
  data() {
    return {
      api_key: "12eec63ec0ae476880e39ea2fd6500d9",
      url_base: `https://api.weatherbit.io/v2.0/forecast/`,
      cityName: "",
      days: null,
      weather: {},
      weatherByLocation: {},
      loading: true,
      lat: null,
      lot: null,
    };
  },
  created() {
    this.getLocation();
  },
  methods: {
    getLocation() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(this.showPosition);
      }
    },
    showPosition(position) {
      let customWeather = axios
        .get(
          `https://api.weatherbit.io/v2.0/current?lat=${position.coords.latitude}&lon=${position.coords.longitude}&key=${this.api_key}`
        )
        .then((res) => {
          this.weatherByLocation = res.data.data;
          console.log(res);
        });
      return customWeather;
    },
    fetchWeather() {
      this.loading = true;
      axios
        .get(
          `${this.url_base}daily?city=${this.cityName}&days=${this.days}&key=${this.api_key}`
        )
        .then((res) => {
          if (res && res.data) {
            this.weather = res.data.data;
            this.city = res.data.city_name;
            console.log(res);
            this.loading = false;
          }
        })
        .catch((err) => {
          console.log(err);
          this.loading = false;
        });
    },
    dateBuilder() {
      let d = new Date();
      let months = [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December",
      ];
      let days = [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday",
      ];
      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();
      return `${day} ${date} ${month} ${year}`;
    },
  },
};
</script>
<style lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,100;0,300;0,400;0,500;0,600;0,900;1,400;1,900&display=swap");
.btn {
  background-color: #31feae !important;
  box-shadow: 0 0 2rem 0.15rem rgb(0 0 255 / 10%) !important;
  color: white !important;
  // border:none !important;
  // outline: none !important;
  &:hover{
    cursor: pointer;
  }
  &:disabled{
    cursor:not-allowed;
    background-color: red !important;
    box-shadow:0 0 25px rgba(240, 34, 34, 0.8) !important;
  }
}
  @-webkit-keyframes spaceboots {
	0% { -webkit-transform: translate(2px, 1px) rotate(0deg); }
	10% { -webkit-transform: translate(-1px, -2px) rotate(-.5deg); }
	20% { -webkit-transform: translate(-2px, 0px) rotate(1deg); }
	30% { -webkit-transform: translate(0px, 2px) rotate(0deg); }
	40% { -webkit-transform: translate(1px, -1px) rotate(.5deg); }
	50% { -webkit-transform: translate(-1px, 2px) rotate(-.5deg); }
	60% { -webkit-transform: translate(-2px, 1px) rotate(0deg); }
	70% { -webkit-transform: translate(2px, 1px) rotate(-.5deg); }
	80% { -webkit-transform: translate(-1px, -1px) rotate(.5deg); }
	90% { -webkit-transform: translate(2px, 2px) rotate(0deg); }
	100% { -webkit-transform: translate(1px, -2px) rotate(-.5deg); }
}
.shake{
	-webkit-animation-name: spaceboots;
	-webkit-animation-duration: 0.8s;
	-webkit-transform-origin:50% 50%;
	-webkit-animation-iteration-count: 2;
	-webkit-animation-timing-function: linear;
}
.Eroranim{
  border:1px solid red !important;
  *>{
    transform: translateX(-100px);
  }
}
.error{
  font-size: 14px;
  color:white;
}
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  margin: 0;
  padding: 0;
  font-family: "Montserrat", sans-serif;
}
.weather__sidebar-searchbox {
  width: 100%;
  display: flex;
  margin-bottom: 30px;
  position: relative;
  z-index: 111111;
  justify-content: center;
  form {
    width: 75%;
  }
}
.weather__sidebar-searchbox .search-bar {
  display: block;
  width: 100%;
  padding: 15px;
  color: #313131;
  font-size: 16px;
  appearance: none;
  border: none;
  outline: none;
  background: none;
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.25);
  border-radius: 0px 16px 0px 16px;
  background-color: #fff;
  transition: 0.4s;
  margin-top: 10px;
}
.weather__sidebar-searchbox .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 0px 16px 0px;
}

.weather {
  &__inner {
    display: flex;
  }
  &__sidebar {
    width: 100%;
    margin: 0 auto;
    flex: 0 0 20%;
    min-width: 300px;
    min-height: 100vh;
    background-color: red;
    background-image: url("./assets/warm-bg.jpg");
    background-repeat: no-repeat;
    background-size: cover;
    &-weather {
      color:white;
      background-color: rgba(255, 255, 255, 0.25);
      height: 35%;
      padding: 30px 15px;
      text-align: center;
      &-city,
      &-timezone {
        font-size: 22px;
        font-weight: 400;
      }
      &-temp {
        font-size: 48px;
        font-weight: 300;
        sup {
          font-size: 32px;
        }
      }
      &-description {
        font-weight: 500;
        margin-top: 10px;
      }
    }
  }
  &__info {
    display: flex;
    align-items: flex-start;
    justify-content: center;
    flex-wrap: wrap;
    width: 100%;
    &-item {
      text-align: center;
      border-radius: 1.75rem;
      flex: 0 0 22%;
      width: 100%;
      box-sizing: border-box;
      margin: 10px;
      padding: 15px;
      box-shadow: rgba(149, 157, 165, 0.2) 0px 8px 24px;
      &-temp {
        font-size: 48px;
        font-weight: 300;
        sup {
          font-size: 32px;
        }
      }
      &-description {
        font-weight: 500;
        margin-top: 10px;
        margin-bottom: 20px;
      }
      &-city {
        font-size: 22px;
        font-weight: 400;
      }
      &-roughly {
        display: flex;
        justify-content: space-evenly;
        font-size: 24px;
        i.fa-caret-down,
        .min {
          color: rgb(0, 255, 155);
        }
        i.fa-caret-up,
        .up {
          color: red;
        }
        span {
          font-size: 16px;
          color: rgb(0, 255, 155);
        }
      }
    }
  }
}

// media query
@media screen and (max-width:1100px){
  .weather__sidebar{
    min-width: 300px;
  }
  .weather__info-item{
    flex: 0 0 45%;
  }

}
@media screen and (max-width:686px){

  .weather__info-item{
    flex: 0 0 85%;
  }

}
</style>
