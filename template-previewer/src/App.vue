<script setup>
import { ref, watch } from 'vue'
import MonacoEditor from './components/MonacoEditor.vue'
import { Splitpanes, Pane } from 'splitpanes'
import 'splitpanes/dist/splitpanes.css'

const template = ref('')
const dataSource = ref('{}')
const activeTab = ref('html')
const renderedHtml = ref('')
const pdfUrl = ref('')
const isDarkMode = ref(false)
const errorMessage = ref('')
const showError = ref(false)

const tabs = [
  { id: 'html', label: 'HTML Preview' },
  { id: 'pdf', label: 'PDF Preview' }
]

const editorOptions = {
  minimap: { enabled: false },
  fontSize: 14,
  lineNumbers: 'on',
  roundedSelection: false,
  scrollBeyondLastLine: false,
  readOnly: false,
  automaticLayout: true
}

const toggleTheme = () => {
  isDarkMode.value = !isDarkMode.value
  document.documentElement.setAttribute('data-theme', isDarkMode.value ? 'dark' : 'light')
}

// Utility function to render template with data
const renderTemplate = (template, data) => {
  return template.replace(/\{\{([^}]+)\}\}/g, (match, key) => {
    // Handle nested properties using dot notation
    const value = key.split('.').reduce((obj, k) => obj?.[k], data)
    return value !== undefined ? value : match
  })
}

const runPreview = async () => {
  try {
    errorMessage.value = ''
    showError.value = false
    
    // Prepare request data with raw string values
    const requestData = {
      dataSource: dataSource.value,
      template: template.value
    }

    // Make API request
    const response = await previewAPI(requestData)
    
    // Handle the response
    if (response.htmlContent) {
      renderedHtml.value = response.htmlContent
      updateIframeContent(response.htmlContent)
    }

    if (response.pdfContent) {
      // Convert byte array to blob
      const pdfBlob = new Blob([new Uint8Array(response.pdfContent)], { type: 'application/pdf' })
      pdfUrl.value = URL.createObjectURL(pdfBlob)
    }

  } catch (error) {
    console.error('Error running preview:', error)
    errorMessage.value = error.message
    showError.value = true
    
    // Clear the contents
    renderedHtml.value = ''
    updateIframeContent('')
    
    // Clear PDF URL
    if (pdfUrl.value) {
      URL.revokeObjectURL(pdfUrl.value)
      pdfUrl.value = ''
    }
  }
}

// Utility function to update iframe content
const updateIframeContent = (content) => {
  const iframe = document.getElementById('preview-iframe')
  if (iframe) {
    const iframeDoc = iframe.contentDocument || iframe.contentWindow.document
    iframeDoc.open()
    iframeDoc.write(content)
    iframeDoc.close()
  }
}

// Watch for tab changes
watch(activeTab, (newTab) => {
  if (newTab === 'html' && renderedHtml.value) {
    // Restore HTML content when switching back to HTML tab
    updateIframeContent(renderedHtml.value)
  }
})

// API endpoint constant
const API_ENDPOINT = '/report/pdfTest'

// API function
const previewAPI = async (requestData) => {
  try {
    const apiUrl = `${import.meta.env.VITE_API_BASE_URL}${API_ENDPOINT}`
    
    const response = await fetch(apiUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(requestData)
    })

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }

    const data = await response.json()
    
    if (!data.htmlContent || !data.pdfContent) {
      throw new Error('Invalid response format from server')
    }

    return {
      htmlContent: data.htmlContent,
      pdfContent: data.pdfContent
    }
  } catch (error) {
    console.error('API Error:', error)
    throw new Error(`API request failed: ${error.message}`)
  }
}

// Utility function to convert base64 to blob
const base64ToBlob = (base64, type = '') => {
  const binStr = atob(base64)
  const len = binStr.length
  const arr = new Uint8Array(len)
  for (let i = 0; i < len; i++) {
    arr[i] = binStr.charCodeAt(i)
  }
  return new Blob([arr], { type })
}
</script>

