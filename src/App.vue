<template>
  <div class="weather" :class="weatherClass">
    <div class="container">
      <div class="card weather-form">
        <input
          type="text"
          class="weather-form__input"
          v-model="searchQuery"
          @keyup.enter="weatherSearch"
          placeholder="Enter city"
        />
        <button class="weather-form__btn" @click="weatherSearch">Search</button>
      </div>

      <div class="card weather-load" v-if="loading">Loading...</div>

      <div
        class="weather-info"
        v-show="!error && location && temperature !== null && description"
      >
        <div class="card" v-if="error">Error</div>

        <div class="weather-info__text">
          <p class="card">{{ location }}</p>
          <p class="card">{{ temperature }}°C</p>
          <p class="card">{{ description }}</p>
        </div>
      </div>
    </div>
    <div class="weather-bg">
      <img
        class="weather-bg__img bg"
        src="./assets/bg.jpg"
        alt="App Background"
      />
      <img
        class="weather-bg__img overcast"
        src="./assets/overcast.jpg"
        alt="App Background"
      />
      <img
        class="weather-bg__img partly-cloudy"
        src="./assets/partly-cloudy.jpg"
        alt="App Background"
      />
      <img
        class="weather-bg__img sunny"
        src="./assets/sunny.jpg"
        alt="App Background"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

// Get API key from .env
const apiKey = import.meta.env.VITE_WEATHER_API_KEY;

// State
const searchQuery = ref("");
const location = ref("");
const temperature = ref(null);
const description = ref("");
const loading = ref(false);
const error = ref(false);


// Return background class based on weather description
const weatherClass = computed(() => {
  if (description.value.includes("Sunny")) return "sunny";
  if (description.value.includes("Overcast")) return "overcast";
  if (description.value.includes("Partly Cloudy")) return "partly-cloudy";
  return "";
});

// Weather search method
const weatherSearch = async () => {
  if (!searchQuery.value.trim()) return;
  loading.value = true;
  error.value = false;

  try {
    const url = `https://api.weatherapi.com/v1/current.json?key=${apiKey}&q=${encodeURIComponent(
      searchQuery.value
    )}&aqi=no`;
    const res = await fetch(url);
    const data = await res.json();

    if (data.error) throw new Error(data.error.message);

    location.value = data.location.name;
    temperature.value = data.current.temp_c;
    description.value = data.current.condition.text;
    resetSearchQuery();
  } catch (err) {
    error.value = true;
    console.error(err);
  } finally {
    loading.value = false;
  }
};

// Reset search input
const resetSearchQuery = () => {
  searchQuery.value = '';
};
</script>
