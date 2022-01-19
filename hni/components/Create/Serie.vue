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
            Initial information of {{ serie.h_id }}
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
              v-model="serie.genreList"
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
      title: '123',
      title_english: '123',
      synopsis: '123',
      censorship: false,
      next_episode: null,
      visits: 0,
      featured: false,
      hasEpisodes: false,
      h_id: null,
      genreList: null,
      status: null,
      language: null,
      serie_type: null
    },
    cover: [],
    background_cover: [],
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
    this.serie.h_id = Math.floor(Math.random() * (666666 - 333333) + 333333).toString()
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
      await this.uploadImageToStrapi()
      this.isSubmitting = !this.isSubmitting
      if (this.cover < 1 || this.background_cover < 1) {
        this.error = true
        this.errorMessage = 'You must define an cover and screenshot image.'
        this.isSubmitting = false
      }
      if (this.serie.genreList.length < 1) {
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
          data: this.serie
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
    },
    async uploadImageToStrapi () {
      const formData = new FormData()
      formData.append('files', this.cover, `${this.serie.title}_cover`)
      formData.append('files', this.background_cover, `${this.serie.title}_screenshot`)
      await fetch(`${process.env.API_STRAPI_ENDPOINT}upload`, {
        method: 'POST',
        headers: {
          Authorization: `Bearer ${this.$store.state.auth.accessToken}`
        },
        body: formData
      }).then((input) => {
        if (input.status === 200) {
          Promise.resolve(input.json())
            .then((res) => {
              return res
            })
        } else {
          throw new Error('Upload failed')
        }
      }).catch((error) => {
        // eslint-disable-next-line no-console
        console.error(error)
      })
    },
    coverSelected (e) {
      this.cover = this.$refs.cover.$refs.input.files[0]
      if (this.cover !== undefined) {
        this.coverPreview = URL.createObjectURL(this.$refs.cover.$refs.input.files[0])
      }
    },
    background_coverSelected (e) {
      this.background_cover = this.$refs.background_cover.$refs.input.files[0]
      this.screenshotPreview = URL.createObjectURL(this.$refs.background_cover.$refs.input.files[0])
    },
    initialCoverClear () {
      this.cover = []
      this.error = false
      this.isSubmitting = false
    },
    initialScreenshotClear () {
      this.background_cover = []
      this.error = false
      this.isSubmitting = false
    }
  }
}
</script>
