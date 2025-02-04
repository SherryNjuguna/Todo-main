<script setup>
import { onMounted, computed, ref } from "vue";


const api_key = "c73622a39f2d4383a72ce87dbb5cc01e";
const weatherData = ref({});
const isLoading = ref(false);
const city_name = ref("Nairobi");
const searchQuery = ref("");
const searchedCityData = ref(null); 
let error = ref(null);
const currentTime = ref("");
const currentDate = ref("");

const getWeather = async (city) => {
  isLoading.value = true;
  try {
    const res = await fetch(
      `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${api_key}&units=metric`
    );

    if (!res.ok) {
      throw new Error(`HTTP error! status: ${res.status}`);
    }

    const data = await res.json();
    weatherData.value[city] = data;
    searchedCityData.value = data; 

  } catch (err) {
    error.value = err.message;
    console.error("Fetching error:", err);
  } finally {
    isLoading.value = false;
  }
};

const getWeatherByCoords = async (lat, lon) => {
  try {
    const res = await fetch(
      `https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${api_key}&units=metric`
    );

    if (!res.ok) {
      throw new Error("HTTP error! status: ${res.status}");
    }
    const data = await res.json();
    weatherData.value.currentLocation = data;
  } catch (err) {
    error.value = err.message;
    console.error("fetching error:", err);
  } finally {
    isLoading.value = false;
  }
};

const getCurrentLocation = () => {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(
      (position) => {
        const { latitude, longitude } = position.coords;
        getWeatherByCoords(latitude, longitude);
      },
      (err) => {
        error.value = "Unable to retrieve your location.";
        console.error("Geolocation error:", err);
        getWeather("Nairobi");
      }
    );
  } else {
    error.value = "Geolocation is not supported by this browser.";
    getWeather("Nairobi");
  }
};

const handleSearch = (event) => {
  event.preventDefault();
  if (searchQuery.value.trim()) {
    getWeather(searchQuery.value.trim());
    searchQuery.value = "";
  }
};

const updateTimeAndDate = () => {
  const now = new Date();
  currentTime.value = now.toLocaleTimeString([], {
    hour: "2-digit",
    minute: "2-digit",
  });
  currentDate.value = now.toLocaleDateString("en-GB", {
    day: "2-digit",
    month: "short",
    year: "numeric",
  });
};

import sunny from "@/assets/sunny.jpg";
import cloud from "@/assets/cloud4.jpg";
import rain from "@/assets/rain.jpg";
import cloud1 from "@/assets/cloud4.jpg";

const weatherBackgrounds = {
  Clear: sunny,
  Clouds: cloud,
  Rain: rain,
  Default: cloud1,
};

const backgroundImage = computed(() => {
  const weather = searchedCityData.value?.weather?.[0]?.main || weatherData.currentLocation?.weather?.[0]?.main;
  return weatherBackgrounds[weather] || weatherBackgrounds.Default;
});

onMounted(async () => {
  await getWeather(city_name.value);
  await getWeather("Mombasa");
  await getWeather("Naivasha");
  getCurrentLocation();
  updateTimeAndDate();
  setInterval(updateTimeAndDate, 1000);
});
</script>

