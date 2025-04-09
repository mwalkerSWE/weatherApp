<script setup>
  import { computed, ref, watchEffect } from 'vue';
  import SearchBar from './components/SearchBar.vue';
  import HourlyWeather from './components/HourlyWeather.vue';
  import cities from './assets/cities.json';

  // OpenWeather API key
  const WEATHER_APP_ID = '9170e0e85794088df319259526c55afd';

  // Transform list of cities into a format suitable for Autocomplete search
  const cityOptions = computed(() => {
    return cities.map((city, index) => ({
      title: `${city.city_name}, ${city.state_code} (${city.country_code})`,
      value: index,
    }));
  });

  // Default cities to display in the Tabs
  const defaultCities = computed(() => {
    const rio = cities.findIndex((city) => city.city_name.toUpperCase() === 'RIO DE JANEIRO');
    const beijing = cities.findIndex((city) => city.city_name.toUpperCase() === 'BEIJING');
    const losAngeles = cities.findIndex((city) => city.city_name.toUpperCase() === 'LOS ANGELES');

    return [
      { text: cities[rio].city_name, value: rio },
      { text: cities[beijing].city_name, value: beijing },
      { text: cities[losAngeles].city_name, value: losAngeles },
    ];
  });
  const tabs = ref(defaultCities.value);
  const cityIndex = ref(defaultCities.value[0].value);

  // On search, add the selected city to displayed tabs (if necessary) and select tab
  const onSearchChange = (index) => {
    const tab = tabs.value.find((tab) => tab.value === index);
    if (!tab) {
      tabs.value.push({ text: cities[index].city_name, value: index });
    }
    cityIndex.value = index;
  };

  // Remove the selected tab and select the first tab (if selected tab is removed)
  const onRemoveTab = (tab) => {
    tabs.value = tabs.value.filter((t) => t !== tab);
    if (cityIndex.value === tab.value) {
      cityIndex.value = tabs.value.length > 0 ? tabs.value[0].value : -1;
    }
  };

  const fetching = ref(false);
  const hourlyWeather = ref({});
  const fetchWeather = async () => {
    if (cityIndex.value < 0) {
      hourlyWeather.value = {};
      return;
    }

    fetching.value = true;
    const city = cities[cityIndex.value];
    try {
      const result = await fetch(`https://api.openweathermap.org/data/2.5/forecast?lat=${city.lat}&lon=${city.lon}&units=imperial&cnt=8&appid=${WEATHER_APP_ID}`);
      hourlyWeather.value = await result.json();
    } catch {
      hourlyWeather.value = {};
    }
    fetching.value = false;
  };

  // Fetch hourly weather data for the selected city
  watchEffect(fetchWeather);
</script>

<template>
  <v-app>
    <v-layout width="550px">
      <SearchBar 
        :items="cityOptions"
        label="City"
        title="Weather"
        @change="onSearchChange"
      />
      <v-main>
        <v-card>
          <v-tabs v-model="cityIndex">
            <v-tab
              v-for="tab in tabs"
              :key="tab.value"
              :text="tab.text"
              :value="tab.value"
            >
              <template v-slot:append>
                <v-icon opacity="0.5" icon="mdi-close" @click="(e) => { e.stopPropagation(); onRemoveTab(tab); }" />
              </template>
            </v-tab>
          </v-tabs>
          <v-card-text style="background-color:rgb(24,103,192)">
            <v-tabs-window v-model="cityIndex">
              <v-tabs-window-item
                v-for="tab in tabs"
                :key="tab.value"
                :value="tab.value"
              >
              <v-container>
                <v-row>
                  <v-col cols="12">
                    <v-card title="Next Hours" :loading="fetching">
                      <template v-slot:append>
                        <v-btn icon="mdi-refresh" title="Refresh" variant="text" @click="fetchWeather" />
                      </template>
                      <v-divider />
                      <v-card-text>
                        <span v-if="fetching">Loading...</span>
                        <v-slide-group v-else-if="hourlyWeather.list">
                          <v-slide-group-item v-for="weather in hourlyWeather.list">
                            <HourlyWeather :data="weather" />
                          </v-slide-group-item>
                        </v-slide-group>
                        <span v-else>No data available</span>
                      </v-card-text>
                    </v-card>
                  </v-col>
                </v-row>
                <v-row>
                  <v-col cols="12">
                    <v-card title="Next 5 days" :loading="fetching">
                      <v-divider />
                      <v-card-text>
                        No data available
                      </v-card-text>
                    </v-card>
                  </v-col>
                </v-row>
              </v-container>  
              </v-tabs-window-item>
            </v-tabs-window>
          </v-card-text>
        </v-card>
      </v-main>
    </v-layout>
  </v-app>
</template>

<style scoped>
  .v-tab-item--selected * {
    font-weight: bold;
  }

  .v-icon:hover {
    opacity: 1;
  }
</style>