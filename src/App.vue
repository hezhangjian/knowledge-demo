<template>
  <n-config-provider :theme="theme">
    <n-message-provider>
      <n-layout class="app-layout">
      <n-layout-header class="header" bordered>
        <n-space align="center" justify="space-between" style="width: 100%">
          <n-space align="center">
            <n-icon size="32" color="#18a058">
              <BookIcon />
            </n-icon>
            <n-h1 style="margin: 0; font-size: 24px">Knowledge Base Demo</n-h1>
          </n-space>
          <n-space>
            <n-switch v-model:value="isDark" @update:value="handleThemeChange">
              <template #checked-icon>
                <n-icon><MoonIcon /></n-icon>
              </template>
              <template #unchecked-icon>
                <n-icon><SunnyIcon /></n-icon>
              </template>
            </n-switch>
          </n-space>
        </n-space>
      </n-layout-header>
      
      <n-layout-content class="content">
        <div class="content-wrapper">
          <!-- Steps Indicator -->
          <n-card style="margin-bottom: 24px">
            <n-steps :current="currentStep" :status="stepStatus">
              <n-step title="Maintain Documents" description="Upload Word or PDF files">
                <template #icon>
                  <n-icon><DocumentIcon /></n-icon>
                </template>
              </n-step>
              <n-step title="Generate/Refresh" description="Process documents and create knowledge base">
                <template #icon>
                  <n-icon><SparklesIcon /></n-icon>
                </template>
              </n-step>
              <n-step title="Confirm Schema" description="Review and confirm knowledge schema">
                <template #icon>
                  <n-icon><CheckmarkCircleIcon /></n-icon>
                </template>
              </n-step>
              <n-step title="Knowledge Graph & Q&A" description="View graph and ask questions">
                <template #icon>
                  <n-icon><ChatbubbleIcon /></n-icon>
                </template>
              </n-step>
            </n-steps>
          </n-card>

          <!-- Step 1: File Upload -->
          <FileUpload
            ref="fileUploadRef"
            @files-changed="handleFilesChanged"
          />

          <!-- Step 2: Generate/Refresh Knowledge Base -->
          <GenerateKnowledge
            :uploaded-files="uploadedFiles"
            @generate-success="handleGenerateSuccess"
          />

          <!-- Step 3: Schema Confirmation -->
          <SchemaConfirmation
            :documents="documents"
            @schema-confirmed="handleSchemaConfirmed"
          />

          <!-- Step 4: Knowledge Graph & Q&A -->
          <KnowledgeGraph :documents="documents" />
        </div>
      </n-layout-content>
      
      <n-layout-footer class="footer" bordered>
        <n-text depth="3">Knowledge Base POC - Built with Vue 3 & Naive UI</n-text>
      </n-layout-footer>
    </n-layout>
    </n-message-provider>
  </n-config-provider>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import {
  NConfigProvider,
  NMessageProvider,
  NLayout,
  NLayoutHeader,
  NLayoutContent,
  NLayoutFooter,
  NSpace,
  NIcon,
  NH1,
  NText,
  NSwitch,
  NCard,
  NSteps,
  NStep,
  darkTheme
} from 'naive-ui'
import {
  BookOutline as BookIcon,
  MoonOutline as MoonIcon,
  SunnyOutline as SunnyIcon,
  DocumentOutline as DocumentIcon,
  SparklesOutline as SparklesIcon,
  ChatbubbleOutline as ChatbubbleIcon,
  CheckmarkCircleOutline as CheckmarkCircleIcon
} from '@vicons/ionicons5'
import FileUpload from './components/FileUpload.vue'
import GenerateKnowledge from './components/GenerateKnowledge.vue'
import SchemaConfirmation from './components/SchemaConfirmation.vue'
import KnowledgeGraph from './components/KnowledgeGraph.vue'

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

const isDark = ref(false)
const theme = computed(() => isDark.value ? darkTheme : null)

const fileUploadRef = ref<InstanceType<typeof FileUpload> | null>(null)
const uploadedFiles = ref<File[]>([])
const documents = ref<Document[]>([])
const currentStep = ref(0)
const stepStatus = ref<'process' | 'finish' | 'error' | 'wait'>('process')

const handleThemeChange = (value: boolean) => {
  isDark.value = value
}

const handleFilesChanged = (files: File[]) => {
  uploadedFiles.value = files
  if (files.length > 0 && currentStep.value === 0) {
    currentStep.value = 1
    stepStatus.value = 'process'
  }
}

const handleGenerateSuccess = (docs: Document[]) => {
  documents.value = docs
  currentStep.value = 2
  stepStatus.value = 'process'
}

const handleSchemaConfirmed = () => {
  currentStep.value = 3
  stepStatus.value = 'finish'
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Fira Sans', 'Droid Sans', 'Helvetica Neue', sans-serif;
}

.app-layout {
  min-height: 100vh;
}

.header {
  padding: 16px 24px;
  background: var(--n-color);
}

.content {
  padding: 0;
  background: var(--n-color);
  min-height: calc(100vh - 120px);
}

.content-wrapper {
  max-width: 1200px;
  margin: 0 auto;
  padding: 24px;
}

.footer {
  padding: 16px 24px;
  text-align: center;
  background: var(--n-color);
}
</style>
