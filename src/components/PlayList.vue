<template>
  <div class="playlist bg-gray-900 text-gray-200 p-6 rounded-lg shadow-lg mt-6">
    <h3 class="text-2xl mb-4 border-b-2 border-gray-700 pb-2">Playlist</h3>
    <ul class="list-none p-0 m-0">
      <li
        v-for="(song, index) in songs"
        :key="index"
        @click="playSong(song)"
        class="cursor-pointer py-2 px-4 border-b border-gray-700 hover:bg-gray-800 hover:text-red-400 transition-colors duration-300"
      >
        {{ song.title }} - {{ song.artist }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  props: {
    songs: Array
  },
  methods: {
    playSong(song) {
      this.$emit('play', song)
    }
  }
}
</script>

<style scoped>
.playlist {
  background: #1a1a1a; /* Dark background for a grungy look */
}
</style>
