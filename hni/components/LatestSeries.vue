<template>
  <v-container>
    <v-row>
      <v-col>
        <h5>
          <a href="/explore">{{ $t('landpage.latest_series_little') }}</a>
        </h5>
        <h1>{{ $t('landpage.latest_series') }}</h1>
      </v-col>
    </v-row>
    <v-row>
      <v-col
        v-for="(serie) in series"
        :key="serie._id"
        cols="6"
        lg="2"
        md="4"
        sm="6"
        xs="6"
      >
        <SerieCard
          :title="serie.attributes.title"
          :synopsis="serie.attributes.synopsis"
          :genres="serie.attributes.genres"
          :status="serie.attributes.statuses.data[0].attributes.name"
          :url="serie.attributes.h_id"
          :screenshot="'https://picsum.photos/720/1280'"
        />
      </v-col>
    </v-row>
  </v-container>
</template>

<script>

export default {
  data () {
    return {
      series: []
    }
  },
  mounted () {
    this.getLatestSeries()
  },
  methods: {
    async getLatestSeries () {
      const qs = require('qs')
      const query = qs.stringify({
        populate: [
          'statuses'
        ],
        sort: ['createdAt:desc'],
        pagination: {
          limit: 24
        }
      },
      {
        encodeValuesOnly: true
      })
      await fetch(`${process.env.API_STRAPI_ENDPOINT}series?${query}`)
        .then(res => res.json())
        .then((input) => {
          const res = input.data.map((serie) => {
            serie.attributes.genres = JSON.parse(serie.attributes.genres)
            return {
              ...serie
            }
          })
          this.series = res
        })
    }
  }
}
</script>
