<template>
  <div class='main'>
    <h1 class='title'>投资小助手</h1>
    <div>
      <h2>量化交易生成器</h2>
      <h3>策略描述</h3>
      <el-input
        v-model="textarea"
        style="width: 100%;margin:20px 0"
        :rows="5"
        type="textarea"
        placeholder="请详细描述您想要的交易策略"
      />
      <el-button type="primary" @click="generate">生成策略</el-button>
      <el-input
        v-model="answer"
        style="width: 100%;margin:20px 0"
        :rows="10"
        type="textarea"
        placeholder="答案..."
      />
      <h2>代码完善助手</h2>
      <h3>遇到错误了？请描述问题：</h3>
      <el-input
        v-model="error"
        style="width: 100%;margin:20px 0"
        :rows="5"
        type="textarea"
        placeholder="请详细描述您遇到的错误"
      />
      <el-button type="primary" @click="improve">智能完善</el-button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { ElMessage } from 'element-plus'
import { CaretBottom, Plus, Microphone, Operation } from '@element-plus/icons-vue'
const textarea = ref('')
const answer = ref('')
const error = ref('')

</script>

<style scoped>
.main {
  width: 80%;
  margin: 0 auto;
}
.title {
  text-align: center;
  margin: 20px 0;
  color: blueviolet;
}
</style>
