<template>
  <v-container fluid>
    <v-row
      v-for="(episode, index) in episodes"
      :key="index"
      class="episode"
    >
      <v-icon x-large>
        mdi-play
      </v-icon>
      <h3 class="mt-2">
        <a :href="`/h/${serie.h_id}/${episode.episode_number}`">
          {{ serie.title }} episode {{ episode.episode_number }}
        </a>
      </h3>
    </v-row>
  </v-container>
</template>
<script>
export default {
  name: 'SerieEpisodeList',
  props: {
    serie: {
      type: Object,
      default: null
    },
    episodes: {
      type: Array,
      default: () => []
    }
  },
  data () {
    return {
    }
  }
}
</script>
<style scoped>
  .episode {
    border-bottom: 2px solid rgb(255 255 240 / 33%);
    padding:10px;
  }
  .episode > h3 > a {
    color: white!important;
  }
  .episode:hover {
    background-color: rgb(255 255 240 / 33%);
    cursor:pointer;
  }
</style>
