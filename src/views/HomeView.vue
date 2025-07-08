<template>
  <div class="p-4 px-16 text-sm text-[#62560E]">
    <!-- Filters and Info Row -->
    <div class="flex flex-wrap items-center gap-4 mb-2">
      <div class="flex items-center gap-2">
        <label>Дата от</label>
        <input type="date" v-model="filters.dateFrom" class="border rounded px-2 py-1" />
      </div>
      <div class="flex items-center gap-2">
        <label>Дата до</label>
        <input type="date" v-model="filters.dateTo" class="border rounded px-2 py-1" />
      </div>
      <span>КОЛИЧЕСТВО ЗАПИСЕЙ: {{ jobs.length }}</span>
      <select v-model="pagination.limit" class="border rounded px-2 py-1">
        <option :value="1000">1000</option>
        <option :value="500">500</option>
        <option :value="100">100</option>
      </select>
      <div class="flex items-center gap-2">
        <button @click="goToPage(1)" :disabled="pagination.page === 1">⏮</button>
        <button @click="goToPage(pagination.page - 1)" :disabled="pagination.page === 1">◀</button>
        <span>Страница {{ pagination.page }}</span>
        <button @click="goToPage(pagination.page + 1)" :disabled="jobs.length < pagination.limit">▶</button>
      </div>
    </div>

    <!-- Status Filters Row -->
    <div class="flex flex-wrap items-center gap-4 mb-4">
      <div class="flex items-center gap-2">
        <label class="flex items-center gap-1">
          <input type="radio" v-model="filters.status" value="" /> Не опубликованные
        </label>
        <label class="flex items-center gap-1">
          <input type="radio" v-model="filters.status" value="Активные" /> Активные
        </label>
      </div>
      <label class="flex items-center gap-1">
        <input type="checkbox" v-model="filters.onlyWithResponses" /> Показать только с откликами
      </label>
      <button @click="exportToXLSX" class="flex items-center justify-center border border-yellow-800 bg-yellow-400 space-x-3 px-4 py-2">
        <img width="20" src="../assets/download.png" alt="">  
        <div>ЭКСПОРТ В XLSX</div>
      
      </button>
    </div>

    <!-- Table always rendered -->
    <div class="overflow-auto">
      <table class="min-w-full border bg-white text-[#62560E]">
        <thead class="bg-gray-100">
          <tr>
            <th class="border px-2 py-1">№</th>
            <th v-for="col in columns" :key="col.key" class="border px-2 py-1">{{ col.label }}</th>
          </tr>
          <tr>
            <th class="border px-2 py-1"></th>
            <th v-for="col in columns" :key="col.key" class="border px-2 py-1">
              <div v-if="col.searchable" class="relative">
                <input
                  type="text"
                  v-model="columnSearch[col.key]"
                  @input="debouncedFetch"
                  class="border pl-6 pr-1 py-0.5 w-full"
                />
                <img src="../assets/search-icon.png" class="absolute left-1 top-1/2 -translate-y-1/2 h-4 w-4" alt="search" />
              </div>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="loading">
            <td :colspan="columns.length + 1" class="text-center text-gray-500 py-4">Загрузка...</td>
          </tr>
          <tr v-else-if="!jobs.length">
            <td :colspan="columns.length + 1" class="text-center text-gray-500 py-4">Нет данных по выбранным фильтрам</td>
          </tr>
          <tr v-else v-for="(job, index) in jobs" :key="job.id">
            <td class="border px-2 py-1 align-top">{{ (pagination.page - 1) * pagination.limit + index + 1 }}</td>
            <td v-for="col in columns" :key="col.key" class="border px-2 py-1 align-top whitespace-pre-line break-words">
              {{ job[col.key] }}
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue';
import { createClient } from '@supabase/supabase-js';
import debounce from 'lodash.debounce';
import * as XLSX from 'xlsx';  // import SheetJS

