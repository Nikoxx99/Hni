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
        v-for="(serie) in Series"
        :key="serie._id"
        cols="6"
        lg="2"
        md="4"
        sm="6"
        xs="6"
      >
        <SerieCard
          :title="serie.title"
          :synopsis="serie.synopsis"
          :genres="serie.genres"
          :status="serie.status"
          :url="serie.episodes[0].urlName"
          :screenshot="serie.coverUrl"
        />
      </v-col>
    </v-row>
  </v-container>
</template>

<script>

export default {
  apollo: {
    Series: {
      query: `
        query ($limit: Int){
          Series(limit: $limit){
            _id
            episodes{
              episode_number
              urlName
            }
            genres{
              name
              url
            }
            title
            synopsis
            status
            coverUrl
          }
        }
      `,
      variables: {
        limit: 24
      }
    }
  },
  data () {
    return {
      Series: {
        episodes: {
          urlName: ''
        }
      }
    }
  }
}
</script>
