<template>
  <NuxtLayout>
    <v-app>
      <v-app-bar color="primary" elevation="2"><v-app-bar-title>Pokemon TGC Lister </v-app-bar-title>
      </v-app-bar>
      <v-main>
        <v-container>
          <v-row>
            <v-col cols="12" >
              <v-sheet width="100%">
                <p class="text-md-center">To get started select a Image of a Pokemon card. You can select multiple images that you'd like to use for a listing. Only the first image will be used to match. </p>
              </v-sheet>
            </v-col>
            <v-col cols="12" >
              <v-sheet width="100%">
                <CardUpload />
              </v-sheet>
            </v-col>
          </v-row>
        </v-container>
      </v-main>
      <v-footer color="secondary">
        {{ new Date().getFullYear() }} — <strong>MadSunshineCreations</strong>
      </v-footer>
    </v-app>
  </NuxtLayout>
</template>