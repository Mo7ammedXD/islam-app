<script lang="ts" setup>
// Interfaces
import type { ISchedule } from '@/models/IPrayerTime';

defineProps<{
  dataSchedule: ISchedule[] | undefined
  loading: boolean
}>()

// Variables
const tableColumns = ref<{ key: string; label: string }[]>([
  { key: 'tanggal', label: 'التاريخ' },
  { key: 'imsak', label: 'الإمساك' },
  { key: 'subuh', label: 'الفجر' },
  { key: 'terbit', label: 'الشروق' },
  { key: 'dhuha', label: 'الضحى' },
  { key: 'dzuhur', label: 'الظهر' },
  { key: 'ashar', label: 'العصر' },
  { key: 'maghrib', label: 'المغرب' },
  { key: 'isya', label: 'العشاء' },
])
</script>

<template>
  <div class="rounded-xl bg-teal-600/5 p-4 dark:bg-background-dark-soft md:p-5">
    <UTable
      :loading
      :loading-state="{ icon: 'i-heroicons-arrow-path-20-solid', label: 'جاري التحميل...' }"
      :empty-state="{ icon: 'i-heroicons-circle-stack-20-solid', label: 'لا توجد بيانات.' }"
      :columns="tableColumns"
      :rows="dataSchedule"
    />
  </div>
</template>
