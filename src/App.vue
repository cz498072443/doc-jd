<script setup>
import { ref } from 'vue'
import axios from 'axios'

const inputText = ref('')
const uploadedFiles = ref([])
const proofreadResults = ref('')
const filteredProofreadResults = ref('')
const hideThinkTags = ref(false)

const toggleThinkTags = () => {
  hideThinkTags.value = !hideThinkTags.value
  updateFilteredResults()
}

const updateFilteredResults = () => {
  if (hideThinkTags.value) {
    filteredProofreadResults.value = proofreadResults.value.replace(/<think>.*?<\/think>/gs, '')
  } else {
    filteredProofreadResults.value = proofreadResults.value
  }
}
const fileInputRef = ref(null)
const isLoading = ref(false)
let taskId = ref('')

const handleFileChange = (event) => {
  uploadedFiles.value = Array.from(event.target.files)
}

const handleDrop = (e) => {
  e.preventDefault()
  if (e.dataTransfer.files && e.dataTransfer.files.length > 0) {
    uploadedFiles.value = Array.from(e.dataTransfer.files)
  }
}

const startProofreading = async () => {
  isLoading.value = true
  try {
    // const response = await fetch('https://api.deepseek.com/chat/completions', {
    //   method: 'POST',
    //   headers: {
    //     'Content-Type': 'application/json',
    //     'Authorization': 'Bearer sk-94322f0edbd64367af06d7c02d7ce375'
    //   },
    //   body: JSON.stringify({
    //     model: "deepseek-chat",
    //     messages: [
    //       {
    //         role: "user",
    //         content: "请校对以下文本：\n\n" + inputText.value
    //       }
    //     ],
    //     stream: false
    //   })
    // })

    const response = await fetch('https://api.dify.ai/v1/chat-messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer app-cFshei9I0pUawcwjX61pmVKI'
      },
      body: JSON.stringify({
        query: inputText.value,
        inputs: {},
        response_mode: "streaming",
        conversation_id: "",
        user: "abc-123"
      })
    })

    if (!response.ok) {
      throw new Error('校对请求失败')
    }

    const reader = response.body.getReader()
    const decoder = new TextDecoder()
    proofreadResults.value = ''
    
    while (true) {
      isLoading.value = false
      const { done, value } = await reader.read()
      if (done) break
      const chunk = decoder.decode(value, { stream: true })
      const lines = chunk.split('\n')
      for (const line of lines) {
        if (line.startsWith('data: ')) {
          const dataStr = line.slice(6)
          if (dataStr === '[DONE]') break
          try {
            const data = JSON.parse(dataStr)
            if (data.answer) {
            proofreadResults.value += data.answer
            updateFilteredResults()
            taskId.value = data.task_id
            console.log('任务ID:', taskId.value)
          }
          } catch (error) {
            console.error('解析流式数据失败:', error)
          }
        }
      }
    }
  } catch (error) {
    console.error('校对失败:', error)
  } finally {
    isLoading.value = false
  }
}

const handleFileUpload = async () => {
  try {
    const formData = new FormData();
    uploadedFiles.value.forEach((file) => {
      formData.append('files', file);
    });

    const response = await fetch('http://localhost:3000/upload', {
      method: 'POST',
      body: formData
    });
    const data = await response.json();
    if (response.ok) {
      inputText.value = data.content;
      console.log('文件解析成功:', data.content);
    } else {
      throw new Error(data.error || '文件解析失败');
    }
  } catch (error) {
    console.error('文件上传解析失败:', error);
  }
}
</script>

