<template>
  <div class='container'>
    <header class="app-header fade-in">
      <h1 class='app-title slide-in-down'>投资助手</h1>
      <p class="app-desc slide-in-up">AI 驱动的量化交易平台</p>
    </header>

    <main class="main-content">
      <!-- 策略生成器 -->
      <section class="section card-enter" style="animation-delay: 0.2s;">
        <div class="section-header">
          <div class="section-icon bounce-in">💡</div>
          <div>
            <h2 class="section-title">策略生成器</h2>
            <p class="section-subtitle">描述您的交易策略，让 AI 为您生成代码</p>
          </div>
        </div>

        <div class="input-group">
          <label class="input-label">策略描述</label>
          <el-input
            v-model="textarea"
            class="strategy-input animate-input"
            :rows="8"
            type="textarea"
            placeholder="请详细描述您的交易策略，例如：'基于RSI和移动平均线的动量策略...'"
          />
        </div>

        <el-button
          type="primary"
          class="action-btn primary-btn pulse-btn"
          @click="generate"
          :loading="generating"
        >
          <span class="btn-text">生成策略</span>
          <span class="btn-loading" v-if="generating">
            <span class="loading-dots">
              <span></span>
              <span></span>
              <span></span>
            </span>
          </span>
        </el-button>

        <transition name="code-reveal" appear>
          <div class="code-output" v-if="answer">
            <div class="output-header">
              <span class="output-title">生成的代码</span>
              <el-button
                size="small"
                type="success"
                @click="copyCode"
                class="copy-btn copy-animation"
                ref="copyButtonRef"
              >
                <span class="copy-icon">📋</span>
                复制代码
              </el-button>
            </div>
            <el-input
              v-model="displayedAnswer"
              class="code-area"
              :rows="25"
              type="textarea"
              readonly
            />
          </div>
        </transition>
      </section>

      <!-- 代码优化助手 -->
      <section class="section card-enter" style="animation-delay: 0.4s;">
        <div class="section-header">
          <div class="section-icon bounce-in" style="animation-delay: 0.5s;">🔧</div>
          <div>
            <h2 class="section-title">代码优化助手</h2>
            <p class="section-subtitle">描述代码问题，获得智能化改进建议</p>
          </div>
        </div>

        <div class="input-group">
          <label class="input-label">问题描述</label>
          <el-input
            v-model="error"
            class="error-input animate-input"
            :rows="4"
            type="textarea"
            placeholder="请描述您在代码中遇到的错误或问题..."
          />
        </div>

        <el-button
          type="success"
          class="action-btn success-btn pulse-btn"
          @click="improve"
          :loading="improving"
        >
          <span class="btn-text">优化代码</span>
          <span class="btn-loading" v-if="improving">
            <span class="loading-dots">
              <span></span>
              <span></span>
              <span></span>
            </span>
          </span>
        </el-button>
      </section>
    </main>

    <!-- 浮动粒子背景 -->
    <div class="particles">
      <div class="particle" v-for="n in 15" :key="n" :style="getParticleStyle(n)"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, nextTick } from 'vue'
import { ElMessage } from 'element-plus'

const textarea = ref('')
const answer = ref('')
const displayedAnswer = ref('')
const error = ref('')
const generating = ref(false)
const improving = ref(false)
const copyButtonRef = ref(null)

// 打字机效果
const typewriterEffect = (text, callback) => {
  displayedAnswer.value = ''
  let i = 0
  const timer = setInterval(() => {
    if (i < text.length) {
      displayedAnswer.value += text.charAt(i)
      i++
    } else {
      clearInterval(timer)
      if (callback) callback()
    }
  }, 20)
}

// 监听answer变化，触发打字机效果
watch(answer, (newAnswer) => {
  if (newAnswer) {
    nextTick(() => {
      typewriterEffect(newAnswer)
    })
  }
})

