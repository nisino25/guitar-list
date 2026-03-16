<template>
  <div class="p-8 bg-gray-100 min-h-screen">
    <div class="text-center relative">
      <h1 class="text-3xl font-bold text-center text-indigo-600 mb-6 inline-flex">🎸Song List</h1>
      <button
          @click="openModal"
          class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 text-sm absolute right-0 top-0 transition"
      >
          Open Tuner
      </button>
    </div>
    <div class="flex flex-wrap gap-2 justify-center mb-4">
      <button
        v-for="key in ['lastPlayedAt', 'artistName', 'plays', 'songName']"
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


    <div>

        <!-- Modal -->
        <div
            v-if="showModal"
            class="fixed inset-0 flex items-center justify-center bg-black/60 z-50"
        >

            <div class="bg-white rounded-xl shadow-xl w-[420px] p-6">

                <div class="flex justify-between items-center mb-4">
                    <h2 class="text-xl font-bold">Guitar Tuner</h2>

                    <button
                        @click="closeModal"
                        class="text-gray-500 hover:text-black"
                    >
                        ✕
                    </button>
                </div>

                <button
                    @click="toggleTuner"
                    class="w-full py-2 mb-6 rounded text-white"
                    :class="isTuning ? 'bg-red-500 hover:bg-red-600' : 'bg-green-500 hover:bg-green-600'"
                >
                    {{ isTuning ? 'Stop Tuner' : 'Start Tuner' }}
                </button>
                
                <!-- audio graph -->
                <div v-if="currentTuningNote" class="text-center mb-4">
                  {{ currentTuningNote.note }} : {{ currentTuningNote.pitch }} Hz
                </div>
                <hr>
                <div class="text-center">
                  <div class="text-lg text-gray-500">
                      {{ detectedWave?.toFixed(2) }} Hz
                  </div>

                  <div class="text-lg">
                      Diff: {{ (detectedWave - currentTuningNote?.pitch)?.toFixed(2) }} Hz
                  </div>
                </div>


                <!-- <div class="grid grid-cols-2 gap-6 text-center">

                    <div>
                        <h3 class="text-gray-500 text-sm">Target</h3>
                        <div class="text-center space-y-2">

                          <div class="text-5xl font-bold">
                              {{ currentNote || '-' }}
                          </div>

                          <div class="text-lg text-gray-500">
                              {{ detectedWave?.toFixed(2) }} Hz
                          </div>

                          <div class="text-lg">
                              Diff: {{ diff?.toFixed(2) }} Hz
                          </div>

                          <div
                              class="text-xl font-bold"
                              :class="{
                                  'text-green-500': Math.abs(diff) < 0.5,
                                  'text-red-500': diff > 0.5,
                                  'text-blue-500': diff < -0.5
                              }"
                          >
                              {{ getTuningStatus(diff) }}
                          </div>

                      </div>

                    </div>

                    <div>
                        <h3 class="text-gray-500 text-sm">Detected</h3>
                        <div class="text-3xl font-bold text-blue-600">
                            {{ detectedNote || '-' }}
                        </div>
                    </div>

                </div> -->

                <!-- show the visual -->
                <div class="mt-6 mx-auto grid grid-cols-2 gap-4 text-center w-[50%] justify-items-center">
                  <template v-for="(note,index) in tempStatus" :key="index" >
                    <div class="flex items-center gap-2">
                        <div
                          class='w-6 aspect-square rounded-full'
                          :class="{
                              'bg-gray-300': note.status === 'not tuned',
                              'bg-yellow-400': note.status === 'tuning',
                              'bg-green-500': note.status === 'tuned'
                          }"
                          @click="selectNote(note)"
                        ></div>
                        <span class="text-gray-700 font-medium">{{ note.note }}</span>
                    </div>  
                  </template>
                </div>

            </div>

        </div>

    </div>



    <div v-if="filteredData?.length > 0" class="grid grid-cols-3 gap-4">
      <div
        v-for="(item, index) in filteredData"
        :key="index"
        class="bg-white rounded-lg shadow-md p-3 hover:shadow-xl transition duration-300 block cursor-pointer"
        @click="handleCardClick(item)"
      >
        <!-- Row 1: Song Name + Plays -->
        <div class="grid grid-cols-5 gap-1 items-center mb-1">
          <!-- Like Button -->
          <h2 class="col-span-4 text-lg font-semibold text-gray-800 truncate flex items-center gap-1">
            <!-- Like Button -->
            <button
              @click.stop="toggleStar(item)"
              :class="[
                item.star == 1 ? 'text-yellow-500 hover:text-yellow-400' : 'text-black hover:text-gray-600',
                'transition transform scale-110 font-normal'
              ]"
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
          <p class="text-xs"><strong class="text-sm">{{ daysAgo(item.lastPlayedAt) }}</strong>日前</p>
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
      baseUrl: 'https://script.google.com/macros/s/AKfycbznvSmhM5j1qnEzuA8Dxkd_Ms9REW1A0zurB7RkG7SkM1VSeHLT4RQfXs2fnybys8vleA/exec',
      sortKey: 'lastPlayedAt',
      sortAsc: false,
      filteredArtist: null,
      starFilter: false,

      showModal: false,

      audioContext: null,
      mediaStream: null,
      analyser: null,

      isTuning: false,

      currentNote: '',
      detectedNote: '',

      targetWave: '',
      detectedWave: null,

      diff: null,
      


      tolerance: 5,

      targetFrequencies: {
          E2: 82.41,
          A2: 110.00,
          D3: 146.83,
          G3: 196.00,
          B3: 246.94,
          E4: 329.63
      },

      tempStatus: [
        {note: 'D', status: "not tuned", pitch: 146.83},
        {note: 'G', status: "not tuned", pitch: 196.00},

        {note: 'A', status: "not tuned", pitch: 110.00},
        {note: 'B', status: "not tuned", pitch: 246.94},
        
        {note: 'E', status: "not tuned", pitch: 82.41},
        {note: 'E', status: "not tuned", pitch: 329.63},
      ],

    };
  },
  methods: {
    fetchData() {
      const url = `${this.baseUrl}?callback=jsonpCallback&action=fetchData`;
      window.jsonpCallback = (data) => {
        console.log("API Response (fetchData):", data);
        if (data.success) {
          this.fetchedData = data.data.sort((a, b) => b.lastPlayedAt - a.lastPlayedAt);
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
      console.log(song)
      this.incrementWord(song);
      song.lastPlayedAt = Math.floor(Date.now() / 1000);
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

    openModal() {
      this.tempStatus.forEach(item => item.status = "not tuned")

      this.tempStatus[4].status = "tuning"
      // this.tempStatus[0].status = true
      this.showModal = true
    },

    closeModal() {
        this.stopTuner()
        this.showModal = false
    },

    toggleTuner() {

        if (this.isTuning) {
            this.stopTuner()
        } else {
            this.startTuner()
        }

    },

    async startTuner() {

        this.isTuning = true

        this.audioContext = new (window.AudioContext || window.webkitAudioContext)()

        this.mediaStream = await navigator.mediaDevices.getUserMedia({ audio: true })

        const source = this.audioContext.createMediaStreamSource(this.mediaStream)

        this.analyser = this.audioContext.createAnalyser()
        source.connect(this.analyser)

        this.analyser.fftSize = 4096

        const bufferLength = this.analyser.frequencyBinCount
        const dataArray = new Uint8Array(bufferLength)

        

        const detect = () => {

            this.analyser.getByteTimeDomainData(dataArray)

            const volumeThreshold = 0.02
            const volume = this.getVolume(dataArray)
            if(volume > volumeThreshold){
              // console.log(dataArray)
              // console.log('Volume:', volume)
              // this.detectedWave = this.getPitch(dataArray, this.audioContext.sampleRate)
              // this.detectedWave = this.detectedWave * 0.8 + newFreq * 0.2
              const newFreq = this.getPitch(dataArray, this.audioContext.sampleRate)

              if (newFreq) {
                  if (!this.detectedWave) {
                      this.detectedWave = newFreq
                  } else {
                      // smoothing
                      this.detectedWave = this.detectedWave * 0.8 + newFreq * 0.2
                  }
              }


  
              const note = this.getClosestNote(this.detectedWave)
  
              this.detectedNote = note
  
              }
                if (this.isTuning) {
                    requestAnimationFrame(detect)
                }

        }

        detect()
    },

    stopTuner() {

        this.isTuning = false

        if (this.mediaStream) {
            this.mediaStream.getTracks().forEach(track => track.stop())
        }

        if (this.audioContext) {
            this.audioContext.close()
        }

        this.currentNote = ''
        this.detectedNote = ''
    },

    getPitch(dataArray, sampleRate) {

        let SIZE = dataArray.length
        let rms = 0

        for (let i = 0; i < SIZE; i++) {
            let val = (dataArray[i] - 128) / 128
            rms += val * val
        }

        rms = Math.sqrt(rms / SIZE)
        if (rms < 0.01) return null

        let r1 = 0, r2 = SIZE - 1
        for (let i = 0; i < SIZE / 2; i++) {
            if (Math.abs(dataArray[i] - 128) > 10) { r1 = i; break }
        }
        for (let i = 1; i < SIZE / 2; i++) {
            if (Math.abs(dataArray[SIZE - i] - 128) > 10) { r2 = SIZE - i; break }
        }

        dataArray = dataArray.slice(r1, r2)
        SIZE = dataArray.length

        let c = new Array(SIZE).fill(0)

        for (let i = 0; i < SIZE; i++) {
            for (let j = 0; j < SIZE - i; j++) {
                c[i] = c[i] + (dataArray[j] - 128) * (dataArray[j + i] - 128)
            }
        }

        let d = 0
        while (c[d] > c[d + 1]) d++

        let maxval = -1
        let maxpos = -1
        for (let i = d; i < SIZE; i++) {
            if (c[i] > maxval) {
                maxval = c[i]
                maxpos = i
            }
        }

        let T0 = maxpos
        let frequency = sampleRate / T0

        return frequency
    },

    getVolume(dataArray) {

        let sum = 0

        for (let i = 0; i < dataArray.length; i++) {

            const value = (dataArray[i] - 128) / 128
            sum += value * value

        }

        const rms = Math.sqrt(sum / dataArray.length)

        return rms
    },

    findPeak(dataArray) {

        let peakIndex = 0
        let peakValue = 0

        for (let i = 0; i < dataArray.length; i++) {

            if (dataArray[i] > peakValue) {

                peakIndex = i
                peakValue = dataArray[i]
            }
        }

        return peakIndex
    },

    getClosestNote(frequency) {

        let closestNote = ''
        let minDiff = Infinity

        for (const note in this.targetFrequencies) {

            const target = this.targetFrequencies[note]

            const diff = Math.abs(frequency - target)

            if (diff < minDiff && diff <= this.tolerance) {

                closestNote = note
                minDiff = diff
            }
        }

        this.currentNote = closestNote

        return closestNote
    },
    getTuningStatus(diff){

        if(Math.abs(diff) < 0.5) return "Perfect"

        if(diff > 0) return "Sharp"

        return "Flat"

    },

    selectNote(note){
      this.tempStatus.forEach(item => {
        if(item.status === "tuning"){
          item.status = "not tuned"
        }
      })
      note.status = "tuning"
    }


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
    currentTuningNote() {
      return this.tempStatus.find(item => item.status === "tuning");
    }
  },
  mounted() {
    console.clear();

    // this.openModal()
    this.fetchData();
  },
  beforeUnmount() {
    this.stopTuner()
  }
};
</script>

<style>
body {
  background: #f3f4f6;
  @apply font-sans;
}
</style>
