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

    <div class="title-container pirata-one-regular">
      <h2
        v-if="currentSong.title.length <= 20"
        class="title flex items-start text-2xl font-bold mb-1 text-red-500"
      >
        {{ currentSong.title }}
      </h2>

      <h2
        v-else-if="
          currentSong.title.length > 20 && currentSong.title.length <= 30
        "
        class="title flex items-start text-xl font-semibold mb-1 text-red-500"
      >
        {{ currentSong.title }}
      </h2>

      <h2
        v-else
        class="title flex items-start text-lg font-semibold mb-1 text-red-500"
      >
        {{ currentSong.title }}
      </h2>
    </div>

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

<script setup>
import { ref, watch, nextTick } from 'vue'

const emit = defineEmits(['update:currentSong'])

const props = defineProps({
  currentSong: Object,
  songs: Array
})

const isPlaying = ref(false)
const isLooping = ref(false)
const isShuffling = ref(false)
const currentTime = ref('0:00')
const duration = ref('0:00')
const progress = ref(0)
const volume = ref(1)
const playedSongs = ref([])

watch(
  () => props.currentSong,
  async (newSong) => {
    await nextTick()
    stop()
    isPlaying.value = false
    const audio = document.querySelector('audio')
    if (audio) {
      audio.src = newSong.src
      audio.load()
      audio.addEventListener(
        'canplaythrough',
        () => {
          audio
            .play()
            .then(() => {
              isPlaying.value = true
            })
            .catch((error) => {
              console.error('Failed to play the audio:', error)
            })
        },
        { once: true }
      )
    }
  },
  { immediate: true }
)

// Watch for volume changes
watch(volume, (newVolume) => {
  const audio = document.querySelector('audio')
  if (audio) {
    audio.volume = newVolume
  }
})

// Methods
const playPause = () => {
  const audio = document.querySelector('audio')
  if (audio) {
    if (isPlaying.value) {
      audio.pause()
    } else {
      audio.play()
    }
    isPlaying.value = !isPlaying.value
  }
}

const stop = () => {
  const audio = document.querySelector('audio')
  if (audio) {
    audio.pause()
    audio.currentTime = 0
    isPlaying.value = false
  }
}

const handleEnd = () => {
  const currentIndex = props.songs.findIndex(
    (song) => song.src === props.currentSong.src
  )

  if (currentIndex === props.songs.length - 1) {
    if (isLooping.value) {
      emit('update:currentSong', props.songs[0])
      isPlaying.value = false
    } else {
      isPlaying.value = false
    }
  } else {
    next()
  }
}

const updateTime = () => {
  const audio = document.querySelector('audio')
  if (audio) {
    currentTime.value = formatTime(audio.currentTime)
    progress.value = (audio.currentTime / audio.duration) * 100
  }
}

const setDuration = () => {
  const audio = document.querySelector('audio')
  if (audio) {
    duration.value = formatTime(audio.duration)
  }
}

const formatTime = (seconds) => {
  const mins = Math.floor(seconds / 60)
  const secs = Math.floor(seconds % 60)
  return `${mins}:${secs.toString().padStart(2, '0')}`
}

const previous = () => {
  const currentIndex = props.songs.findIndex(
    (song) => song.src === props.currentSong.src
  )

  if (currentIndex === 0) {
    emit('update:currentSong', props.songs[props.songs.length - 1])
  } else {
    emit('update:currentSong', props.songs[currentIndex - 1])
  }
}

const next = () => {
  let nextSong

  if (isShuffling.value) {
    const unplayedSongs = props.songs.filter(
      (song) => !playedSongs.value.includes(song)
    )

    if (unplayedSongs.length === 0) {
      playedSongs.value = []
      nextSong = props.songs[Math.floor(Math.random() * props.songs.length)]
    } else {
      nextSong = unplayedSongs[Math.floor(Math.random() * unplayedSongs.length)]
    }
  } else {
    const currentIndex = props.songs.findIndex(
      (song) => song.src === props.currentSong.src
    )
    nextSong =
      currentIndex < props.songs.length - 1
        ? props.songs[currentIndex + 1]
        : props.songs[0]
  }

  playedSongs.value.push(nextSong)
  emit('update:currentSong', nextSong)
}

const toggleLoop = () => {
  isLooping.value = !isLooping.value
}

const toggleShuffle = () => {
  isShuffling.value = !isShuffling.value
  playedSongs.value = []
}

const seek = (event) => {
  const progressBar = document.querySelector('.progress-bar')
  const audio = document.querySelector('audio')
  const rect = progressBar.getBoundingClientRect()
  const offsetX = event.clientX - rect.left
  const width = rect.width
  const percentage = offsetX / width
  const seekTime = audio.duration * percentage
  audio.currentTime = seekTime
  updateTime()
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

.music-player h2 {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.progress-bar,
.volume-control input {
  cursor: pointer;
}

.title-container {
  position: relative;
  height: 2.5rem;
  overflow: hidden;
}

.title {
  position: absolute;
  width: 100%;
  text-align: center;
  line-height: 2.5rem;
}
</style>