const generate = async () => {
  if (!textarea.value.trim()) {
    ElMessage.warning('请先描述您的策略')
    return
  }

  generating.value = true
  try {
    const response = await fetch('https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `sk-6bb6ef9228d44550acc26a94174718d7`,
      },
      body: JSON.stringify({
        model: "qwen-plus-2025-04-28",
        messages: [
          {
            role: "system",
            content: "你是一个专业的量化交易策略开发助手，能够根据用户描述生成完整的Python交易策略代码。请确保代码包含必要的导入、策略逻辑、信号生成和使用示例。"
          },
          {
            role: "user",
            content: `请根据以下描述生成一个完整的量化交易策略代码：${textarea.value}，我只需要代码，其他的都不需要`
          }
        ],
        temperature: 0.7,
        max_tokens: 2000
      })
    })

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }

    const data = await response.json()
    answer.value = data.choices[0].message.content
    ElMessage.success('策略生成成功！')

  } catch (error) {
    console.error('API调用失败:', error)
    ElMessage.error('生成失败，请重试')
  } finally {
    generating.value = false
  }
}

const improve = async () => {
  if (!error.value.trim()) {
    ElMessage.warning('请先描述遇到的问题')
    return
  }

  improving.value = true
  try {
    await new Promise(resolve => setTimeout(resolve, 1500))
    ElMessage.success('代码优化建议已准备好！')
  } finally {
    improving.value = false
  }
}

const copyCode = async () => {
  try {
    await navigator.clipboard.writeText(answer.value)

    // 复制成功动画
    if (copyButtonRef.value) {
      copyButtonRef.value.$el.classList.add('copy-success')
      setTimeout(() => {
        copyButtonRef.value.$el.classList.remove('copy-success')
      }, 800)
    }

    ElMessage.success('代码已复制到剪贴板！')
  } catch (err) {
    const textArea = document.createElement('textarea')
    textArea.value = answer.value
    document.body.appendChild(textArea)
    textArea.select()
    document.execCommand('copy')
    document.body.removeChild(textArea)
    ElMessage.success('代码已复制到剪贴板！')
  }
}

// 生成粒子样式
const getParticleStyle = (index) => {
  const delay = Math.random() * 20
  const duration = 15 + Math.random() * 10
  const size = 2 + Math.random() * 4
  const leftPosition = Math.random() * 100

  return {
    left: `${leftPosition}%`,
    animationDelay: `${delay}s`,
    animationDuration: `${duration}s`,
    width: `${size}px`,
    height: `${size}px`,
  }
}
</script>

<style scoped>
/* 基础动画定义 */
@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

