<script setup lang="ts">
// Interface
import type { IHadithItems, IHadithList } from "@/models/IHadith";
// استيراد البيانات الثابتة
import { getHadithByNumber, getHadithsByCollection } from "@/data/hadith";

// Variable
const hadithHistorySelected = ref<string>("abu-dawud");
const page = ref<number>(1);
const limit: number = 20;
const search = ref<string>("");
const isSearching = ref<boolean>(false);
const isErrorSearch = ref<boolean>(false);
const isFinishSearch = ref<boolean>(false);
const searchResult = ref<IHadithItems | null>(null);
const debounceSearch = ref();
const dataListHadith = ref<IHadithList | null>(null);
const statusDataList = ref<string>("success");

// Watch if page is change
watch(page, () => {
  window.scrollTo({
    top: 0,
    behavior: "smooth",
  });
  loadHadiths();
});

watch(search, () => {
  if (debounceSearch.value) clearTimeout(debounceSearch.value);
  debounceSearch.value = setTimeout(() => {
    searchHadithNumber();
  }, 1000);
});

// تحميل الأحاديث من البيانات الثابتة
const loadHadiths = () => {
  statusDataList.value = "pending";
  // محاكاة تأخير الشبكة
  setTimeout(() => {
    dataListHadith.value = getHadithsByCollection(
      hadithHistorySelected.value,
      page.value,
      limit
    );
    statusDataList.value = "success";
  }, 500);
};

// Change hadit history selected
const changeHadithHistory = (value: string) => {
  page.value = 1;
  search.value = "";
  clearResultSearch();
  hadithHistorySelected.value = value;
  loadHadiths();
};

// Search hadith using number of hadith
const searchHadithNumber = () => {
  if (search.value) {
    isErrorSearch.value = false;
    isFinishSearch.value = false;
    isSearching.value = true;

    const history = hadithHistorySelected.value;
    const number = parseInt(search.value);

    // محاكاة تأخير الشبكة
    setTimeout(() => {
      const result = getHadithByNumber(history, number);
      if (result) {
        searchResult.value = result;
        isErrorSearch.value = false;
      } else {
        isErrorSearch.value = true;
      }
      isSearching.value = false;
      isFinishSearch.value = true;
    }, 500);
  } else {
    clearResultSearch();
  }
};

// Clear result search
const clearResultSearch = () => {
  searchResult.value = null;
  isErrorSearch.value = false;
  isFinishSearch.value = false;
};

// تحميل الأحاديث عند تحميل الصفحة
onMounted(() => {
  loadHadiths();
});

// Meta
useHead({
  title: "الأحاديث | تطبيق إسلام",
});
</script>

<template>
  <div>
    <!-- Header -->
    <HadithHeaderList
      v-model="search"
      :hadith-history-selected
      @change-hadith-history="changeHadithHistory"
    />

    <div class="container">
      <!-- Skeleton -->
      <SkeletonHadith v-if="statusDataList === 'pending' || isSearching" />

      <div v-else-if="statusDataList === 'success'" class="mt-5">
        <div class="mb-3">
          <!-- Information total & search hadith -->
          <p
            v-if="!searchResult && dataListHadith?.pagination"
            class="text-sm text-yami dark:text-slate-200 md:text-base"
          >
            يوجد <b>{{ dataListHadith.pagination.totalItems }}</b> حديث
            من
            {{ dataListHadith.name }}
          </p>

          <p v-else class="text-sm text-yami dark:text-slate-200 md:text-base">
            عرض حديث {{ dataListHadith?.name }} رقم
            <b>{{ search }}</b>
          </p>
        </div>

        <!-- List hadith -->
        <div v-if="!isSearching">
          <!-- List all hadith -->
          <div v-if="!isFinishSearch" class="grid grid-cols-1 gap-4">
            <HadithCard
              v-for="list in dataListHadith?.items"
              :key="list.number"
              :hadith-history-icon="dataListHadith?.name"
              :hadith="list"
            />
          </div>

          <!-- Show search result -->
          <HadithCard
            v-if="isFinishSearch && searchResult && !isErrorSearch"
            :hadith-history-icon="dataListHadith?.name"
            :hadith="searchResult"
          />

          <div
            v-if="isFinishSearch && isErrorSearch"
            class="rounded-lg bg-slate-200/50 p-5 text-sm font-normal text-yami dark:bg-slate-700/50 dark:text-slate-200 sm:text-base"
          >
            لم يتم العثور على حديث برقم <b>{{ search }}</b>
          </div>
        </div>

        <!-- Pagination -->
        <div v-if="!search && !isFinishSearch && dataListHadith?.pagination" class="mt-5 flex justify-end">
          <UPagination
            v-model="page"
            :page-count="limit"
            :total="dataListHadith.pagination.totalItems"
            size="sm"
            :max="5"
            :ui="{
              wrapper: 'flex items-center space-x-1',
              base: 'min-w-[32px] h-8 flex items-center justify-center rounded-full',
              rounded: 'first:rounded-full last:rounded-full',
            }"
          />
        </div>
      </div>
    </div>
  </div>
</template>
