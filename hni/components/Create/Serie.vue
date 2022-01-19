<template>
  <v-container>
    <v-alert
      v-if="error || genreError"
      type="error"
      tile
      dismissible
      outlined
    >
      {{ errorMessage }}
    </v-alert>
    <v-row>
      <v-col cols="6">
        <v-card
          elevation
        >
          <v-card-title>
            Initial information
          </v-card-title>
          <v-container>
            <v-text-field
              v-model="serie.title"
              label="Hentai Title"
              required
              outlined
            />
            <v-text-field
              v-model="serie.title_english"
              label="Hentai English Title"
              outlined
            />
            <v-textarea
              v-model="serie.synopsis"
              name="synopsis"
              label="Synopsis"
              value="Como comenzo con el que tenia el peinado follador..."
              hint="Describe the Hentai"
              outlined
            />
            <v-autocomplete
              v-model="serie.genres"
              :items="genreList"
              label="Hentai Genres"
              item-text="name"
              item-value="id"
              multiple
              clearable
              deletable-chips
              outlined
              chips
              :return-object="false"
            />
            <v-select
              v-model="serie.serie_type"
              :items="serie_typeList"
              outlined
              item-text="name"
              item-value="id"
              filled
              label="Serie Type"
            />
            <v-select
              v-model="serie.status"
              :items="statusList"
              item-text="name"
              item-value="id"
              outlined
              label="Status"
            />
            <v-select
              v-model="serie.language"
              :items="languageList"
              item-text="name"
              item-value="id"
              outlined
              label="Language"
            />
            <v-switch
              v-model="serie.censorship"
              outlined
              label="Censorship"
            />
          </v-container>
        </v-card>
      </v-col>
      <v-col cols="6">
        <v-card
          elevation
        >
          <v-card-title>
            Image Settings
          </v-card-title>
          <v-container>
            <v-file-input
              ref="cover"
              show-size
              :error="error"
              label="Cover Image"
              :clearable="false"
              @change="coverSelected"
              @click="initialCoverClear"
            />
            <v-file-input
              ref="background_cover"
              show-size
              :error="error"
              label="Screenshot Image"
              :clearable="false"
              @change="background_coverSelected"
              @click="initialScreenshotClear"
            />
            <v-menu
              :close-on-content-click="false"
              :nudge-right="40"
              transition="scale-transition"
              offset-y
              min-width="290px"
            >
              <template #activator="{ on }">
                <v-text-field
                  v-model="serie.next_episode"
                  label="Next episode on"
                  prepend-icon="mdi-calendar"
                  readonly
                  v-on="on"
                />
              </template>
              <v-date-picker v-model="serie.next_episode" />
            </v-menu>
            <v-btn
              class="mr-4 blue darken-4"
              :loading="isSubmitting"
              :disabled="isSubmitting"
              large
              @click="createSerie"
            >
              submit
            </v-btn>
          </v-container>
          <v-container>
            <v-row>
              <v-col
                v-if="coverPreview"
                cols="6"
              >
                <v-img :src="coverPreview" />
              </v-col>
              <v-col
                v-if="screenshotPreview"
                cols="6"
              >
                <v-img :src="screenshotPreview" />
              </v-col>
            </v-row>
          </v-container>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: 'CreateSerie',
  data: () => ({
    serie: {
      title: '',
      title_english: '',
      synopsis: '',
      censorship: false,
      next_episode: '',
      visits: 0,
      featured: false,
      hasEpisodes: false,
      h_id: () => { return Math.floor(Math.random() * (666666 - 333333) + 333333) },
      genres: null,
      status: null,
      language: null,
      serie_type: null,
      cover: [],
      background_cover: []
    },
    genreList: [],
    categories: [],
    statusList: [],
    languageList: [],
    serie_typeList: [],
    genreError: false,
    coverPreview: '',
    screenshotPreview: '',
    error: false,
    errorMessage: '',
    alertBoxColor: '',
    isSubmitting: false
  }),
  mounted () {
    this.getGenres()
    this.getSerie_Types()
    this.getLanguageList()
    this.getStatuses()
  },
  methods: {
    async getGenres () {
      await fetch(`${process.env.API_STRAPI_ENDPOINT}genres`)
        .then(res => res.json())
        .then((input) => {
          const res = input.data.map((genre) => {
            genre.attributes.id = genre.id
            return {
              ...genre.attributes
            }
          })
          this.genreList = res
        })
    },
    async getSerie_Types () {
      await fetch(`${process.env.API_STRAPI_ENDPOINT}serie-types`)
        .then(res => res.json())
        .then((input) => {
          const res = input.data.map((serietype) => {
            serietype.attributes.id = serietype.id
            return {
              ...serietype.attributes
            }
          })
          this.serie_typeList = res
        })
    },
    async getLanguageList () {
      await fetch(`${process.env.API_STRAPI_ENDPOINT}languages`)
        .then(res => res.json())
        .then((input) => {
          const res = input.data.map((language) => {
            language.attributes.id = language.id
            return {
              ...language.attributes
            }
          })
          this.languageList = res
        })
    },
    async getStatuses () {
      await fetch(`${process.env.API_STRAPI_ENDPOINT}statuses`)
        .then(res => res.json())
        .then((input) => {
          const res = input.data.map((status) => {
            status.attributes.id = status.id
            return {
              ...status.attributes
            }
          })
          this.statusList = res
        })
    },
    async createSerie () {
      this.isSubmitting = !this.isSubmitting
      if (this.serie.cover < 1 || this.serie.background_cover < 1) {
        this.error = true
        this.errorMessage = 'You must define an cover and screenshot image.'
        this.isSubmitting = false
      }
      if (this.serie.genres < 1) {
        this.genreError = true
        this.errorMessage = 'You must select one or more genres.'
        this.isSubmitting = false
      }

      await fetch(`${process.env.API_STRAPI_ENDPOINT}series`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Bearer ${this.$store.state.auth.accessToken}`
        },
        body: JSON.stringify({
          data: this.serie.map((serie) => {
            serie.genres = JSON.stringify(serie.genres)
            return {
              ...serie
            }
          })
        })
      }).then((input) => {
        console.log(input)
        if (input.status === 200) {
          Promise.resolve(input.json())
            .then((res) => {
              console.log(res)
            })
        } else {
          this.loginFailed = true
        }
      }).catch((error) => {
        // eslint-disable-next-line no-console
        console.error(error)
      })

      // this.$apollo.mutate({
      //   mutation: gql`mutation ($input: SerieInput){
      //     createSerie(input: $input){
      //       success
      //       errors{
      //         path
      //         message
      //       }
      //     }
      //   }`,
      //   variables: {
      //     input: {
      //       title: this.serie.title,
      //       title_english: this.serie.title_english,
      //       synopsis: this.serie.synopsis,
      //       genres: this.serie.genres,
      //       status: this.serie.status,
      //       serie_type: this.serie.serie_type,
      //       next_episode: this.serie.next_episode,
      //       censorship: this.serie.censorship,
      //       cover: this.serie.cover,
      //       background_cover: this.serie.background_cover
      //     }
      //   }
      // }).then((input) => {
      //   if (input.data.createSerie.success) {
      //     this.$router.push({ path: '/panel/serie', query: { created: true } }, () => { window.location.reload(true) }, () => { window.location.reload(true) })
      //   } else {
      //     this.error = true
      //     this.alertBoxColor = 'red darken-4'
      //     this.errorMessage = input.data.createSerie.errors[0].message
      //     this.isSubmitting = false
      //   }
      // }).catch((error) => {
      //   // eslint-disable-next-line no-console
      //   this.error = true
      //   this.errorMessage = error
      // })
    },
    coverSelected (e) {
      this.serie.cover = []
      this.serie.cover.push(this.$refs.cover.$refs.input.files[0])
      this.serie.cover.push(this.serie.title)
      if (this.serie.cover !== undefined) {
        this.coverPreview = URL.createObjectURL(this.$refs.cover.$refs.input.files[0])
      }
    },
    background_coverSelected (e) {
      this.serie.background_cover.push(this.$refs.background_cover.$refs.input.files[0])
      this.serie.background_cover.push(this.serie.title)
      this.screenshotPreview = URL.createObjectURL(this.$refs.background_cover.$refs.input.files[0])
    },
    initialCoverClear () {
      this.serie.cover = []
      this.error = false
      this.isSubmitting = false
    },
    initialScreenshotClear () {
      this.serie.background_cover = []
      this.error = false
      this.isSubmitting = false
    }
  }
}
</script>
