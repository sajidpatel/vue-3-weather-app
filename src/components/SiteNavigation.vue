<template>
  <header class="sticky top-0 shadow-lg bg-day-primary">
    <nav
      class="container flex flex-col items-center gap-4 py-6 text-white sm:flex-row"
    >
      <div class="flex items-center flex-1 gap-3">
        <i class="text-2xl fa-solid fa-temperature-three-quarters"></i>
        <RouterLink :to="{ name: 'home' }">
          <h1 class="text-2xl">The Local Weather</h1>
        </RouterLink>
      </div>

      <div class="flex gap-3">
        <i
          @click="toggleModal"
          class="text-xl duration-150 cursor-pointer fa-solid fa-circle-info hover:text-day-secondary"
        ></i>

        <!-- Add City -->
        <i
          v-if="route.query.preview"
          @click="addCity"
          class="text-xl duration-150 cursor-pointer fa-solid fa-plus hover:text-day-secondary"
        >
        </i>
      </div>

      <BaseModal
        :modalActive="modalActive"
        @close-modal="toggleModal"
      >
        <!-- Modal Slot Content -->
        <div class="text-black">
          <h1 class="mb-1 text-2xl">About:</h1>
          <p class="mb-4">
            The Local Weather allows you to track the current and
            future weather of cities of your choosing.
          </p>
          <h2 class="text-2xl">How it works:</h2>
          <ol class="mb-4 list-decimal list-inside">
            <li>
              Search for your city by entering the name into the
              search bar.
            </li>
            <li>
              Select a city within the results, this will take
              you to the current weather for your selection.
            </li>
            <li>
              Track the city by clicking on the "+" icon in the
              top right. This will save the city to view at a
              later time on the home page.
            </li>
          </ol>

          <h2 class="text-2xl">Removing a city</h2>
          <p>
            If you no longer wish to track a city, simply click
            on the "" icon within the home page. This will enable
            a edit mode. A "" icon should now be visible and
            allow you to delete a city.
          </p>
        </div>
      </BaseModal>
    </nav>
  </header>
</template>

<script setup>
import { ref } from "vue";
import { RouterLink, useRoute, useRouter } from "vue-router";
import { uid } from "uid";
import BaseModal from "./BaseModal.vue";

const route = useRoute();
const router = useRouter();
const savedCities = ref([]);

const addCity = async (searchResult) => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(
      localStorage.getItem("savedCities")
    );
  }

  const locationObj = {
    id: uid(),
    state: route.params.state,
    city: route.params.city,
    coords: {
      lat: route.query.lat,
      lng: route.query.lng,
    },
  };

  savedCities.value.push(locationObj);
  localStorage.setItem(
    "savedCities",
    JSON.stringify(savedCities.value)
  );

  // get query
  let query = Object.assign({}, route.query);
  delete query.preview;
  router.replace({ query });
};

const modalActive = ref(null);
const toggleModal = () => {
  modalActive.value = !modalActive.value;
};
</script>
