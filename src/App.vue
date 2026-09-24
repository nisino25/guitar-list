<template>
  <div class="p-8 bg-gray-100 min-h-screen">
    <div class="text-center relative">
      <button
          @click="showAddModal = true"
          class="px-4 py-2 bg-green-600 text-white rounded hover:bg-green-700 text-sm absolute left-0 top-0 transition"
      >
          + Add Song
      </button>

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
          sortKey === key ? 'bg-indigo-600 text-white' : 'bg-white text-gray-700'
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

              <!-- tuning meter -->
              <div v-if="cents !== null" class="relative w-64 h-3 mx-auto mt-4 bg-gray-200 rounded-full overflow-hidden">
                  <div class="absolute top-0 bottom-0 bg-green-200" :style="tunedZoneStyle"></div>
                  <div class="absolute top-0 bottom-0 left-1/2 w-px bg-gray-400"></div>
                  <div
                      class="absolute top-1/2 w-4 h-4 rounded-full -mt-2 -ml-2 transition-all duration-100"
                      :class="Math.abs(cents) < tunedThresholdCents ? 'bg-green-500' : 'bg-red-500'"
                      :style="{ left: needlePositionPercent + '%' }"
                  ></div>
              </div>

              <div v-if="cents !== null && Math.abs(cents) < tunedThresholdCents" class="text-sm text-green-600 mt-2">
                  Perfect!
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
                      @click="selectNote(index)"
                    ></div>
                    <span class="text-gray-700 font-medium">{{ note.note }}</span>
                </div>  
              </template>
            </div>

        </div>

    </div>

    <!-- Add Song Modal -->
    <div
        v-if="showAddModal"
        class="fixed inset-0 flex items-center justify-center bg-black/60 z-50"
      >
        <div class="bg-white rounded-xl shadow-xl w-[420px] p-6">

            <!-- header -->
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-xl font-bold">Add Song</h2>
                <button
                    @click="showAddModal = false"
                    class="text-gray-500 hover:text-black"
                >
                    ✕
                </button>
            </div>

            <!-- form -->
            <div class="space-y-3">

                <input
                    v-model="newSong.url"
                    placeholder="URL"
                    class="w-full border rounded px-3 py-2"
                />

                <input
                    v-model="newSong.songName"
                    placeholder="Song Name"
                    class="w-full border rounded px-3 py-2"
                />

                <input
                    v-model="newSong.artistName"
                    placeholder="Artist Name"
                    class="w-full border rounded px-3 py-2"
                />

                <!-- type toggle -->
                <div class="flex gap-2">
                    <button
                        @click="newSong.type = 'ultimate-guitar'"
                        :class="[
                            'flex-1 py-2 rounded border',
                            newSong.type === 'ultimate-guitar'
                                ? 'bg-indigo-600 text-white'
                                : 'bg-white'
                        ]"
                    >
                        ultimate-guitar
                    </button>

                    <button
                        @click="newSong.type = 'Ufret'"
                        :class="[
                            'flex-1 py-2 rounded border',
                            newSong.type === 'Ufret'
                                ? 'bg-indigo-600 text-white'
                                : 'bg-white'
                        ]"
                    >
                        Ufret
                    </button>
                </div>

            </div>

            <!-- actions -->
            <div class="mt-5 flex justify-end gap-2">
                <button
                    @click="showAddModal = false"
                    class="px-4 py-2 bg-gray-200 rounded hover:bg-gray-300"
                >
                    Cancel
                </button>

                <button
                    @click="addSong"
                    class="px-4 py-2 bg-indigo-600 text-white rounded hover:bg-indigo-700"
                >
                    Add
                </button>
            </div>

        </div>
    </div>


    <div v-if="!isLoading" class="grid grid-cols-3 gap-4">
      <div
        v-for="(item, index) in filteredData"
        :key="index"
        class="bg-white rounded-lg shadow-md p-3 hover:shadow-xl transition duration-300 block cursor-pointer"
        :class="getBgColor(item)"
        @click="handleCardClick(item)"
      >
        <!-- Row 1: Song Name + Plays -->
        <div class="grid grid-cols-5 gap-1 items-center mb-1">
          <!-- Like Button -->
          <h2 class="col-span-4 text-lg font-semibold text-gray-800 truncate flex items-center gap-1">
            <!-- Like Button -->
            <!-- <button
              @click.stop="toggleStar(item)"
              :class="[
                item.star == 1 ? 'text-yellow-500 hover:text-yellow-400' : 'text-black hover:text-gray-600',
                'transition transform scale-110 font-normal'
              ]"
            >
              {{ item.star == 1 ? '★' : '☆' }}
            </button> -->
            <button
                @click.stop="toggleStar(item)"
                :class="[
                    item.star == 1
                        ? 'text-yellow-500 hover:text-yellow-400'
                        : item.star == 2
                            ? 'text-red-500 hover:text-red-400 scale-90'
                            : 'text-black hover:text-gray-600',
                    'transition transform font-normal'
                ]"
            >
                {{
                    item.star == 2
                        ? '❤️'
                        : item.star == 1
                            ? '★'
                            : '☆'
                }}
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
      baseUrl: 'https://script.google.com/macros/s/AKfycbxB0_D-Q46UKqrTq-fShg6aEBRp87Fdofc_e_X08vvWQMCrtfsMvamLUfmxKTktijYv_g/exec',
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

      tolerance: 5,
      tunedThresholdCents: 5,

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
      selectedNoteIndex: 4,
      // tempStatus indices ordered from lowest to highest string (E2, A2, D3, G3, B3, E4)
      tuningOrder: [4, 2, 0, 1, 3, 5],
      tunedSinceTimestamp: null,
      holdToAdvanceMs: 1500,

      showAddModal: false,
      newSong: {
          url: "",
          songName: "",
          artistName: "",
          type: "ultimate-guitar"
      },
      isLoading: true,


    };
  },
  methods: {
    fetchData() {
      const url = `${this.baseUrl}?callback=jsonpCallback&action=fetchData`;
      window.jsonpCallback = (data) => {
        console.log("API Response (fetchData):", data);
        if (data.success) {
          this.isLoading = false;
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
      const url = `${this.baseUrl}?callback=jsonpCallback&action=increment&songName=${encodeURIComponent(song?.songName.toString())}&artistName=${encodeURIComponent(song.artistName)}`;
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
        // const newStar = song.star == 1 ? 0 : 1;
        const newStar = (song.star + 1) % 3;
        song.star = newStar; // Optimistically update UI

        const url = `${this.baseUrl}?callback=jsonpCallback&action=toggleStar&songName=${encodeURIComponent(song.songName)}&artistName=${encodeURIComponent(song.artistName)}`;

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

    // ------------------- Tuner Methods -------------------

    openModal() {
      this.tempStatus.forEach(item => item.status = "not tuned")

      this.selectedNoteIndex = 4
      this.tempStatus[4].status = "tuning"
      this.tunedSinceTimestamp = null
      // this.tempStatus[0].status = true
      this.showModal = true
      this.startTuner()
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

        try {
            this.mediaStream = await navigator.mediaDevices.getUserMedia({ audio: true })
        } catch (err) {
            alert("Couldn't access the microphone. Please allow mic access and try again.")
            this.stopTuner()
            return
        }

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

              if (newFreq && isFinite(newFreq)) {
                  if (!this.detectedWave) {
                      this.detectedWave = newFreq
                  } else {
                      // smoothing
                      this.detectedWave = this.detectedWave * 0.8 + newFreq * 0.2
                  }
              }



              const note = this.getClosestNote(this.detectedWave)

              this.detectedNote = note

              if (this.currentTuningNote && this.detectedWave) {
                  const isTuned = Math.abs(this.cents) < this.tunedThresholdCents

                  if (isTuned) {
                      this.currentTuningNote.status = "tuned"
                      if (!this.tunedSinceTimestamp) {
                          this.tunedSinceTimestamp = Date.now()
                      }
                  } else {
                      this.tunedSinceTimestamp = null
                      this.currentTuningNote.status = "tuning"
                  }
              }

              }

              // Checked every frame regardless of current volume, so a note that
              // rings out (drops below volumeThreshold) doesn't stall the countdown
              // before it reaches holdToAdvanceMs.
              if (this.tunedSinceTimestamp && Date.now() - this.tunedSinceTimestamp >= this.holdToAdvanceMs) {
                  this.playSuccessSound()
                  this.advanceToNextNote()
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
            this.mediaStream = null
        }

        if (this.audioContext && this.audioContext.state !== 'closed') {
            this.audioContext.close()
        }
        this.audioContext = null

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
        if (!T0 || T0 <= 0) return null

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

    playSuccessSound() {
        if (!this.audioContext) return

        const ctx = this.audioContext
        const now = ctx.currentTime

        const playTone = (freq, startTime, duration) => {
            const osc = ctx.createOscillator()
            const gain = ctx.createGain()

            osc.type = 'sine'
            osc.frequency.value = freq

            gain.gain.setValueAtTime(0, startTime)
            gain.gain.linearRampToValueAtTime(0.3, startTime + 0.02)
            gain.gain.exponentialRampToValueAtTime(0.001, startTime + duration)

            osc.connect(gain)
            gain.connect(ctx.destination)

            osc.start(startTime)
            osc.stop(startTime + duration)
        }

        // pleasant two-note chime (A5 -> C#6)
        playTone(880.00, now, 0.25)
        playTone(1108.73, now + 0.08, 0.3)
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
    selectNote(index){
      this.tempStatus.forEach(item => {
        if(item.status === "tuning"){
          item.status = "not tuned"
        }
      })
      this.selectedNoteIndex = index
      this.tunedSinceTimestamp = null
      if (this.tempStatus[index].status !== "tuned") {
        this.tempStatus[index].status = "tuning"
      }
    },

    advanceToNextNote(){
      const currentPos = this.tuningOrder.indexOf(this.selectedNoteIndex)
      if (currentPos === -1 || currentPos === this.tuningOrder.length - 1) return

      const nextIndex = this.tuningOrder[currentPos + 1]
      this.selectedNoteIndex = nextIndex
      this.detectedWave = null
      this.tunedSinceTimestamp = null
      if (this.tempStatus[nextIndex].status !== "tuned") {
        this.tempStatus[nextIndex].status = "tuning"
      }
    },

    getBgColor(item) {
      if (item.star == 1) {
        return 'bg-yellow-50';
      } else if (item.star == 2) {
        return 'bg-rose-50';
      } else if (this.filteredArtist && item.artistName === this.filteredArtist) {
        return 'bg-blue-50';
      } else {
        return '';
      }
    },

    addSong() {
        const { url, songName, artistName, type } = this.newSong;

        if (!url || !songName || !artistName) {
            alert("fill all fields bro");
            return;
        }

        this.isLoading = true;

        const apiUrl = `${this.baseUrl}?callback=jsonpCallback&action=addData&url=${encodeURIComponent(url)}&songName=${encodeURIComponent(songName)}&artistName=${encodeURIComponent(artistName)}&type=${encodeURIComponent(type)}`;
        this.showAddModal = false;

        window.jsonpCallback = (data) => {
            console.log("API Response (addData):", data);

            if (data.success) {
                // this.isLoading = false;
                

                // reset form
                this.newSong = {
                    url: "",
                    songName: "",
                    artistName: "",
                    type: "ultimate-guitar"
                };

                // refresh list
                this.fetchData();
            } else {
                alert(data.message);
            }
        };

        const script = document.createElement("script");
        script.src = apiUrl;
        script.async = true;
        document.body.appendChild(script);

        script.onload = () => {
            document.body.removeChild(script);
        };
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
            data = data.filter(item => item.star === 1 || item.star === 2);
        }

        return data;
    },
    currentTuningNote() {
      return this.tempStatus[this.selectedNoteIndex];
    },
    cents() {
      if (!this.detectedWave || !this.currentTuningNote) return null
      return 1200 * Math.log2(this.detectedWave / this.currentTuningNote.pitch)
    },
    clampedCents() {
      if (this.cents === null) return 0
      return Math.max(-50, Math.min(50, this.cents))
    },
    needlePositionPercent() {
      return this.clampedCents + 50
    },
    tunedZoneStyle() {
      const widthPercent = this.tunedThresholdCents * 2
      return {
        left: (50 - this.tunedThresholdCents) + '%',
        width: widthPercent + '%'
      }
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