<template>
  <div class="app-container" :class="{ 'dark-mode': isDarkMode }">
    <!-- Error Notification -->
    <div v-if="showError" class="error-notification">
      <div class="error-content">
        <span class="error-icon">⚠️</span>
        <span class="error-text">{{ errorMessage }}</span>
        <button class="error-close" @click="showError = false">×</button>
      </div>
    </div>
    <div class="theme-toggle">
      <button @click="toggleTheme" class="theme-button">
        {{ isDarkMode ? '☀️' : '🌙' }}
      </button>
    </div>
    
    <Splitpanes class="main-layout">
      <!-- Sol Panel -->
      <Pane min-size="20" size="50">
        <div class="left-panel">
          <div class="editor-section">
            <h2>HTML Template</h2>
            <div class="editor-container">
              <MonacoEditor
                v-model="template"
                language="html"
                :theme="isDarkMode ? 'vs-dark' : 'vs-light'"
                :options="editorOptions"
              />
            </div>
          </div>
          <div class="editor-section">
            <h2>Data Source</h2>
            <div class="editor-container">
              <MonacoEditor
                v-model="dataSource"
                language="json"
                :theme="isDarkMode ? 'vs-dark' : 'vs-light'"
                :options="editorOptions"
              />
            </div>
          </div>
          <div class="control-buttons">
            <button @click="runPreview" class="run-button">Run</button>
          </div>
        </div>
      </Pane>

      <!-- Sağ Panel -->
      <Pane min-size="20" size="50">
        <div class="right-panel">
          <div class="tabs">
            <button 
              v-for="tab in tabs" 
              :key="tab.id"
              :class="{ active: activeTab === tab.id }"
              @click="activeTab = tab.id"
            >
              {{ tab.label }}
            </button>
          </div>
          <div class="preview-container">
            <div v-show="activeTab === 'html'" class="html-preview">
              <iframe id="preview-iframe" frameborder="0" width="100%" height="100%"></iframe>
            </div>
            <div v-show="activeTab === 'pdf'" class="pdf-preview">
              <iframe v-if="pdfUrl" :src="pdfUrl" frameborder="0"></iframe>
            </div>
          </div>
        </div>
      </Pane>
    </Splitpanes>
  </div>
</template>

<style>
/* Scrollbar gizleme ve düzenleme */
::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background: transparent;
}

::-webkit-scrollbar-thumb {
  background: var(--border-color);
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: var(--accent-color);
}

/* Root stilleri */
:root {
  --bg-primary: #ffffff;
  --bg-secondary: #f5f5f5;
  --text-primary: #333333;
  --text-secondary: #666666;
  --border-color: #dddddd;
  --accent-color: #4CAF50;
  --accent-hover: #45a049;
  --editor-border: #e0e0e0;
}

[data-theme="dark"] {
  --bg-primary: #1e1e1e;
  --bg-secondary: #252525;
  --text-primary: #ffffff;
  --text-secondary: #cccccc;
  --border-color: #333333;
  --editor-border: #333333;
}

/* Reset margin ve padding */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  margin: 0;
  padding: 0;
  overflow: hidden;
  background-color: var(--bg-primary);
}

.app-container {
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  background-color: var(--bg-primary);
  color: var(--text-primary);
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}

.main-layout {
  flex: 1;
  display: flex;
  overflow: hidden;
  background-color: var(--bg-primary);
  height: 100%;
  width: 100%;
}

.left-panel, .right-panel {
  height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  background-color: var(--bg-primary);
  width: 100%;
}

/* Splitpanes custom styles */
.splitpanes {
  background-color: var(--bg-primary) !important;
  height: 100% !important;
  width: 100% !important;
}

.splitpanes__pane {
  background-color: var(--bg-primary) !important;
  height: 100% !important;
}

.splitpanes__splitter {
  background-color: var(--border-color) !important;
  position: relative;
  width: 6px !important;
  margin: 0 -3px;
  cursor: col-resize;
  z-index: 10;
}

.splitpanes__splitter:hover {
  background-color: var(--accent-color) !important;
}

.dark-mode .splitpanes__splitter {
  background-color: var(--border-color) !important;
}

