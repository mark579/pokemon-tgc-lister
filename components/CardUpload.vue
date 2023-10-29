
<script lang="ts">
import { defineComponent } from 'vue'

interface CardUpload {
  valid: boolean;
  images: File[];
  imageRules: any[];
  previews: string[];
  matchLoading: boolean;
  match: Match | null;
}

interface Match {
  id: string;
  name: string;
  set: {
    id: string;
    name: string;
  }
  images: {
    small: string;
    large: string;
  }
}

export default defineComponent({
  data: (): CardUpload => ({
    valid: false,
    images: [],
    imageRules: [
      (value: File) => {
        if (value) return true

        return 'At least one image is required.'
      },
    ],
    previews: [],
    matchLoading: false,
    match: null,
  }),
  methods: {
    cardTitle() {
      if (this.match) {
        return `${this.match.name} - ${this.match.set.name}`
      }
      return 'No Match'
    },
    previewImage() {
      this.previews = this.images.map(image => { return URL.createObjectURL(image) })
      this.loadMatch()
    },
    clearPreview() {
      this.previews = []
      this.match = null
    },
    async loadMatch() {
      this.matchLoading = true;

      const headers = {
        "Accept": "application/json",
      };

      const formData = new FormData();
      formData.append('file', this.images[0]);

      const { data } = await useFetch<Match>('/api/match', {
        method: 'POST',
        headers: headers,
        body: formData,
      })

      if (data.value) {
        this.match = data.value;
      }
    }
  }
})
</script>

<template>
  <v-form action="/api/match" v-model="valid">
    <v-container>
      <v-row>
        <v-col cols="12">
          <v-file-input @click:clear="clearPreview" @change="previewImage" prepend-icon="mdi-camera" chips multiple
            accept="image/*" v-model="images" :rules="imageRules" label="Card Images"></v-file-input>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12">
          <div>
            <p> Select an image to load listing information</p>
          </div>
        </v-col>
      </v-row>
      <v-row>
        <template v-for="preview in previews">
          <v-col cols="4">
            <v-card elevation="2" max-height="250" title="">
              <div class="pa-6">
                <v-img :height="150" :src="preview"></v-img>
              </div>
            </v-card>
          </v-col>
        </template>
      </v-row>
      <v-row>
        <v-col cols="12">
          <template v-if="match">
            <v-card class="mx-auto" elevation="16" :title="cardTitle()">
              <div class="pa-6">
                <v-img :height="200" :src="match.images.small"></v-img>
              </div>
            </v-card>
          </template>
        </v-col>
      </v-row>
    </v-container>
  </v-form>
</template>