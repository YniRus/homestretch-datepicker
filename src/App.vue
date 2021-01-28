<template>
  <v-app>
    <v-main>
      <v-container d-flex justify-center align-center>
        <v-sheet d-flex justify-center align-center flex-column>
          <v-date-picker v-model="current" multiple no-title scrollable>
          </v-date-picker>
          <v-flex d-flex mt-2 justify-end align-center flex-row>
            <v-progress-circular indeterminate v-if="loading"></v-progress-circular>
            <v-btn text v-on:click="reset">Reset</v-btn>
            <v-btn text color="primary" v-on:click="save">Save</v-btn>
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
  methods: {
    request: async function (url, options = {}) {
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
    load: async function () {
      const dates = await this.request('http://test.unit.homestretch.ch/')
      this.setDates(dates)
    },
    save: async function () {
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
    reset: function () {
      this.current = this.dates
    },
    setDates: function (dates) {
      if (Array.isArray(dates)) {
        this.current = this.dates = dates
      }
    },
    getChanges: function () {
      const changes = []
      const added = this.current.filter(date => !this.dates.includes(date))
      for (const date of added) {
        changes.push({ date: date, value: true })
      }
      const removed = this.dates.filter(date => !this.current.includes(date))
      for (const date of removed) {
        changes.push({ date: date, value: false })
      }
      return changes
    }
  },
  mounted: function () {
    this.load()
  }
}
</script>

<style>
  .container {
    height: 100%;
  }
</style>
