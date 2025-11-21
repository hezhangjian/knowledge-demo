<template>
  <n-card title="Knowledge Base" class="knowledge-card">
    <template #header-extra>
      <n-space>
        <n-input
          v-model:value="searchQuery"
          placeholder="Search knowledge base..."
          clearable
          style="width: 300px"
          @keyup.enter="handleSearch"
        >
          <template #prefix>
            <n-icon><SearchIcon /></n-icon>
          </template>
        </n-input>
        <n-button type="primary" @click="handleSearch">
          Search
        </n-button>
      </n-space>
    </template>

    <n-empty v-if="documents.length === 0" description="No documents uploaded yet">
      <template #extra>
        <n-text depth="3">Upload Word or PDF files to build your knowledge base</n-text>
      </template>
    </n-empty>

    <n-list v-else>
      <n-list-item v-for="doc in filteredDocuments" :key="doc.id">
        <n-card hoverable>
          <template #header>
            <n-space justify="space-between" align="center">
              <n-space>
                <n-icon>
                  <DocumentIcon />
                </n-icon>
                <n-text strong>{{ doc.filename }}</n-text>
              </n-space>
              <n-tag type="info" size="small">{{ doc.type }}</n-tag>
            </n-space>
          </template>
          
          <n-space vertical :size="12">
            <n-descriptions label-placement="left" :column="2" size="small">
              <n-descriptions-item label="Size">
                {{ formatFileSize(doc.size) }}
              </n-descriptions-item>
              <n-descriptions-item label="Uploaded">
                {{ formatDate(doc.uploadedAt) }}
              </n-descriptions-item>
            </n-descriptions>
            
            <n-divider />
            
            <n-collapse>
              <n-collapse-item name="content" title="View Content">
                <n-space vertical>
                  <n-text v-for="(chunk, index) in doc.chunks" :key="index" depth="3">
                    <n-card size="small" style="margin-bottom: 8px">
                      <n-text>{{ chunk.text }}</n-text>
                      <template #footer>
                        <n-text depth="3" style="font-size: 12px">
                          Page {{ chunk.metadata.page }} • Source: {{ chunk.metadata.source }}
                        </n-text>
                      </template>
                    </n-card>
                  </n-text>
                </n-space>
              </n-collapse-item>
            </n-collapse>
          </n-space>
        </n-card>
      </n-list-item>
    </n-list>

    <n-pagination
      v-if="allFilteredDocuments.length > pageSize"
      v-model:page="currentPage"
      :page-size="pageSize"
      :item-count="allFilteredDocuments.length"
      show-size-picker
      :page-sizes="[5, 10, 20, 50]"
      @update:page-size="handlePageSizeChange"
      style="margin-top: 16px; justify-content: center"
    />
  </n-card>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import {
  NCard,
  NEmpty,
  NList,
  NListItem,
  NTag,
  NSpace,
  NText,
  NIcon,
  NInput,
  NButton,
  NDescriptions,
  NDescriptionsItem,
  NDivider,
  NCollapse,
  NCollapseItem,
  NPagination
} from 'naive-ui'
import { DocumentOutline as DocumentIcon, SearchOutline as SearchIcon } from '@vicons/ionicons5'

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

const searchQuery = ref('')
const currentPage = ref(1)
const pageSize = ref(10)

const allFilteredDocuments = computed(() => {
  let docs = props.documents
  
  if (searchQuery.value.trim()) {
    const query = searchQuery.value.toLowerCase()
    docs = docs.filter(doc => 
      doc.filename.toLowerCase().includes(query) ||
      doc.content.toLowerCase().includes(query) ||
      doc.chunks.some(chunk => chunk.text.toLowerCase().includes(query))
    )
  }
  
  return docs
})

const filteredDocuments = computed(() => {
  const docs = allFilteredDocuments.value
  const start = (currentPage.value - 1) * pageSize.value
  const end = start + pageSize.value
  return docs.slice(start, end)
})

const handleSearch = () => {
  currentPage.value = 1
}

const handlePageSizeChange = (size: number) => {
  pageSize.value = size
  currentPage.value = 1
}

const formatFileSize = (bytes: number): string => {
  if (bytes === 0) return '0 Bytes'
  const k = 1024
  const sizes = ['Bytes', 'KB', 'MB', 'GB']
  const i = Math.floor(Math.log(bytes) / Math.log(k))
  return Math.round(bytes / Math.pow(k, i) * 100) / 100 + ' ' + sizes[i]
}

const formatDate = (dateString: string): string => {
  const date = new Date(dateString)
  return date.toLocaleString('en-US', {
    year: 'numeric',
    month: 'short',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  })
}
</script>

<style scoped>
.knowledge-card {
  margin-top: 24px;
}
</style>

