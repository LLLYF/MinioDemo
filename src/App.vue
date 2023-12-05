<template>
  <div>
    <h1 class="title">视频上传 && 下载</h1>
    <input type="file" id="fileInput" ref="fileInput" @change="handleFileChange">
    <button @click="handleFileUpload">上传文件</button>
    <button @click="handleFileDownload">下载文件</button>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import axios from 'axios'
import { ChangeEvent } from 'react'

const file = ref<any>()

const handleFileChange = (event: ChangeEvent<HTMLInputElement>) => {
  file.value = event.target.files[0]
  console.log('点击上传啦！')
  console.log('文件名: ', file.value.name)
  console.log('文件信息: ', file.value)
};
const handleFileUpload = () => {
  console.log('准备去获取上传链接啦！')
  axios
    .post('http://127.0.0.1:8081/minio/upload',{ file: file.value.name, bucket: "dem"})  // 替换为实际的获取上传文件链接的接口地址
    .then((response) => {
      const uploadUrl = response.data.url  // 假设服务端返回的数据中包含名为 uploadUrl 的字段
      console.log('url:', uploadUrl)
      if (uploadUrl) {
        // 使用获取到的 uploadUrl 进行文件上传
        const formData = new FormData()
        formData.append('file', file.value)

        axios
          .put(uploadUrl, formData, {
            headers: {
              'Content-Type': 'multipart/form-data',
            },
          })
          .then((response) => {
            console.log('文件上传成功', response);
          })
          .catch((error) => {
            console.error('文件上传失败', error);
          });
      } else {
        console.error('未能获取上传文件链接');
      }
    })
    .catch((error) => {
      console.error('获取上传文件链接失败', error);
    });
};

const handleFileDownload = () => {
  axios
    .get('/api/download', {
      responseType: 'blob',
    })
    .then((response) => {
      const url = window.URL.createObjectURL(new Blob([response.data]));
      const link = document.createElement('a');
      link.href = url;
      link.setAttribute('download', 'file.txt');
      document.body.appendChild(link);
      link.click();
    })
    .catch((error) => {
      console.error(error);
    });
};
</script>

<style scoped>
.title {
  margin-bottom: 200px;
}
</style>