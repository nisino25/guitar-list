<template>
  <div class="px-8 pb-8 bg-gray-200 min-h-screen">
    <!-- Sticky header: one row -->
    <!-- Top padding includes the iPad status bar area (black-translucent status bar draws over the page) -->
    <div
      class="sticky top-0 z-40 -mx-8 px-8 pb-3 mb-4 bg-gray-200/95 backdrop-blur transition-shadow duration-200"
      :style="{
        paddingTop: 'calc(1.5rem + env(safe-area-inset-top))',
        boxShadow: isScrolled ? '0 6px 16px -8px rgba(0, 0, 0, 0.3)' : 'none'
      }"
    >
      <div class="flex items-center gap-2">
        <!-- Title doubles as the Songs / Lab switch -->
        <button @click="page = page === 'songs' ? 'lab' : 'songs'" class="shrink-0 px-1 active:scale-95 transition">
          <h1 class="text-2xl font-black tracking-tighter text-gray-900 leading-none">
            {{ page === 'songs' ? 'setlist' : 'lab' }}<span class="text-indigo-600">.</span>
          </h1>
        </button>

        <div v-if="page === 'songs'" class="flex-1 min-w-0 flex items-center justify-center gap-2">
          <!-- Sort: one pill, options open on tap (tap the active one again to flip direction) -->
          <div class="relative">
            <button
              @click="showSortMenu = !showSortMenu"
              class="h-9 pl-3 pr-2 rounded-full bg-white text-gray-700 text-sm font-medium shadow-sm flex items-center gap-1.5 whitespace-nowrap"
            >
              <AppIcon name="sort" :size="15" class="text-gray-400" />
              {{ currentSortLabel }}
              <span class="text-xs text-indigo-600">{{ sortAsc ? '↑' : '↓' }}</span>
            </button>
            <template v-if="showSortMenu">
              <div class="fixed inset-0 z-40" @click="showSortMenu = false"></div>
              <div class="absolute left-0 mt-2 w-44 bg-white rounded-xl shadow-xl py-1 z-50">
                <button
                  v-for="option in sortOptions"
                  :key="option.key"
                  @click="sortBy(option.key); showSortMenu = false"
                  class="w-full flex justify-between px-4 py-2 text-sm hover:bg-indigo-50"
                  :class="sortKey === option.key ? 'font-semibold text-indigo-700' : 'text-gray-700'"
                >
                  {{ option.label }}
                  <span v-if="sortKey === option.key">{{ sortAsc ? '↑' : '↓' }}</span>
                </button>
              </div>
            </template>
          </div>

          <button
            @click="starFilter = !starFilter"
            class="w-9 h-9 shrink-0 rounded-full shadow-sm flex items-center justify-center transition"
            :class="starFilter ? 'bg-amber-400 text-white' : 'bg-white text-gray-400'"
            title="Liked only"
          >
            <AppIcon name="star" :filled="starFilter" :size="17" />
          </button>

          <!-- Playlist: icon button, list opens on tap -->
          <div class="relative">
            <button
              @click="showPlaylistMenu = !showPlaylistMenu"
              class="h-9 shrink-0 rounded-full shadow-sm flex items-center justify-center gap-1.5 transition whitespace-nowrap max-w-[10rem]"
              :class="playlistFilter ? 'bg-purple-600 text-white px-3' : 'bg-white text-gray-400 w-9'"
              title="Playlists"
            >
              <AppIcon name="folder" :filled="!!playlistFilter" :size="16" class="shrink-0" />
              <span v-if="playlistFilter" class="text-sm font-medium truncate">{{ playlistLabel(playlistFilter) }}</span>
            </button>
            <template v-if="showPlaylistMenu">
              <div class="fixed inset-0 z-40" @click="showPlaylistMenu = false"></div>
              <div class="absolute left-1/2 -translate-x-1/2 mt-2 w-52 bg-white rounded-xl shadow-xl py-1 z-50">
                <button
                  @click="playlistFilter = null; showPlaylistMenu = false"
                  class="w-full text-left px-4 py-2 text-sm hover:bg-purple-50"
                  :class="!playlistFilter ? 'font-semibold text-purple-700' : 'text-gray-700'"
                >
                  All songs
                </button>
                <button
                  v-for="playlist in playlists"
                  :key="playlist"
                  @click="playlistFilter = playlist; showPlaylistMenu = false"
                  class="w-full text-left px-4 py-2 text-sm hover:bg-purple-50"
                  :class="playlistFilter === playlist ? 'font-semibold text-purple-700' : 'text-gray-700'"
                >
                  {{ playlistLabel(playlist) }}
                </button>
              </div>
            </template>
          </div>

            <button
              v-if="filteredArtist"
              @click="filteredArtist = null"
              class="h-9 px-4 rounded-full bg-rose-100 text-rose-800 text-sm font-medium shadow-sm"
            >
              ✕ {{ filteredArtist }}
            </button>

            <div v-if="luckyPicks" class="inline-flex items-center h-9 rounded-full bg-emerald-100 text-emerald-800 text-sm font-medium shadow-sm overflow-hidden">
              <span class="pl-3 pr-2 flex items-center gap-1"><AppIcon name="shuffle" :size="14" /> Lucky {{ luckyPicks.length }}</span>
              <button @click="pickLucky" class="px-2 h-full hover:bg-emerald-200" title="Shuffle again">↻</button>
              <button @click="luckyPicks = null" class="pl-2 pr-4 h-full hover:bg-emerald-200" title="Back to all songs">✕</button>
            </div>
        </div>
        <div v-else class="flex-1"></div>

        <button
            v-if="page === 'songs'"
            @click="pickLucky"
            class="h-9 px-3 shrink-0 rounded-full bg-emerald-500 text-white text-sm font-semibold shadow-sm hover:bg-emerald-600 transition flex items-center gap-1.5"
            title="Feeling lucky: 6 songs you haven't played today"
        >
            <AppIcon name="shuffle" :size="15" />
            Lucky
        </button>
        <button
            @click="openModal"
            class="w-9 h-9 shrink-0 rounded-full bg-white text-gray-600 shadow-sm hover:text-indigo-600 transition flex items-center justify-center"
            title="Tuner"
        >
            <AppIcon name="tuner" :size="18" />
        </button>
        <button
            @click="showAddModal = true"
            class="w-9 h-9 shrink-0 rounded-full bg-gray-900 text-white shadow-sm hover:bg-gray-700 transition flex items-center justify-center"
            title="Add song"
        >
            <AppIcon name="plus" :size="18" />
        </button>
      </div>
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

    <!-- Playlist Modal -->
    <div
        v-if="showPlaylistModal"
        class="fixed inset-0 flex items-center justify-center bg-black/60 z-50"
    >
        <div class="bg-white rounded-xl shadow-xl w-[420px] p-6">

            <div class="flex justify-between items-center mb-4">
                <h2 class="text-xl font-bold">Playlists</h2>
                <button
                    @click="closePlaylistModal"
                    class="text-gray-500 hover:text-black"
                >
                    ✕
                </button>
            </div>

            <div class="space-y-2">
                <label
                    v-for="playlist in sortedPlaylistsForModal"
                    :key="playlist"
                    class="flex items-center gap-3 px-3 py-2 rounded border cursor-pointer hover:bg-gray-50"
                >
                    <input
                        type="checkbox"
                        :checked="(playlistModalSong.playlists || []).includes(playlist)"
                        @change="togglePlaylist(playlistModalSong, playlist)"
                    />
                    <span>{{ playlistLabel(playlist) }}</span>
                </label>
            </div>

            <div class="flex gap-2 mt-4">
                <input
                    v-model="newPlaylistName"
                    @keyup.enter="addNewPlaylist"
                    placeholder="+ New Playlist"
                    class="flex-1 border rounded px-3 py-2 text-sm"
                />
                <button
                    @click="addNewPlaylist"
                    class="px-3 py-2 bg-purple-600 text-white rounded hover:bg-purple-700 text-sm"
                >
                    Add
                </button>
            </div>

        </div>
    </div>


    <div v-if="page === 'songs' && !isLoading" class="grid grid-cols-3 gap-2">
        <div
          v-for="(item, index) in filteredData"
          :key="index"
          class="group transition px-2 py-1.5 border-l-4 flex items-center gap-2 cursor-pointer"
          :class="d3CardClass(item, 'ticket3')"
          :style="d3CardStyle(item, index, 'ticket3')"
          @click="handleCardClick(item)"
        >
          <div class="shrink-0 flex flex-col items-center -ml-1">
            <button
              @click.stop="toggleStar(item)"
              class="w-7 h-6 flex items-center justify-center transition active:scale-125"
              :class="item.star == 1 ? 'text-amber-500' : item.star == 2 ? 'text-rose-500' : songCardStyle.emptyStar"
              :title="item.star == 2 ? 'Loved' : item.star == 1 ? 'Liked' : 'Like'"
            >
              <AppIcon :name="item.star == 2 ? 'heart' : 'star'" :filled="item.star == 1 || item.star == 2" :size="16" />
            </button>
            <button
              @click.stop="openPlaylistModal(item)"
              class="w-7 h-6 flex items-center justify-center transition"
              :class="(item.playlists || []).length ? 'text-purple-500' : songCardStyle.emptyStar"
              title="Add to playlist"
            >
              <AppIcon name="folder" :filled="(item.playlists || []).length > 0" :size="14" />
            </button>
          </div>
          <div class="flex-1 min-w-0">
            <div class="truncate" :class="songCardStyle.title">{{ item.songName }}</div>
            <div class="flex items-center gap-2 text-[11px]" :class="songCardStyle.sub">
              <span class="flex-1 min-w-0 truncate" @click.stop="filterArtist(item.artistName)">{{ item.artistName }}</span>
              <span class="shrink-0 flex gap-1" :class="songCardStyle.meta">
                <span class="w-7 text-right font-semibold whitespace-nowrap">{{ item.plays || 0 }}回</span>
                <span class="w-12 text-right whitespace-nowrap">{{ playedAgo(item.lastPlayedAt, '日前') }}</span>
              </span>
            </div>
          </div>
        </div>
    </div>

    <div v-if="page === 'songs' && !isLoading && !filteredData.length" class="text-center text-gray-500 mt-12">No songs here.</div>
    <div v-if="page === 'songs' && isLoading" class="text-center text-gray-500 mt-12">Loading songs...</div>

    <!-- Lab page: song card style experiments (preview only, clicks don't count plays) -->
    <div v-if="page === 'lab'">
      <div class="flex flex-wrap gap-2 justify-center mb-4">
        <button
          v-for="style in labStyles"
          :key="style.key"
          @click="labStyle = style.key"
          class="px-3 py-1 rounded border text-sm font-medium transition"
          :class="labStyle === style.key
            ? 'bg-purple-600 text-white border-purple-600'
            : 'bg-white text-gray-700 border-gray-300 hover:bg-purple-50'"
        >
          {{ style.label }}
        </button>
      </div>

      <div v-if="isLoading" class="text-center text-gray-500 mt-12">Loading songs...</div>

      <!-- A: compact list -->
      <div v-else-if="labStyle === 'list'" class="max-w-3xl mx-auto bg-white rounded-lg shadow-md divide-y">
        <div
          v-for="(item, index) in filteredData"
          :key="index"
          class="flex items-center gap-3 px-4 py-2 hover:bg-gray-50"
        >
          <span class="w-5 text-center" :class="item.star == 1 ? 'text-yellow-500' : 'text-gray-300'">
            {{ item.star == 2 ? '❤️' : item.star == 1 ? '★' : '☆' }}
          </span>
          <div class="flex-1 min-w-0">
            <div class="font-semibold text-gray-800 truncate">{{ item.songName }}</div>
            <div class="text-xs text-gray-500 truncate">{{ item.artistName }}</div>
          </div>
          <span class="text-sm text-gray-500 w-12 text-right">{{ item.plays || 0 }}回</span>
          <span class="text-xs text-gray-400 w-14 text-right">{{ playedAgo(item.lastPlayedAt, '日前') }}</span>
        </div>
      </div>

      <!-- B: album tiles -->
      <div v-else-if="labStyle === 'tiles'" class="grid grid-cols-4 gap-4">
        <div v-for="(item, index) in filteredData" :key="index" class="bg-white rounded-lg shadow-md overflow-hidden">
          <div
            class="aspect-square flex items-center justify-center text-5xl font-bold text-white relative"
            :class="artistColor(item.artistName)"
          >
            {{ (item.songName || '?').toString().charAt(0).toUpperCase() }}
            <span v-if="item.star" class="absolute top-2 right-2 text-xl">{{ item.star == 2 ? '❤️' : '★' }}</span>
            <span class="absolute bottom-2 left-2 text-xs bg-black/30 rounded px-2 py-0.5">{{ item.plays || 0 }}回</span>
          </div>
          <div class="p-2">
            <div class="font-semibold text-gray-800 truncate">{{ item.songName }}</div>
            <div class="text-xs text-gray-500 truncate">{{ item.artistName }}</div>
          </div>
        </div>
      </div>

      <!-- C: minimal with freshness bar -->
      <div v-else-if="labStyle === 'minimal'" class="grid grid-cols-3 gap-4">
        <div v-for="(item, index) in filteredData" :key="index" class="bg-white rounded-xl p-4 border border-gray-200">
          <div class="flex justify-between items-start gap-2">
            <h2 class="text-lg font-semibold text-gray-800 leading-tight">{{ item.songName }}</h2>
            <span v-if="item.star" class="shrink-0">{{ item.star == 2 ? '❤️' : '⭐' }}</span>
          </div>
          <p class="text-sm text-gray-500 mb-3">{{ item.artistName }}</p>
          <div class="flex items-center gap-2 text-xs text-gray-500">
            <span class="px-2 py-0.5 rounded-full bg-indigo-50 text-indigo-700 font-medium">{{ item.plays || 0 }} plays</span>
            <div class="flex-1 h-1.5 bg-gray-100 rounded-full overflow-hidden">
              <div class="h-full bg-indigo-400 rounded-full" :style="{ width: freshness(item) + '%' }"></div>
            </div>
            <span>{{ playedAgo(item.lastPlayedAt, 'd') }}</span>
          </div>
        </div>
      </div>

      <!-- D: recency accent -->
      <div v-else-if="labStyle === 'accent'" class="grid grid-cols-3 gap-4">
        <div
          v-for="(item, index) in filteredData"
          :key="index"
          class="bg-white shadow-md p-3 border-l-4"
          :class="recencyBorder(item)"
        >
          <div class="flex justify-between items-center">
            <h2 class="text-lg font-semibold text-gray-800 truncate">{{ item.songName }}</h2>
            <span class="text-2xl font-bold text-gray-300">{{ item.plays || 0 }}</span>
          </div>
          <div class="flex justify-between text-sm text-gray-600">
            <span class="truncate">{{ item.star == 2 ? '❤️ ' : item.star == 1 ? '★ ' : '' }}{{ item.artistName }}</span>
            <span class="text-xs shrink-0">{{ playedAgo(item.lastPlayedAt, '日前') }}</span>
          </div>
        </div>
      </div>

      <!-- D2: recency border + star tint, rounded -->
      <div v-else-if="labStyle === 'accent2'" class="grid grid-cols-3 gap-3">
        <div
          v-for="(item, index) in filteredData"
          :key="index"
          class="rounded-r-lg shadow-sm hover:shadow-md transition p-3 border-l-4"
          :class="[recencyBorder(item), getBgColor(item) || 'bg-white']"
        >
          <div class="flex justify-between items-center gap-2">
            <h2 class="font-semibold text-gray-800 truncate">
              <span v-if="item.star" class="mr-1">{{ item.star == 2 ? '❤️' : '★' }}</span>{{ item.songName }}
            </h2>
            <span class="text-xs font-semibold text-gray-500 shrink-0">{{ item.plays || 0 }}回</span>
          </div>
          <div class="flex justify-between text-xs text-gray-500 mt-0.5">
            <span class="truncate">{{ item.artistName }}</span>
            <span class="shrink-0">{{ playedAgo(item.lastPlayedAt, '日前') }}</span>
          </div>
        </div>
      </div>

      <!-- D3: dense 3 columns (no hover, iPad friendly). Click card = open link, star = like/love, 📁 = playlists -->
      <div v-else-if="labStyle === 'accent3'">
      <div class="flex flex-wrap gap-2 justify-center mb-4">
        <button
          v-for="theme in d3Themes"
          :key="theme.key"
          @click="d3Theme = theme.key"
          class="px-3 py-1 rounded-full border text-xs font-medium transition"
          :class="d3Theme === theme.key
            ? 'bg-gray-800 text-white border-gray-800'
            : 'bg-white text-gray-700 border-gray-300'"
        >
          {{ theme.label }}
        </button>
      </div>
      <div class="grid grid-cols-3 gap-2 rounded-xl" :class="d3ThemeStyle.wrap" :style="d3ThemeStyle.wrapStyle">
        <div
          v-for="(item, index) in filteredData"
          :key="index"
          class="group transition px-2 py-1.5 border-l-4 flex items-center gap-2 cursor-pointer"
          :class="d3CardClass(item)"
          :style="d3CardStyle(item, index)"
          @click="handleCardClick(item)"
        >
          <div class="shrink-0 flex flex-col items-center -ml-1">
            <button
              @click.stop="toggleStar(item)"
              class="w-7 h-6 flex items-center justify-center transition active:scale-125"
              :class="item.star == 1 ? 'text-amber-500' : item.star == 2 ? 'text-rose-500' : d3ThemeStyle.emptyStar"
              :title="item.star == 2 ? 'Loved' : item.star == 1 ? 'Liked' : 'Like'"
            >
              <AppIcon :name="item.star == 2 ? 'heart' : 'star'" :filled="item.star == 1 || item.star == 2" :size="16" />
            </button>
            <button
              @click.stop="openPlaylistModal(item)"
              class="w-7 h-6 flex items-center justify-center transition"
              :class="(item.playlists || []).length ? 'text-purple-500' : d3ThemeStyle.emptyStar"
              title="Add to playlist"
            >
              <AppIcon name="folder" :filled="(item.playlists || []).length > 0" :size="14" />
            </button>
          </div>
          <div class="flex-1 min-w-0">
            <div class="truncate" :class="d3ThemeStyle.title">{{ item.songName }}</div>
            <div class="flex items-center gap-2 text-[11px]" :class="d3ThemeStyle.sub">
              <span class="flex-1 min-w-0 truncate" @click.stop="filterArtist(item.artistName)">{{ item.artistName }}</span>
              <span class="shrink-0 flex gap-1" :class="d3ThemeStyle.meta">
                <span class="w-7 text-right font-semibold whitespace-nowrap">{{ item.plays || 0 }}回</span>
                <span class="w-12 text-right whitespace-nowrap">{{ playedAgo(item.lastPlayedAt, '日前') }}</span>
              </span>
            </div>
          </div>
        </div>
      </div>
      </div>

      <!-- D4: big play count block on the left, colored by recency -->
      <div v-else-if="labStyle === 'accent4'" class="grid grid-cols-3 gap-3">
        <div v-for="(item, index) in filteredData" :key="index" class="bg-white shadow-sm rounded-lg flex overflow-hidden">
          <div class="w-14 shrink-0 flex flex-col items-center justify-center" :class="recencyBlock(item)">
            <span class="text-xl font-bold leading-none">{{ item.plays || 0 }}</span>
            <span class="text-[10px]">回</span>
          </div>
          <div class="p-2 min-w-0 flex-1">
            <h2 class="font-semibold text-gray-800 truncate">{{ item.songName }}</h2>
            <div class="flex justify-between text-xs text-gray-500">
              <span class="truncate">{{ item.star == 2 ? '❤️ ' : item.star == 1 ? '★ ' : '' }}{{ item.artistName }}</span>
              <span class="shrink-0">{{ playedAgo(item.lastPlayedAt, '日前') }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- D5: no border, small recency dot next to the title -->
      <div v-else-if="labStyle === 'accent5'" class="grid grid-cols-3 gap-3">
        <div v-for="(item, index) in filteredData" :key="index" class="bg-white shadow-sm rounded-lg p-3">
          <div class="flex items-center gap-2">
            <span class="w-2 h-2 rounded-full shrink-0" :class="recencyBlock(item)"></span>
            <h2 class="font-semibold text-gray-800 truncate flex-1">{{ item.songName }}</h2>
            <span class="text-lg font-bold text-gray-300 shrink-0">{{ item.plays || 0 }}</span>
          </div>
          <div class="flex justify-between text-xs text-gray-500 pl-4">
            <span class="truncate">{{ item.star == 2 ? '❤️ ' : item.star == 1 ? '★ ' : '' }}{{ item.artistName }}</span>
            <span class="shrink-0">{{ playedAgo(item.lastPlayedAt, '日前') }}</span>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
import AppIcon from './components/AppIcon.vue';

export default {
  components: { AppIcon },
  data() {
    return {
      fetchedData: null,
      baseUrl: 'https://script.google.com/macros/s/AKfycbxB0_D-Q46UKqrTq-fShg6aEBRp87Fdofc_e_X08vvWQMCrtfsMvamLUfmxKTktijYv_g/exec',
      sortKey: 'lastPlayedAt',
      sortAsc: false,
      filteredArtist: null,
      starFilter: false,

      basePlaylists: ['Becca', 'Cozy', 'Practicing'],
      playlistLabels: {
        Becca: 'Sing with Becca',
        Cozy: 'Cozy Songs',
        Practicing: 'Practicing'
      },
      newPlaylistName: '',
      playlistFilter: null,
      showPlaylistMenu: false,
      showSortMenu: false,
      isScrolled: false,
      luckyPicks: null,
      sortOptions: [
        { key: 'lastPlayedAt', label: 'Recent' },
        { key: 'artistName', label: 'Artist' },
        { key: 'plays', label: 'Plays' },
        { key: 'songName', label: 'Song' }
      ],
      showPlaylistModal: false,
      playlistModalSong: null,

      page: 'songs',
      labStyle: 'accent3',
      d3Theme: 'wood',
      d3Themes: [
        { key: 'plain', label: 'Plain' },
        { key: 'wood', label: '🪵 Wood' },
        { key: 'fretboard', label: '🎸 Fretboard' },
        { key: 'setlist', label: '📝 Setlist' },
        { key: 'stage', label: '🌙 Stage' },
        { key: 'cassette', label: '📼 Cassette' },
        { key: 'vinyl', label: '💿 Vinyl' },
        { key: 'ticket', label: '🎟️ Ticket-1' },
        { key: 'ticket2', label: '🎫 Ticket-2' },
        { key: 'ticket3', label: '🎫 Ticket-3' },
        { key: 'ticket4', label: '🎫 Ticket-4' },
        { key: 'ticket5', label: '🎫 Ticket-5' },
        { key: 'ticket6', label: '🎫 Ticket-6' },
        { key: 'neon', label: '💡 Neon' },
        { key: 'chalk', label: '🖍️ Chalkboard' },
        { key: 'sunset', label: '🌅 Sunset glass' }
      ],
      labStyles: [
        { key: 'list', label: 'A · List' },
        { key: 'tiles', label: 'B · Tiles' },
        { key: 'minimal', label: 'C · Minimal' },
        { key: 'accent', label: 'D1 · Recency' },
        { key: 'accent2', label: 'D2 · Tinted' },
        { key: 'accent3', label: 'D3 · Dense' },
        { key: 'accent4', label: 'D4 · Big count' },
        { key: 'accent5', label: 'D5 · Dot' }
      ],
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
          this.fetchedData = data.data
            .map(item => ({
              ...item,
              playlists: (item.playlists || '').toString().split(',').filter(Boolean)
            }))
            .sort((a, b) => b.lastPlayedAt - a.lastPlayedAt);
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

      if (!this.fetchedData) return;
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

    openPlaylistModal(song) {
        this.playlistModalSong = song;
        this.showPlaylistModal = true;
    },

    closePlaylistModal() {
        this.showPlaylistModal = false;
        this.playlistModalSong = null;
        this.newPlaylistName = '';
    },

    playlistLabel(playlist) {
        return this.playlistLabels[playlist] || playlist;
    },

    addNewPlaylist() {
        const name = this.newPlaylistName.trim();
        if (!name || !this.playlistModalSong) return;

        this.togglePlaylist(this.playlistModalSong, name);
        this.newPlaylistName = '';
    },

    togglePlaylist(song, playlist) {
        const current = song.playlists || [];
        song.playlists = current.includes(playlist)
            ? current.filter(p => p !== playlist)
            : [...current, playlist]; // Optimistically update UI

        const url = `${this.baseUrl}?callback=jsonpCallback&action=togglePlaylist&songName=${encodeURIComponent(song.songName)}&artistName=${encodeURIComponent(song.artistName)}&playlist=${encodeURIComponent(playlist)}`;

        window.jsonpCallback = (data) => {
            console.log("API Response (togglePlaylist):", data);
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

    artistColor(name) {
      const colors = ['bg-indigo-500', 'bg-rose-500', 'bg-amber-500', 'bg-emerald-500', 'bg-sky-500', 'bg-purple-500', 'bg-teal-500', 'bg-orange-500'];
      const str = (name || '').toString();
      let hash = 0;
      for (let i = 0; i < str.length; i++) hash = (hash * 31 + str.charCodeAt(i)) >>> 0;
      return colors[hash % colors.length];
    },
    // 100% = played today, 0% = 60+ days ago
    freshness(item) {
      const days = Number(this.daysAgo(item.lastPlayedAt));
      return isNaN(days) ? 0 : Math.max(0, 100 - (days / 60) * 100);
    },
    recencyBorder(item) {
      const days = Number(this.daysAgo(item.lastPlayedAt));
      if (days <= 7) return 'border-emerald-500';
      if (days <= 30) return 'border-amber-400';
      return 'border-gray-300';
    },
    // "12日前" style label, or a dash when the song has never been played
    playedAgo(timestamp, suffix) {
      const days = this.daysAgo(timestamp);
      return days === 'N/A' ? '—' : days + suffix;
    },
    // Card look for the D3 lab style, per theme. Recency still drives the left border color.
    d3CardClass(item, theme = this.d3Theme) {
      if (theme === 'plain') return ['shadow-sm hover:shadow-md', this.recencyBorder(item), this.getBgColor(item) || 'bg-white'];
      if (theme === 'wood') return ['rounded-r-md shadow hover:shadow-lg', this.recencyBorder(item)];
      if (theme === 'fretboard') return ['rounded-r-md shadow-md hover:shadow-xl', this.recencyBorder(item)];
      if (theme === 'setlist') return ['shadow hover:shadow-md border-l-red-300'];
      if (theme === 'cassette') return ['rounded-md shadow border-l-0 pt-3'];
      if (theme === 'vinyl') return ['rounded-l-md rounded-r-full shadow-lg border-l-0 pr-10'];
      if (theme === 'ticket2') return ['rounded-md shadow-sm border-l-0 border-r-2 border-dashed', this.recencyBorder(item).replace('border-', 'border-r-')];
      if (/^ticket\d?$/.test(theme)) return ['rounded-md shadow-sm border-l-0 border-r-2 border-dashed border-r-gray-300'];
      if (theme === 'neon') return ['rounded-md border border-l'];
      if (theme === 'chalk') return ['rounded-sm border-l-2', this.recencyBorder(item)];
      if (theme === 'sunset') return ['rounded-xl border border-white/40 border-l-4 backdrop-blur-md shadow-lg', this.recencyBorder(item)];
      return ['rounded-r-md hover:brightness-125', this.recencyBorder(item)];
    },
    d3CardStyle(item, index, theme = this.d3Theme) {
      const loved = item.star == 2, liked = item.star == 1;
      if (theme === 'wood') {
        const base = loved ? '#f2d2c4' : liked ? '#f4dfa8' : '#efdcb8';
        return {
          background: `repeating-linear-gradient(92deg, rgba(110,60,20,.07) 0 1px, transparent 1px 7px),
            repeating-linear-gradient(88deg, rgba(110,60,20,.05) 0 2px, transparent 2px 23px), ${base}`
        };
      }
      if (theme === 'fretboard') {
        return {
          background: `repeating-linear-gradient(180deg, transparent 0 7px, rgba(235,215,175,.22) 7px 8px),
            linear-gradient(90deg, transparent 0 calc(100% - 3px), rgba(220,220,220,.35) calc(100% - 3px)),
            ${loved ? '#4a1f1f' : liked ? '#4a3518' : '#3a2418'}`
        };
      }
      if (theme === 'setlist') {
        return {
          background: `repeating-linear-gradient(180deg, transparent 0 15px, #dbe7f5 15px 16px), ${loved ? '#fff1f2' : liked ? '#fffbe6' : '#fffdf7'}`,
          transform: `rotate(${index % 3 === 0 ? -0.6 : index % 3 === 1 ? 0.4 : -0.2}deg)`
        };
      }
      const days = Number(this.daysAgo(item.lastPlayedAt));
      const recent = days <= 7 ? 0 : days <= 30 ? 1 : 2;
      if (theme === 'cassette') {
        const stripes = [['#f97316', '#facc15'], ['#14b8a6', '#38bdf8'], ['#a78bfa', '#f472b6']][recent];
        return {
          background: `linear-gradient(180deg, ${stripes[0]} 0 4px, ${stripes[1]} 4px 8px, transparent 8px),
            ${loved ? '#fde8e4' : liked ? '#fdf3d6' : '#f5efe2'}`
        };
      }
      if (theme === 'vinyl') {
        const label = loved ? '#e11d48' : liked ? '#f59e0b' : '#64748b';
        return {
          background: `radial-gradient(circle at calc(100% - 22px) 50%, ${label} 0 6px, #111 6px 8px, transparent 8px),
            repeating-radial-gradient(circle at calc(100% - 22px) 50%, transparent 0 2px, rgba(255,255,255,.07) 2px 3px), #18181b`
        };
      }
      if (theme === 'ticket') {
        const color = loved ? '#ffe4e6' : liked ? '#fef3c7' : ['#dcfce7', '#e0f2fe', '#f1f5f9'][recent];
        return {
          background: `radial-gradient(circle at 0 50%, #e5e7eb 0 7px, transparent 7.5px), radial-gradient(circle at 100% 50%, #e5e7eb 0 7px, transparent 7.5px), ${color}`
        };
      }
      // Ticket-2..6: variations on Ticket-1 (same shape: side notches + dashed right edge)
      const notches = (wrapColor) => `radial-gradient(circle at 0 50%, ${wrapColor} 0 7px, transparent 7.5px), radial-gradient(circle at 100% 50%, ${wrapColor} 0 7px, transparent 7.5px)`;
      if (theme === 'ticket2') {
        return { background: `${notches('#e5e7eb')}, ${loved ? '#fff1f2' : liked ? '#fffbeb' : '#ffffff'}` };
      }
      if (theme === 'ticket3') {
        const color = loved ? '#ffe4e6' : liked ? '#fef3c7' : ['#dcfce7', '#e0f2fe', '#f1f5f9'][recent];
        return {
          background: `radial-gradient(circle at 5px 50%, #cbd5e1 0 2.5px, transparent 3px) 0 0 / 10px 9px repeat-y, ${color}`
        };
      }
      if (theme === 'ticket4') {
        const color = loved ? '#ead1cf' : liked ? '#ece0c2' : ['#d5e3d2', '#d3dfe8', '#e3e1dc'][recent];
        return { background: `${notches('#e5e7eb')}, ${color}` };
      }
      if (theme === 'ticket5') {
        const color = loved ? '#ffe4e6' : liked ? '#fef3c7' : ['#dcfce7', '#e0f2fe', '#f1f5f9'][recent];
        return {
          background: `${notches('#e5e7eb')}, repeating-linear-gradient(135deg, rgba(255,255,255,.55) 0 2px, transparent 2px 7px), ${color}`
        };
      }
      if (theme === 'ticket6') {
        const color = loved ? '#ffe4e6' : liked ? '#fef3c7' : ['#dcfce7', '#e0f2fe', '#f1f5f9'][recent];
        const corner = (pos) => `radial-gradient(circle at ${pos}, #e5e7eb 0 8px, transparent 8.5px)`;
        return { background: `${corner('0 0')}, ${corner('100% 0')}, ${corner('0 100%')}, ${corner('100% 100%')}, ${color}` };
      }
      if (theme === 'neon') {
        const c = loved ? '244,63,94' : liked ? '250,204,21' : ['34,211,238', '217,70,239', '100,116,139'][recent];
        return {
          background: '#09090f',
          borderColor: `rgb(${c})`,
          boxShadow: `0 0 8px rgba(${c},.6), inset 0 0 8px rgba(${c},.25)`
        };
      }
      if (theme === 'chalk') {
        return {
          background: `radial-gradient(ellipse at 30% 20%, rgba(255,255,255,.06), transparent 60%), ${loved ? '#3d3a2c' : liked ? '#344232' : '#2f3d33'}`
        };
      }
      if (theme === 'sunset') {
        return { background: loved ? 'rgba(255,228,230,.55)' : liked ? 'rgba(254,243,199,.55)' : 'rgba(255,255,255,.35)' };
      }
      if (theme === 'stage') {
        const days = Number(this.daysAgo(item.lastPlayedAt));
        const glow = days <= 7 ? '16,185,129' : days <= 30 ? '251,191,36' : '148,163,184';
        return {
          background: loved ? '#2a1424' : liked ? '#231f12' : '#141826',
          boxShadow: `-6px 0 14px -6px rgba(${glow},.8)`
        };
      }
      return {};
    },
    themeStyle(key) {
      const themes = {
        plain: { wrap: '', title: 'text-sm font-semibold text-gray-800', sub: 'text-gray-500', meta: 'text-gray-400', emptyStar: 'text-gray-300' },
        wood: { wrap: 'p-3', wrapStyle: { background: '#5b3a22' }, title: 'text-sm font-bold text-amber-950', sub: 'text-amber-900/70', meta: 'text-amber-900/60', emptyStar: 'text-amber-900/30' },
        fretboard: { wrap: 'p-3', wrapStyle: { background: '#1f1410' }, title: 'text-sm font-semibold text-amber-50', sub: 'text-amber-100/60', meta: 'text-amber-100/50', emptyStar: 'text-amber-100/30' },
        setlist: { wrap: 'p-3', wrapStyle: { background: '#c9a77c' }, title: 'text-xl leading-6 text-gray-900 [font-family:Caveat,cursive] font-bold', sub: 'text-gray-500', meta: 'text-gray-400', emptyStar: 'text-gray-300' },
        stage: { wrap: 'p-3', wrapStyle: { background: '#0a0d16' }, title: 'text-sm font-semibold text-slate-100', sub: 'text-slate-400', meta: 'text-slate-500', emptyStar: 'text-slate-600' },
        cassette: { wrap: 'p-3', wrapStyle: { background: '#3f3a36' }, title: 'text-sm font-bold uppercase tracking-wide text-stone-800 [font-family:ui-monospace,monospace]', sub: 'text-stone-500', meta: 'text-stone-400', emptyStar: 'text-stone-300' },
        vinyl: { wrap: 'p-3', wrapStyle: { background: '#e7e2d8' }, title: 'text-sm font-semibold text-zinc-100', sub: 'text-zinc-400', meta: 'text-zinc-500', emptyStar: 'text-zinc-600' },
        ticket: { wrap: 'p-3', wrapStyle: { background: '#e5e7eb' }, title: 'text-sm font-extrabold uppercase tracking-tight text-gray-800', sub: 'text-gray-500', meta: 'text-gray-500', emptyStar: 'text-gray-300' },
        ticket2: { wrap: 'p-3', wrapStyle: { background: '#e5e7eb' }, title: 'text-sm font-extrabold uppercase tracking-tight text-gray-800', sub: 'text-gray-500', meta: 'text-gray-500', emptyStar: 'text-gray-300' },
        ticket3: { wrap: 'p-3', wrapStyle: { background: '#e5e7eb' }, title: 'text-sm font-extrabold uppercase tracking-tight text-gray-800', sub: 'text-gray-500', meta: 'text-gray-500', emptyStar: 'text-gray-400/70' },
        ticket4: { wrap: 'p-3', wrapStyle: { background: '#e5e7eb' }, title: 'text-sm font-serif font-bold text-stone-800', sub: 'text-stone-500', meta: 'text-stone-500', emptyStar: 'text-stone-400' },
        ticket5: { wrap: 'p-3', wrapStyle: { background: '#e5e7eb' }, title: 'text-sm font-extrabold uppercase tracking-tight text-gray-800', sub: 'text-gray-500', meta: 'text-gray-500', emptyStar: 'text-gray-300' },
        ticket6: { wrap: 'p-3', wrapStyle: { background: '#e5e7eb' }, title: 'text-sm font-extrabold uppercase tracking-tight text-gray-800', sub: 'text-gray-500', meta: 'text-gray-500', emptyStar: 'text-gray-300' },
        neon: { wrap: 'p-4 gap-3', wrapStyle: { background: '#030307' }, title: 'text-sm font-semibold text-white [text-shadow:0_0_6px_rgba(255,255,255,.7)]', sub: 'text-fuchsia-300/70', meta: 'text-cyan-300/70', emptyStar: 'text-zinc-600' },
        chalk: { wrap: 'p-3', wrapStyle: { background: '#24302a', border: '8px solid #7a5a3a' }, title: 'text-xl leading-6 text-white/90 [font-family:Caveat,cursive] font-bold', sub: 'text-white/50', meta: 'text-white/40', emptyStar: 'text-white/30' },
        sunset: { wrap: 'p-4', wrapStyle: { background: 'linear-gradient(135deg, #f97316 0%, #ec4899 50%, #8b5cf6 100%)' }, title: 'text-sm font-bold text-gray-900', sub: 'text-gray-800/70', meta: 'text-gray-800/60', emptyStar: 'text-white/70' }
      };
      return themes[key] || themes.plain;
    },
    onScroll() {
      this.isScrolled = window.scrollY > 4;
    },
    // 6 random songs that haven't been played since midnight today
    pickLucky() {
      const startOfToday = new Date().setHours(0, 0, 0, 0) / 1000;
      const pool = (this.fetchedData || []).filter(song => !song.lastPlayedAt || song.lastPlayedAt < startOfToday);
      for (let i = pool.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [pool[i], pool[j]] = [pool[j], pool[i]];
      }
      this.luckyPicks = pool.slice(0, 6);
    },
    recencyBlock(item) {
      const days = Number(this.daysAgo(item.lastPlayedAt));
      if (days <= 7) return 'bg-emerald-500 text-white';
      if (days <= 30) return 'bg-amber-400 text-white';
      return 'bg-gray-200 text-gray-600';
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
    currentSortLabel() {
      return (this.sortOptions.find(option => option.key === this.sortKey) || {}).label;
    },
    d3ThemeStyle() {
      return this.themeStyle(this.d3Theme);
    },
    // Songs page uses the Ticket-3 look picked in the Lab
    songCardStyle() {
      return this.themeStyle('ticket3');
    },
    filteredData() {
        if (!this.fetchedData) return [];
        if (this.luckyPicks) return this.luckyPicks;

        let data = this.fetchedData;

        if (this.filteredArtist) {
            data = data.filter(item => item.artistName === this.filteredArtist);
        }

        if (this.starFilter) {
            data = data.filter(item => item.star === 1 || item.star === 2);
        }

        if (this.playlistFilter) {
            data = data.filter(item => (item.playlists || []).includes(this.playlistFilter));
        }

        return data;
    },
    currentTuningNote() {
      return this.tempStatus[this.selectedNoteIndex];
    },
    playlists() {
      const fromSongs = (this.fetchedData || []).flatMap(s => s.playlists || []);
      return [...new Set([...this.basePlaylists, ...fromSongs])];
    },
    sortedPlaylistsForModal() {
      if (!this.playlistModalSong) return this.playlists;
      const checked = this.playlistModalSong.playlists || [];
      return [...this.playlists].sort((a, b) => checked.includes(b) - checked.includes(a));
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
    window.addEventListener('scroll', this.onScroll, { passive: true });
  },
  beforeUnmount() {
    this.stopTuner()
    window.removeEventListener('scroll', this.onScroll);
  }
};
</script>

<style>
  html, body {
    background: #e5e7eb;
    @apply font-sans;
  }
</style>
