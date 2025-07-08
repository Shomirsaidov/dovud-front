<template>
  <div class=" bg-[#FFFCEC] p-6 px-16 space-y-6">
    <!-- Access Token -->
    <div class="w-1/2">
      <label class=" block bg-gray-100 px-4 py-2 font-bold border border-[#A89957] text-left">
        Access Token VK
      </label>
      <input
        v-model="serviceKey"
        class="w-full border border-[#A89957] px-4 py-2 text-[#7C6C17] outline-none"
        placeholder="Введите VK Access Token"
      />
    </div>

    <!-- Owner ID -->
    <div class="w-1/2">
      <label class="block bg-gray-100 px-4 py-2 font-bold border border-[#A89957] text-left">
        ID VK сообщества
      </label>
      <input
        v-model="ownerId"
        class="w-full border border-[#A89957] px-4 py-2 text-[#7C6C17] outline-none"
        placeholder="Введите ID сообщества"
      />
    </div>

    <!-- Save Button -->
    <div class="text-start pt-4">
      <button
        @click="saveTokenData"
        class="bg-[#D4AF37] hover:bg-[#c29e2e] text-white font-semibold px-6 py-2 rounded shadow"
        :disabled="saving"
      >
        {{ saving ? 'Сохраняем...' : 'Сохранить' }}
      </button>

      <p v-if="successMessage" class="text-green-600 mt-2">{{ successMessage }}</p>
      <p v-if="errorMessage" class="text-red-600 mt-2">{{ errorMessage }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { createClient } from '@supabase/supabase-js'

// ✅ Supabase client
const supabase = createClient(
  process.env.VUE_APP_SUPABASE_URL,
  process.env.VUE_APP_SUPABASE_KEY
)

// 🌱 Form state
const serviceKey = ref('')
const ownerId = ref('')
const tokenRowId = ref(null)
const saving = ref(false)
const successMessage = ref('')
const errorMessage = ref('')

// 🔄 Load the only row
const loadTokenData = async () => {
  const { data, error } = await supabase
    .from('access_tokens')
    .select('*')
    .limit(1)
    .single()

  if (error) {
    console.error('Ошибка при загрузке:', error.message)
    errorMessage.value = 'Ошибка при загрузке токенов.'
    return
  }

  if (data) {
    serviceKey.value = data.service_key || ''
    ownerId.value = data.owner_id || ''
    tokenRowId.value = data.id
  }
}

// 💾 Save both fields
const saveTokenData = async () => {
  if (!tokenRowId.value) return

  saving.value = true
  successMessage.value = ''
  errorMessage.value = ''

  const { error } = await supabase
    .from('access_tokens')
    .update({
      service_key: serviceKey.value,
      owner_id: ownerId.value
    })
    .eq('id', tokenRowId.value)

  saving.value = false

  if (error) {
    console.error('Ошибка при сохранении:', error.message)
    errorMessage.value = 'Не удалось сохранить. Проверьте консоль.'
  } else {
    successMessage.value = 'Успешно сохранено.'
  }
}

// 🔄 Fetch on mount
onMounted(() => {
  loadTokenData()
})
</script>
