<template>
    <div class="editor-container" :class="{ 'dark-theme': isDarkTheme }">
      <div class="toolbar">
        <select v-model="language">
          <option value="javascript">JavaScript</option>
          <option value="python">Python</option>
          <option value="html">HTML</option>
          <option value="css">CSS</option>
          <option value="java">Java</option>
        </select>
        
        <div class="toolbar-actions">
          <button @click="saveCode">Save</button>
          <button @click="copyCode">Copy</button>
          <button @click="toggleTheme">
            {{ isDarkTheme ? '☀️' : '🌙' }}
          </button>
        </div>
      </div>
      
      <div class="editor-wrapper">
        <textarea
          ref="editorRef"
          v-model="code"
          @input="updateHighlight"
          @keydown="handleKeydown"
          :spellcheck="false"
          class="editor-textarea"
        ></textarea>
        <pre 
          class="editor-highlight"
          v-html="highlightedCode"
        ></pre>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, computed, onMounted } from 'vue'
  import hljs from 'highlight.js'
  import 'highlight.js/styles/github-dark.css' // Add this to your main.js instead if preferred
  
  const code = ref('')
  const language = ref('javascript')
  const isDarkTheme = ref(false)
  const editorRef = ref(null)
  
  // Sample starter code for each language
  const starters = {
    javascript: `// Welcome to the Code Editor
  function greeting(name) {
    return \`Hello, \${name}!\`;
  }
  
  console.log(greeting('World'));`,
    python: `# Welcome to the Code Editor
  def greeting(name):
      return f"Hello, {name}!"
  
  print(greeting("World"))`,
    html: `<!DOCTYPE html>
  <html>
    <head>
      <title>Hello World</title>
    </head>
    <body>
      <h1>Hello, World!</h1>
    </body>
  </html>`,
    css: `/* Welcome to the Code Editor */
  .container {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    background: #f0f0f0;
  }`,
    java: `// Welcome to the Code Editor
  public class HelloWorld {
      public static void main(String[] args) {
          System.out.println("Hello, World!");
      }
  }`
  }
  
  const highlightedCode = computed(() => {
    if (!code.value) return ''
    const highlighted = hljs.highlight(code.value, { language: language.value })
    return highlighted.value
  })
  
  function handleKeydown(e) {
    // Handle tab key
    if (e.key === 'Tab') {
      e.preventDefault()
      const start = e.target.selectionStart
      const end = e.target.selectionEnd
      
      code.value = code.value.substring(0, start) + '  ' + code.value.substring(end)
      
      // Set cursor position after tab
      nextTick(() => {
        e.target.selectionStart = e.target.selectionEnd = start + 2
      })
    }
  }
  
  function saveCode() {
    const blob = new Blob([code.value], { type: 'text/plain' })
    const url = URL.createObjectURL(blob)
    const a = document.createElement('a')
    a.href = url
    a.download = `code.${language.value}`
    a.click()
    URL.revokeObjectURL(url)
  }
  
  function copyCode() {
    navigator.clipboard.writeText(code.value)
  }
  
  function toggleTheme() {
    isDarkTheme.value = !isDarkTheme.value
  }
  
  // Watch for language changes to update starter code
  watch(language, (newLang) => {
    if (!code.value || code.value === starters[language.value]) {
      code.value = starters[newLang]
    }
  })
  
  onMounted(() => {
    code.value = starters[language.value]
  })
  </script>
  
  <style scoped>
  .editor-container {
    height: 100vh;
    display: flex;
    flex-direction: column;
    background: #ffffff;
    color: #1a1a1a;
  }
  
  .editor-container.dark-theme {
    background: #1a1a1a;
    color: #ffffff;
  }
  
  .toolbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0.5rem 1rem;
    background: #f5f5f5;
    border-bottom: 1px solid #ddd;
  }
  
  .dark-theme .toolbar {
    background: #2a2a2a;
    border-color: #444;
  }
  
  .toolbar select,
  .toolbar button {
    padding: 0.5rem 1rem;
    border: 1px solid #ddd;
    border-radius: 4px;
    background: white;
    cursor: pointer;
  }
  
  .dark-theme select,
  .dark-theme button {
    background: #1a1a1a;
    border-color: #444;
    color: white;
  }
  
  .toolbar-actions {
    display: flex;
    gap: 0.5rem;
  }
  
  .editor-wrapper {
    position: relative;
    flex: 1;
    overflow: hidden;
  }
  
  .editor-textarea,
  .editor-highlight {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    padding: 1rem;
    font-family: 'Fira Code', monospace;
    font-size: 14px;
    line-height: 1.5;
    tab-size: 2;
  }
  
  .editor-textarea {
    color: transparent;
    background: transparent;
    caret-color: #000;
    resize: none;
    border: none;
    outline: none;
    z-index: 1;
  }
  
  .dark-theme .editor-textarea {
    caret-color: #fff;
  }
  
  .editor-highlight {
    pointer-events: none;
    white-space: pre-wrap;
    word-wrap: break-word;
    z-index: 0;
  }
  </style>