<template>
  <div class="container image m-0 p-0" :style="{ backgroundImage: `url(${backgroundImage})` }">
    <div class="row">
      <div class="col d-flex align-items-center">
        <div class="logo">
          <img
            src="@/assets/logo.png"
            alt=""
            class="img-fluid p-2"
            style="width: 50px; height: 50px"
          />
        </div>
        <div>
          <p class="text-dark fw-semibold me-5 pe-4 pt-2">Weather Forecast</p>
        </div>
        <div class="navbar pt-3">
          <ul class="d-flex align-items-center list-unstyled">
            <li class="me-3"><a href="#" class="text-dark">Home</a></li>
            <li><a href="#" class="text-dark">Application</a></li>
          </ul>
        </div>

        <div class="date d-flex pt-2">
          <h4 class="me-2 ms-5">{{ currentTime }}</h4>
          <h6>
            {{ currentTime.includes("AM") ? "AM" : "PM" }}
          </h6>
        </div>

        <div class="btn pt-2">
          <button class="btn rounded-pill" style="background-color: lightblue">
            <i class="fas fa-cloud me-2"></i>
            <i class="fas fa-sun"></i>
          </button>
        </div>
      </div>

      <div class="col">
        <div class="date d-flex pt-2">
          <h4 class="me-2">
            {{ currentDate.split(" ")[0] }} {{ currentDate.split(" ")[1] }}
          </h4>
          <h6>
            {{ currentDate.split(" ")[2] }} <br />
            Date
          </h6>
        </div>

        <div class="greet ms-3">
          <h6 class="me-2 ms-5 pt-2">
            Hey 👋 <br />
            Rosemary
          </h6>
        </div>

        <div class="btn pt-2 ms-5">
          <button class="btn rounded-pill" style="background-color: lightblue">
            <i class="fas fa-sun me-2"></i>
            <i class="fa-solid fa-circle-plus me-2"></i>
            <i class="fa-solid fa-face-smile"></i>
          </button>
        </div>

        <div class="icons">
          <i
            class="fa-solid fa-globe pt-2 me-5 ms-5"
            style="width: 40px; height: 40px"
          ></i>
          <i
            class="fa-solid fa-bars pt-2 me-3"
            style="width: 40px; height: 40px"
          ></i>
        </div>
      </div>
    </div>

    <div class="row">
      <div class="col d-block">
        <div class="col">
        <div class="search-bar mt-3 mb-3">
          <form @submit="handleSearch" class="d-flex justify-content-center">
            <input
              v-model="searchQuery"
              type="text"
              placeholder="Enter a city name"
              class="form-control w-50 me-2"
            />
            <button type="submit" class="btn btn-primary">Search</button>
          </form>
          <div v-if="searchedCityData">
      <p class="ms-5 fw-bold">
        City: {{ searchedCityData.name }} <br />
        Temp: {{ searchedCityData.main.temp.toFixed(1) }}&deg; <br />
        Weather: {{ searchedCityData.weather[0].description }}
      </p>
    </div>   
        </div>
      </div>
        <div v-if="isLoading">Loading...</div>
        <div v-else-if="error">Error: {{ error }}</div>
        <div v-else></div>
        <div v-if="weatherData.currentLocation" class="firstsection">
          <p class="fw-bold ms-5" style="font-size: 5rem">
            {{ searchedCityData.weather[0].main }} 
          </p>
          <p class="fs-1 ms-5 fw-semibold">
            {{ searchedCityData.weather[0].description }}
          </p>
        </div>
        <!-- weatherData.currentLocation.weather[0].main ||  -->
        <div class="secondsection d-block">
          <div class="d-flex">
            <div class="first ms-5 pt-3">
              <p style="font-size: 3rem">
                {{
                  weatherData.Mombasa?.main?.temp.toFixed(1) || "Loading..."
                }}&deg;
              </p>
              <p class="border-bottom border-dark pb-2">
                {{ weatherData.Mombasa?.coord?.lon || "Loading..." }}W <br />
                Mombasa
              </p>
            </div>
            <div class="first ms-5 pt-3">
              <p style="font-size: 3rem">
                {{
                  weatherData.Nairobi?.main?.temp.toFixed(1) || "Loading..."
                }}&deg;
              </p>
              <p class="border-bottom border-dark pb-2">
                {{ weatherData.Nairobi?.coord?.lon || "Loading..." }}W
                <br />
                Nairobi
              </p>
            </div>
            <div class="first ms-5 pt-3">
              <p style="font-size: 3rem">
                {{
                  weatherData.Naivasha?.main?.temp.toFixed(1) || "Loading..."
                }}&deg;
              </p>
              <p class="border-bottom border-dark pb-2">
                {{ weatherData.Naivasha?.coord?.lon || "Loading..." }}W<br />
                Naivasha
              </p>
            </div>
          </div>

          <div class="second flex-column align-items-center position-absolute">
            <div class="temperature-tracker ms-5 mt-2">
              <div class="progress-bar" style="--progress: 50%">
                <div
                  class="indicator-container flex-column align-items-center position-absolute"
                >
                  <div class="indicator bg-white rounded-circle"></div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>


      <div class="col d-block">
        <div class="topsection d-flex">
          <div class="icons d-inline">
            <div>
              <button class="btn-sm rounded-pill mb-2" style="margin-top: 4rem">
                {{ searchedCityData?.coord?.lon  || "Loading..." }}W
              </button>
            </div>
            <button class="btn-sm rounded-pill">
              {{ searchedCityData?.coord?.lat || "Loading..." }}W
            </button>
          </div>

          <div class="temperature d-flex">
            <p style="font-size: 10rem">
              {{
               searchedCityData?.main?.temp.toFixed(1) || "Loading..."
              }}&deg;
            </p>
            <i
              class="fa-solid fa-plus-minus"
              style="padding-top: 9rem; height: 30px"
            ></i>
          </div>
        </div>

        <div class="middlesection d-flex">
          <div class="me-5 pe-5">
            <p>intensity <br />zones</p>
          </div>

          <div class="ms-5 ps-5">
            <p>
              WIND:WSW {{ weatherData.Mombasa?.wind?.speed || "Loading..." }}MPH
              <br />
              UV INDEX:0 OF 10 <br />
              -MOMBASA
            </p>
          </div>
        </div>

        <div class="bottomsection d-block">
          <div class="d-flex">
            <div class="me-5" style="font-size: 30px">
              <i class="fa-solid fa-temperature-quarter"></i>
            </div>
            <div>
              <p style="font-size: 30px">
                {{ weatherData.Mombasa?.main?.humidity || "Loading..." }}%
              </p>
            </div>
          </div>

          <div class="d-flex">
            <div>
              <button class="btn btn-transparent" style="width: 140px">
                Nairobi[NB]
              </button>
            </div>
            <div class="me-5 pe-5 ms-5 ps-5">
              <p>
                {{
                  weatherData.Nairobi?.main?.temp.toFixed(1) || "Loading..."
                }}&deg;
              </p>
            </div>
            <div><i class="fa-solid fa-cloud"></i></div>
          </div>

          <div class="d-flex">
            <div>
              <button class="btn btn-transparent" style="width: 140px">
                Naivasha [NV]
              </button>
            </div>
            <div class="me-5 pe-5 ms-5 ps-5">
              <p>
                {{
                  weatherData.Naivasha?.main?.temp.toFixed(1) || "Loading..."
                }}&deg;
              </p>
            </div>
            <div><i class="fa-solid fa-cloud-rain"></i></div>
          </div>

          <div class="d-flex">
            <div>
              <button class="btn btn-transparent" style="width: 140px">
                Mombasa [MB]
              </button>
            </div>
            <div class="me-5 pe-5 ms-5 ps-5">
              <p>
                {{
                  weatherData.Mombasa?.main?.temp.toFixed(1) || "Loading..."
                }}&deg;
              </p>
            </div>
            <div><i class="fa-regular fa-sun"></i></div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <div class="wave-container">
    <div class="wave"></div>
  </div>
