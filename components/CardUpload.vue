
<script lang="ts">
import { defineComponent } from 'vue'

interface CardUpload {
  valid: boolean;
  images: File[];
  imageRules: any[];
  previews: string[];
  matchLoading: boolean;
  match: Match | null;
  selectedPrice: number;
}

interface Match {
  id: string;
  name: string;
  number: string;
  set: {
    id: string;
    name: string;
    total: number;
    printedTotal: number;
  }
  images: {
    small: string;
    large: string;
  }
  tcgplayer: {
    url: string;
    updatedAt: string;
    prices: {
      holofoil: {
        low: number;
        mid: number;
        high: number;
        market: number;
        directLow: number;
      }
      reverseHolofoil: {
        low: number;
        mid: number;
        high: number;
        market: number;
        directLow: number;
      }
      normal: {
        low: number;
        mid: number;
        high: number;
        market: number;
        directLow: number;
      }
    }
  }
  cardmarket: {
    url: string;
    updatedAt: string;
    prices: {
      averageSellPrice: number;
      trendPrice: number;
      lowPrice: number;
      trendPriceExponential: number;
      germanProLow: number;
      suggestedPrice: number;
      reverseHoloSell: number;
      reverseHoloLow: number;
      lowPriceExponential: number;
      avg1: number;
      avg7: number;
      avg30: number;
      reverseHoloAvg1: number;
      reverseHoloAvg7: number;
      reverseHoloAvg30: number;
    }
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
    selectedPrice: 0,
  }),
  methods: {
    cardTitle() {
      if (this.match) {
        return `${this.match.name} - ${this.match.set.name} | ${this.match.number}/${this.match.set.printedTotal
          }`
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
    setSelectionPrice(price: number) {
      this.selectedPrice = price
    },
    listCard() {
      if (!this.match) {
        return;
      }

      const headers = {
        "Accept": "application/json",
      };

      const params = new URLSearchParams();
      params.append('cardId', this.match.id);
      params.append('price', this.selectedPrice.toString());

      const { data } = useFetch<Match>('/api/list?' + params.toString(), {
        method: 'POST',
        headers: headers,
      })

      if (data.value) {
        this.match = data.value;
      }
      this.matchLoading = false;
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
      this.matchLoading = false;
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
            <v-card variant="outlined" elevation="2" max-height="250" title="">
              <div class="pa-6">
                <v-img :height="150" :src="preview"></v-img>
              </div>
            </v-card>
          </v-col>
        </template>
      </v-row>
      <v-row>
        <v-col cols="12">
          <template v-if="match && !matchLoading">
            <v-card class="mx-auto" elevation="2" :title="cardTitle()">
              <div class="pa-6">
                <v-img :height="200" :src="match.images.small"></v-img>
              </div>
              <v-card-actions>
                <v-text-field v-model="selectedPrice" label="Listing Price" required hide-details></v-text-field>
                <v-btn @click="listCard()" variant="outlined" color="primary"> List Card </v-btn>
              </v-card-actions>
              <v-divider />
              <v-table>
                <thead>
                  <tr>
                    <th class="text-left">Market</th>
                    <th class="text-left">Type</th>
                    <th class="text-left">Low Price</th>
                    <th class="text-left">Suggested Price</th>
                    <th></th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-if="match.tcgplayer.prices.reverseHolofoil">
                    <td><a :href="match.tcgplayer.url">TCGPlayer</a></td>
                    <td>Reverse Holo</td>
                    <td>{{ match.tcgplayer.prices.reverseHolofoil.low }}</td>
                    <td>{{ match.tcgplayer.prices.reverseHolofoil.market }}</td>
                    <td><v-btn @click="setSelectionPrice(match.tcgplayer.prices.reverseHolofoil.market)">Select</v-btn>
                    </td>
                  </tr>
                  <tr v-if="match.tcgplayer.prices.holofoil">
                    <td><a :href="match.tcgplayer.url">TCGPlayer</a></td>
                    <td>Holofoil</td>
                    <td>{{ match.tcgplayer.prices.holofoil.low }}</td>
                    <td>{{ match.tcgplayer.prices.holofoil.market }}</td>
                    <td><v-btn @click="setSelectionPrice(match.tcgplayer.prices.holofoil.market)">Select</v-btn></td>
                  </tr>
                  <tr v-if="match.tcgplayer.prices.normal">
                    <td><a :href="match.tcgplayer.url">TCGPlayer</a></td>
                    <td>Normal</td>
                    <td>{{ match.tcgplayer.prices.normal.low }}</td>
                    <td>{{ match.tcgplayer.prices.normal.market }}</td>
                    <td><v-btn @click="setSelectionPrice(match.tcgplayer.prices.normal.market)">Select</v-btn></td>
                  </tr>
                </tbody>
              </v-table>
              <v-divider />

            </v-card>
          </template>
          <template v-if="matchLoading">
            <v-skeleton-loader type="heading, image"></v-skeleton-loader>
          </template>
        </v-col>
      </v-row>
    </v-container>
  </v-form>
</template>