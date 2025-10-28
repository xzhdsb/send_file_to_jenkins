<script setup>
import axios from 'axios'
import { onMounted, ref } from 'vue'

const fileInputRef = ref(null)
const selectedFileName = ref('')
const isUploading = ref(false)
const formData = ref({
  app_type: '',
  file: null,
})

// 处理文件选择
const handleFileChange = (event) => {
  const file = event.target.files[0]
  if (file) {
    if (file.name.endsWith('.jmx')) {
      selectedFileName.value = file.name
      formData.value.file = file // 绑定文件到 formData
      console.log('Selected file:', file.name)
      console.log('FormData updated:', formData.value)
    } else {
      alert('请选择 .jmx 格式的文件')
      event.target.value = ''
      selectedFileName.value = ''
      formData.value.file = null // 清空 formData 中的文件
    }
  }
}

// 触发文件选择
const triggerFileSelect = () => {
  if (fileInputRef.value) {
    fileInputRef.value.click()
  }
}

// 清除选中的文件
const clearFile = () => {
  selectedFileName.value = ''
  formData.value.file = null // 清空 formData 中的文件
  if (fileInputRef.value) {
    fileInputRef.value.value = ''
  }
  console.log('File cleared, FormData updated:', formData.value)
}

// 处理项目类型选择
const handleAppTypeChange = () => {
  console.log('App type changed:', formData.value.app_type)
  console.log('FormData updated:', formData.value)

  // 更新选择样式
  const selectElement = document.getElementById('app-type')
  if (selectElement) {
    if (formData.value.app_type && formData.value.app_type !== '') {
      selectElement.classList.add('has-selection')
    } else {
      selectElement.classList.remove('has-selection')
    }
  }
}

const handleSubmit = async () => {
  // 验证表单数据
  if (!formData.value.app_type) {
    alert('请选择一个项目')
    return
  }

  if (!formData.value.file) {
    alert('请选择一个 .jmx 文件')
    return
  }

  if (isUploading.value) {
    return // 防止重复提交
  }

  try {
    isUploading.value = true
    console.log('Submitting form data:', formData.value)

    const subData = new FormData()

    // 方案1：当前格式
    subData.append('app_type', formData.value.app_type)
    subData.append('file', formData.value.file)

    const res = await axios.post('https://more-master-walrus.ngrok-free.app/file/upload', subData, {
      headers: {
        'Content-Type': 'multipart/form-data',
        'ngrok-skip-browser-warning': 'true',
      },
      timeout: 30000, // 30秒超时
    })

    console.log('Upload response:', res)

    if (res.status === 200) {
      formData.value.app_type = ''
      clearFile()
      alert(
        `上传成功！`,
      )
    } else {
      throw new Error(`服务器返回状态码: ${res.status}`)
    }
  } catch (error) {
    console.error('Upload error:', error)
    let errorMessage = '上传失败'

    if (error.response) {
      // 特殊处理 422 错误
      if (error.response.status === 422) {
        const errorData = error.response.data
        if (errorData && errorData.message) {
          errorMessage = `请求格式错误：${errorData.message}`
        } else if (errorData && errorData.errors) {
          errorMessage = `请求验证失败：${JSON.stringify(errorData.errors)}`
        } else {
          errorMessage = `请求格式错误 (422)：服务器无法处理请求内容`
        }
      } else {
        errorMessage = `上传失败：${error.response.status} - ${error.response.statusText}`
      }
    } else if (error.request) {
      // 请求已发出但没有收到响应
      errorMessage = '上传失败：网络连接超时或服务器无响应'
    } else {
      // 其他错误
      errorMessage = `上传失败：${error.message}`
    }

    alert(errorMessage)
  } finally {
    isUploading.value = false
  }
}

// 处理 select 选中状态的样式
onMounted(() => {
  // 初始化时检查是否已经有选择
  const selectElement = document.getElementById('app-type')
  if (selectElement && formData.value.app_type) {
    selectElement.classList.add('has-selection')
  }
})
</script>