</template>

<style>

.image {
  background-image: url("@/assets/cloud1.jpeg");
  background-repeat: no-repeat;
  background-size: cover;
  min-height: 100vh;
  min-width: 100%;
}

.col {
  display: flex;
}

ul,
li,
a {
  text-decoration: none;
}

.temperature-tracker {
  width: 250px;
  height: 10px;
  border: 2px solid #000;
  border-radius: 50px;
  position: relative;
  background: linear-gradient(to right, red, yellow, green, red);
  overflow: visible;
}

.progress-bar {
  height: 100%;
  width: calc(var(--progress, 0%));
  background: #254d75;
  position: relative;
  z-index: 2;
  transition: width 0.3s ease-in-out;
  border-radius: 50px;
}

.indicator-container {
  top: 50%;
  right: 0;
  transform: translate(50%, -50%);
  z-index: 3;
}

.indicator {
  width: 20px;
  height: 50px;
  background: #fff;
  border-radius: 100%;
  z-index: 3;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.2);
}

.temperature-text {
  font-size: 14px;
  color: #333;
  font-weight: bold;
}

.status-text {
  font-size: 12px;
  color: #555;
}

.btn-sm {
  backdrop-filter: blur(60px);
  background-color: rgba(147, 250, 147, 0.1);
}

.search-bar {
  margin: 20px 0;
}

.weather-card {
  background: rgba(255, 255, 255, 0.8);
  border-radius: 10px;
  padding: 20px;
  margin: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}
</style>
