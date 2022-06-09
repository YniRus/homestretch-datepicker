<template>
  <v-app>
    <v-main>
      <v-container d-flex justify-center align-center>
        <v-sheet d-flex justify-center align-center flex-column>
          <v-date-picker v-model="current" multiple no-title scrollable />
          <v-flex d-flex mt-2 justify-end align-center flex-row>
            <v-progress-circular v-if="loading" indeterminate />
            <v-btn text @click="reset">Reset</v-btn>
            <v-btn text color="primary" @click="save">Save</v-btn>
          </v-flex>
        </v-sheet>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: 'App',

  data () {
    return {
      loading: false,
      dates: [],
      current: []
    }
  },

  mounted () {
    this.load()
  },

  methods: {
    async request (url, options = {}) {
      try {
        this.loading = true
        const response = await fetch(url, options)
        this.loading = false
        if (!response.ok) {
          throw new Error(`Ошибка HTTP: ${response.status}`)
        }
        return await response.json()
      } catch (error) {
        this.loading = false
        throw new Error(`Ошибка fetch запроса: ${error.message}`)
      }
    },

    async load () {
      const dates = await this.request('http://test.unit.homestretch.ch/')
      this.setDates(dates)
    },

    async save () {
      const changes = this.getChanges()
      const dates = await this.request('http://test.unit.homestretch.ch/save', {
        headers: {
          Accept: 'application/json',
          'Content-Type': 'application/json'
        },
        method: 'POST',
        body: JSON.stringify(changes)
      })
      this.setDates(dates)
    },

    reset () {
      this.current = this.dates
    },

    setDates (dates) {
      if (Array.isArray(dates)) {
        this.current = this.dates = dates
      }
    },

    getChanges () {
      const added = this.current
        .filter(date => !this.dates.includes(date))
        .map(date => ({ date: date, value: true }))

      const removed = this.dates
        .filter(date => !this.current.includes(date))
        .map(date => ({ date: date, value: false }))

      return [...added, ...removed]
    }
  }
}
</script>

<style lang="scss">
  .container {
    height: 100%;
  }
</style>
