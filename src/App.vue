<script setup lang="ts">
import { API_KEY, BASE_URL } from "./constans/index";
import HighLights from "./components/HighLights.vue";
import WeatherSummary from "./components/WeatherSummary.vue";
import Coords from "./components/Coords.vue";
import Humidity from "./components/Humidity.vue";
import { ref, onMounted, computed } from "vue";

interface WeatherData {
  cod?: number;
  message?: string;
  coord?: { lat: number; lon: number };
  main?: { temp: number; humidity: number };
  weather?: Array<{ description: string }>;
  name?: string;
  sys?: { country: string };
}

const city = ref('Timashevsk');
const weatherInfo = ref<WeatherData | null>(null);

const getWeather = () => {
  fetch(`${BASE_URL}?q=${city.value}&units=metric&appid=${API_KEY}`)
    .then(response => response.json())
    .then(data => weatherInfo.value = data);
};

onMounted(getWeather);

const isError = computed(() => weatherInfo.value?.cod !== 200);

const doUperCase = (str: string): string => {
  if (!str) return '';
  return str.charAt(0).toUpperCase() + str.slice(1);
};
</script>

<template>
  <div class="page">
    <main class="main">
      <div class="container">
        <div class="laptop">
          <div class="sections">
            <section :class="['section', 'section-left', { 'section-error': isError }]">
              <div class="info">
                <h1>{{ doUperCase(city) }}</h1>
                <div class="city-inner">
                  <input
                    v-model="city"
                    type="text"
                    class="search"
                    @keyup.enter="getWeather"
                  />
                  <button class="search-button" @click="getWeather"></button>
                </div>
                <weather-summary v-if="!isError && weatherInfo" :weatherInfo="weatherInfo" />
                <div v-else class="error">
                  <div class="error-title">Oooops! Something went wrong</div>
                  <div v-if="weatherInfo?.message" class="error-message">{{ doUperCase(weatherInfo.message) }}</div>
                </div>
              </div>
            </section>
            <section v-if="!isError && weatherInfo" class="section section-right">
              <high-lights :weatherInfo="weatherInfo" />
            </section>
          </div>
          <div v-if="!isError && weatherInfo" class="sections">
            <coords v-if="weatherInfo.coord" :coord="weatherInfo.coord" />
            <humidity v-if="weatherInfo.main" :main="weatherInfo.main" />
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<style lang="sass" scoped>
@import './assets/styles/main'
.search-button
  position: absolute
  right: 10px
  width: 25px
  height: 25px
  background: url('./assets/img/search.svg') no-repeat center
  background-size: contain
  border: none
  cursor: pointer
  outline: none
.page
  position: relative
  display: flex
  justify-content: center
  align-items: center
  min-height: 100vh
  padding: 20px 0
  background-color: #59585d

.laptop
  width: 100%
  padding: 20px
  background-color: #0e100f
  border-radius: 25px

.sections
  display: flex
  width: 100%

  @media (max-width: 767px)
    flex-direction: column

.section-left
  width: 30%
  padding-right: 10px

  @media (max-width: 767px)
    width: 100%
    padding-right: 0
    
  &.section-error
    min-width: 235px
    width: auto
    padding-right: 0

.section-right
  width: 70%
  padding-left: 10px

  @media (max-width: 767px)
    width: 100%
    margin-top: 16px
    padding-left: 0

.city-inner
  position: relative
  display: flex
  align-items: center
  width: 100%

.info
  height: 100%
  padding: 16px
  background: url('./assets/img/gradient-1.jpg') no-repeat 50% 50%
  background-size: cover
  border-radius: 25px

.search
  width: 100%
  padding: 16px
  font-family: 'Inter', Arial, sans-serif
  color: $white
  background-color: rgba(0, 0, 0, 0.75)
  border-radius: 16px
  border: none
  outline: none
  cursor: pointer

.section-bottom
  width: 50%
  margin-top: 16px

  @media (max-width: 767px)
    width: 100%

.error
  padding-top: 20px
  color: #ff5454
  &-title
    font-size: 18px
    font-weight: 700
  &-message
    padding-top: 10px
</style>