<!-- Display App Bar w/ title at the top of the screen with Autocomplete Search functionality -->
<script setup>
  defineProps({
    items: {
      type: Array,
      required: true,
    },
    label: {
      type: String,
      default: 'Search...',
    },
    title: {
      type: String,
      default: '',
    },
  });

  const emit = defineEmits(['change']);
  const onChange = (value) => emit('change', value);
</script>

<template>
  <v-app-bar color="primary" :elevation="2">
    <v-app-bar-title>{{title}}</v-app-bar-title>
    <template v-slot:append>
      <v-autocomplete
        density="compact"
        min-width="200"
        rounded
        :label="label"
        :items="items"
        menu-icon=""
        @update:model-value="onChange"
        style="margin-top:20px; margin-right:1rem;"
        variant="solo"
      >
        <template v-slot:append-inner>
          <v-icon icon="mdi-magnify"></v-icon>
        </template>
      </v-autocomplete>
    </template>
  </v-app-bar>
</template>