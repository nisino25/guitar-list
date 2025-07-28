<template>
  <div class="p-8 bg-gray-100 min-h-screen">
    <h1 class="text-3xl font-bold text-center text-indigo-600 mb-6">
      🎵 🎸Song List
    </h1>
    <div class="flex flex-wrap gap-2 justify-center mb-4">
      <button
        v-for="key in ['timestamp', 'artistName', 'plays', 'songName']"
        :key="key"
        @click="sortBy(key)"
        :class="[
          'px-3 py-1 rounded border text-sm font-medium',
          sortKey === key ? 'bg-indigo-600 text-white' : 'bg-white text-gray-700',
          'hover:bg-indigo-100 transition'
        ]"
      >
        <span class="capitalize">
          {{ key === 'artistName' ? 'Artist' : key === 'songName' ? 'Song' : key }}
        </span>
        <span v-if="sortKey === key">
          &nbsp;{{ sortAsc ? '▲' : '▼' }}
        </span>
      </button>
      <!-- Filter Tag (only show if filterArtist is set) -->
      <button
        v-if="filteredArtist"
        @click="filteredArtist = null"
        class="px-3 py-1 rounded bg-rose-200 text-rose-800 font-medium text-sm border border-rose-400 hover:bg-rose-300 transition"
      >
        ✖ {{ filteredArtist }}
      </button>
      <button
        @click="starFilter = !starFilter"
        class="px-2 py-1 rounded border text-sm font-medium transition hover:bg-yellow-200"
        :class="starFilter
          ? 'bg-yellow-300 text-yellow-900 border-yellow-500 shadow-sm'
          : 'bg-white text-gray-700 border-gray-300'"
      >
        {{ starFilter ? '★' : '☆' }}
      </button>



    </div>



    <div v-if="filteredData?.length > 0" class="grid grid-cols-3 gap-4">
      <div
        v-for="(item, index) in filteredData"
        :key="index"
        class="bg-white rounded-lg shadow-md p-4 hover:shadow-xl transition duration-300 block cursor-pointer"
        @click="handleCardClick(item)"
      >
        <!-- Row 1: Song Name + Plays -->
        <div class="grid grid-cols-5 gap-1 items-center mb-2">
          <!-- Like Button -->
          <h2 class="col-span-4 text-lg font-semibold text-gray-800 truncate flex items-center gap-1">
            <!-- Like Button -->
            <button
              @click.stop="toggleStar(item)"
              class="text-yellow-500 hover:text-yellow-400 transition transform scale-110"
              :title="item.star == 1 ? 'Unstar' : 'Star'"
            >
              {{ item.star == 1 ? '★' : '☆' }}
            </button>

            {{ item.songName }}
          </h2>

          <div class="text-right text-sm text-gray-500">
            <strong>{{ item.plays || '0' }}回</strong>
          </div>
        </div>

        <!-- Row 2: Artist + Timestamp -->
        <div class="flex justify-between text-gray-600 text-sm">
          <p @click.stop="filterArtist(item.artistName)">{{ item.artistName }}</p>
          <p><strong>{{ daysAgo(item.timestamp) }}</strong> days</p>
        </div>
      </div>

    </div>


    <div v-else class="text-center text-gray-500 mt-12">Loading songs...</div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      fetchedData: null,
      baseUrl: 'https://script.google.com/macros/s/AKfycbwIrCP2wbwF36WcQn5ogTu3nfLBFSiNdsBxm60XOG39c7QcTOIl35G4tOnXUFRHN3v6Bg/exec',
      sortKey: 'timestamp',
      sortAsc: false,
      filteredArtist: null,
      starFilter: false,
    };
  },
  methods: {
    fetchData() {
      const url = `${this.baseUrl}?callback=jsonpCallback&action=fetchData`;
      window.jsonpCallback = (data) => {
        console.log("API Response (fetchData):", data);
        if (data.success) {
          this.fetchedData = data.data.sort((a, b) => b.timestamp - a.timestamp);
        } else {
          console.error("Error fetching data:", data.message);
        }
      };

      const script = document.createElement("script");
      script.src = url;
      script.async = true;
      document.body.appendChild(script);

      script.onload = () => {
        document.body.removeChild(script);
      };
    },
    daysAgo(unixTimestamp) {
      if (!unixTimestamp) return 'N/A';
      const now = Math.floor(Date.now() / 1000); // current time in seconds
      const diffInSeconds = now - unixTimestamp;
      const diffInDays = Math.floor(diffInSeconds / 86400); // 86400 sec/day
      return `${diffInDays}`;
    },
    sortBy(key) {
      if (this.sortKey === key) {
        this.sortAsc = !this.sortAsc;
      } else {
        this.sortKey = key;
        this.sortAsc = false;
      }

      this.fetchedData.sort((a, b) => {
        const aVal = a[key] ?? '';
        const bVal = b[key] ?? '';
        if (typeof aVal === 'number' && typeof bVal === 'number') {
          return this.sortAsc ? aVal - bVal : bVal - aVal;
        } else {
          return this.sortAsc
            ? String(aVal).localeCompare(String(bVal))
            : String(bVal).localeCompare(String(aVal));
        }
      });
    },
    handleCardClick(song) {
      this.incrementWord(song);
      window.open(song.url, "_blank");
    },
    incrementWord(song) {
      // Locally increment the counter if you want visual feedback
      song.plays = Number(song.plays || 0) + 1;

      // Build the API URL with the new parameters
      const url = `${this.baseUrl}?callback=jsonpCallback&action=increment&songName=${encodeURIComponent(song.songName)}&artistName=${encodeURIComponent(song.artistName)}`;
      console.log('-------- incrementing play count --------');
      console.log(url);
      console.log('-----------------------------------------');

      // Define the callback function globally
      window.jsonpCallback = (data) => {
        console.log("API Response (increment):", data);
      };

      // Add <script> tag to trigger JSONP request
      const script = document.createElement("script");
      script.src = url;
      script.async = true;
      document.body.appendChild(script);

      // Clean up
      script.onload = () => {
        document.body.removeChild(script);
      };
    },
    filterArtist(name){
      this.filteredArtist = name
    },
    toggleStar(song) {
        const newStar = song.star == 1 ? 0 : 1;
        song.star = newStar; // Optimistically update UI

        const url = `${this.baseUrl}?callback=jsonpCallback&action=toggleStar&songName=${encodeURIComponent(song.songName)}&artistName=${encodeURIComponent(song.artistName)}`;
        console.log('-------- toggling star --------');
        console.log(url);
        console.log('--------------------------------');

        window.jsonpCallback = (data) => {
            console.log("API Response (toggleStar):", data);
        };

        const script = document.createElement("script");
        script.src = url;
        script.async = true;
        document.body.appendChild(script);

        script.onload = () => {
            document.body.removeChild(script);
        };
    },

  },
  computed: {
    filteredData() {
        if (!this.fetchedData) return [];

        let data = this.fetchedData;

        if (this.filteredArtist) {
            data = data.filter(item => item.artistName === this.filteredArtist);
        }

        if (this.starFilter) {
            data = data.filter(item => item.star === 1);
        }

        return data;
    },
},
  mounted() {
    console.clear();
    this.fetchData();
  },
};
</script>

<style>
body {
  background: #f3f4f6;
  @apply font-sans;
}
</style>
