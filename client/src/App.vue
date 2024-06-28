<template>
  <div class="container">
    <div class="header">
      <h1>Weather App</h1>
      <div class="search-bar">
        <input type="text" 
               v-model="city" 
               placeholder="Enter city name" 
               class="search-input" />
        <button @click="searchByCity" class="search-button">Search</button>
      </div>
    </div>
    <main class="main-section">
      <div v-if="weatherData" class="weather">
        <h2>{{ weatherData.name }}, {{ weatherData.sys.country }}</h2>
        <div class="temp-box">
          <img :src="iconUrl" alt="weather icon" class="weather-icon">
          <p class="temperature">{{ temperature }} °C</p>
        </div>
        <span class="clouds">{{ weatherData.weather[0].description }}</span>
      </div>
      <div v-else class="loading">Loading...</div>
      <div class="divider"></div>

      <div class="forecast">
        <div class="cast-header">Upcoming forecast</div>
        <div class="forecast-list">
          <div class="next" v-for="(forecast, index) in hourlyForecast" :key="index">
            <div class="forecast-item">
              <p class="time">{{ forecast.time }}</p>
              <p class="temp">{{ forecast.temp_max }} °C</p>
              <p class="desc">{{ forecast.description }}</p>
            </div>
          </div>
        </div>
      </div>

      <div v-if="dailyForecast.length" class="forecast">
        <div class="cast-header">Next 4 days forecast</div>
        <div class="forecast-list">
          <div class="day" v-for="(forecast, index) in dailyForecast" :key="index">
            <div class="forecast-item">
              <p class="date">{{ forecast.date }}</p>
              <p class="temp">{{ forecast.temp_max }} °C</p>
              <p class="desc">{{ forecast.description }}</p>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import axios from 'axios';

const apikey = "feff206daa60b539abe8fae8f2ab7f29";

export default {
  name: "App",
  data() {
    return {
      city: "",
      weatherData: null,
      hourlyForecast: [],
      dailyForecast: [],
    };
  },
  computed: {
    temperature() {
      return this.weatherData ? Math.floor(this.weatherData.main.temp - 273) : null;
    },
    iconUrl() {
      return this.weatherData ? `http://api.openweathermap.org/img/w/${this.weatherData.weather[0].icon}.png` : null;
    },
  },
  mounted() {
    this.fetchCurrentLocationWeather();
  },
  methods: {
    async fetchCurrentLocationWeather() {
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(async (position) => {
          const { latitude, longitude } = position.coords;
          const url = `http://api.openweathermap.org/data/2.5/weather?lat=${latitude}&lon=${longitude}&appid=${apikey}`;
          await this.fetchWeatherData(url);
        });
      }
    },
    async fetchWeatherData(url) {
      try {
        const response = await axios.get(url);
        this.weatherData = response.data;
        await this.fetchForecast(this.weatherData.name);
      } catch (error) {
        console.error("Error fetching weather data:", error);
      }
    },
    async fetchForecast(city) {
      const urlcast = `http://api.openweathermap.org/data/2.5/forecast?q=${city}&appid=${apikey}`;
      try {
        const response = await axios.get(urlcast);
        const forecast = response.data;
        this.hourlyForecast = this.hourlyForecastData(forecast);
        this.dailyForecast = this.dailyForecastData(forecast);
      } catch (error) {
        console.error("Error fetching forecast data:", error);
      }
    },
    async searchByCity() {
      if (!this.city) return;
      try {
        const urlsearch = `http://api.openweathermap.org/data/2.5/weather?q=${this.city}&appid=${apikey}`;
        const response = await axios.get(urlsearch);
        this.weatherData = response.data;
        await this.fetchForecast(this.weatherData.name);
      } catch (error) {
        console.error("Error fetching weather data:", error);
      }
      this.city = "";
    },
    hourlyForecastData(forecast) {
      const hourlyData = [];
      for (let i = 0; i < 8; i++) {
        const date = new Date(forecast.list[i].dt * 1000);
        hourlyData.push({
          time: date.toLocaleTimeString(undefined, { hour: '2-digit', minute: '2-digit' }),
          temp_max: Math.floor(forecast.list[i].main.temp_max - 273),
          description: forecast.list[i].weather[0].description,
        });
      }
      return hourlyData;
    },
    dailyForecastData(forecast) {
      const dailyData = [];
      for (let i = 8; i < forecast.list.length; i += 8) {
        const date = new Date(forecast.list[i].dt * 1000);
        dailyData.push({
          date: date.toDateString().slice(0, 10),
          temp_max: Math.floor(forecast.list[i].main.temp_max - 273),
          description: forecast.list[i].weather[0].description,
        });
      }
      return dailyData;
    }
  }
};
</script>

<style scoped>
/* Scoped styles */
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f0f2f5;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.header h1 {
  font-size: 2.5rem;
  color: #2b6cb0;
  margin: 0;
  font-family: 'Orbitron', sans-serif;
}

.search-bar {
  display: flex;
  align-items: center;
  margin-top: 20px;
}

.search-input {
  flex: 1;
  padding: 10px;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  margin-right: 10px;
}

.search-button {
  padding: 10px 20px;
  background-color: #4a90e2;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.search-button:hover {
  background-color: #357dbf;
}

.main-section {
  margin-top: 20px;
}

.weather {
  background-color: #4a7ea0;
  color: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  text-align: center;
}

.temp-box {
  margin-top: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.weather-icon {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  background-color: rgba(240, 248, 255, 0.5);
}

.temperature {
  font-size: 3rem;
  margin-left: 10px;
}

.clouds {
  display: inline-block;
  background-color: rgb(18, 32, 59);
  padding: 8px 16px;
  border-radius: 4px;
  font-size: 1.2rem;
  margin-top: 10px;
}

.forecast {
  margin-top: 20px;
}

.cast-header {
  font-size: 1.5rem;
  color: #333;
  background-color: #a5c6e6;
  padding: 10px;
  border-radius: 8px;
  margin-bottom: 10px;
}

.forecast-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}

.next,
.day {
  flex-basis: calc(20% - 10px);
  margin-bottom: 20px;
  padding: 20px;
  border-radius: 8px;
  background-color: #fff;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease;
}

.next:hover,
.day:hover {
  transform: translateY(-5px);
}

.time,
.date {
  font-size: 1.2rem;
  color: #4a90e2;
  margin-bottom: 10px;
}

.temp,
.desc {
  font-size: 1.2rem;
  margin-bottom: 10px;
}

@media screen and (max-width: 768px) {
  .next,
  .day {
    flex-basis: 100%;
  }
}
</style>
