<template>
  <n-card title="Step 4: Knowledge Graph & Q&A" class="knowledge-card">
    <n-space vertical :size="16">
      <n-alert v-if="documents.length === 0" type="warning">
        Please complete Step 1, Step 2, and Step 3 first to generate and confirm the knowledge base.
      </n-alert>

      <div v-else>
        <!-- Knowledge Graph Visualization -->
        <n-card title="Knowledge Graph" size="small" style="margin-bottom: 24px">
          <div class="graph-container">
            <n-empty v-if="documents.length === 0" description="No knowledge graph available">
              <template #extra>
                <n-text depth="3">Generate knowledge base in Step 2 and confirm schema in Step 3 to see the graph</n-text>
              </template>
            </n-empty>
            <div v-else class="graph-visualization">
              <n-space vertical :size="12">
                <n-text strong>Document Relationships</n-text>
                <n-list>
                  <n-list-item v-for="doc in documents" :key="doc.id">
                    <n-space align="center">
                      <n-icon color="#18a058"><DocumentIcon /></n-icon>
                      <n-text>{{ doc.filename }}</n-text>
                      <n-tag size="small" type="info">{{ doc.chunks.length }} chunks</n-tag>
                    </n-space>
                  </n-list-item>
                </n-list>
                <n-text depth="3" style="font-size: 12px">
                  In a real application, this would show a visual graph of document relationships and entities.
                </n-text>
              </n-space>
            </div>
          </div>
        </n-card>

        <!-- Q&A Section -->
        <n-card title="Ask Questions" size="small">
          <n-space vertical :size="16">
            <n-input
              v-model:value="question"
              type="textarea"
              placeholder="Ask a question about your documents..."
              :rows="3"
              :disabled="documents.length === 0 || answering"
              @keyup.ctrl.enter="handleAsk"
            />
            <n-space>
              <n-button
                type="primary"
                :disabled="!question.trim() || documents.length === 0 || answering"
                :loading="answering"
                @click="handleAsk"
              >
                <template #icon>
                  <n-icon><ChatbubbleIcon /></n-icon>
                </template>
                Ask Question
              </n-button>
              <n-button @click="question = ''" :disabled="answering">
                Clear
              </n-button>
            </n-space>

            <n-divider />

            <!-- Answer Display -->
            <div v-if="answers.length > 0">
              <n-text strong style="font-size: 16px">Answers:</n-text>
              <n-space vertical :size="12" style="margin-top: 12px">
                <n-card
                  v-for="(answer, index) in answers"
                  :key="index"
                  size="small"
                  hoverable
                >
                  <n-space vertical :size="8">
                    <n-text strong style="color: #18a058">Q: {{ answer.question }}</n-text>
                    <n-text>A: {{ answer.answer }}</n-text>
                    <n-divider style="margin: 8px 0" />
                    <n-space>
                      <n-tag size="small" type="info">Source: {{ answer.source }}</n-tag>
                      <n-text depth="3" style="font-size: 12px">
                        Confidence: {{ answer.confidence }}%
                      </n-text>
                    </n-space>
                  </n-space>
                </n-card>
              </n-space>
            </div>

            <n-empty v-else-if="!answering" description="No questions asked yet">
              <template #extra>
                <n-text depth="3">Ask a question to get answers from your knowledge base</n-text>
              </template>
            </n-empty>
          </n-space>
        </n-card>
      </div>
    </n-space>
  </n-card>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import {
  NCard,
  NSpace,
  NAlert,
  NInput,
  NButton,
  NIcon,
  NText,
  NTag,
  NEmpty,
  NDivider,
  NList,
  NListItem,
  useMessage
} from 'naive-ui'
import {
  DocumentOutline as DocumentIcon,
  ChatbubbleOutline as ChatbubbleIcon
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

interface Answer {
  question: string
  answer: string
  source: string
  confidence: number
}

const props = defineProps<{
  documents: Document[]
}>()

const message = useMessage()

const question = ref('')
const answering = ref(false)
const answers = ref<Answer[]>([])

const handleAsk = async () => {
  if (!question.value.trim() || props.documents.length === 0) {
    return
  }

  answering.value = true

  try {
    // Simulate Q&A search
    await new Promise(resolve => setTimeout(resolve, 1500))

    // Find relevant chunks (simulated search)
    const relevantChunks = props.documents
      .flatMap(doc => doc.chunks.map(chunk => ({ ...chunk, docName: doc.filename })))
      .filter(chunk => 
        chunk.text.toLowerCase().includes(question.value.toLowerCase()) ||
        question.value.toLowerCase().includes(chunk.text.toLowerCase().split(' ')[0])
      )
      .slice(0, 2)

    if (relevantChunks.length === 0) {
      answers.value.unshift({
        question: question.value,
        answer: 'I could not find relevant information in the knowledge base to answer this question. Please try rephrasing your question or check if the documents contain the information you are looking for.',
        source: 'N/A',
        confidence: 0
      })
    } else {
      const bestMatch = relevantChunks[0]
      answers.value.unshift({
        question: question.value,
        answer: `Based on the document "${bestMatch.docName}", ${bestMatch.text.substring(0, 200)}...`,
        source: bestMatch.docName,
        confidence: Math.floor(Math.random() * 30) + 70 // 70-100%
      })
    }

    question.value = ''
    message.success('Answer generated')
  } catch (error) {
    console.error('Q&A error:', error)
    message.error('Failed to generate answer')
  } finally {
    answering.value = false
  }
}
</script>

<style scoped>
.knowledge-card {
  margin-top: 24px;
}

.graph-container {
  min-height: 200px;
}

.graph-visualization {
  padding: 16px;
  background: var(--n-color);
  border-radius: 4px;
}
</style>