<template>
  <div class="container">
    <h1>韶关移动智能文本校对系统</h1>

    <!-- 文字输入区域 -->
    <div class="input-area">
      <textarea v-model="inputText" placeholder="在此输入需要校对的文本..." class="text-input"></textarea>
      <button @click="startProofreading" class="proofread-btn">开始校对</button>
    </div>

    <!-- 文件上传区域 -->
    <div class="upload-area">
      <div @dragover.prevent @drop.prevent="handleDrop" class="drop-zone">
        拖放文件到此处或点击上传  <span style="color: gray;font-weight: 800;font-size: 18px;">支持.docx、.pdf、.txt </span>
        <input ref="fileInputRef" type="file" class="hidden" @change="handleFileChange" accept=".doc,.docx,.pdf,.wps" />
      </div>
      <button @click="handleFileUpload" class="upload-btn">上传文件</button>
      <ul class="file-list">
        <li v-for="(file, index) in uploadedFiles" :key="index">{{ file.name }}</li>
      </ul>
    </div>

    <!-- 原文和校对结果展示区域 -->
    <div class="result-container">
      <div class="result-area">
      <h2>原文</h2>
      <pre class="result-output" style="color: black;">{{ inputText }}</pre>
      </div>
      <div class="result-area">
      <h2>校对结果</h2>
      <button @click="toggleThinkTags" class="think-toggle-btn">隐藏think内容</button>
      <div v-if="isLoading" class="loading-container">
        <div class="spinner"></div>
      </div>
      <pre v-else-if="proofreadResults" class="result-output">{{ filteredProofreadResults }}</pre>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.input-area {
  margin-bottom: 20px;
  text-align: center;
}

.upload-area {
  text-align: center;
}

.text-input {
  width: 100%;
  height: 200px;
  padding: 10px;
  margin-bottom: 10px;
  border-radius: 8px;
  border: 1px solid #dcdfe6;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  font-size: 14px;
  transition: border-color 0.2s;
}

.text-input:hover {
  border-color: #c0c4cc;
}

.text-input:focus {
  outline: none;
  border-color: #409eff;
  box-shadow: 0 0 0 2px rgba(64, 158, 255, 0.2);
}

.proofread-btn {
  background-color: #4CAF50;
  color: white;
  padding: 10px 20px;
  border: none;
  cursor: pointer;
  display: inline-block;
  transition: all 0.3s ease;
}

.proofread-btn:hover {
  background-color: #45a049;
  transform: translateY(-2px);
}

.proofread-btn:active {
  transform: translateY(0);
}

.upload-btn {
  background-color: #2196F3;
  color: white;
  padding: 10px 20px;
  border: none;
  cursor: pointer;
  display: inline-block;
  transition: all 0.3s ease;
}

.result-output {
  font-size: 15px;
  font-weight: bold;
  font-family: 'Microsoft YaHei', sans-serif;
  color:#006400;
}


.upload-btn:hover {
  background-color: #0b7dda;
  transform: translateY(-2px);
}

.upload-btn:active {
  transform: translateY(0);
}

.upload-area {
  margin-bottom: 20px;
}

.drop-zone {
  border: 2px dashed #ccc;
  padding: 40px;
  text-align: center;
  margin-bottom: 10px;
  cursor: pointer;
}

.upload-btn {
  background-color: #2196F3;
  color: white;
  padding: 10px 20px;
  border: none;
  cursor: pointer;
}

.file-list {
  list-style-type: none;
  padding: 0;
}

.result-container {
  display: flex;
  gap: 20px;
}



.think-toggle-btn {
  position: fixed;
  right: 20px;
  bottom: 20px;
  background-color: #ff5722;
  color: white;
  padding: 12px 24px;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  font-size: 16px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  transition: all 0.3s ease;
}

.think-toggle-btn:hover {
  background-color: #e64a19;
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.25);
}

.think-toggle-btn:active {
  transform: translateY(0);
}

.result-area {
  flex: 1;
  min-width: 0;
}

.result-output {
  white-space: pre-wrap;
  word-wrap: break-word;
  min-height: 200px;
}

.result-area {
  margin-top: 20px;
}

.result-item {
  border: 1px solid #ddd;
  padding: 10px;
  margin-bottom: 10px;
}

.loading-container {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

.spinner {
  border: 4px solid rgba(0, 0, 0, 0.1);
  border-radius: 50%;
  border-top: 4px solid #4CAF50;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.result-output {
  border: 1px solid #ddd;
  padding: 10px;
  margin-bottom: 10px;
  white-space: pre-wrap;
  word-wrap: break-word;
}
</style>
