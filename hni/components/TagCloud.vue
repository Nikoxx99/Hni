<template>
  <v-container>
    <v-chip-group
      v-if="genres"
      style="border-bottom: 1px solid rgb(84 84 84);padding-bottom: 10px;padding-top: 5px;"
    >
      <v-chip v-for="tag in genres" :key="tag.text" :href="`/explore?genre=${tag.url}`">
        {{ tag.text }}
      </v-chip>
    </v-chip-group>
  </v-container>
</template>
<script>
export default {
  name: 'TagCloud',
  data: () => ({
    genres: [],
    h: 'e'
  }),
  mounted () {
    this.getTags()
  },
  methods: {
    async getTags () {
      await this.$strapi.graphql({
        query: `query {
          genres(limit: 20) {
            name
            url
          }
        }`
      })
    }
  }
}
</script>
<style>
.v-slide-group__prev {
display: none !important;
}
</style>
