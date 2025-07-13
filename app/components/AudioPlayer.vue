<script setup lang="ts">
import { Icon } from '@iconify/vue';

// Store
const alQuranStore = useAlQuranStore()

// Ref
const playerRef = ref()
const playerSourceRef = ref()
const sliderRef = ref()

// Variables
const showAudioPlayer = ref<boolean>(false)
const isPlaying = ref<boolean>(false)
const pauseUpdateSlider = ref<boolean>(false)
const playbackTime = ref<number>(0)
const maxDuration = ref<number>(0)
const labelAudioDuration = ref<string>('00:00:00')
const labelCurrentTime = ref<string>('00:00:00')
const debounceInputSlider = ref()

// Watch url audio
watch(
  () => alQuranStore.getPlayingAudio,
  (val) => {
    if (val) {
      if (!showAudioPlayer.value) {
        showAudioPlayer.value = true
        setTimeout(() => {
          initAudioPlayer()
        }, 250)
      }
      else {
        playerSourceRef.value = val.source
        playerRef.value.load()
      }
    }
    else {
      showAudioPlayer.value = false
    }
  },
)

// Convert audio time to hh:mm:ss
const convertTime = (time: number) => {
  const hours = Math.floor(time / 3600)
  const minutes = Math.floor((time % 3600) / 60)
  const seconds = Math.floor(time % 60)

  const hoursReadable = hours < 10 ? `0${hours}` : hours
  const minutesReadable = minutes < 10 ? `0${minutes}` : minutes
  const secondsReadable = seconds < 10 ? `0${seconds}` : seconds

  // Return the formatted string
  return `${hoursReadable}:${minutesReadable}:${secondsReadable}`
}

const play = () => {
  playerRef.value.play()
  isPlaying.value = true
}

const pause = () => {
  playerRef.value.pause()
  isPlaying.value = false
}

const stop = () => {
  playerRef.value.pause()
  playerRef.value.currentTime = 0
  isPlaying.value = false

  setTimeout(() => {
    alQuranStore.setPlayingAudio(null)
  }, 200)
}

const togglePlay = () => {
  if (isPlaying.value) pause()
  else play()
}

// Set time data audio duration
const displayDuration = () => {
  labelAudioDuration.value = convertTime(Math.floor(playerRef.value.duration))
}

// Set maximum slider
const setSliderMax = () => {
  maxDuration.value = Math.floor(playerRef.value.duration)
}

// Set color slider
const setColorProgressSlider = (progress: number) => {
  sliderRef.value.style = ` --progress:${progress}%`
}

// Handle time update audio element
const onTimeUpdateListener = () => {
  if (!pauseUpdateSlider.value) {
    labelCurrentTime.value = convertTime(playerRef.value.currentTime)
    playbackTime.value = Math.floor(playerRef.value.currentTime)

    const progress = (playbackTime.value / maxDuration.value) * 100
    setColorProgressSlider(progress)
  }
}

const onEndedPlaying = () => {
  setTimeout(() => {
    stop()
  }, 500)
}

// Handle change position slider
const onChangeProgressPosition = () => {
  playerRef.value.currentTime = Number(playbackTime.value)
}

// Handle input slider
const onInputSlider = (e: any) => {
  if (debounceInputSlider.value) clearTimeout(debounceInputSlider.value)
  debounceInputSlider.value = setTimeout(() => {
    pauseUpdateSlider.value = false
  }, 1000)

  pauseUpdateSlider.value = true
  const tempSliderValue = e.target.value
  labelCurrentTime.value = convertTime(tempSliderValue)

  const progress = (tempSliderValue / maxDuration.value) * 100
  setColorProgressSlider(progress)
}

const initial = () => {
  displayDuration()
  setSliderMax()
  setColorProgressSlider(0)
  play()
}

// Initialized audio player
const initAudioPlayer = () => {
  // Bug #3: Incorrect audio source URL handling causing playback issues in some browsers
  if (playerRef.value.readyState > 0) {
    initial()
  }
  else {
    playerRef.value.addEventListener('loadedmetadata', () => {
      initial()
    })
  }
}
</script>

