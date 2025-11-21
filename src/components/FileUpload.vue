<template>
  <n-card title="Step 1: Maintain Documents" class="upload-card">
    <n-upload
      v-model:file-list="fileList"
      :accept="acceptTypes"
      :max="10"
      :multiple="true"
      :on-before-upload="beforeUpload"
      :on-remove="handleRemove"
      :custom-request="customRequest"
      list-type="text"
    >
      <n-upload-dragger>
        <div style="margin-bottom: 12px">
          <n-icon size="48" :depth="3">
            <DocumentIcon />
          </n-icon>
        </div>
        <n-text style="font-size: 16px">
          Click or drag files to this area to upload
        </n-text>
        <n-p depth="3" style="margin: 8px 0 0 0">
          Support for Word (.docx) and PDF (.pdf) files
        </n-p>
      </n-upload-dragger>
    </n-upload>
    
    <n-space v-if="uploading" style="margin-top: 16px">
      <n-spin size="small" />
      <n-text>Uploading...</n-text>
    </n-space>

    <n-alert v-if="fileList.length > 0" type="info" style="margin-top: 16px">
      {{ fileList.length }} document(s) uploaded. Please proceed to Step 2 to generate knowledge base.
    </n-alert>
  </n-card>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { NCard, NUpload, NIcon, NText, NP, NSpace, NSpin, useMessage } from 'naive-ui'
import { DocumentOutline as DocumentIcon } from '@vicons/ionicons5'
import type { UploadFileInfo, UploadCustomRequestOptions } from 'naive-ui'

const message = useMessage()

const props = defineProps<{
  onUploadSuccess?: (file: File) => void
}>()

const emit = defineEmits<{
  (e: 'upload-success', file: File): void
  (e: 'file-removed', file: File): void
  (e: 'files-changed', files: File[]): void
}>()

defineExpose({
  getFiles: () => fileList.value.map(f => f.file).filter(Boolean) as File[]
})

const fileList = ref<UploadFileInfo[]>([])
const uploading = ref(false)
const acceptTypes = '.docx,.doc,.pdf'

interface FileItem {
  file: File
  status: 'pending' | 'uploading' | 'success' | 'error'
  result?: any
}

const fileItems = ref<Map<string, FileItem>>(new Map())

const beforeUpload = (data: { file: UploadFileInfo }) => {
  const file = data.file.file
  if (!file) return false
  
  const ext = file.name.split('.').pop()?.toLowerCase()
  if (!['doc', 'docx', 'pdf'].includes(ext || '')) {
    message.error('Only Word (.doc, .docx) and PDF (.pdf) files are supported')
    return false
  }
  
  if (file.size > 50 * 1024 * 1024) {
    message.error('File size should not exceed 50MB')
    return false
  }
  
  return true
}

const customRequest = async (options: UploadCustomRequestOptions) => {
  const { file, onFinish, onError } = options
  const uploadFile = file.file
  
  if (!uploadFile) {
    onError()
    return
  }
  
  uploading.value = true
  
  try {
    // Simulate file upload only (no processing)
    await simulateUpload(uploadFile)
    
    fileItems.value.set(uploadFile.name, {
      file: uploadFile,
      status: 'success'
    })
    
    onFinish()
    message.success(`Successfully uploaded ${uploadFile.name}`)
    
    emit('upload-success', uploadFile)
    emit('files-changed', fileList.value.map(f => f.file).filter(Boolean) as File[])
    if (props.onUploadSuccess) {
      props.onUploadSuccess(uploadFile)
    }
  } catch (error) {
    console.error('Upload error:', error)
    onError()
    message.error(`Failed to upload ${uploadFile.name}`)
  } finally {
    uploading.value = false
  }
}

const simulateUpload = (file: File): Promise<void> => {
  return new Promise((resolve) => {
    // Simulate upload delay only
    setTimeout(() => {
      resolve()
    }, 800)
  })
}

const handleRemove = (options: { file: UploadFileInfo }) => {
  const file = options.file.file
  if (file) {
    fileItems.value.delete(file.name)
    emit('file-removed', file)
    emit('files-changed', fileList.value.map(f => f.file).filter(Boolean) as File[])
  }
}
</script>

<style scoped>
.upload-card {
  margin-bottom: 24px;
}
</style>