.dark-mode .splitpanes__splitter:hover {
  background-color: var(--accent-color) !important;
}

.editor-section {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  padding: 1rem;
  min-height: 0; /* Önemli: Flex child overflow için */
}

.editor-section h2 {
  color: var(--text-primary);
  margin-bottom: 0.5rem;
}

.editor-container {
  flex: 1;
  overflow: hidden;
  border: 1px solid var(--editor-border);
  border-radius: 4px;
  min-height: 0; /* Önemli: Flex child overflow için */
}

.preview-container {
  flex: 1;
  overflow: hidden;
  padding: 1rem;
  min-height: 0; /* Önemli: Flex child overflow için */
}

.control-buttons {
  padding: 1rem;
  display: flex;
  justify-content: flex-end;
}

.run-button {
  padding: 0.5rem 1rem;
  background-color: var(--accent-color);
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
  font-size: 1rem;
  font-weight: 500;
}

.run-button:hover {
  background-color: var(--accent-hover);
}

.theme-toggle {
  position: fixed;
  top: 1rem;
  right: 1rem;
  z-index: 1000;
}

.theme-button {
  padding: 0.5rem;
  font-size: 1.2rem;
  background: none;
  border: 1px solid var(--border-color);
  border-radius: 50%;
  cursor: pointer;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background-color 0.3s;
}

.theme-button:hover {
  background-color: var(--bg-secondary);
}

.tabs {
  display: flex;
  border-bottom: 1px solid var(--border-color);
  background-color: var(--bg-secondary);
}

.tabs button {
  padding: 0.5rem 1rem;
  border: none;
  background: none;
  cursor: pointer;
  color: var(--text-secondary);
  transition: color 0.3s;
}

.tabs button.active {
  color: var(--accent-color);
  border-bottom: 2px solid var(--accent-color);
}

.html-preview {
  height: 100%;
  border: 1px solid var(--border-color);
  border-radius: 4px;
  background-color: var(--bg-primary);
  overflow: hidden;
}

.html-preview iframe {
  border: none;
  background-color: white;
}

.pdf-preview {
  height: 100%;
}

.pdf-preview iframe {
  width: 100%;
  height: 100%;
  border: 1px solid var(--border-color);
  border-radius: 4px;
}

/* Responsive Design */
@media (max-width: 768px) {
  .main-layout {
    flex-direction: column;
    height: auto;
  }

  .left-panel,
  .right-panel {
    width: 100%;
    height: 50vh;
  }

  .left-panel {
    border-right: none;
    border-bottom: 1px solid var(--border-color);
  }

  .theme-toggle {
    top: 0.5rem;
    right: 0.5rem;
  }

  .editor-section {
    min-height: 200px;
  }

  .preview-container {
    height: calc(50vh - 41px);
  }
}

@media (max-width: 480px) {
  .left-panel,
  .right-panel {
    padding: 0.5rem;
  }

  .control-buttons {
    padding: 0.5rem;
  }

  .tabs button {
    padding: 0.5rem;
    font-size: 0.9rem;
  }
}

.error-notification {
  position: fixed;
  top: 1rem;
  left: 50%;
  transform: translateX(-50%);
  z-index: 1001;
  background-color: #ff5252;
  color: white;
  padding: 0.75rem 1rem;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  max-width: 90%;
  animation: slideDown 0.3s ease-out;
}

.error-content {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.error-icon {
  font-size: 1.2rem;
}

.error-text {
  flex: 1;
  font-size: 0.9rem;
}

.error-close {
  background: none;
  border: none;
  color: white;
  font-size: 1.2rem;
  cursor: pointer;
  padding: 0 0.25rem;
  opacity: 0.8;
  transition: opacity 0.2s;
}

.error-close:hover {
  opacity: 1;
}

@keyframes slideDown {
  from {
    transform: translate(-50%, -100%);
    opacity: 0;
  }
  to {
    transform: translate(-50%, 0);
    opacity: 1;
  }
}

/* Responsive adjustments for error notification */
@media (max-width: 768px) {
  .error-notification {
    width: 90%;
  }
}
</style>
