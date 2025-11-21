<template>
  <n-card title="Step 2: Generate/Refresh" class="generate-card">
    <n-space vertical :size="16">
      <n-alert type="info">
        Click the button below to process uploaded documents and generate the knowledge base.
      </n-alert>

      <n-list v-if="uploadedFiles.length > 0">
        <n-list-item v-for="(file, index) in uploadedFiles" :key="index">
          <n-space align="center" justify="space-between" style="width: 100%">
            <n-space align="center">
              <n-icon><DocumentIcon /></n-icon>
              <n-text>{{ file.name }}</n-text>
              <n-tag size="small" type="info">{{ formatFileSize(file.size) }}</n-tag>
            </n-space>
          </n-space>
        </n-list-item>
      </n-list>

      <n-empty v-else description="No documents uploaded">
        <template #extra>
          <n-text depth="3">Please upload documents in Step 1 first</n-text>
        </template>
      </n-empty>

      <n-space>
        <n-button
          type="primary"
          size="large"
          :disabled="uploadedFiles.length === 0 || generating"
          :loading="generating"
          @click="handleGenerate"
        >
          <template #icon>
            <n-icon><SparklesIcon /></n-icon>
          </template>
          Generate/Refresh Knowledge Base
        </n-button>
      </n-space>

      <n-progress
        v-if="generating"
        type="line"
        :percentage="progress"
        :indicator-placement="'inside'"
        status="success"
      >
        Processing documents...
      </n-progress>

      <n-alert v-if="generated && !generating" type="success">
        Knowledge base generated successfully! You can now proceed to Step 3 to confirm the schema.
      </n-alert>
    </n-space>
  </n-card>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'
import {
  NCard,
  NSpace,
  NAlert,
  NList,
  NListItem,
  NIcon,
  NText,
  NTag,
  NButton,
  NEmpty,
  NProgress,
  useMessage
} from 'naive-ui'
import { DocumentOutline as DocumentIcon, SparklesOutline as SparklesIcon } from '@vicons/ionicons5'

const message = useMessage()

const props = defineProps<{
  uploadedFiles: File[]
}>()

const emit = defineEmits<{
  (e: 'generate-success', documents: any[]): void
}>()

const generating = ref(false)
const generated = ref(false)
const progress = ref(0)

const handleGenerate = async () => {
  if (props.uploadedFiles.length === 0) {
    message.warning('Please upload documents first')
    return
  }

  generating.value = true
  generated.value = false
  progress.value = 0

  try {
    // Simulate knowledge base generation
    const documents = []
    
    for (let i = 0; i < props.uploadedFiles.length; i++) {
      const file = props.uploadedFiles[i]
      
      // Simulate processing each file
      await new Promise(resolve => {
        const interval = setInterval(() => {
          progress.value = Math.min(progress.value + 10, (i + 1) * (100 / props.uploadedFiles.length))
        }, 100)
        
        setTimeout(() => {
          clearInterval(interval)
          progress.value = (i + 1) * (100 / props.uploadedFiles.length)
          
          documents.push({
            id: Date.now().toString() + i,
            filename: file.name,
            type: file.type || 'application/pdf',
            size: file.size,
            uploadedAt: new Date().toISOString(),
            content: `Extracted content from ${file.name}`,
            chunks: [
              {
                id: `${i}-1`,
                text: `This is a simulated chunk extracted from ${file.name}. In a real application, this would be the actual content extracted from the document and stored in a vector database.`,
                metadata: {
                  page: 1,
                  source: file.name
                }
              },
              {
                id: `${i}-2`,
                text: `Another chunk from ${file.name} demonstrating how documents are split into smaller pieces for better search and retrieval.`,
                metadata: {
                  page: 2,
                  source: file.name
                }
              }
            ]
          })
          
          resolve(undefined)
        }, 1500)
      })
    }

    progress.value = 100
    generated.value = true
    message.success('Knowledge base generated successfully!')
    
    emit('generate-success', documents)
  } catch (error) {
    console.error('Generation error:', error)
    message.error('Failed to generate knowledge base')
  } finally {
    generating.value = false
  }
}

const formatFileSize = (bytes: number): string => {
  if (bytes === 0) return '0 Bytes'
  const k = 1024
  const sizes = ['Bytes', 'KB', 'MB', 'GB']
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  return Math.round(bytes / Math.pow(k, i) * 100) / 100 + ' ' + sizes[i]
}

watch(() => props.uploadedFiles, () => {
  generated.value = false
  progress.value = 0
}, { deep: true })
</script>

<style scoped>
.generate-card {
  margin-bottom: 24px;
}
</style>

