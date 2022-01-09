<template>
  <v-carousel v-if="featuredSeries" v-model="model" :show-arrows="false">
    <v-carousel-item
      v-for="serie in featuredSeries"
      :key="serie.attributes.title"
      :src="`${serie.attributes.images.data[0].attributes.path}`"
      :href="`h/${serie.attributes.h_id}`"
      gradient="to top right, rgba(0,0,0,.8), rgba(0,0,0,.2)"
    >
      <v-row align="end" class="lightbox white--text pa-2 fill-height">
        <v-col class="mb-10">
          <v-container>
            <v-chip
              color="blue darken-2 white--text"
            >
              {{ serie.attributes.statuses.data[0].attributes.name }}
            </v-chip>
            <h1>{{ serie.attributes.title }}</h1>
            <p>{{ serie.attributes.synopsis }}</p>
            <v-chip-group
              active-class="blue darken-3"
              column
            >
              <v-chip
                v-for="genre in serie.attributes.genres"
                :key="genre.text"
                :href="`/explore?genre=${genre.url}`"
              >
                {{ genre.text }}
              </v-chip>
            </v-chip-group>
          </v-container>
        </v-col>
      </v-row>
    </v-carousel-item>
  </v-carousel>
</template>

<script>
export default {
  data () {
    return {
      featuredSeries: [],
      model: 0,
      CDN: process.env.CDN_URI
    }
  },
  mounted () {
    this.getFeaturedSeries()
  },
  methods: {
    async getFeaturedSeries () {
      const query = 'filters[featured][$eq]=true&populate[0]=images&populate[1]=statuses'
      await fetch(`${process.env.API_STRAPI_ENDPOINT}series?${query}`)
        .then(res => res.json())
        .then((series) => { // Genres came in a string, so we map the array to convert it to an object
          const res = series.data.map((serie) => {
            serie.attributes.genres = JSON.parse(serie.attributes.genres)
            return {
              ...serie
            }
          })
          this.featuredSeries = res
        })
    }
  }
}
</script>