@keyframes slideInDown {
  from {
    opacity: 0;
    transform: translateY(-30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes slideInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes cardEnter {
  from {
    opacity: 0;
    transform: translateY(50px) scale(0.95);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

@keyframes bounceIn {
  0% {
    opacity: 0;
    transform: scale(0.3) rotate(0deg);
  }
  50% {
    opacity: 1;
    transform: scale(1.1) rotate(180deg);
  }
  100% {
    opacity: 1;
    transform: scale(1) rotate(360deg);
  }
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
  100% {
    transform: scale(1);
  }
}

@keyframes loadingDots {
  0%, 80%, 100% {
    opacity: 0;
    transform: scale(0.8);
  }
  40% {
    opacity: 1;
    transform: scale(1);
  }
}

@keyframes copySuccess {
  0% {
    transform: scale(1);
    background: #10b981;
  }
  50% {
    transform: scale(1.1);
    background: #059669;
  }
  100% {
    transform: scale(1);
    background: #10b981;
  }
}

@keyframes floatUp {
  0% {
    opacity: 0.7;
    transform: translateY(100vh) rotate(0deg);
  }
  100% {
    opacity: 0;
    transform: translateY(-100px) rotate(360deg);
  }
}

@keyframes glow {
  0% {
    box-shadow: 0 0 5px rgba(59, 130, 246, 0.3);
  }
  50% {
    box-shadow: 0 0 20px rgba(59, 130, 246, 0.6);
  }
  100% {
    box-shadow: 0 0 5px rgba(59, 130, 246, 0.3);
  }
}

/* 应用动画类 */
.fade-in {
  animation: fadeIn 1s ease-out;
}

.slide-in-down {
  animation: slideInDown 0.8s ease-out;
}

.slide-in-up {
  animation: slideInUp 0.8s ease-out 0.2s both;
}

.card-enter {
  animation: cardEnter 0.6s ease-out both;
}

.bounce-in {
  animation: bounceIn 1s ease-out;
}

.pulse-btn:hover {
  animation: pulse 0.6s ease-in-out infinite;
}

/* 全局容器 */
.container {
  min-height: 100vh;
  background: #f8fafc;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', system-ui, sans-serif;
  position: relative;
  overflow-x: hidden;
}

/* 粒子背景 */
.particles {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 1;
}

.particle {
  position: absolute;
  background: rgba(59, 130, 246, 0.3);
  border-radius: 50%;
  animation: floatUp linear infinite;
}

/* 头部样式 */
.app-header {
  text-align: center;
  padding: 60px 20px 40px;
  background: linear-gradient(135deg, #1e3a8a 0%, #3b82f6 100%);
  color: white;
  margin-bottom: 40px;
  position: relative;
  z-index: 2;
  overflow: hidden;
}

.app-header::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: linear-gradient(45deg, transparent, rgba(255,255,255,0.1), transparent);
  animation: shimmer 3s ease-in-out infinite;
}

@keyframes shimmer {
  0% {
    transform: translateX(-100%) translateY(-100%) rotate(45deg);
  }
  100% {
    transform: translateX(100%) translateY(100%) rotate(45deg);
  }
}

.app-title {
  font-size: 2.5rem;
  font-weight: 700;
  margin: 0 0 8px 0;
  letter-spacing: -0.025em;
  position: relative;
  z-index: 3;
}

.app-desc {
  font-size: 1.1rem;
  opacity: 0.9;
  margin: 0;
  font-weight: 400;
  position: relative;
  z-index: 3;
}

/* 主内容区域 */
.main-content {
  max-width: 900px;
  margin: 0 auto;
  padding: 0 20px 60px;
  position: relative;
  z-index: 2;
}

/* 区块样式 */
.section {
  background: white;
  border-radius: 12px;
  padding: 32px;
  margin-bottom: 32px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  border: 1px solid #e5e7eb;
  position: relative;
  transition: all 0.3s ease;
}

.section:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.15);
}

.section-header {
  display: flex;
  align-items: flex-start;
  gap: 16px;
  margin-bottom: 28px;
}

.section-icon {
  font-size: 2rem;
  flex-shrink: 0;
  margin-top: 4px;
  transition: all 0.3s ease;
}

.section-icon:hover {
  transform: scale(1.2) rotate(10deg);
}

.section-title {
  font-size: 1.5rem;
  font-weight: 600;
  color: #111827;
  margin: 0 0 4px 0;
}

.section-subtitle {
  color: #6b7280;
  font-size: 0.95rem;
  margin: 0;
}

/* 输入组样式 */
.input-group {
  margin-bottom: 24px;
}

.input-label {
  display: block;
  font-weight: 500;
  color: #374151;
  margin-bottom: 8px;
  font-size: 0.95rem;
}

/* 自定义输入框 */
.animate-input {
  transition: all 0.3s ease;
}

:deep(.animate-input .el-textarea__inner) {
  border: 2px solid #e5e7eb;
  border-radius: 8px;
  font-size: 14px;
  line-height: 1.5;
  transition: all 0.3s ease;
  resize: vertical;
}

:deep(.animate-input .el-textarea__inner:focus) {
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
  transform: scale(1.02);
  animation: glow 2s ease-in-out infinite;
}

/* 按钮样式 */
.action-btn {
  width: 100%;
  height: 48px;
  font-size: 15px;
  font-weight: 500;
  border-radius: 8px;
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.action-btn::before {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  width: 0;
  height: 0;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  transform: translate(-50%, -50%);
  transition: all 0.5s ease;
}

.action-btn:hover::before {
  width: 300px;
  height: 300px;
}

.primary-btn {
  background: linear-gradient(135deg, #3b82f6 0%, #1e40af 100%);
  border: none;
}

.primary-btn:hover {
  background: linear-gradient(135deg, #2563eb 0%, #1e3a8a 100%);
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(59, 130, 246, 0.4);
}

.success-btn {
  background: linear-gradient(135deg, #10b981 0%, #047857 100%);
  border: none;
}

.success-btn:hover {
  background: linear-gradient(135deg, #059669 0%, #065f46 100%);
  transform: translateY(-2px);
  box-shadow: 0 8px 25px rgba(16, 185, 129, 0.4);
}

/* 加载动画 */
.loading-dots {
  display: inline-flex;
  gap: 4px;
}

.loading-dots span {
  width: 8px;
  height: 8px;
  background: white;
  border-radius: 50%;
  animation: loadingDots 1.4s ease-in-out infinite both;
}

.loading-dots span:nth-child(1) { animation-delay: -0.32s; }
.loading-dots span:nth-child(2) { animation-delay: -0.16s; }
.loading-dots span:nth-child(3) { animation-delay: 0s; }

/* 代码输出区域动画 */
.code-reveal-enter-active {
  transition: all 0.8s ease;
}

.code-reveal-enter-from {
  opacity: 0;
  transform: translateY(30px) scale(0.95);
}

.code-reveal-enter-to {
  opacity: 1;
  transform: translateY(0) scale(1);
}

.code-output {
  margin-top: 28px;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  overflow: hidden;
  position: relative;
}

.code-output::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 2px;
  background: linear-gradient(90deg, transparent, #3b82f6, transparent);
  animation: scan 2s ease-in-out infinite;
}

@keyframes scan {
  0% {
    left: -100%;
  }
  100% {
    left: 100%;
  }
}

.output-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  background: #f9fafb;
  border-bottom: 1px solid #e5e7eb;
}

.output-title {
  font-weight: 500;
  color: #374151;
  font-size: 0.9rem;
}

.copy-btn {
  height: 32px;
  padding: 0 12px;
  font-size: 13px;
  transition: all 0.3s ease;
}

.copy-animation.copy-success {
  animation: copySuccess 0.8s ease;
}

.copy-icon {
  margin-right: 4px;
  transition: all 0.3s ease;
}

.copy-btn:hover .copy-icon {
  transform: scale(1.2);
}

:deep(.code-area .el-textarea__inner) {
  border: none;
  border-radius: 0;
  background: #1f2937;
  color: #e5e7eb;
  font-family: 'Monaco', 'SF Mono', 'Cascadia Code', 'Roboto Mono', monospace;
  font-size: 13px;
  line-height: 1.5;
  resize: vertical;
  transition: all 0.3s ease;
}

:deep(.code-area .el-textarea__inner:focus) {
  box-shadow: none;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .app-header {
    padding: 40px 20px 30px;
  }

  .app-title {
    font-size: 2rem;
  }

  .section {
    padding: 24px 20px;
    margin-bottom: 24px;
  }

  .section-header {
    gap: 12px;
  }

  .section-icon {
    font-size: 1.5rem;
  }

  .section-title {
    font-size: 1.25rem;
  }
}

/* 加载状态优化 */
:deep(.el-button.is-loading) {
  pointer-events: none;
}

/* 滚动条美化 */
::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-track {
  background: #f1f5f9;
  border-radius: 4px;
}

::-webkit-scrollbar-thumb {
  background: linear-gradient(135deg, #3b82f6, #1e40af);
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: linear-gradient(135deg, #2563eb, #1e3a8a);
}
</style>
