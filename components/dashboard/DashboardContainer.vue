<template>
  <v-container fluid class="pa-10">
    <!-- Top Row: Sensor Value + Reference Thresholds -->
    <v-row class="mb-8" align="stretch">
      <!-- Sensor Value -->
      <v-col cols="12" md="6">
        <v-card color="#e8f5e9" elevation="6" class="pa-6 rounded-lg text-center">
          <v-icon size="48" color="green darken-2">mdi-thermometer</v-icon>
          <div class="text-h3 font-weight-bold mt-3">{{ latestTemperature }}</div>
          <div class="text-subtitle-1 mt-1">Sensor Value</div>
        </v-card>
      </v-col>

      <!-- Reference Thresholds -->
      <v-col cols="12" md="6">
        <v-card class="pa-4 elevation-6 rounded-lg" color="#fff3e0">
          <v-card-title class="font-weight-bold">🔥 Reference Thresholds</v-card-title>
          <v-card-text>
            <v-list dense>
              <v-list-item>
                <v-list-item-icon><v-icon color="blue darken-1">mdi-gas-cylinder</v-icon></v-list-item-icon>
                <v-list-item-content>
                  <v-list-item-title>Standard Atmospheric Condition: 2,000–3,000</v-list-item-title>
                </v-list-item-content>
              </v-list-item>
              <v-list-item>
                <v-list-item-icon><v-icon color="orange">mdi-fire</v-icon></v-list-item-icon>
                <v-list-item-content>
                  <v-list-item-title>Lighter Flame: 5,000 and Above</v-list-item-title>
                </v-list-item-content>
              </v-list-item>
              <v-list-item>
                <v-list-item-icon><v-icon color="orange">mdi-fire</v-icon></v-list-item-icon>
                <v-list-item-content>
                  <v-list-item-title>Burning Paper: 5,000 and Above</v-list-item-title>
                </v-list-item-content>
              </v-list-item>
              <v-list-item>
                <v-list-item-icon><v-icon color="deep-orange">mdi-fire</v-icon></v-list-item-icon>
                <v-list-item-content>
                  <v-list-item-title>LPG: 25,000 and Above</v-list-item-title>
                </v-list-item-content>
              </v-list-item>
              <v-list-item>
                <v-list-item-icon><v-icon color="deep-orange">mdi-fire</v-icon></v-list-item-icon>
                <v-list-item-content>
                  <v-list-item-title>Butane Gas: 25,000 and Above</v-list-item-title>
                </v-list-item-content>
              </v-list-item>
            </v-list>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <!-- Live Logs Table -->
    <v-row>
      <v-col cols="12">
        <v-card elevation="4" class="rounded-lg">
          <v-card-title class="font-weight-bold primary--text">
            📋 Live Sensor Logs
          </v-card-title>
          <v-data-table
            :headers="headers"
            :items="logs"
            :items-per-page="5"
            class="elevation-1"
            dense
            hide-default-footer
            style="border-radius: 0 0 12px 12px;"
          >
            <template v-slot:item.timestamp="{ item }">
              {{ new Date(item.timestamp).toLocaleString() }}
            </template>
          </v-data-table>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>


<script>
import Pusher from 'pusher-js'

export default {
  data() {
    return {
      logs: [],
      headers: [
        { text: 'Timestamp', value: 'timestamp', align: 'start' },
        { text: 'Sensor Value', value: 'logs' }
      ]
    }
  },
  computed: {
    latestTemperature() {
      return this.logs.length > 0 ? this.logs[0].logs : 'N/A'
    }
  },
  methods: {
    async getAllData() {
      try {
        const data = await this.$axios.$get('/logs/')
        this.logs = data
      } catch (err) {
        console.error('Error fetching logs:', err)
      }
    },
    reload() {
      this.getAllData()
    }
  },
  created() {
    this.getAllData()
  },
  mounted() {
    const pusher = new Pusher('29e101e910ebcfc920cf', { cluster: 'eu' })
    const channel = pusher.subscribe('my-channel')
    channel.bind('my-event', () => this.reload())
  }
}
</script>

<style scoped>
.text-subtitle-1 {
  font-size: 1.1rem;
}
</style>
