<template>
  <div class="weather" :class="weatherClass">
    <div class="container">
      <div class="language-switcher">
        <select v-model="language">
          <option value="en">EN</option>
          <option value="uk">UA</option>
        </select>
      </div>
      <div class="card weather-form">
        <input
          type="text"
          class="weather-form__input"
          v-model="searchQuery"
          @keyup.enter="weatherSearch"
          :placeholder="t.placeholder"
        />
        <button class="weather-form__btn" @click="weatherSearch">
          {{ t.search }}
        </button>
      </div>

      <div class="card weather-load" v-if="loading">Loading...</div>

      <div class="card weather-error" v-if="error">Error</div>

      <div
        class="weather-info"
        v-show="!error && location && temperature !== null && description"
      >
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
      <img class="weather-bg__img sunny" src="./assets/sunny.jpg" alt="Sunny" />
      <img
        class="weather-bg__img nightClear"
        src="./assets/night.jpg"
        alt="Night Clear"
      />
      <img
        class="weather-bg__img cloudy"
        src="./assets/cloudy.jpg"
        alt="Cloudy"
      />
      <img class="weather-bg__img rain" src="./assets/rain.jpg" alt="Rain" />
      <img class="weather-bg__img snow" src="./assets/snow.jpg" alt="Snow" />
      <img class="weather-bg__img fog" src="./assets/fog.jpg" alt="Fog" />
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
const language = ref("en");

// Simple mapping for transliteration of Ukrainian letters
const transliterate = (text) => {
  const map = {
    а: "a",
    б: "b",
    в: "v",
    г: "h",
    ґ: "g",
    д: "d",
    е: "e",
    є: "ye",
    ж: "zh",
    з: "z",
    и: "y",
    і: "i",
    ї: "yi",
    й: "y",
    к: "k",
    л: "l",
    м: "m",
    н: "n",
    о: "o",
    п: "p",
    р: "r",
    с: "s",
    т: "t",
    у: "u",
    ф: "f",
    х: "kh",
    ц: "ts",
    ч: "ch",
    ш: "sh",
    щ: "shch",
    ю: "yu",
    я: "ya",
    ь: "",
    "'": "",
  };
  return text
    .toLowerCase()
    .split("")
    .map((char) => map[char] ?? char)
    .join("");
};

// Weather condition groups with support for both English and Ukrainian
const weatherGroups = {
  sunny: ["sunny", "сонячно"],
  nightClear: ["clear", "ясно"],
  cloudy: [
    "cloudy",
    "хмарно",
    "похмуро",
    "overcast",
    "partly cloudy",
    "невелика хмарність",
    "хмарність",
  ],
  rain: ["rain", "дощ", "shower", "гроза", "thunder", "drizzle", "мряка"],
  snow: ["snow", "сніг", "blizzard", "freezing"],
  fog: ["fog", "mist", "туман", "haze"],
};

// Compute CSS background class based on weather description
const weatherClass = computed(() => {
  const desc = description.value.toLowerCase();
  for (const [className, keywords] of Object.entries(weatherGroups)) {
    if (keywords.some((k) => desc.includes(k))) {
      return className;
    }
  }
  return "";
});

// Weather search method
const weatherSearch = async () => {
  if (!searchQuery.value.trim()) return;
  loading.value = true;
  error.value = false;

  try {
    // If language is Ukrainian — transliterate query
    const query =
      language.value === "uk"
        ? transliterate(searchQuery.value)
        : searchQuery.value;

    const url = `https://api.weatherapi.com/v1/current.json?key=${apiKey}&q=${encodeURIComponent(
      query
    )}&aqi=no&lang=${language.value}`;
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

// Localized UI text for supported languages
const translations = {
  en: {
    placeholder: "Enter city",
    search: "Search",
  },
  uk: {
    placeholder: "Введіть місто",
    search: "Пошук",
  },
};

// Reactive computed value for current language translations
const t = computed(() => translations[language.value]);

// Reset search input
const resetSearchQuery = () => {
  searchQuery.value = "";
};
</script>
