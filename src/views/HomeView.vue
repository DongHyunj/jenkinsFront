<template>
  <div class="dashboard">
    <!-- Header -->
    <header class="header">
      <h1 class="title">Jenkins Pipeline Monitor</h1>
      <div class="status-dot" :class="globalStatus"></div>
    </header>

    <!-- Stats Bar -->
    <div class="stats-bar">
      <div class="stat-card" v-for="stat in stats" :key="stat.label">
        <span class="stat-value" :style="{ color: stat.color }">{{ stat.value }}</span>
        <span class="stat-label">{{ stat.label }}</span>
      </div>
    </div>

    <!-- Controls -->
    <div class="controls">
      <button class="btn btn-success" @click="addBuild('SUCCESS')">+ 성공 빌드</button>
      <button class="btn btn-fail" @click="addBuild('FAILURE')">+ 실패 빌드</button>
      <button class="btn btn-running" @click="addBuild('RUNNING')">+ 진행 빌드</button>
      <button class="btn btn-reset" @click="reset">초기화</button>
    </div>

    <!-- Pipeline Jobs -->
    <div class="section-title">파이프라인 현황</div>
    <div class="pipeline-grid">
      <div class="pipeline-card" v-for="job in jobs" :key="job.id" :class="job.status.toLowerCase()"
        @click="selectJob(job)">
        <div class="pipeline-header">
          <span class="job-name">{{ job.name }}</span>
          <span class="job-badge" :class="job.status.toLowerCase()">{{ job.status }}</span>
        </div>
        <div class="pipeline-bar">
          <div class="pipeline-progress" :class="job.status.toLowerCase()" :style="{ width: job.progress + '%' }"></div>
        </div>
        <div class="pipeline-meta">
          <span>#{{ job.buildNumber }}</span>
          <span>{{ job.duration }}</span>
          <span>{{ job.branch }}</span>
        </div>
      </div>
    </div>

    <!-- Build History -->
    <div class="section-title">빌드 히스토리</div>
    <div class="history-list">
      <transition-group name="slide">
        <div class="history-item" v-for="build in buildHistory" :key="build.id" :class="build.status.toLowerCase()">
          <div class="history-icon" :class="build.status.toLowerCase()">
            <span v-if="build.status === 'SUCCESS'">✓</span>
            <span v-else-if="build.status === 'FAILURE'">✗</span>
            <span v-else>⟳</span>
          </div>
          <div class="history-info">
            <span class="history-job">{{ build.job }}</span>
            <span class="history-time">{{ build.time }}</span>
          </div>
          <span class="history-badge" :class="build.status.toLowerCase()">{{ build.status }}</span>
        </div>
      </transition-group>
    </div>

    <!-- Detail Panel -->
    <transition name="fade">
      <div class="detail-panel" v-if="selectedJob">
        <div class="detail-header">
          <span>{{ selectedJob.name }} 상세</span>
          <button class="close-btn" @click="selectedJob = null">✕</button>
        </div>
        <div class="detail-body">
          <div class="detail-row">
            <span class="detail-key">상태</span>
            <span class="job-badge" :class="selectedJob.status.toLowerCase()">{{ selectedJob.status }}</span>
          </div>
          <div class="detail-row">
            <span class="detail-key">빌드 번호</span>
            <span>#{{ selectedJob.buildNumber }}</span>
          </div>
          <div class="detail-row">
            <span class="detail-key">브랜치</span>
            <span>{{ selectedJob.branch }}</span>
          </div>
          <div class="detail-row">
            <span class="detail-key">진행률</span>
            <span>{{ selectedJob.progress }}%</span>
          </div>
          <div class="detail-row">
            <span class="detail-key">소요 시간</span>
            <span>{{ selectedJob.duration }}</span>
          </div>
          <div class="log-box">
            <div v-for="(line, i) in selectedJob.logs" :key="i" class="log-line">
              <span class="log-time">[{{ line.time }}]</span>
              <span :class="'log-' + line.level">{{ line.message }}</span>
            </div>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const jobNames = ['frontend-build', 'backend-test', 'docker-push', 'deploy-prod', 'e2e-test']
const branches = ['main', 'develop', 'feature/auth', 'hotfix/bug-123']
const statusList = ['SUCCESS', 'FAILURE', 'RUNNING']

let idCounter = 100

