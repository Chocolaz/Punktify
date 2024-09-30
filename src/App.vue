<template>
  <video autoplay muted loop>
    <source src="/src/assets/bgvideo.mp4" type="video/mp4" />
  </video>

  <div id="app" class="min-h-screen py-10">
    <MusicPlayer
      ref="player"
      :currentSong="currentSong"
      :songs="songs"
      @update:currentSong="updateCurrentSong"
    />
    <Playlist :songs="songs" @play="setSong" />
  </div>
</template>

<script>
import MusicPlayer from './components/MusicPlayer.vue'
import Playlist from './components/PlayList.vue'

const musicFiles = import.meta.glob('@/assets/music/*.mp3', { eager: true })
const coverFiles = import.meta.glob('@/assets/covers/*.jpg', { eager: true })

const songs = Object.keys(musicFiles).map((key) => {
  const fileName = key.split('/').pop().replace('.mp3', '')
  const [artist, title] = fileName.split(' - ')

  const coverKey = Object.keys(coverFiles).find((coverKey) =>
    coverKey.endsWith(`${fileName}.jpg`)
  )

  const coverPath = coverKey
    ? coverFiles[coverKey].default
    : '/src/assets/covers/default-cover.jpg'

  return {
    title: title || fileName,
    artist: title ? artist : 'Unknown Artist',
    src: musicFiles[key].default,
    albumArt: coverPath
  }
})

export default {
  components: {
    MusicPlayer,
    Playlist
  },
  data() {
    return {
      songs: songs,
      currentSong: songs[0]
    }
  },
  methods: {
    setSong(song) {
      this.currentSong = song
      this.$refs.player.playPause()
    },
    updateCurrentSong(song) {
      this.currentSong = song
    }
  }
}
</script>

<style>
body {
  margin: 0;
  padding: 0;
  font-family: 'Courier New', Courier, monospace;
}

video {
  width: 100vw;
  height: 100vh;
  object-fit: cover;
  position: fixed;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  z-index: -1;
}
</style>
