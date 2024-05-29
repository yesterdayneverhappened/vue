<style scoped>

</style>

<template>
  <header :class="{ show: !showWrapper, hide: showWrapper }">
    <div style="display: flex; justify-content:baseline">
      <img src="..\public\img\logo.png" width="40px">
      <p class="logo">Blah-Blah</p>
    </div>
    <div class="userProfile">
      <div class="avatar"> <img src="../public/img/user.png" width="40px" v-if="showLogin"/> </div>
      <button type="button" @click="openDialog" class="login-button" v-if="!showLogin">Войти</button>
      <p class="userName" v-if="isOpen == false">{{ login }}</p>
      <div v-if="isOpen" class="dialog-container">
        <div class="dialog-content">
          <h2>Авторизация</h2>
          <form @submit.prevent="handleSubmit">
            <label>
              Логин:
              <input v-model="login" type="text" />
            </label>
            <label>
              Пароль:
              <input v-model="password" type="password" />
            </label>
            <div class="dialog-buttons">
              <button type="submit">Войти</button>
              <button type="button" @click="closeDialog">Отмена</button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </header>
  <div class="pereh">
    <button v-if="showWrapper === false" @click="showWrapper = true" type="button"><img src="../public/img/loop.png" width="17px"> Поиск</button>
  </div>
  <div class="city" :class="{ show: !showWrapper, hide: showWrapper }">
    <div class="all">
      <p class="cityName">{{city}}</p>
      
      
      <div class="info">
        <p v-if="info != null" class="temperature">{{ temperature }}</p>
        <p v-if="info != null" class="weather">{{ info.weather[0].description }}</p>
      </div>
      
    </div>
    <div class="moreInform">
      <!-- Добавляем блок для прогноза погоды -->
      <div class="forecast">
        <!-- Прогнозы погоды будут добавлены сюда -->
      </div>
    </div>
  </div>

  <div class="moreInfoWeather" :class="{ show: !showWrapper, hide: showWrapper }">
    <div class="moreInfo">
      <div class="infoFillLike" v-if="info != null">
        <p v-if="info != null" class="feel"> Feels like<br> <span>{{this.info.main.feels_like + " °С" }}</span></p>
      </div>
      <div class="infoFillLike" v-if="info != null">
        <p v-if="info != null" class="minTemp"> Min temp<br> <span>{{this.info.main.temp_min + " °С" }}</span></p>
      </div>
      <div class="infoFillLike" v-if="info != null">
        <p v-if="info != null" class="maxTemp"> Min temp<br> <span>{{ this.info.main.temp_max + " °С" }}</span></p>
      </div>
    </div>
  </div>

  <div class="otherInfo" v-if="!showWrapper" >
    <div class="row" v-if="info != null">
      <div class="wind"> <img src="../public/img/wind.png" width="60px" style="filter:invert(1)"> <br> <span>Wind</span><br>{{wind.speed}} M/seks</div>
      <div class="preassure"><img src="../public/img/pressure.png" width="60px" style="filter:invert(1)"> <br> <span>Pressure</span><br>{{ wind.pressure }}</div>
    </div>
    <div class="row" v-if="info != null">
      <div class="humidity"> <img src="../public/img/humidity.png" width="60px" style="filter:invert(1)"> <br> <span>Humidity</span><br>{{ wind.humidity }}%</div>
      <div class="sun">
        <p class="left"><span>Sunrise</span><br>{{ wind.sunrise }}</p>
        <img src="../public/img/sunset.png" width="60px" style="filter:invert(1)">
        <p class="right"><span>Sunset</span><br>{{wind.sunset}}</p>
      </div>
    </div>
    
  </div>

  <div class="wrapper" :class="{ show: showWrapper, hide: !showWrapper }">
    <h1 v-if="city.trim() !== ''">{{ city }}</h1>
    <h1 v-else>Погодное приложение</h1>
    <p>Узнать погоду в вашем городе</p>
    <input type="text" v-model="city" placeholder="Введите город">
    <button v-show="city != ''" @click="getWeather()">Найти</button>
    <p class="err">{{ err }}</p>
  </div>
  <video
    v-if="info && info.weather[0].description.includes('clear')"
    autoplay
    muted
    loop
    id="myVideo" :class="{ show: !showWrapper, hide: showWrapper }">
    <source src="../public/img/sun.mp4" type="video/mp4">
</video>
<video
    v-if="info && info.weather[0].description.includes('clouds')"
    autoplay
    muted
    loop
    id="myVideo" :class="{ show: !showWrapper, hide: showWrapper }">
    <source src="../public/img/clouds.mp4" type="video/mp4">
