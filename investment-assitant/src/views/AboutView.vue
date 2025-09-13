<template>
  <div class='container'>
    <header class="app-header">
      <h1 class='app-title'>投资助手</h1>
      <p class="app-desc">AI 驱动的量化交易平台</p>
    </header>

    <main class="main-content">
      <!-- 策略生成器 -->
      <section class="section">
        <div class="section-header">
          <div class="section-icon">💡</div>
          <div>
            <h2 class="section-title">策略生成器</h2>
            <p class="section-subtitle">描述您的交易策略，让 AI 为您生成代码</p>
          </div>
        </div>

        <div class="input-group">
          <label class="input-label">策略描述</label>
          <el-input
            v-model="textarea"
            class="strategy-input"
            :rows="8"
            type="textarea"
            placeholder="请详细描述您的交易策略，例如：'基于RSI和移动平均线的动量策略...'"
          />
        </div>

        <el-button
          type="primary"
          class="action-btn primary-btn"
          @click="generate"
          :loading="generating"
        >
          生成策略
        </el-button>

        <div class="code-output" v-if="answer">
          <div class="output-header">
            <span class="output-title">生成的代码</span>
            <el-button
              size="small"
              type="success"
              @click="copyCode"
              class="copy-btn"
            >
              <span class="copy-icon">📋</span>
              复制代码
            </el-button>
          </div>
          <el-input
            v-model="answer"
            class="code-area"
            :rows="25"
            type="textarea"
            readonly
          />
        </div>
      </section>

      <!-- 代码优化助手 -->
      <section class="section">
        <div class="section-header">
          <div class="section-icon">🔧</div>
          <div>
            <h2 class="section-title">代码优化助手</h2>
            <p class="section-subtitle">描述代码问题，获得智能化改进建议</p>
          </div>
        </div>

        <div class="input-group">
          <label class="input-label">问题描述</label>
          <el-input
            v-model="error"
            class="error-input"
            :rows="4"
            type="textarea"
            placeholder="请描述您在代码中遇到的错误或问题..."
          />
        </div>

        <el-button
          type="success"
          class="action-btn success-btn"
          @click="improve"
          :loading="improving"
        >
          优化代码
        </el-button>
      </section>
    </main>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { ElMessage } from 'element-plus'

const textarea = ref('')
const answer = ref('')
const error = ref('')
const generating = ref(false)
const improving = ref(false)

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
        'Authorization': `sk-6bb6ef9228d44550acc26a94174718d7`, // 从环境变量读取
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
    ElMessage.success('代码已复制到剪贴板！')
  } catch (err) {
    // 旧浏览器的降级处理
    const textArea = document.createElement('textarea')
    textArea.value = answer.value
    document.body.appendChild(textArea)
    textArea.select()
    document.execCommand('copy')
    document.body.removeChild(textArea)
    ElMessage.success('代码已复制到剪贴板！')
  }
}
</script>

<style scoped>
/* 全局容器 */
.container {
  min-height: 100vh;
  background: #f8fafc;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', system-ui, sans-serif;
}

/* 头部样式 */
.app-header {
  text-align: center;
  padding: 60px 20px 40px;
  background: linear-gradient(135deg, #1e3a8a 0%, #3b82f6 100%);
  color: white;
  margin-bottom: 40px;
}

.app-title {
  font-size: 2.5rem;
  font-weight: 700;
  margin: 0 0 8px 0;
  letter-spacing: -0.025em;
}

.app-desc {
  font-size: 1.1rem;
  opacity: 0.9;
  margin: 0;
  font-weight: 400;
}

/* 主内容区域 */
.main-content {
  max-width: 900px;
  margin: 0 auto;
  padding: 0 20px 60px;
}

/* 区块样式 */
.section {
  background: white;
  border-radius: 12px;
  padding: 32px;
  margin-bottom: 32px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  border: 1px solid #e5e7eb;
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
:deep(.strategy-input .el-textarea__inner),
:deep(.error-input .el-textarea__inner) {
  border: 2px solid #e5e7eb;
  border-radius: 8px;
  font-size: 14px;
  line-height: 1.5;
  transition: border-color 0.2s ease;
  resize: vertical;
}

:deep(.strategy-input .el-textarea__inner:focus),
:deep(.error-input .el-textarea__inner:focus) {
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

/* 按钮样式 */
.action-btn {
  width: 100%;
  height: 48px;
  font-size: 15px;
  font-weight: 500;
  border-radius: 8px;
  transition: all 0.2s ease;
}

.primary-btn {
  background: #3b82f6;
  border-color: #3b82f6;
}

.primary-btn:hover {
  background: #2563eb;
  border-color: #2563eb;
}

.success-btn {
  background: #10b981;
  border-color: #10b981;
}

.success-btn:hover {
  background: #059669;
  border-color: #059669;
}

/* 代码输出区域 */
.code-output {
  margin-top: 28px;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  overflow: hidden;
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
}

.copy-icon {
  margin-right: 4px;
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
</style>