function makeJob(name, status) {
  const progress = status === 'SUCCESS' ? 100 : status === 'FAILURE' ? Math.floor(Math.random() * 80) + 10 : Math.floor(Math.random() * 70) + 10
  return {
    id: idCounter++,
    name,
    status,
    buildNumber: Math.floor(Math.random() * 500) + 1,
    branch: branches[Math.floor(Math.random() * branches.length)],
    progress,
    duration: `${Math.floor(Math.random() * 10) + 1}m ${Math.floor(Math.random() * 59)}s`,
    logs: [
      { time: '10:01:00', level: 'info', message: 'Checkout SCM 완료' },
      { time: '10:01:12', level: 'info', message: 'npm install 시작...' },
      { time: '10:02:44', level: status === 'FAILURE' ? 'error' : 'info', message: status === 'FAILURE' ? 'ERROR: 빌드 실패 - 의존성 오류' : 'npm run build 완료' },
      { time: '10:03:10', level: status === 'RUNNING' ? 'warn' : 'info', message: status === 'RUNNING' ? 'WARN: 테스트 진행 중...' : '아티팩트 업로드 완료' },
    ]
  }
}

const jobs = ref(jobNames.map((name, i) => makeJob(name, statusList[i % 3])))
const buildHistory = ref([])
const selectedJob = ref(null)

const stats = computed(() => [
  { label: '전체 빌드', value: jobs.value.length + buildHistory.value.length, color: '#94a3b8' },
  { label: '성공', value: jobs.value.filter(j => j.status === 'SUCCESS').length, color: '#22c55e' },
  { label: '실패', value: jobs.value.filter(j => j.status === 'FAILURE').length, color: '#ef4444' },
  { label: '진행 중', value: jobs.value.filter(j => j.status === 'RUNNING').length, color: '#f59e0b' },
])

const globalStatus = computed(() => {
  if (jobs.value.some(j => j.status === 'FAILURE')) return 'fail'
  if (jobs.value.some(j => j.status === 'RUNNING')) return 'running'
  return 'success'
})

function addBuild(status) {
  const name = jobNames[Math.floor(Math.random() * jobNames.length)]
  const job = makeJob(name, status)
  jobs.value.unshift(job)

  buildHistory.value.unshift({
    id: idCounter++,
    job: name,
    status,
    time: new Date().toLocaleTimeString('ko-KR'),
  })

  if (buildHistory.value.length > 10) buildHistory.value.pop()
  if (jobs.value.length > 8) jobs.value.pop()
}

function selectJob(job) {
  selectedJob.value = job
}

function reset() {
  jobs.value = jobNames.map((name, i) => makeJob(name, statusList[i % 3]))
  buildHistory.value = []
  selectedJob.value = null
  idCounter = 100
}

// Simulate running job progress
onMounted(() => {
  setInterval(() => {
    jobs.value.forEach(job => {
      if (job.status === 'RUNNING' && job.progress < 99) {
        job.progress = Math.min(99, job.progress + Math.floor(Math.random() * 5) + 1)
      }
    })
  }, 1500)
})
</script>

<style scoped>
.dashboard {
  max-width: 1100px;
  margin: 0 auto;
  padding: 24px 16px;
}

/* Header */
.header {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 24px;
}

