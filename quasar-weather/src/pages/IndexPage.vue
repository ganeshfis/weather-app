<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        @keyup.enter="getWeatherBySearch()"
        v-model="search"
        placeholder="Search"
        dark
        borderless
        :error="error"
        :error-message="error_message"
      >
        <template v-slot:before>
          <q-icon @click="getLocation()" name="my_location" />
        </template>

        <template v-slot:append>
          <q-btn round dense flat icon="search" @click="getWeatherBySearch" />
        </template>
      </q-input>
    </div>
    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">{{ weatherData.name }}</div>
        <div class="text-h6 text-weight-light">
          {{ weatherData.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;</span>
        </div>
      </div>
      <div class="col text-center">
        <img
          :src="`http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`"
        />
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">Project<br />Weather</div>
        <q-btn @click="getLocation" class="col" flat>
          <q-icon left size="3em" name="my_location" />
          <div>FIND MY LOCATION</div>
        </q-btn>
      </div>
    </template>
    <div class="col skyline"></div>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  watch: {
    search() {
      if (!this.search == "") {
        this.error = false;
      }
    },
  },
  name: "IndexPage",
  data() {
    return {
      search: "",
      weatherData: null,
      lat: null,
      long: null,
      apiUrl: "https://api.openweathermap.org/data/2.5/weather",
      apiKey: "4525edf5c30127b0d124cd4d4df4c98d",
      newApiKey: "93ce8aeb1384ba729d951d974ed40ba2",
      YOUR_ACCESS_KEY: "4d432bb292dc45d2b8fdab096599c908",
      error: false,
      error_message: "",
    };
  },
  computed: {
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith("n")) {
          return "bg-night";
        } else {
          return "bg-day";
        }
      }
    },
  },
  methods: {
    getLocation() {
      this.$q.loading.show();
      if (!this.$q.platform.is.electron) {
        this.$axios
          .get(
            `http://api.ipstack.com/check?access_key=${this.YOUR_ACCESS_KEY}`
          )
          .then((response) => {
            this.lat = response.data.latitude;
            this.long = response.data.longitude;
            this.getWeatherByCoords();
          });
      } else {
        navigator.geolocation.getCurrentPosition((position) => {
          console.log("position:", position);
          this.lat = position.coords.latitude;
          this.long = position.coords.longitude;
          this.getWeatherByCoords();
        });
      }
    },
    getWeatherByCoords() {
      this.$q.loading.show();
      this.$axios(
        `${this.apiUrl}?lat=${this.lat}&
	lon=${this.long}&
	appid=${this.apiKey}&units=metric`
      ).then((response) => {
        console.log(response);
        this.weatherData = response.data;
        this.$q.loading.hide();
      });
    },
    getWeatherBySearch() {
      if (this.search == "") {
        this.error = true;
        this.error_message = "Please enter the city name";
        console.log(this);
        return;
      }
      this.$q.loading.show();
      this.$axios(
        `${this.apiUrl}?q=${this.search}&
	appid=${this.apiKey}&units=metric`
      )
        .then((response) => {
          console.log("response", response);
          this.weatherData = response.data;
          this.$q.loading.hide();
        })
        .catch(function (error) {
          if (error.response) {
            // The request was made and the server responded with a status code
            // that falls out of the range of 2xx
            console.log(error.response.data.message);
          } else if (error.request) {
            // The request was made but no response was received
            // `error.request` is an instance of XMLHttpRequest in the browser and an instance of
            // http.ClientRequest in node.js
            console.log(error.request);
          } else {
            // Something happened in setting up the request that triggered an Error
            console.log("Error", error.message);
          }
        });
    },
  },
});
</script>
