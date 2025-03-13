<template>
    <!-- Barre de navigation -->
    <v-app-bar app color="primary" dark>
      <v-toolbar-title>Le jeu de la vie QrCode</v-toolbar-title>
      <v-spacer></v-spacer>

      <!-- Liens de navigation -->
      <v-btn text to="/">Accueil</v-btn>
      <v-btn text to="/documentation">Documentation</v-btn>

    </v-app-bar>
</template>

<script setup lang="ts">

</script>

<style>
/* Ajoute un peu d'espace pour le contenu sous la barre de navigation */
.v-main {
  margin-top: 2rem;
}
</style>
