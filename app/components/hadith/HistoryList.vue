<script setup lang="ts">
import type { IHadithHistory } from '@/models/IHadith';
// استيراد مجموعات الأحاديث من البيانات الثابتة
import { hadithCollections } from '@/data/hadith';

// Props
defineProps<{
  selected: string
}>()

// Emits
const emits = defineEmits<{
  (e: 'change-hadith-history', value: string): void
}>()

// تحويل مجموعات الأحاديث إلى الصيغة المطلوبة
const hadithHistoryList: IHadithHistory[] = hadithCollections.map(collection => ({
  name: collection.name,
  slug: collection.id
}))
</script>

<template>
  <div
    class="no-scrollbar flex w-full items-center gap-x-2 overflow-x-auto scroll-smooth lg:justify-center"
  >
    <div
      v-for="hadithHistory in hadithHistoryList"
      :key="hadithHistory.slug"
      :class="
        selected === hadithHistory.slug
          ? 'bg-teal-100 dark:bg-teal-800 text-teal-700 dark:text-teal-100'
          : 'bg-gray-100/90 dark:bg-slate-600/60 text-slate-500/80 dark:text-slate-400'
      "
      class="w-min cursor-pointer whitespace-nowrap rounded-full px-3 py-1 text-sm font-medium"
      @click="emits('change-hadith-history', hadithHistory.slug)"
    >
      {{ hadithHistory.name }}
    </div>
  </div>
</template>
