<template>
  <link
    href="https://fonts.googleapis.com/icon?family=Material+Icons"
    rel="stylesheet"
  />

  <div
    class="music-player text-gray-200 p-6 rounded-lg shadow-2xl max-w-md mx-auto"
  >
    <div class="album-art mb-4">
      <img
        :src="currentSong.albumArt"
        :alt="currentSong.title + ' album art'"
        class="w-full h-64 object-cover rounded-md shadow-lg"
      />
    </div>
    <h2 class="text-2xl font-bold mb-2 text-red-500">
      {{ currentSong.title }}
    </h2>
    <h3 class="text-lg font-semibold mb-4 text-gray-400">
      {{ currentSong.artist }}
    </h3>

    <div
      class="progress-bar bg-gray-700 rounded-full h-2 mb-4"
      @click="seek"
      ref="progressBar"
    >
      <div
        :style="{ width: `${progress}%` }"
        class="bg-red-500 h-full rounded-full"
      ></div>
    </div>

    <div class="time flex justify-between text-sm mb-4">
      <span>{{ currentTime }}</span>
      <span>{{ duration }}</span>
    </div>

    <div class="controls relative flex items-center mb-6">
      <!-- Center Controls -->
      <div class="center-controls flex items-center space-x-4 mx-auto">
        <button
          @click="previous"
          class="text-3xl hover:text-red-500 transition-colors duration-300"
        >
          <i class="material-icons" style="font-size: 2.5rem; line-height: 1"
            >skip_previous</i
          >
        </button>
        <button
          @click="playPause"
          class="hover:text-red-500 transition-colors duration-300"
        >
          <i
            class="material-icons custom-icon"
            style="font-size: 3.5rem; line-height: 1"
          >
            {{ isPlaying ? 'pause_circle_filled' : 'play_circle_filled' }}
          </i>
        </button>
        <button
          @click="next"
          class="text-3xl hover:text-red-500 transition-colors duration-300"
        >
          <i class="material-icons" style="font-size: 2.5rem; line-height: 1"
            >skip_next</i
          >
        </button>
      </div>
      <!-- Loop Button -->
      <button
        @click="toggleLoop"
        :class="{ 'text-slate-500': isLooping, 'text-red-500': !isLooping }"
        class="absolute right-0 text-3xl hover:text-red-500 transition-colors duration-300"
      >
        <i class="material-icons" style="font-size: 2.5rem; line-height: 1"
          >loop</i
        >
      </button>
    </div>

    <div class="volume-control flex items-center space-x-2 mb-4">
      <span class="text-lg">🔈</span>
      <input
        type="range"
        min="0"
        max="1"
        step="0.1"
        v-model="volume"
        class="w-full"
      />
    </div>

    <audio
      ref="audio"
      :src="currentSong.src"
      @timeupdate="updateTime"
      @ended="handleEnd"
      @canplay="setDuration"
    ></audio>
  </div>
</template>

<script>
export default {
  props: {
    currentSong: {
      type: Object,
      required: true
    },
    songs: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      isPlaying: false,
      isLooping: false,
      currentTime: '0:00',
      duration: '0:00',
      progress: 0,
      volume: 1
    }
  },
  watch: {
    currentSong: {
      handler(newSong) {
        this.$nextTick(() => {
          this.stop()
          this.isPlaying = false
          const audio = this.$refs.audio
          if (audio) {
            audio.src = newSong.src
            audio.load()
            audio.addEventListener(
              'canplaythrough',
              () => {
                audio
                  .play()
                  .then(() => {
                    this.isPlaying = true
                  })
                  .catch((error) => {
                    console.error('Failed to play the audio:', error)
                  })
              },
              { once: true }
            )
          }
        })
      },
      immediate: true
    },
    volume(newVolume) {
      if (this.$refs.audio) {
        this.$refs.audio.volume = newVolume
      }
    }
  },
  methods: {
    playPause() {
      const audio = this.$refs.audio
      if (audio) {
        if (this.isPlaying) {
          audio.pause()
        } else {
          audio.play()
        }
        this.isPlaying = !this.isPlaying
      }
    },
    stop() {
      const audio = this.$refs.audio
      if (audio) {
        audio.pause()
        audio.currentTime = 0
        this.isPlaying = false
      }
    },
    handleEnd() {
      if (this.isLooping) {
        this.next()
      } else {
        this.isPlaying = false
      }
    },
    updateTime() {
      const audio = this.$refs.audio
      if (audio) {
        this.currentTime = this.formatTime(audio.currentTime)
        this.progress = (audio.currentTime / audio.duration) * 100
      }
    },
    setDuration() {
      const audio = this.$refs.audio
      if (audio) {
        this.duration = this.formatTime(audio.duration)
      }
    },
    formatTime(seconds) {
      const mins = Math.floor(seconds / 60)
      const secs = Math.floor(seconds % 60)
      return `${mins}:${secs.toString().padStart(2, '0')}`
    },
    previous() {
      const currentIndex = this.songs.findIndex(
        (song) => song.src === this.currentSong.src
      )
      if (currentIndex > 0) {
        this.$emit('update:currentSong', this.songs[currentIndex - 1])
      }
    },
    next() {
      const currentIndex = this.songs.findIndex(
        (song) => song.src === this.currentSong.src
      )
      if (currentIndex < this.songs.length - 1) {
        this.$emit('update:currentSong', this.songs[currentIndex + 1])
      } else if (this.isLooping) {
        this.$emit('update:currentSong', this.songs[0])
      }
    },
    toggleLoop() {
      this.isLooping = !this.isLooping
    },
    seek(event) {
      const progressBar = this.$refs.progressBar
      const audio = this.$refs.audio
      const rect = progressBar.getBoundingClientRect()
      const offsetX = event.clientX - rect.left
      const width = rect.width
      const percentage = offsetX / width
      const seekTime = audio.duration * percentage
      audio.currentTime = seekTime
      this.updateTime()
    }
  }
}
</script>

<style>
.music-player {
  background: #1a1a1a;
}
.controls {
  position: relative;
}
.center-controls {
  margin: 0 auto;
}
</style>
