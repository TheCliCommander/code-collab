<template>
  <div class="editor-container" :class="{ 'dark-theme': isDarkTheme }">
    <EditorToolbar 
      :language="language"
      :isDarkTheme="isDarkTheme"
      @update:language="updateLanguage"
      @toggle-theme="toggleTheme"
      @save="saveCode"
      @share="copyCode"
      @load="loadFile"
    />
    
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
import { ref, computed, onMounted, watch, nextTick } from 'vue'
import hljs from 'highlight.js'
import javascript from 'highlight.js/lib/languages/javascript'
import python from 'highlight.js/lib/languages/python'
import xml from 'highlight.js/lib/languages/xml'
import css from 'highlight.js/lib/languages/css'
import java from 'highlight.js/lib/languages/java'
import csharp from 'highlight.js/lib/languages/csharp'
import EditorToolbar from './EditorToolbar.vue'
import 'highlight.js/styles/github-dark.css'

// Register the languages
hljs.registerLanguage('javascript', javascript)
hljs.registerLanguage('python', python)
hljs.registerLanguage('html', xml)
hljs.registerLanguage('css', css)
hljs.registerLanguage('java', java)
hljs.registerLanguage('csharp', csharp)

const code = ref('')
const language = ref('javascript')
const isDarkTheme = ref(false)
const editorRef = ref(null)

// Sample starter code for each language
const starters = {
  javascript: `// JavaScript Example
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

// Calculate first 10 Fibonacci numbers
for (let i = 0; i < 10; i++) {
  console.log(\`Fibonacci(\${i}) = \${fibonacci(i)}\`);
}`,
  python: `# Python Example
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

# Calculate first 10 Fibonacci numbers
for i in range(10):
    print(f"Fibonacci({i}) = {fibonacci(i)}")`,
  html: `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Interactive Example</title>
    <style>
        .container { text-align: center; padding: 2rem; }
        button { padding: 1rem; cursor: pointer; }
    </style>
</head>
<body>
    <div class="container">
        <h1>Hello, World!</h1>
        <button onclick="alert('Clicked!')">Click me</button>
    </div>
</body>
</html>`,
  css: `/* Modern CSS Example */
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.card {
  padding: 2rem;
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transform: translateY(0);
  transition: transform 0.2s ease;
}

.card:hover {
  transform: translateY(-5px);
}`,
  java: `// Java Example
public class SortingExample {
    public static void bubbleSort(int[] arr) {
        int n = arr.length;
        for (int i = 0; i < n - 1; i++) {
            for (int j = 0; j < n - i - 1; j++) {
                if (arr[j] > arr[j + 1]) {
                    // Swap elements
                    int temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                }
            }
        }
    }

    public static void main(String[] args) {
        int[] numbers = {64, 34, 25, 12, 22, 11, 90};
        bubbleSort(numbers);
        
        System.out.println("Sorted array:");
        for (int num : numbers) {
            System.out.print(num + " ");
        }
    }
}`,
  csharp: `// C# Example
using System;

public class Program
{
    public static int Fibonacci(int n)
    {
        if (n <= 1)
            return n;
        return Fibonacci(n - 1) + Fibonacci(n - 2);
    }

    public static void Main()
    {
        Console.WriteLine("First 10 Fibonacci numbers:");
        for (int i = 0; i < 10; i++)
        {
            Console.WriteLine($"Fibonacci({i}) = {Fibonacci(i)}");
        }
    }
}`
}

const highlightedCode = computed(() => {
  if (!code.value) return ''
  const highlighted = hljs.highlight(code.value, { language: language.value })
  return highlighted.value
})

function updateLanguage(newLang) {
  language.value = newLang
  code.value = starters[newLang]
}

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

function updateHighlight() {
  nextTick(() => {
    highlightedCode.value = hljs.highlight(code.value, { language: language.value }).value
  })
}

// Watch for language changes
watch(language, (newLang) => {
  code.value = starters[newLang]
  updateHighlight()
})

onMounted(() => {
  code.value = starters[language.value]
})

async function loadFile(event) {
  const file = event.target.files[0]
  if (!file) return

  // Update language based on file extension
  const extension = file.name.split('.').pop()
  const extensionToLanguage = {
    'js': 'javascript',
    'py': 'python',
    'html': 'html',
    'css': 'css',
    'java': 'java',
    'cs': 'csharp'
  }
  
  if (extensionToLanguage[extension]) {
    language.value = extensionToLanguage[extension]
  }

  // Read file content
  const text = await file.text()
  code.value = text
  updateHighlight()
  
  // Reset file input
  event.target.value = ''
}
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

.editor-wrapper {
  position: relative;
  flex: 1;
  overflow: auto;
}

.editor-textarea,
.editor-highlight {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  min-height: 100%;
  padding: 1rem;
  font-family: 'Fira Code', monospace;
  font-size: 14px;
  line-height: 1.5;
  tab-size: 2;
  white-space: pre-wrap;
  word-wrap: break-word;
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