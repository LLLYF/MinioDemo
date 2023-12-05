<template>
  <div>
    <h1 class="title">视频上传 && 下载</h1>
    <input type="file" id="fileInput" ref="fileInput" @change="handleFileChange">
    <button @click="handleFileUpload">上传文件</button>
    <button class="dropbtn" @click="downloadClick">下载文件</button>
    <div class="dropdown-content" v-if="showDropDowm">
      <a @click="handleFileDownload" v-if="isUploaded">下载原文件</a>
      <a @click="handleCompressedFileDownload" v-if="isUploaded && isCompressed">下载压缩文件</a>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import axios from 'axios'
import { ChangeEvent } from 'react'

const file = ref<any>()
const isUploaded = ref<Boolean>(false)
const isCompressed = ref<Boolean>(false)
const showDropDowm = ref<Boolean>(false)
const downloadUrl = ref<string>('')
const downloadCompressUrl = ref<string>('')
const bucketName = ref<string>('')
const objectName = ref<string>('')
const infoUrl = ref<string>('')
const frameUrl = ref<string>('')

// 转base64
const fileToBase64 = (file) => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.readAsDataURL(file)
    reader.onload = () => resolve(reader.result)
    reader.onerror = (error) => reject(error)
  })
}
// 接收文件上传事件
const handleFileChange = (event: ChangeEvent<HTMLInputElement>) => {
  file.value = event.target.files[0]
  console.log('点击上传啦！')
  console.log('文件名: ', file.value.name)
  console.log('文件信息: ', file.value)
}

const handleFileUpload = () => {
  console.log('准备去获取上传链接啦！')
  axios
    .post('http://127.0.0.1:8081/minio/get_url', { fileName: file.value.name })  // 文件名和桶名
    .then((response) => {
      const uploadUrl = response.data.url  // 返回 uploadUrl
      console.log('url:', uploadUrl)
      if (uploadUrl) {
        // 使用获取到的 uploadUrl 进行文件上传
        fileToBase64(file.value)
          .then((base64Data) => {
            // console.log('文件转换成 base64 数据:', base64Data)
            const formData = new FormData()
            formData.append('file', file.value)
            // 上传到MinIO
            axios
              .put(uploadUrl, formData, {
                headers: {
                  'Content-Type': 'multipart/form-data',
                },
              })
              .then((response) => {
                console.log('文件上传成功', response)
                // 发送消息通知后端上传成功
                if (response.status !== 200) {
                  response.status = 500
                  uploadUrl.value = ''
                }
                // 通知服务端上传成功，请求原视频下载地址
                axios
                  .post('http://127.0.0.1:8081/minio/upload_success', { status: response.status, url: uploadUrl })
                  .then((response) => {
                    downloadUrl.value = 'http://127.0.0.1:9000/vuetest/mov1.mp4?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=KB937TUUMTQOL1X91G4U%2F20231205%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231205T085051Z&X-Amz-Expires=604800&X-Amz-Security-Token=eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJhY2Nlc3NLZXkiOiJLQjkzN1RVVU1UUU9MMVg5MUc0VSIsImV4cCI6MTcwMTc4ODAxNSwicGFyZW50IjoiYWRtaW4ifQ.DRbrriI_IzfgSYrCtcSJ1dp6Eu3S0WPI2AdGXwklsdaFP5fgLjjcEue2ZIJlKkXq1nwhUUQp5QD3zYDR93cqZQ&X-Amz-SignedHeaders=host&versionId=null&X-Amz-Signature=97301dc639b86c122b9719bc58e9a4ab23157a9020266331262e14ac58daecc6'
                    console.log(response)
                    if (downloadUrl.value) {
                      console.log('下载链接: ', downloadUrl.value)
                      isUploaded.value = true
                      console.log('isUploaded: ', isUploaded.value)
                      bucketName.value = response.data.bucketName
                      objectName.value = response.data.objectName
                      console.log('桶名:', bucketName.value, '文件名:', objectName.value)
                      handleCompressedFile()
                    }
                  })
                  .catch((error) => {
                    console.error('通知失败', error)
                  })
              })
              .catch((error) => {
                console.error('文件上传失败', error)
              });
          })
          .catch((error) => {
            console.error('文件转换成 base64 数据时出错:', error)
          })
      } else {
        console.error('未能获取上传文件链接')
      }
    })
    .catch((error) => {
      console.error('获取上传文件链接失败', error)
    })
}

const handleFileDownload = () => {
  console.log('下载原视频文件！')
  axios
    .get(downloadUrl.value, { responseType: 'blob', })
    .then((response) => {
      const contentType = response.headers['content-type']
      const blob = new Blob([response.data], { type: contentType })
      const link = document.createElement('a')

      link.href = window.URL.createObjectURL(blob)
      link.setAttribute('download', file.value.name)
      document.body.appendChild(link)
      link.click()
      document.body.removeChild(link)
      URL.revokeObjectURL(link.href)
    })
    .catch((error) => {
      console.error(error)
    })
}

const handleCompressedFileDownload = () => {
  console.log('下载压缩后的视频文件！')
  axios
    .get(downloadCompressUrl.value, { responseType: 'blob', })
    .then((response) => {
      const contentType = response.headers['content-type']
      const blob = new Blob([response.data], { type: contentType })
      const link = document.createElement('a')

      link.href = window.URL.createObjectURL(blob)
      link.setAttribute('download', file.value.name)
      document.body.appendChild(link)
      link.click()
      document.body.removeChild(link)
      URL.revokeObjectURL(link.href)
    })
    .catch((error) => {
      console.error(error)
    })
}

const handleCompressedFile = () => {
  console.log('请求下载压缩文件')
  const pollBackendService = setInterval(() => {
    axios
      .post('http://127.0.0.1:8081/minio/download_url', { bucketName: bucketName.value, objectName: objectName.value }) // 请求压缩文件下载链接
      .then((response) => {
        downloadCompressUrl.value = response.data.compressUrl  // 压缩文件下载链接
        infoUrl.value = response.data.infoUrl  // 文件信息下载链接
        frameUrl.value = response.data.frameUrl  //封面图下载链接

        console.log('压缩文件下载链接', downloadCompressUrl.value)
        console.log('文件信息下载链接', infoUrl.value)
        console.log('封面图下载链接', frameUrl.value)
        const downloadLink = downloadCompressUrl.value // 从后端服务获取的下载链接
        if (downloadLink) {
          clearInterval(pollBackendService) // 停止轮询
          isCompressed.value = !isCompressed.value
          console.log('可以下载压缩视频了！')
        }
      })
      .catch((error) => {
        console.log(error)
      })
  }, 5000); // 每隔8秒轮询一次
}

const downloadClick = () => {
  showDropDowm.value = !showDropDowm.value
  console.log('点击下载', showDropDowm.value)
}

</script>

<style scoped>
.title {
  margin-bottom: 200px;
}

a {
  display: block;
  /* 将链接显示为块级元素，使其占据一整行 */
  margin-top: 10px;
  margin-left: 370px;
}
</style>