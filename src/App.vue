<template>
  <div>
    <h1>视频上传 && 下载</h1>
    <button @click="handleFileUpload">上传文件</button>
    <button @click="handleFileDownload">下载文件</button>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import axios from 'axios';

const file = ref<File>();

const handleFileUpload = () => {
  axios
    .post('127.0.0.1:8081/minio/upload',{ file: "filename", bucket: "dem"})  // 替换为实际的获取上传文件链接的接口地址
    .then((response) => {
      const uploadUrl = response.data.url;  // 假设服务端返回的数据中包含名为 uploadUrl 的字段
      if (uploadUrl) {
        // 使用获取到的 uploadUrl 进行文件上传
        const formData = new FormData();
        formData.append('file', file.value as Blob);
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

<style scoped lang="scss">

</style>