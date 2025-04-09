<!-- Display data returned from weather service as a single-column Card -->
<script setup>
  import { computed } from 'vue';
  
  const WEATHER_IMG_URL =  'https://openweathermap.org/img/wn';

  const props = defineProps({
    data: {
      type: Object,
      required: true,
    },
  });

  // Convert date string to local time string
  const time = computed(() => new Date(props.data.dt_txt).toLocaleTimeString('en-US', { hour: 'numeric', minute: '2-digit' }));
</script>

<template>
  <v-card>
    <v-container>
      <v-row align="center">
        <v-col class="bold center-text" cols="12">
          {{Math.round(data.main.temp)}}°
        </v-col>
      </v-row>
      <v-row align="center">
        <v-col class="center-text light-text" cols="12">
          {{data.main.humidity}}%
        </v-col>
      </v-row>
      <v-row align="center">
        <v-col class="center-text" cols="12">
          <v-img width="50px" :src="`${WEATHER_IMG_URL}/${data.weather[0].icon}.png`" :title="data.weather[0].description" />
        </v-col>
      </v-row>
      <v-row align="center">
        <v-col class="center-text grey-text" cols="12">
          {{time}}
        </v-col>
      </v-row>
    </v-container>
  </v-card>
</template>

<style scoped>
  .bold {
    font-weight: bold;
  }
  
  .center-text {
    text-align: center;
  }

  .light-text {
    color: darkturquoise;
  }

  .grey-text {
    color: grey;
  }
</style>