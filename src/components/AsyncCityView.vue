<template>
  <div
    v-if="route.query.preview"
    class="w-full p-4 text-center text-white bg-day-secondary"
  >
    <p>
      You are currently previewing this city, click the "+" icon
      to start tracking this city.
    </p>
  </div>
  <div class="flex flex-col items-center py-12 text-white">
    <h1 class="text-4xl">{{ route.params.name }}</h1>
    <p class="mt-2 text-sm">
      {{
        new Date(weatherData.currentTime).toLocaleDateString(
          "en-us",
          {
            weekday: "short",
            day: "2-digit",
            month: "long",
          }
        )
      }}
      {{
        new Date(weatherData.currentTime).toLocaleTimeString(
          "en-us",
          {
            timeStyle: "short",
          }
        )
      }}
    </p>
    <p class="mt-12 text-8xl">
      {{ Math.round(weatherData.current.temp) }}&deg;
    </p>
    <div class="mt-8 text-center">
      <p>
        Feels like
        {{ Math.round(weatherData.current.feels_like) }} &deg;
      </p>
      <p class="capitalize">
        {{ weatherData.current.weather[0].description }}
      </p>
    </div>
    <img
      class="w-[150px] h-auto"
      :src="
        `http://openweathermap.org/img/wn/${weatherData.current.weather[0].icon}@2x.png`
      "
      alt=""
    />
  </div>
  <hr
    class="border-white border-opacity-10 border-[1px] w-full"
  />
  <!-- Hourly Weather -->
  <div class="w-full max-w-screen-md py-12">
    <div class="mx-8 text-white">
      <p class="mb-3">Hourly Forecast</p>
      <div class="flex gap-10 overflow-x-scroll ">
        <div
          v-for="(hourData, index) in weatherData.hourly"
          :key="index"
          class="flex flex-col items-center gap-4"
        >
          <p class="text-lg whitespace-nowrap">
            {{
              new Date(hourData.currentTime).toLocaleTimeString(
                "en-us",
                {
                  hour: "numeric",
                }
              )
            }}
          </p>
          <p class="text-xl">
            {{ Math.round(hourData.temp) }}&deg;
          </p>
        </div>
      </div>
    </div>
  </div>
  <hr
    class="border-white border-opacity-10 border-[1px] w-full"
  />
  <!-- Weekly Weather -->
  <div class="w-full max-w-screen-md py-12">
    <div class="mx-8 text-white">
      <p>7 Day Forecast</p>
      <div v-for="day in weatherData.daily" :key="day.dt">
        <div class="flex items-center">
          <p class="flex-1">
            {{
              new Date(day.dt * 1000).toLocaleDateString(
                "en-us",
                {
                  weekday: "long",
                }
              )
            }}
          </p>
          <img
            class="w-[50px] h-auto"
            :src="
              `http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`
            "
            alt=""
          />
          <div class="flex justify-end flex-1 gap-2">
            <p>H: {{ Math.round(day.temp.max) }}</p>
            <p>L: {{ Math.round(day.temp.min) }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { useRoute } from "vue-router";
import { ref } from "vue";

const route = useRoute();
const getWeatherData = async () => {
  try {
    const weatherData = await axios.get(
      `https://api.openweathermap.org/data/2.5/onecall?lat=${
        route.query.lat
      }&lon=${route.query.lng}&exclude={part}&appid=${
        import.meta.env.VITE_OPEN_WEATHER_API
      }&units=metric`
    );

    // cal current date & time
    const localOffset = new Date().getTimezoneOffset() * 60000;
    const utc = weatherData.data.current.dt * 1000 + localOffset;
    weatherData.data.currentTime =
      utc + 1000 * weatherData.data.timezone_offset;

    // cal hourly weather offset
    weatherData.data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset;
      hour.currentTime =
        utc + 1000 * weatherData.data.timezone_offset;
    });

    // Flicker Delay
    await new Promise((res) => setTimeout(res, 1000));
    console.log(weatherData);
    return weatherData.data;
  } catch (err) {
    console.log(err);
  }
};

const weatherData = ref(await getWeatherData());
</script>
