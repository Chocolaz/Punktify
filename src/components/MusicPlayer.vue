<template>
  <link
    href="https://fonts.googleapis.com/icon?family=Material+Icons"
    rel="stylesheet"
  />

  <div
    class="music-player text-gray-200 p-4 rounded-lg shadow-lg max-w-sm mx-auto"
  >
    <div class="album-art mb-3 flex items-center justify-center">
      <img
        :src="currentSong.albumArt"
        :alt="currentSong.title + ' album art'"
        class="h-60 object-cover rounded-md shadow-md"
      />
    </div>
    <h2 class="text-xl font-bold mb-1 text-red-500">
      {{ currentSong.title }}
    </h2>
    <h3 class="text-base font-semibold mb-3 text-gray-400">
      {{ currentSong.artist }}
    </h3>

    <div
      class="progress-bar bg-gray-700 rounded-full h-1 mb-3"
      @click="seek"
      ref="progressBar"
    >
      <div
        :style="{ width: `${progress}%` }"
        class="bg-red-500 h-full rounded-full"
      ></div>
    </div>

    <div class="time flex justify-between text-xs mb-3">
      <span>{{ currentTime }}</span>
      <span>{{ duration }}</span>
    </div>

    <div class="controls relative flex items-center mb-4">
      <!-- Shuffle Button -->
      <button
        @click="toggleShuffle"
        :class="{ 'text-red-500': isShuffling, 'text-slate-500': !isShuffling }"
        class="absolute left-0 text-xl hover:text-red-500 transition-colors duration-300"
      >
        <i class="material-icons" style="font-size: 1.5rem; line-height: 1"
          >shuffle</i
        >
      </button>
      <!-- Center Controls -->
      <div class="center-controls flex items-center space-x-3 mx-auto">
        <button
          @click="previous"
          class="text-2xl hover:text-red-500 transition-colors duration-300"
        >
          <i class="material-icons" style="font-size: 1.5rem; line-height: 1"
            >skip_previous</i
          >
        </button>
        <button
          @click="playPause"
          class="hover:text-red-500 transition-colors duration-300"
        >
          <i
            class="material-icons custom-icon"
            style="font-size: 2.5rem; line-height: 1"
          >
            {{ isPlaying ? 'pause_circle_filled' : 'play_circle_filled' }}
          </i>
        </button>
        <button
          @click="next"
          class="text-2xl hover:text-red-500 transition-colors duration-300"
        >
          <i class="material-icons" style="font-size: 1.5rem; line-height: 1"
            >skip_next</i
          >
        </button>
      </div>
      <!-- Loop Button -->
      <button
        @click="toggleLoop"
        :class="{ 'text-red-500': isLooping, 'text-slate-500': !isLooping }"
        class="absolute right-0 text-2xl hover:text-red-500 transition-colors duration-300"
      >
        <i class="material-icons" style="font-size: 1.5rem; line-height: 1"
          >loop</i
        >
      </button>
    </div>

    <div class="volume-control flex items-center space-x-2 mb-3">
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
      isShuffling: false,
      currentTime: '0:00',
      duration: '0:00',
      progress: 0,
      volume: 1,
      playedSongs: []
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
      let nextSong

      if (this.isShuffling) {
        // Pick a random song that hasn't been played yet
        const unplayedSongs = this.songs.filter(
          (song) => !this.playedSongs.includes(song)
        )

        if (unplayedSongs.length === 0) {
          // All songs have been played; reset the playedSongs list
          this.playedSongs = []
          nextSong = this.songs[Math.floor(Math.random() * this.songs.length)]
        } else {
          nextSong =
            unplayedSongs[Math.floor(Math.random() * unplayedSongs.length)]
        }
      } else {
        const currentIndex = this.songs.findIndex(
          (song) => song.src === this.currentSong.src
        )
        nextSong =
          currentIndex < this.songs.length - 1
            ? this.songs[currentIndex + 1]
            : this.songs[0] // Go back to the first song if it's the last in the list
      }

      this.playedSongs.push(nextSong)
      this.$emit('update:currentSong', nextSong)
    },
    toggleLoop() {
      this.isLooping = !this.isLooping
    },
    toggleShuffle() {
      this.isShuffling = !this.isShuffling
      this.playedSongs = [] // Reset the played songs when shuffling is toggled
    },
    shuffleArray(array) {
      const shuffledArray = [...array]
      for (let i = shuffledArray.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1))
        ;[shuffledArray[i], shuffledArray[j]] = [
          shuffledArray[j],
          shuffledArray[i]
        ]
      }
      return shuffledArray
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
