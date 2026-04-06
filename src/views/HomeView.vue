<template>
  <div class="page">
    <div class="card">
      <h1 class="title">CI/CD Test v6</h1>
      <p class="subtitle">백엔드 연결 테스트</p>

      <button class="btn" :class="{ loading: isLoading }" :disabled="isLoading" @click="fetchTest">
        <span v-if="isLoading" class="spinner"></span>
        <span v-else>GET /cicd/test</span>
      </button>

      <transition name="fade">
        <div v-if="response !== null" class="result" :class="isError ? 'error' : 'success'">
          <div class="result-label">{{ isError ? '오류' : '응답' }}</div>
          <div class="result-body">{{ response }}</div>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const response = ref(null)
const isLoading = ref(false)
const isError = ref(false)

async function fetchTest() {
  isLoading.value = true
  response.value = null
  isError.value = false
  try {
    const res = await fetch('http://10.10.10.202/cicd/test')
    const text = await res.text()
    response.value = text
    isError.value = !res.ok
  } catch (e) {
    response.value = e.message
    isError.value = true
  } finally {
    isLoading.value = false
  }
}
</script>

<style scoped>
.page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
}

.card {
  background: #1e293b;
  border: 1px solid #334155;
  border-radius: 16px;
  padding: 48px 40px;
  width: 100%;
  max-width: 480px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}

.title {
  font-size: 1.8rem;
  font-weight: 700;
  background: linear-gradient(135deg, #60a5fa, #a78bfa);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  margin-bottom: 2px;
}

.subtitle {
  font-size: 0.85rem;
  color: #64748b;
  margin-bottom: 20px;
}

.btn {
  width: 100%;
  padding: 14px;
  border-radius: 10px;
  border: 1px solid #60a5fa;
  background: #1e3a5f;
  color: #93c5fd;
  font-size: 0.95rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  min-height: 48px;
}

.btn:hover:not(:disabled) {
  background: #1d4ed8;
  color: #fff;
  transform: translateY(-1px);
}

.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.spinner {
  width: 18px;
  height: 18px;
  border: 2px solid #334155;
  border-top-color: #60a5fa;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
}

.result {
  width: 100%;
  border-radius: 10px;
  padding: 16px;
  margin-top: 8px;
}

.result.success {
  background: #0f2818;
  border: 1px solid #22c55e;
}

.result.error {
  background: #1f0a0a;
  border: 1px solid #ef4444;
}

.result-label {
  font-size: 0.7rem;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  margin-bottom: 8px;
}

.result.success .result-label {
  color: #22c55e;
}

.result.error .result-label {
  color: #ef4444;
}

.result-body {
  font-family: 'Courier New', monospace;
  font-size: 0.9rem;
  color: #e2e8f0;
  word-break: break-all;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s, transform 0.3s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(8px);
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