<template>
  <div class="app-container">
    <div class="file-info">
      <div class="title">上传Jmeter脚本</div>
      <div class="desc">
        <p>1.选择将脚本上传到哪个项目</p>
        <p>2.选择本地jmx脚本上传</p>
        <p>3.上传后,可前往http://tool.enerjoy.life:8080对应项目下,检查是否上传成功</p>
      </div>
      <div class="input_wrapper">
        <p>App <span>*</span></p>
        <select
          name="app-type"
          id="app-type"
          v-model="formData.app_type"
          @change="handleAppTypeChange"
        >
          <option value="" disabled selected>select an option...</option>
          <option value="eato">eato</option>
          <option value="justfit_w2s">justfit_w2s</option>
          <option value="justfit">justfit</option>
          <option value="meplus">meplus</option>
        </select>
        <p>Jmx文件 <span>*</span></p>

        <!-- 文件上传区域 -->
        <div class="file-upload-container">
          <input
            ref="fileInputRef"
            type="file"
            id="file-input"
            accept=".jmx"
            @change="handleFileChange"
            style="display: none"
          />
          <div
            class="file-picker"
            :class="{ 'has-file': selectedFileName }"
            @click="triggerFileSelect"
          >
            <span class="file-name" v-if="selectedFileName">{{ selectedFileName }}</span>
            <button
              v-if="selectedFileName"
              class="clear-file"
              @click.stop="clearFile"
              title="清除文件"
            >
              ✕
            </button>
          </div>
        </div>

        <div class="send" :class="{ uploading: isUploading }" @click="handleSubmit">
          {{ isUploading ? '上传中...' : '提交' }}
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.app-container {
  height: 100%;
  width: 100%;
  display: flex;
  flex: 1;
  align-items: self-start;
  justify-content: center;
  font-family: 'Open Sans', sans-serif;
  font-weight: 400;
  font-size: 14px;
  color: #535356;
  .file-info {
    width: 510px;
    margin: 24px 0 0;
    box-sizing: border-box;
    padding: 24px;
    border: 1px solid #dbdfe7;
    border-radius: 8px;
    background-color: #ffffff;
    box-shadow: 0px 4px 16px 0px rgba(99, 77, 255, 0.06);
    .title {
      font-size: 20px;
      font-weight: 500;
      color: #535356;
      text-align: center;
    }
    .desc {
      margin-top: 36px;
      color: #535356;
      font-size: 14.5px;
    }
    .input_wrapper {
      margin-top: 20px;
      p {
        font-size: 16px;
        font-weight: 500;
        span {
          color: #ff6d5a;
        }
      }
      #app-type {
        width: 100%;
        padding: 12px;
        border: 1px solid #dbdfe7;
        border-radius: 4px;
        font-size: 14px;
        color: #535356;
        transition: all 0.3s ease;

        &:active {
          border-color: #ff6c59;
        }
        &:hover {
          border-color: #ff6c59;
        }
        &:focus {
          outline: none;
          border-color: #ff6c59;
          box-shadow: 0 0 0 2px rgba(255, 108, 89, 0.1);
        }

        option {
          color: #535356;
          font-weight: 400;

          &:disabled {
            font-style: italic;
            color: #999;
          }
        }
      }

      /* 文件上传样式 */
      .file-upload-container {
        margin-top: 8px;
        margin-bottom: 46px;
      }

      .file-picker {
        width: 100%;
        height: 48px;
        box-sizing: border-box;
        border: 1px solid #dbdfe7;
        border-radius: 4px;
        background-color: #fafbfc;
        position: relative;
        cursor: pointer;
        transition: all 0.3s ease;
        display: flex;
        align-items: center;
        padding: 0 16px;

        &:hover {
          border-color: #ff6c59;
          background-color: #fff5f4;
        }

        &.has-file {
          border-color: #ff6c59;
        }

        &::before {
          content: '选择 .jmx 文件';
          position: absolute;
          left: 16px;
          top: 50%;
          transform: translateY(-50%);
          color: #999;
          font-size: 14px;
          pointer-events: none;
        }

        &::after {
          content: '浏览';
          position: absolute;
          right: 16px;
          top: 50%;
          transform: translateY(-50%);
          background-color: #ff6c59;
          color: white;
          padding: 4px 12px;
          border-radius: 3px;
          font-size: 12px;
          font-weight: 500;
        }

        &.has-file::before {
          display: none;
        }

        .file-name {
          color: #535356;
          font-size: 14px;
          flex: 1;
          white-space: nowrap;
          overflow: hidden;
          text-overflow: ellipsis;
          margin-right: 8px;
        }

        .clear-file {
          background: #ff6d5a;
          color: white;
          border: none;
          border-radius: 50%;
          width: 20px;
          height: 20px;
          cursor: pointer;
          font-size: 12px;
          display: flex;
          align-items: center;
          justify-content: center;
          transition: all 0.2s ease;
          margin-right: 54px; /* 为浏览按钮留出空间 */
          flex-shrink: 0;

          &:hover {
            background-color: #ff5542;
            transform: scale(1.1);
          }
        }
      }

      .send {
        background-color: #ff6c59;
        color: white;
        padding: 16px 24px;
        border-radius: 6px;
        text-align: center;
        cursor: pointer;
        font-size: 14px;
        font-weight: 500;
        transition: all 0.3s ease;
        border: none;

        &:hover:not(.uploading) {
          background-color: #ff5542;
          transform: translateY(-2px);
          box-shadow: 0 4px 12px rgba(255, 108, 89, 0.3);
        }

        &:active:not(.uploading) {
          transform: translateY(0);
        }

        &.uploading {
          background-color: #ccc;
          cursor: not-allowed;
          opacity: 0.7;

          &::after {
            content: '';
            display: inline-block;
            width: 12px;
            height: 12px;
            margin-left: 8px;
            border: 2px solid transparent;
            border-top: 2px solid white;
            border-radius: 50%;
            animation: spin 1s linear infinite;
          }
        }
      }

      @keyframes spin {
        0% {
          transform: rotate(0deg);
        }
        100% {
          transform: rotate(360deg);
        }
      }

      .test-api-btn {
        background-color: #6c757d;
        color: white;
        border: none;
        padding: 8px 16px;
        border-radius: 4px;
        cursor: pointer;
        font-size: 12px;
        transition: background-color 0.3s ease;

        &:hover {
          background-color: #5a6268;
        }
      }
    }
  }
}
</style>
