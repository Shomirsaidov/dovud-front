<div class="p-6 max-w-3xl mx-auto bg-white">
    <h1 class="text-2xl text-start font-bold mb-6">Параметры публикаций у объявлений (постов)</h1>

    <form @submit.prevent="handleSubmit" class="space-y-4 text-[#62560E]">
      <div class="space-y-2">
        <div v-for="(label, key) in checkboxes" :key="key" class="flex items-center space-x-2">
          <input type="checkbox" :id="key" v-model="form[key]" class="w-4 h-4 text-yellow-600 border-gray-300 rounded text-[#62560E]" />
          <label :for="key" class="text-[#62560E]">{{ label }}</label>
        </div>

        <div class="flex items-center space-x-2">
          <label for="salaryThreshold" class="text-[#62560E]">У зарплата руб./месяц если сумма менее</label>
          <input type="number" v-model="form.salaryThreshold" id="salaryThreshold" class="w-24 px-2 py-1 border rounded" />
          <span>менять на текст "По договоренности"</span>
        </div>
      </div>

      <div class="flex items-center space-x-8 py-8">


        

        <button type="submit" class="border border-yellow-800 bg-yellow-400 space-x-3 px-4 py-2" :disabled="isLoading">
          {{ isLoading ? 'Отправляется...' : 'ПАРСИНГ' }}
        </button>

        <div class="">
          <label class="block mb-1 font-medium text-gray-700">Загрузите XML файл:</label>
          <input type="file" @change="handleFileUpload" accept=".xml" class="border rounded px-2 py-1" />
        </div>

      

      </div>
    </form>

    <!-- Loading spinner -->
    <div v-if="isLoading" class="mt-6 flex items-center space-x-2">
      <svg class="animate-spin h-6 w-6 text-yellow-500" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v8z"></path>
      </svg>
      <span class="text-yellow-700 font-medium">Публикация может занять несколько минут. Пожалуйста, не закрывайте окно.</span>
    </div>

    <div v-if="response && !isLoading" class="mt-6">
      <h2 class="text-xl font-bold mb-2">Результаты публикации</h2>
      <div v-for="(result, idx) in response.results" :key="idx" class="mb-4 p-4 border rounded">
        <p class="font-medium">Вакансии: {{ result.job_titles.join(', ') }}</p>
        <p v-if="result.success" class="text-green-600">✅ Успешно опубликовано: <a :href="result.link" target="_blank" class="underline text-blue-600">Смотреть пост</a></p>
        <p v-else class="text-red-600">❌ Ошибка: {{ result.error }}</p>
      </div>
    </div>
  </div>



  <script>
import axios from 'axios';

export default {
  data() {
    return {
      form: {
        groupByVacancyName: false,
        groupByAccountNumber: false,
        hideCompanyName: false,
        salaryThreshold: 50000,
        includeConditions: false,
        includeResponsibilities: false,
        includeRequirements: false,
        hideAddress: false,
        hideEmail: false
      },
      file: null,
      response: null,
      isLoading: false,
      checkboxes: {
        groupByVacancyName: 'Объединений объявления по названию вакансии.',
        groupByAccountNumber: 'Режим объединений объявления по номеру счета',
        hideCompanyName: 'Не публиковать названия компаний',
        includeConditions: 'Добавлять <DOPINFORMSUSLOVIY>',
        includeResponsibilities: 'Добавлять <DOPINFORMSOBYAZANOSTI>',
        includeRequirements: 'Добавлять <DOPINFORMSTREBOVANIY>',
        hideAddress: 'Без адреса',
        hideEmail: 'Без email'
      }
    };
  },
  methods: {
    handleFileUpload(event) {
      this.file = event.target.files[0];
    },
    async handleSubmit() {
      if (!this.file) {
        alert('Пожалуйста, выберите XML файл');
        return;
      }

      const formData = new FormData();
      formData.append('file', this.file);

      for (const key in this.form) {
        formData.append(key, this.form[key]);
      }

      this.isLoading = true;
      this.response = null;

      // Fire-and-forget: отправляем и не ждём ответа
      axios.post(`${process.env.VUE_APP_API_URL}/upload-and-publish`, formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        },
        timeout: 10000 // только для избежания висящего запроса в axios
      }).catch(() => {
        // Ничего не делаем, т.к. сервер всё равно обрабатывает
      }).finally(() => {
        this.isLoading = false;
        alert('Файл отправлен на обработку. Публикация займёт до 10 минут.'); // по желанию
      });
    }
  }
};
</script>
