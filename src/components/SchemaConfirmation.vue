<template>
  <n-card title="Step 3: Confirm Knowledge Schema" class="schema-card">
    <n-space vertical :size="16">
      <n-alert v-if="documents.length === 0" type="warning">
        Please complete Step 1 and Step 2 first to generate the knowledge base.
      </n-alert>

      <div v-else>
        <n-alert type="info">
          Please review the knowledge schema below and confirm before proceeding to the knowledge graph.
        </n-alert>

        <!-- Schema Overview -->
        <n-card title="Schema Overview" size="small" style="margin-top: 16px">
          <n-space vertical :size="12">
            <n-descriptions label-placement="left" :column="1" bordered>
              <n-descriptions-item label="Total Documents">
                {{ documents.length }}
              </n-descriptions-item>
              <n-descriptions-item label="Total Chunks">
                {{ totalChunks }}
              </n-descriptions-item>
              <n-descriptions-item label="Schema Version">
                v1.0
              </n-descriptions-item>
              <n-descriptions-item label="Generated At">
                {{ new Date().toLocaleString() }}
              </n-descriptions-item>
            </n-descriptions>
          </n-space>
        </n-card>

        <!-- Document Schema Details -->
        <n-card title="Document Schema" size="small" style="margin-top: 16px">
          <n-list>
            <n-list-item v-for="doc in documents" :key="doc.id">
              <n-space vertical :size="8">
                <n-space align="center" justify="space-between">
                  <n-space align="center">
                    <n-icon color="#18a058"><DocumentIcon /></n-icon>
                    <n-text strong>{{ doc.filename }}</n-text>
                  </n-space>
                  <n-tag size="small" type="info">{{ doc.chunks.length }} chunks</n-tag>
                </n-space>
                <n-collapse>
                  <n-collapse-item title="View Schema Details" name="schema">
                    <n-descriptions label-placement="left" :column="1" size="small">
                      <n-descriptions-item label="Document ID">
                        {{ doc.id }}
                      </n-descriptions-item>
                      <n-descriptions-item label="File Type">
                        {{ doc.type }}
                      </n-descriptions-item>
                      <n-descriptions-item label="File Size">
                        {{ formatFileSize(doc.size) }}
                      </n-descriptions-item>
                      <n-descriptions-item label="Uploaded At">
                        {{ new Date(doc.uploadedAt).toLocaleString() }}
                      </n-descriptions-item>
                    </n-descriptions>
                    <n-divider style="margin: 12px 0" />
                    <n-text strong style="font-size: 14px">Chunk Structure:</n-text>
                    <n-list style="margin-top: 8px">
                      <n-list-item v-for="(chunk, idx) in doc.chunks.slice(0, 3)" :key="chunk.id">
                        <n-space vertical :size="4">
                          <n-text depth="3" style="font-size: 12px">
                            Chunk {{ idx + 1 }}: {{ chunk.text.substring(0, 100) }}...
                          </n-text>
                          <n-space>
                            <n-tag size="tiny" type="info">Page: {{ chunk.metadata.page }}</n-tag>
                            <n-tag size="tiny" type="info">Source: {{ chunk.metadata.source }}</n-tag>
                          </n-space>
                        </n-space>
                      </n-list-item>
                      <n-list-item v-if="doc.chunks.length > 3">
                        <n-text depth="3" style="font-size: 12px">
                          ... and {{ doc.chunks.length - 3 }} more chunks
                        </n-text>
                      </n-list-item>
                    </n-list>
                  </n-collapse-item>
                </n-collapse>
              </n-space>
            </n-list-item>
          </n-list>
        </n-card>

        <!-- Confirmation Actions -->
        <n-space style="margin-top: 24px">
          <n-button
            type="primary"
            size="large"
            :disabled="documents.length === 0 || confirmed"
            @click="handleConfirm"
          >
            <template #icon>
              <n-icon><CheckmarkCircleIcon /></n-icon>
            </template>
            Confirm Schema
          </n-button>
          <n-button
            size="large"
            :disabled="documents.length === 0"
            @click="handleModify"
          >
            <template #icon>
              <n-icon><CreateIcon /></n-icon>
            </template>
            Modify Schema
          </n-button>
        </n-space>

        <n-alert v-if="confirmed" type="success" style="margin-top: 16px">
          Schema confirmed! You can now proceed to Step 4 for Knowledge Graph & Q&A.
        </n-alert>
      </div>
    </n-space>
  </n-card>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import {
  NCard,
  NSpace,
  NAlert,
  NDescriptions,
  NDescriptionsItem,
  NList,
  NListItem,
  NIcon,
  NText,
  NTag,
  NCollapse,
  NCollapseItem,
  NDivider,
  NButton,
  useMessage
} from 'naive-ui'
import {
  DocumentOutline as DocumentIcon,
  CheckmarkCircleOutline as CheckmarkCircleIcon,
  CreateOutline as CreateIcon
} from '@vicons/ionicons5'

interface DocumentChunk {
  id: string
  text: string
  metadata: {
    page: number
    source: string
  }
}

interface Document {
  id: string
  filename: string
  type: string
  size: number
  uploadedAt: string
  content: string
  chunks: DocumentChunk[]
}

const props = defineProps<{
  documents: Document[]
}>()

const emit = defineEmits<{
  (e: 'schema-confirmed'): void
}>()

const message = useMessage()
const confirmed = ref(false)

const totalChunks = computed(() => {
  return props.documents.reduce((sum, doc) => sum + doc.chunks.length, 0)
})

const formatFileSize = (bytes: number): string => {
  if (bytes === 0) return '0 Bytes'
  const k = 1024
  const sizes = ['Bytes', 'KB', 'MB', 'GB']
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  return Math.round(bytes / Math.pow(k, i) * 100) / 100 + ' ' + sizes[i]
}

const handleConfirm = () => {
  confirmed.value = true
  message.success('Schema confirmed successfully!')
  emit('schema-confirmed')
}

const handleModify = () => {
  message.info('Schema modification feature coming soon')
}
</script>

<style scoped>
.schema-card {
  margin-top: 24px;
}
</style>

