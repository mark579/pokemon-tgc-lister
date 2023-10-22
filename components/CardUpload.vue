<script>
export default {
  data: () => ({
    valid: false,
    images: [],
    imageRules: [
      value => {
        if (value) return true

        return 'At least one image is required.'
      },
    ],
    previews: "",
  }),
  methods: {
    previewImage() {
      this.previews= URL.createObjectURL(this.images[0])
    }
  }
}
</script>

<template>
  <v-form v-model="valid">
    <v-container>
      <v-file-input @change="previewImage" prepend-icon="mdi-camera" chips multiple accept="image/*" v-model="images" :rules="imageRules"
        label="Card Images"></v-file-input>
        <v-img :src="previews"></v-img>
      <v-btn type="submit" block class="mt-2">Submit</v-btn>
    </v-container>
  </v-form>
</template>