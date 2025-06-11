<template>
    <v-container fluid class="pa-10">
        <!-- Video Stream and Clips -->
        <v-row class="mb-8" align="stretch">
            <!-- Live Stream -->
            <v-col cols="12" md="6">
                <v-fade-transition mode="in-out">
                    <v-card color="#e3f2fd" elevation="5" class="pa-4 rounded-lg" key="live-stream-card">
                        <v-card-title class="d-flex justify-space-between align-center">
                            <span class="font-weight-bold">🎥 Live Stream</span>
                            <v-btn small color="primary" @click="refreshVideo" elevation="2">
                                Reset Stream
                            </v-btn>
                        </v-card-title>
                        <v-card-text>
                            <video :src="videoSrc" controls autoplay muted loop
                                style="width: 100%; height: 300px; object-fit: cover; border-radius: 8px;"></video>
                        </v-card-text>
                    </v-card>
                </v-fade-transition>
            </v-col>

            <!-- Video Clips -->
            <v-col cols="12" md="6">
                <v-slide-y-transition group>
                    <v-card color="#fff3e0" elevation="5" class="pa-4 rounded-lg" key="video-clips-card">
                        <v-card-title class="font-weight-bold">📂 Recent Video Clips</v-card-title>
                        <v-card-text style="max-height: 300px; overflow-y: auto;">
                            <v-list dense nav>
                                <v-list-item v-for="clip in videoLogs" :key="clip.id"
                                    @click="selectVideo(clip.video_link)" class="rounded hover-grey">
                                    <v-list-item-content>
                                        <v-list-item-title class="text-truncate">
                                            {{ clip.video_link }}
                                        </v-list-item-title>
                                        <v-list-item-subtitle>
                                            {{ new Date(clip.timestamp).toLocaleString() }}
                                        </v-list-item-subtitle>
                                    </v-list-item-content>
                                </v-list-item>
                            </v-list>
                        </v-card-text>
                    </v-card>
                </v-slide-y-transition>
            </v-col>
        </v-row>

        <!-- Sensor Cards -->
        <v-row class="mb-8">
            <v-col cols="12" md="6">
                <v-scale-transition>
                    <v-card color="#e8f5e9" elevation="4" class="pa-4 rounded-lg sensor-card"
                        :key="'temperature-' + temperature">
                        <v-card-title class="font-weight-bold">🌡️ Temperature</v-card-title>
                        <v-card-text class="text-h4 font-weight-bold text-center pulse">
                            {{ temperature }}°C
                        </v-card-text>
                    </v-card>
                </v-scale-transition>
            </v-col>
            <v-col cols="12" md="6">
                <v-scale-transition>
                    <v-card color="#fce4ec" elevation="4" class="pa-4 rounded-lg sensor-card"
                        :key="'sensor-' + sensorValue">
                        <v-card-title class="font-weight-bold">🔬 Sensor Value</v-card-title>
                        <v-card-text class="text-h4 font-weight-bold text-center pulse">
                            {{ sensorValue }}
                        </v-card-text>
                    </v-card>
                </v-scale-transition>
            </v-col>
        </v-row>

        <!-- Air Quality -->
        <v-row>
            <v-col cols="12">
                <v-fade-transition>
                    <v-card color="#ede7f6" elevation="5" class="pa-4 rounded-lg" key="air-quality-card">
                        <v-card-title class="font-weight-bold">🌬️ Air Quality</v-card-title>
                        <v-card-text>
                            <v-slider v-model="airQuality" :min="0" :max="500" step="1" ticks tick-size="4"
                                :color="airQualityColor" track-color="grey lighten-1" thumb-label="always"></v-slider>
                            <div class="d-flex justify-space-between mt-3 air-labels">
                                <span>Good</span>
                                <span>Moderate</span>
                                <span>Poor</span>
                                <span>Unhealthy</span>
                                <span>Severe</span>
                                <span>Hazardous</span>
                            </div>
                        </v-card-text>
                    </v-card>
                </v-fade-transition>
            </v-col>
        </v-row>
    </v-container>
</template>

<script>
import Pusher from 'pusher-js'

export default {
    data() {
        return {
            temperature: 'N/A',
            sensorValue: 'N/A',
            airQuality: 120,
            videoBaseUrl: 'http://fire-smoke-detection-pup.s3.ap-southeast-2.amazonaws.com/',
            videoSrc: '',
            videoLogs: [],
            previousVideoLogsLength: 0  // Track previous logs count
        }
    },
    computed: {
        airQualityColor() {
            const value = this.airQuality
            if (value <= 50) return 'green'
            if (value <= 100) return 'yellow'
            if (value <= 150) return 'orange'
            if (value <= 200) return 'red'
            if (value <= 300) return 'purple'
            return 'brown'
        }
    },
    methods: {
        async getAllData() {
            try {
                const response = await this.$axios.get('/logs/')
                if (response.data?.length) {
                    const latest = response.data[0]
                    this.sensorValue = latest.logs ?? 'N/A'
                    this.temperature = latest.temperature ?? 'N/A'
                    this.airQuality = latest.airQuality ?? 120
                }
            } catch (err) {
                console.error('Error fetching logs:', err)
            }
        },
        async loadVideoLogs() {
            try {
                const response = await this.$axios.get('/videologs/')
                const newLogs = response.data || []

                // Check for new video logs
                const isIncremented = newLogs.length > this.previousVideoLogsLength

                this.videoLogs = newLogs
                this.previousVideoLogsLength = newLogs.length

                // Update video only if it's the first time OR new log has been added
                if (isIncremented || !this.videoSrc) {
                    const latestClip = this.videoLogs[0]?.video_link
                    if (latestClip) {
                        this.videoSrc = `${latestClip}?t=${Date.now()}`
                    }
                }
            } catch (err) {
                console.error('Error fetching video logs:', err)
            }
        },
        refreshVideo() {
            if (this.videoLogs.length > 0) {
                const latestClip = this.videoLogs[0].video_link
                this.videoSrc = `${latestClip}?t=${Date.now()}`
            } else {
                console.warn('No video logs available to reset to.')
            }

        },

        selectVideo(filename) {
            const timestamp = Date.now()
            this.videoSrc = `${filename}?t=${timestamp}`
        }
    },
    mounted() {
        this.getAllData()
        this.loadVideoLogs()

        const pusher = new Pusher('29e101e910ebcfc920cf', { cluster: 'eu' })
        const channel = pusher.subscribe('my-channel')
        channel.bind('my-event', () => {
            this.getAllData()
            this.loadVideoLogs()
        })
    }
}
</script>



<style scoped>
.hover-grey:hover {
    background-color: #f1f1f1 !important;
    cursor: pointer;
    transition: background-color 0.2s ease-in-out;
}

.air-labels>span {
    width: 16%;
    text-align: center;
    font-size: 0.85rem;
}

.pulse {
    animation: pulse 1s ease-in-out;
}

@keyframes pulse {
    0% {
        transform: scale(1);
        opacity: 1;
    }

    50% {
        transform: scale(1.05);
        opacity: 0.8;
    }

    100% {
        transform: scale(1);
        opacity: 1;
    }
}
</style>