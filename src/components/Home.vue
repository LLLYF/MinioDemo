<template>
  <div> {{title}} </div>

</template>

<script setup lang="ts">
import Minio from 'minio'
import { ref } from 'vue'

const title = defineProps({
    title:{
        type: String,
        default: 'MinIO'
    }
})

const minioClient = new Minio.Client({
  endPoint: '192.168.2.13:9000',
  port: 9000,
  useSSL: false,
  accessKey: 'admin',
  secretKey: '12345678'
})

const uploadClient = async (file, targetpath) => {
    try {
        await minioClient.putObject('dem', targetpath,file.path )
        console.log('上传成功') 
    } catch (e) {
        console.error('文件上传失败', e)
    }
}

const downloadFile = async (filePath) => {
    try {
        const fileStream = await minioClient.getObject('dem', filePath)
        console.log('文件下载成功')
    } catch (e){
        console.log('文件下载失败', e)
    }
}
</script>

<style scoped lang="scss">

</style>