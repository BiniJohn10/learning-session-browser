<template>
  <div
    role="alert"
    class="relative overflow-hidden rounded-xl border border-red-200/70 bg-red-50/90 px-4 py-4 sm:px-6 sm:py-5 shadow-sm"
  >
    <div class="absolute inset-y-0 left-0 w-1 bg-gradient-to-b from-red-400 to-red-500"></div>
    <div class="ml-4 sm:ml-5">
      <strong class="text-sm font-semibold text-red-800 flex items-center gap-2">
        <span
          class="inline-flex h-6 w-6 items-center justify-center rounded-full bg-red-100 text-red-600 text-sm"
        >
          !
        </span>
        Something went wrong.
      </strong>
      <div class="mt-1 text-sm text-red-700">
        {{ message || "We couldn't load sessions. Please check your connection and try again." }}
      </div>
      <div class="mt-3">
        <button
          @click="onRetry"
          :disabled="loading"
          class="inline-flex items-center gap-2 rounded-full bg-red-600 px-4 py-2 text-sm font-medium text-white shadow-sm transition hover:bg-red-700 hover:shadow-md disabled:opacity-70"
        >
          <span>Retry</span>
          <span aria-hidden="true">⟳</span>
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  message: {
    type: String,
    default: '',
  },
  loading: {
    type: Boolean,
    default: false,
  },
})

const emit = defineEmits(['retry'])

function onRetry() {
  emit('retry')
}
</script>


