<template>
  <div class="card">
    <h2>{{ t('debugInfo.title') }}</h2>
    <div class="marketplace-actions">
      <button @click="clearDebug">{{ t('debugInfo.clear') }}</button>
      <button @click="exportDebugInfo">{{ t('debugInfo.export') }}</button>
    </div>
    <div ref="debugInfo" class="debug-info"></div>
  </div>
</template>

<script>
import { clearDebug, exportDebugInfo as exportDebug } from '../utils/helpers'
import { getTranslation } from '../utils/i18n'

export default {
  name: 'DebugInfo',
  props: {
    language: {
      type: String,
      default: 'en',
    },
  },
  methods: {
    t(key) {
      return getTranslation(this.language, key)
    },
    addDebugInfo(message) {
      const timestamp = new Date().toLocaleTimeString()
      this.$refs.debugInfo.innerHTML += `[${timestamp}] ${message}<br>`
      this.$refs.debugInfo.scrollTop = this.$refs.debugInfo.scrollHeight
    },
    clearDebug() {
      this.$refs.debugInfo.innerHTML = ''
    },
    exportDebugInfo() {
      const debugInfo = this.$refs.debugInfo.innerText
      const blob = new Blob([debugInfo], { type: 'text/plain' })
      const url = URL.createObjectURL(blob)
      const a = document.createElement('a')
      a.href = url
      a.download = 'debug_info.txt'
      a.click()
      URL.revokeObjectURL(url)
    },
  },
}
</script>

<style scoped></style>