const supabase = createClient('https://zmhjuxowzsbvdflmxwlv.supabase.co', 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InptaGp1eG93enNidmRmbG14d2x2Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTE1MTYxNTYsImV4cCI6MjA2NzA5MjE1Nn0.J4B269cOGY9Ef-FNo5XX654ryOLd5NUi9G-CXhgjJ-I');
const jobs = ref([]);
const loading = ref(false);

const columns = [
  { key: 'id', label: 'ID', searchable: true },
  { key: 'job_title', label: 'НАЗВАНИЕ ВАКАНСИИ', searchable: true },
  { key: 'status', label: 'СТАТУС НА САЙТЕ', searchable: true },
  { key: 'publication_date', label: 'ДАТА ПУБЛИКАЦИЯ', searchable: false },
  { key: 'depub_date', label: 'ДАТА ДЕ-ПУБЛИКАЦИЯ', searchable: false },
  { key: 'account_number', label: 'НОМЕР СЧЕТА', searchable: true },
  { key: 'account_date', label: 'ДАТА СЧЕТА', searchable: false },
  { key: 'rubr_atryb', label: 'АТРИБУТ', searchable: true },
  { key: 'company_inn', label: 'ИНН', searchable: true },
  { key: 'company_name', label: 'НАЗВАНИЕ КОМПАНИИ', searchable: true },
  { key: 'phone', label: 'ТЕЛЕФОН', searchable: true },
  { key: 'email', label: 'EMAIL', searchable: true },
  { key: 'address', label: 'АДРЕС', searchable: true },
  { key: 'conditions', label: 'УСЛОВИЯ', searchable: true },
  { key: 'responsibilities', label: 'ОБЯЗАННОСТИ', searchable: true },
  { key: 'requirements', label: 'ТРЕБОВАНИЯ', searchable: true },
  { key: 'schedule', label: 'ГРАФИК', searchable: true },
  { key: 'salary', label: 'ЗАРПЛАТА', searchable: true },
  { key: 'vk_link', label: 'ССЫЛКА НА САЙТЕ', searchable: true }
];

const filters = ref({
  dateFrom: '',
  dateTo: '',
  status: '',
  onlyWithResponses: false
});

const columnSearch = ref({});
columns.forEach(col => columnSearch.value[col.key] = '');

const pagination = ref({ page: 1, limit: 1000 });

const fetchJobs = async () => {
  loading.value = true;
  let query = supabase.from('jobs').select('*');

  if (filters.value.dateFrom) query = query.gte('publication_date', filters.value.dateFrom);
  if (filters.value.dateTo) query = query.lte('publication_date', filters.value.dateTo);
  if (filters.value.status) query = query.eq('status', filters.value.status);
  if (filters.value.onlyWithResponses) query = query.gt('response_count', 0);

  const orFilters = Object.entries(columnSearch.value)
    .filter(([key, val]) => val.trim())
    .map(([key, val]) => `${key}.ilike.${val.trim()}%`)
    .join(',');
  if (orFilters) query = query.or(orFilters);

  const from = (pagination.value.page - 1) * pagination.value.limit;
  const to = from + pagination.value.limit - 1;
  query = query.range(from, to);

  const { data, error } = await query;
  if (!error) jobs.value = data;
  loading.value = false;
};

const debouncedFetch = debounce(fetchJobs, 500);

watch([filters, () => pagination.value.limit], fetchJobs, { deep: true });
watch(() => pagination.value.page, fetchJobs);

const goToPage = (page) => {
  if (page < 1) return;
  pagination.value.page = page;
};

onMounted(fetchJobs);

// --- Export Function ---
const exportToXLSX = () => {
  if (!jobs.value.length) {
    alert('Нет данных для экспорта');
    return;
  }

  // Prepare data for export
  const exportData = jobs.value.map(job => {
    const obj = {};
    columns.forEach(col => {
      obj[col.label] = job[col.key] ?? '';  // Use label as column header, fallback to empty string if missing
    });
    return obj;
  });

  // Create worksheet and workbook
  const worksheet = XLSX.utils.json_to_sheet(exportData);
  const workbook = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(workbook, worksheet, 'Вакансии');

  // Generate binary string and trigger download
  XLSX.writeFile(workbook, 'jobs_export.xlsx');
};
</script>


<style>
table th, table td {
  white-space: nowrap;
  text-align: left;
}
</style>