</video>
<video
    v-if="info && info.weather[0].description.includes('rain')"
    autoplay
    muted
    loop
    id="myVideo" :class="{ show: !showWrapper, hide: showWrapper }">
    <source src="../public/img/rainy.mp4" type="video/mp4">
</video>
<video
    v-if="info && info.weather[0].description.includes('mist')"
    autoplay
    muted
    loop
    id="myVideo" :class="{ show: !showWrapper, hide: showWrapper }">
    <source src="../public/img/foggy.mp4" type="video/mp4">
</video>


</template>

<script>
import axios from 'axios';
import moment from 'moment';
export default {
  data() {
    return {
      city: '',
      err: '',
      info: null,
      showWrapper: false,
      wind: {
        speed: '',
        pressure: '',
        humidity: '',
        sunrise: '',
        sunset: ''
      },
      isOpen: false,
      login: '',
      password: '',
      showLogin: false
    };
  },
  methods: {
    openDialog() {
      this.isOpen = true;
    },
    closeDialog() {
      this.isOpen = false;
    },
    handleSubmit() {
      // Здесь вы можете обработать отправку данных логина и пароля
      console.log('Login:', this.login);
      console.log('Password:', this.password);
      this.closeDialog();
      this.showLogin = true;
    },

    getWeather() {
      if (this.city.trim().length < 2) {
        this.err = 'Слишком короткое название города';
        return false;
      }
      this.err = '';

      axios
        .get(
          `https://api.openweathermap.org/data/2.5/weather?q=${this.city}&units=metric&appid=e2b448a56fe17e55d5946ccafda82e11`
        )
        .then((res) => {
          this.info = res.data;
          this.showWrapper = false;
          // Получаем информацию о ветре и присваиваем ее свойствам объекта wind
          // Преобразуем Unix-время восхода и заката солнца в обычное время
          const sunrise = res.data.sys.sunrise;
          const sunset = res.data.sys.sunset;
          const sunriseTime = moment.unix(sunrise).format('HH:mm');
          const sunsetTime = moment.unix(sunset).format('HH:mm');

          // Присваиваем данные объекту wind
          this.wind.speed = res.data.wind.speed; // Получить скорость ветра из объекта wind
          this.wind.pressure = res.data.main.pressure; // Получить давление из объекта main
          this.wind.humidity = res.data.main.humidity; // Получить влажность из объекта main
          this.wind.sunrise = sunriseTime; // Преобразованное время восхода солнца
          this.wind.sunset = sunsetTime; // Преобразованное время заката солнца

          this.fetchForecast();
        });
    },
    fetchForecast() {
  var endpoint =
    `https://api.openweathermap.org/data/2.5/forecast?q=${this.city}&units=metric&appid=e2b448a56fe17e55d5946ccafda82e11`;

  axios.get(endpoint)
    .then(response => {
      const forecasts = response.data.list;
      console.log(response);
      const dailyForecasts = {};

      forecasts.forEach(forecast => {
        // Извлекаем дату из прогноза
        const date = forecast.dt_txt.split(' ')[0];

        if (dailyForecasts[date]) {
          dailyForecasts[date].push(forecast.main.temp);
        } else {
          dailyForecasts[date] = [forecast.main.temp];
        }
      });

      const forecastContainer = document.querySelector('.forecast');
      forecastContainer.innerHTML = '';
      var i  = 0;
      for (const date in dailyForecasts) {
        if (dailyForecasts.hasOwnProperty(date)) {
          const temperatures = dailyForecasts[date];
          const averageTemperature = Math.round(
            temperatures.reduce((sum, temp) => sum + temp, 0) / temperatures.length
          );

          const forecastItem = document.createElement('div');
          forecastItem.classList.add('forecast-day');

          const dayOfWeek = new Date(date).toLocaleDateString('en', { weekday: 'long' });

          forecastItem.innerHTML = `
            <p class="day">${dayOfWeek}</p>
            <p><img src="https://openweathermap.org/img/wn/${forecasts[i].weather[0].icon}.png" alt="${forecasts[0].weather[0].description}" width="100px"></p>
            <div class="forecast-day--temp">${averageTemperature}<sup>°C</sup></div>
          `;

          // Добавляем элемент прогноза в блок прогноза погоды
          forecastContainer.appendChild(forecastItem);
          i++;
        }
      }
    })
    .catch(error => {
      console.error('Ошибка при получении прогноза погоды:', error);
    });
},

  },
  computed: {
    temperature() {
      return this.info ? `${this.info.main.temp} °C` : '';
    }
  }
};
</script>