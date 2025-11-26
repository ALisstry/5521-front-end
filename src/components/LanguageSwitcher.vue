<template>
  <div class="language-switcher">
    <button
      v-for="lang in languages"
      :key="lang.code"
      :class="['lang-btn', { active: currentLanguage === lang.code }]"
      @click="switchLanguage(lang.code)"
    >
      {{ lang.name }}
    </button>
  </div>
</template>

<script>
export default {
  name: 'LanguageSwitcher',
  props: {
    modelValue: {
      type: String,
      default: 'en',
    },
  },
  data() {
    return {
      currentLanguage: this.modelValue,
      languages: [
        { code: 'en', name: 'English' },
        { code: 'zh', name: '中文' },
      ],
    }
  },
  watch: {
    modelValue(newVal) {
      this.currentLanguage = newVal
    },
  },
  methods: {
    switchLanguage(lang) {
      this.currentLanguage = lang
      this.$emit('update:modelValue', lang)
      localStorage.setItem('language', lang)
    },
  },
}
</script>

<style scoped>
.language-switcher {
  display: flex;
  gap: 8px;
  position: fixed;
  top: 10px;
  right: 10px;
  z-index: 1000;
  background: rgba(255, 255, 255, 0.95);
  padding: 8px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.lang-btn {
  padding: 8px 12px;
  border: 1px solid #ddd;
  background: #f5f5f5;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
  font-weight: 500;
  transition: all 0.3s ease;
}

.lang-btn:hover {
  background: #e8e8e8;
  border-color: #999;
}

.lang-btn.active {
  background: #007bff;
  color: white;
  border-color: #0056b3;
  box-shadow: 0 2px 6px rgba(0, 123, 255, 0.3);
}
</style>
