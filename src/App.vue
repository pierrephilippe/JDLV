<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import TheNavigation from './components/TheNavigation.vue';
</script>

<template>
  <v-app>
    <TheNavigation />
    <v-main>
      <v-container>
        <RouterView />
      </v-container>
    </v-main>
  </v-app>
</template>

<style scoped>

</style>
