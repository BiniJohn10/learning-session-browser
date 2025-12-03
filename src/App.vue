<template>
  <div class="min-h-screen bg-slate-100 text-slate-900" aria-live="polite">
    <AppHeader
      :query="query"
      :sort-desc="sortDesc"
      :simulate-next-error="simulateNextError"
      :loading="loading"
      @update:query="val => (query = val)"
      @toggle-sort="toggleSort"
      @update:simulateNextError="val => (simulateNextError = val)"
    />

    <section
      :aria-busy="loading ? 'true' : 'false'"
      class="max-w-6xl mx-auto px-4 py-8 space-y-4"
    >
      <ErrorAlert
        v-if="error"
        :message="error"
        :loading="loading"
        @retry="retry"
      />

      <LoadingState
        v-else-if="loading"
      />

      <SessionsList
        v-else
        :sessions="displayedSessions"
        :coerce-mins="coerceMins"
        :format-date="formatDate"
        :is-completed="isCompleted"
        :toggle-completed="toggleCompleted"
      />
    </section>
  </div>
</template>

<script setup>
import { ref, reactive, watch, onMounted, computed } from 'vue'
import AppHeader from './components/AppHeader.vue'
import ErrorAlert from './components/ErrorAlert.vue'
import LoadingState from './components/LoadingState.vue'
import SessionsList from './components/SessionsList.vue'

const SIM_ERROR_KEY = 'next-error'

function readSimError() {
  try { return localStorage.getItem(SIM_ERROR_KEY) === '1' } catch { return false }
}
function writeSimError(v) {
  try { localStorage.setItem(SIM_ERROR_KEY, v ? '1' : '0') } catch {}
}

const loading = ref(false)
const error = ref(null)
const sessions = ref([])
const query = ref('')
const debouncedQuery = ref('')
const sortDesc = ref(true)
const simulateNextError = ref(readSimError())
const completedIds = reactive(new Set())

const initialLoad = ref(true)

let debounceTimer = null

function coerceMins(mins) {
  const n = Number(mins)
  return Number.isFinite(n) ? n : 0
}

function formatDate(dt) {
  try { return new Date(dt).toLocaleDateString() } catch { return '—' }
}

function escapeRegex(s) {
  return s.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')
}

function sortByPopularity(list, desc = true) {
  return [...list].sort((a, b) => {
    const diff = (a.popularity ?? 0) - (b.popularity ?? 0)
    if (diff === 0) return a._originalIndex - b._originalIndex
    return desc ? -diff : diff
  })
}

const displayedSessions = computed(() => {
  let list = sessions.value
  const q = debouncedQuery.value.trim()

  if (q) {
    const re = new RegExp(escapeRegex(q), 'i')
    list = list.filter(s => re.test(s.title || ''))
  }

  return sortByPopularity(list, sortDesc.value)
})

async function fetchSessionsSimulated() {
  loading.value = true
  error.value = null

  try {
    if (initialLoad.value) {
      await new Promise(r => setTimeout(r, 500))
      initialLoad.value = false
    }

    if (simulateNextError.value) {
      simulateNextError.value = false
      writeSimError(false)
      throw new Error('Simulated network error')
    }

    const res = await fetch('/sessions.json')
    if (!res.ok) throw new Error('HTTP ' + res.status)

    const data = await res.json()

    const cloned = data.map((s, i) => ({ ...s, _originalIndex: i }))
    sessions.value = cloned

    completedIds.clear()
    cloned.forEach(s => {
      if (s.completed) completedIds.add(s.id)
    })
  } catch (err) {
    error.value = err?.message ?? 'Error'
    sessions.value = []
  } finally {
    loading.value = false
  }
}

watch(query, () => {
  clearTimeout(debounceTimer)
  debounceTimer = setTimeout(() => {
    debouncedQuery.value = query.value
  }, 300)
})

watch(simulateNextError, v => writeSimError(v))

function toggleSort() {
  sortDesc.value = !sortDesc.value
}

function isCompleted(id) {
  return completedIds.has(id)
}

function toggleCompleted(id) {
  isCompleted(id) ? completedIds.delete(id) : completedIds.add(id)
}

function retry() {
  if (!loading.value) fetchSessionsSimulated()
}

onMounted(() => {
  fetchSessionsSimulated()
})
</script>
