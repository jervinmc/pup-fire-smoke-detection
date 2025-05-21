<template>
    <div>
      <v-row>
        <v-col>
          <v-card class="pa-10" align="center">
            <div>{{ latestTemperature }}</div>
            <div class="text-h4">Sensor Value</div>
          </v-card>
        </v-col>
      </v-row>
  
      <!-- Data Table -->
      <v-card class="mt-5">
        <v-card-title>Live Logs</v-card-title>
        <v-data-table
          :headers="headers"
          :items="logs"
          :items-per-page="5"
          class="elevation-1"
        >
          <template v-slot:item.timestamp="{ item }">
            {{ new Date(item.timestamp).toLocaleString() }}
          </template>
        </v-data-table>
      </v-card>
      <div>
        <v-col cols="6">
          <div>
            <div class="text-h6 pb-1">
              Standard Atmospheric Condition = 2,000-3,000
            </div>
            <div class="text-h6 pb-1">
              Lighter Flame = 5,000 and Above
            </div>
            <div class="text-h6 pb-1">
              Burning Paper = 5,000 and Above
            </div>
            <div class="text-h6 pb-1">
              Liquefied Petroleum Gas (LPG) = 25,000 and Above
            </div>
            <div class="text-h6 pb-1">
              Butane Gas = 25,000 and Above
            </div>
          </div>
        </v-col>
      </div>
    </div>
  </template>
  
  <script>
  import Pusher from 'pusher-js';
  
  export default {
    created() {
      this.getAllData();
    },
    data() {
      return {
        logs: [], // To store incoming logs
        headers: [
          { text: 'Timestamp', value: 'timestamp' },
          { text: 'Sensor Value', value: 'logs' },
        ],
      };
    },
    computed: {
      latestTemperature() {
        return this.logs.length > 0 ? this.logs[0].logs : 'N/A';
      },
      latestStatus() {
        return this.logs.length > 0 ? this.logs[0].logs : 'N/A';
      },
    },
    methods: {
      async getAllData() {
        try {
          const data = await this.$axios.$get('/logs/');
          this.logs = data;
        } catch (err) {
          console.error('Error fetching logs:', err);
        }
      },
      reload() {
        this.getAllData();
      },
    },
    mounted() {
      Pusher.logToConsole = true;
  
      const pusher = new Pusher('29e101e910ebcfc920cf', {
        cluster: 'eu',
      });
  
      const channel = pusher.subscribe('my-channel');
  
      channel.bind('my-event', () => {
        this.reload();
      });
    },
  };
  </script>
  