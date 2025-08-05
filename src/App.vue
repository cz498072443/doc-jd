<script setup>
import { ref } from 'vue'
import axios from 'axios'

const inputText = ref('')
const uploadedFiles = ref([])
const proofreadResults = ref([])
const fileInputRef = ref(null)

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
  try {
    const response = await fetch('https://api.deepseek.com/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer sk-94322f0edbd64367af06d7c02d7ce375'
      },
      body: JSON.stringify({
        model: "deepseek-chat",
        messages: [
          {
            role: "user",
            content: "请校对以下文本：\n\n" + inputText.value
          }
        ],
        stream: false
      })
    })
    proofreadResults.value = response.data.suggestions
  } catch (error) {
    console.error('校对失败:', error)
  }
}

const startProofreadingWithFiles = async () => {
  try {
    const formData = new FormData()
    uploadedFiles.value.forEach((file) => {
      formData.append('files', file)
      text: inputText.value
    })
    proofreadResults.value = response.data.suggestions
  } catch (error) {
    console.error('校对失败:', error)
  }
}



// const handleFileUpload = async () => {
//   // 实际项目中需要实现文件上传逻辑
//   console.log('上传文件:', uploadedFiles.value)
// }
</script>

<template>
  <div class="container">
    <h1>智能文本校对系统</h1>
    
    <!-- 文字输入区域 -->
    <div class="input-area">
      <textarea
        v-model="inputText"
        placeholder="在此输入需要校对的文本..."
        class="text-input"
      ></textarea>
      <button @click="startProofreading" class="proofread-btn">开始校对</button>
    </div>

    <!-- 文件上传区域 -->
    <div class="upload-area">
      <div
        @click="fileInputRef.click()"
        @dragover.prevent
        @drop.prevent="handleDrop"
        class="drop-zone"
      >
        拖放文件到此处或点击上传
        <input
          ref="fileInputRef"
          type="file"
          class="hidden"
          @change="handleFileChange"
          accept=".doc,.docx,.pdf,.wps"
        />
      </div>
      <button @click="handleFileUpload" class="upload-btn">上传文件</button>
      <ul class="file-list">
        <li v-for="(file, index) in uploadedFiles" :key="index">{{ file.name }}</li>
      </ul>
    </div>

    <!-- 校对结果展示区域 -->
    <div class="result-area" v-if="proofreadResults.length > 0">
      <h2>校对结果</h2>
      <div v-for="(result, index) in proofreadResults" :key="index" class="result-item">
        <p>错漏: {{ result.issue }}</p>
        <p>原文: {{ result.originalText }}</p>
        <p>建议: {{ result.suggestedText }}</p>
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
}

.text-input {
  width: 100%;
  height: 200px;
  padding: 10px;
  margin-bottom: 10px;
}

.proofread-btn {
  background-color: #4CAF50;
  color: white;
  padding: 10px 20px;
  border: none;
  cursor: pointer;
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

.result-area {
  margin-top: 20px;
}

.result-item {
  border: 1px solid #ddd;
  padding: 10px;
  margin-bottom: 10px;
}
</style>