.title {
  font-size: 1.6rem;
  font-weight: 700;
  background: linear-gradient(135deg, #60a5fa, #a78bfa);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.status-dot {
  width: 14px;
  height: 14px;
  border-radius: 50%;
  flex-shrink: 0;
}

.status-dot.success {
  background: #22c55e;
  box-shadow: 0 0 8px #22c55e;
}

.status-dot.fail {
  background: #ef4444;
  box-shadow: 0 0 8px #ef4444;
  animation: pulse 1s infinite;
}

.status-dot.running {
  background: #f59e0b;
  box-shadow: 0 0 8px #f59e0b;
  animation: pulse 1s infinite;
}

/* Stats */
.stats-bar {
  display: flex;
  gap: 16px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.stat-card {
  background: #1e293b;
  border: 1px solid #334155;
  border-radius: 12px;
  padding: 16px 24px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  min-width: 100px;
  transition: transform 0.2s;
}

.stat-card:hover {
  transform: translateY(-2px);
}

.stat-value {
  font-size: 2rem;
  font-weight: 700;
}

.stat-label {
  font-size: 0.75rem;
  color: #64748b;
}

/* Controls */
.controls {
  display: flex;
  gap: 10px;
  margin-bottom: 28px;
  flex-wrap: wrap;
}

.btn {
  padding: 8px 18px;
  border-radius: 8px;
  border: none;
  cursor: pointer;
  font-size: 0.85rem;
  font-weight: 600;
  transition: all 0.2s;
}

.btn:hover {
  transform: translateY(-1px);
  opacity: 0.9;
}

.btn-success {
  background: #166534;
  color: #86efac;
  border: 1px solid #22c55e;
}

.btn-fail {
  background: #7f1d1d;
  color: #fca5a5;
  border: 1px solid #ef4444;
}

.btn-running {
  background: #78350f;
  color: #fcd34d;
  border: 1px solid #f59e0b;
}

.btn-reset {
  background: #1e293b;
  color: #94a3b8;
  border: 1px solid #334155;
}

/* Section Title */
.section-title {
  font-size: 0.85rem;
  font-weight: 600;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  margin-bottom: 12px;
}

/* Pipeline Grid */
.pipeline-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 14px;
  margin-bottom: 32px;
}

.pipeline-card {
  background: #1e293b;
  border: 1px solid #334155;
  border-radius: 12px;
  padding: 16px;
  cursor: pointer;
  transition: all 0.2s;
}

.pipeline-card:hover {
  transform: translateY(-2px);
  border-color: #60a5fa;
}

.pipeline-card.success {
  border-left: 3px solid #22c55e;
}

.pipeline-card.failure {
  border-left: 3px solid #ef4444;
}

.pipeline-card.running {
  border-left: 3px solid #f59e0b;
}

.pipeline-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.job-name {
  font-weight: 600;
  font-size: 0.9rem;
}

.job-badge {
  font-size: 0.7rem;
  font-weight: 700;
  padding: 2px 8px;
  border-radius: 99px;
}

.job-badge.success {
  background: #166534;
  color: #86efac;
}

.job-badge.failure {
  background: #7f1d1d;
  color: #fca5a5;
}

.job-badge.running {
  background: #78350f;
  color: #fcd34d;
}

.pipeline-bar {
  height: 6px;
  background: #334155;
  border-radius: 99px;
  overflow: hidden;
  margin-bottom: 10px;
}

.pipeline-progress {
  height: 100%;
  border-radius: 99px;
  transition: width 0.8s ease;
}

.pipeline-progress.success {
  background: #22c55e;
}

.pipeline-progress.failure {
  background: #ef4444;
}

.pipeline-progress.running {
  background: linear-gradient(90deg, #f59e0b, #fcd34d);
  animation: shimmer 1.5s infinite;
}

.pipeline-meta {
  display: flex;
  gap: 12px;
  font-size: 0.75rem;
  color: #64748b;
}

/* History */
.history-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 28px;
}

.history-item {
  display: flex;
  align-items: center;
  gap: 12px;
  background: #1e293b;
  border: 1px solid #334155;
  border-radius: 10px;
  padding: 10px 16px;
  transition: all 0.3s;
}

.history-icon {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.85rem;
  font-weight: 700;
  flex-shrink: 0;
}

.history-icon.success {
  background: #166534;
  color: #86efac;
}

.history-icon.failure {
  background: #7f1d1d;
  color: #fca5a5;
}

.history-icon.running {
  background: #78350f;
  color: #fcd34d;
}

.history-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.history-job {
  font-size: 0.85rem;
  font-weight: 600;
}

.history-time {
  font-size: 0.75rem;
  color: #64748b;
}

.history-badge {
  font-size: 0.7rem;
  font-weight: 700;
  padding: 2px 8px;
  border-radius: 99px;
}

.history-badge.success {
  background: #166534;
  color: #86efac;
}

.history-badge.failure {
  background: #7f1d1d;
  color: #fca5a5;
}

.history-badge.running {
  background: #78350f;
  color: #fcd34d;
}

/* Detail Panel */
.detail-panel {
  position: fixed;
  top: 0;
  right: 0;
  width: 360px;
  height: 100vh;
  background: #1e293b;
  border-left: 1px solid #334155;
  padding: 24px;
  overflow-y: auto;
  z-index: 100;
}

.detail-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  font-weight: 700;
  font-size: 1rem;
}

.close-btn {
  background: none;
  border: none;
  color: #64748b;
  cursor: pointer;
  font-size: 1rem;
}

.close-btn:hover {
  color: #e2e8f0;
}

.detail-body {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.detail-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 0.85rem;
}

.detail-key {
  color: #64748b;
}

.log-box {
  background: #0f172a;
  border-radius: 8px;
  padding: 12px;
  font-size: 0.75rem;
  font-family: 'Courier New', monospace;
  display: flex;
  flex-direction: column;
  gap: 6px;
  margin-top: 8px;
}

.log-line {
  display: flex;
  gap: 8px;
}

.log-time {
  color: #475569;
}

.log-info {
  color: #94a3b8;
}

.log-error {
  color: #f87171;
}

.log-warn {
  color: #fbbf24;
}

/* Transitions */
.slide-enter-active,
.slide-leave-active {
  transition: all 0.3s ease;
}

.slide-enter-from {
  opacity: 0;
  transform: translateX(-20px);
}

.slide-leave-to {
  opacity: 0;
  transform: translateX(20px);
}

.fade-enter-active,
.fade-leave-active {
  transition: all 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateX(40px);
}

/* Animations */
@keyframes pulse {

  0%,
  100% {
    opacity: 1;
  }

  50% {
    opacity: 0.4;
  }
}

@keyframes shimmer {
  0% {
    background-position: -200% center;
  }

  100% {
    background-position: 200% center;
  }
}
</style>
