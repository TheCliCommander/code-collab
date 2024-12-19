<template>
    <div class="toolbar" :class="{ 'dark-theme': isDarkTheme }">
      <select 
        :value="language"
        @input="$emit('update:language', $event.target.value)"
      >
        <option value="javascript">JavaScript</option>
        <option value="python">Python</option>
        <option value="html">HTML</option>
        <option value="css">CSS</option>
        <option value="java">Java</option>
        <option value="cpp">C++</option>
      </select>
      
      <div class="toolbar-actions">
        <button @click="$emit('share')">Share</button>
        <button @click="$emit('save')">Save</button>
        <button @click="$emit('toggle-theme')">
          {{ isDarkTheme ? '☀️' : '🌙' }}
        </button>
      </div>
    </div>
  </template>
  
  <script setup>
  defineProps({
    language: String,
    isDarkTheme: Boolean
  })
  
  defineEmits(['update:language', 'share', 'save', 'toggle-theme'])
  </script>
  
  <style scoped>
  .toolbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0.5rem 1rem;
    background: #f5f5f5;
    border-bottom: 1px solid #ddd;
  }
  
  .dark-theme.toolbar {
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
  </style>