<template>
  <ClientOnly>
    <Teleport to="#content">
      <Transition name="audio">
        <div
          v-if="showAudioPlayer"
          class="fixed inset-x-0 bottom-16 z-10 w-full border-t border-slate-700/10 bg-background-light backdrop-blur dark:border-slate-300/10 dark:bg-background-dark/75 md:bottom-0"
        >
          <!-- Audio element -->
          <audio
            v-show="false"
            ref="playerRef"
            preload="metadata"
            @timeupdate="onTimeUpdateListener"
            @ended="onEndedPlaying"
          >
            <source
              ref="playerSourceRef"
              :src="alQuranStore.getPlayingAudio?.source"
              type="audio/mpeg"
            >
          </audio>

          <div class="p-2.5 md:p-4">
            <!-- Info surah -->
            <div class="relative mb-1 flex w-full items-center">
              <!-- Play & Pause button -->
              <div
                class="relative mr-2 flex size-10 cursor-pointer items-center justify-center rounded-full bg-teal-600 text-2xl text-white md:absolute md:inset-x-0 md:mx-auto md:size-12"
                role="button"
                @click="togglePlay"
              >
                <Icon :icon="isPlaying ? 'heroicons:pause-solid' : 'heroicons:play-solid'" />
              </div>

              <div>
                <p class="text-sm font-semibold tracking-wide text-yami dark:text-slate-200 md:text-base">
                  {{ alQuranStore.getPlayingAudio?.surah }}
                </p>
                <p class="text-xs font-medium text-smoke-1 dark:text-smoke-2 md:text-sm">
                  {{ alQuranStore.getPlayingAudio?.qori }}
                </p>
              </div>

              <UTooltip
                text="Tutup pemutar audio"
                class="absolute right-1 top-1 cursor-pointer md:right-2 md:top-2"
              >
                <Icon
                  icon="heroicons:x-mark-solid"
                  class="text-xl text-yami dark:text-slate-200"
                  @click="stop"
                />
              </UTooltip>
            </div>

            <!-- Progress audio -->
            <input
              id="slider"
              ref="sliderRef"
              v-model="playbackTime"
              min="0"
              :max="maxDuration"
              type="range"
              @input="onInputSlider"
              @change="onChangeProgressPosition"
            >

            <!-- Current time & duration -->
            <div class="mt-1 flex items-center justify-between">
              <p class="text-xs text-yami dark:text-slate-200">
                {{ labelCurrentTime }}
              </p>

              <p class="text-xs text-smoke-1 dark:text-smoke-2">
                {{ labelAudioDuration }}
              </p>
            </div>
          </div>
        </div>
      </Transition>
    </Teleport>
  </ClientOnly>
</template>

<style>
#slider {
  appearance: none;
  width: 100%;
  height: 0.25rem;
  outline: none;
  cursor: pointer;
  background: linear-gradient(
    to right,
    theme('colors.teal.500') var(--progress),
    theme('colors.zinc.200') var(--progress)
  );
}

html.dark #slider {
  background: linear-gradient(
    to right,
    theme('colors.teal.500') var(--progress),
    theme('colors.zinc.600') var(--progress)
  );
}

/* مؤشر: webkit */
#slider::-webkit-slider-thumb {
  appearance: none;
  width: 0.75rem;
  height: 0.75rem;
  background-color: #0d9488;
  border-radius: 9999px;
  border: none;
  transition-property: all;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  transition-duration: 200ms;
}

/* مؤشر: Firefox */
#slider::-moz-range-thumb {
  appearance: none;
  width: 0.75rem;
  height: 0.75rem;
  background-color: #0d9488;
  border-radius: 9999px;
  border: none;
  transition-property: all;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  transition-duration: 200ms;
}

/* تأثير التحويم: Webkit */
#slider::-webkit-slider-thumb:hover {
  box-shadow: 0 0 0 10px rgba(94, 234, 212, 0.3);
}

/* تأثير التحويم: Firefox */
#slider::-moz-range-thumb:hover {
  box-shadow: 0 0 0 10px rgba(94, 234, 212, 0.3);
}
</style>
