<template>
  <v-carousel v-if="featuredSeries" v-model="model" :show-arrows="false">
    <v-carousel-item
      v-for="serie in featuredSeries"
      :key="serie.title"
      :src="`${CDN}/screenshot/${serie.background_coverUrl}`"
      :href="'/episode/' + serie.episodes[0].urlName + '/' + 1"
      gradient="to top right, rgba(0,0,0,.8), rgba(0,0,0,.2)"
    >
      <v-row align="end" class="lightbox white--text pa-2 fill-height">
        <v-col class="mb-10">
          <v-container>
            <v-chip
              color="blue darken-2 white--text"
            >
              {{ serie.status }}
            </v-chip>
            <h1>{{ serie.title }}</h1>
            <p>{{ serie.synopsis }}</p>
            <v-chip-group
              active-class="blue darken-3"
              column
            >
              <v-chip
                v-for="genre in serie.genres"
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
      await this.$strapi.graphql({
        query: `query {
          series(limit: 20, where: { featured: true }) {
            title
            title_english
            synopsis
            genres {
              name
              url
            }
            airing
            types{
              name
            }
            censorship
            episodes {
              number
            }
            nextepisode
            visits
          }
        }`
      })
    }
  }
}
</script>
