<template>
    <div class="app">
      <ImageGallery />
    </div>
  </template>
  
  <script setup>
  import ImageGallery from './components/ImageGallery.vue'
  </script>
  
  <style>
  .app {
    min-height: 100vh;
    /* background: #f5f5f5; */
  }
  
  :root {
  /* 主题色 */
  --accent-color: #e27d4a;
  --accent-hover: #357abd;
  --accent-light: #6366f1;
  --accent-purple: #8b5cf6;

  /* 浅色主题 */
  --bg-primary: #f8f9fa;
  --bg-secondary: #ffffff;
  --bg-tertiary: #f1f3f5;
  
  --text-primary: #333333;
  --text-secondary: #666666;
  --text-tertiary: #999999;
  
  --border-color: #e9ecef;
  --border-hover: #dee2e6;
  
  --shadow-sm: 0 2px 4px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.07);
  --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.1);
  
  --overlay-bg: rgba(255, 255, 255, 0.9);
  --overlay-dark: rgba(0, 0, 0, 0.5);
}

/* 暗色主题 */
@media (prefers-color-scheme: dark) {
  :root {
    --bg-primary: #18191a;
    --bg-secondary: #242526;
    --bg-tertiary: #3a3b3c;
    
    --text-primary: #e4e6eb;
    --text-secondary: #b0b3b8;
    --text-tertiary: #777777;
    
    --border-color: #3e4042;
    --border-hover: #4e5052;
    
    --shadow-sm: 0 2px 4px rgba(0, 0, 0, 0.2);
    --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.3);
    --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.4);
    
    --overlay-bg: rgba(36, 37, 38, 0.9);
    --overlay-dark: rgba(0, 0, 0, 0.7);
  }
}

/* 基础样式重置 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 
    Oxygen-Sans, Ubuntu, Cantarell, 'Helvetica Neue', sans-serif;
  background-color: var(--bg-primary);
  color: var(--text-primary);
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* 平滑滚动 */
html {
  scroll-behavior: smooth;
}

/* 滚动条美化 */
::-webkit-scrollbar {
  width: 10px;
  height: 10px;
}

::-webkit-scrollbar-track {
  background: var(--bg-tertiary);
}

::-webkit-scrollbar-thumb {
  background: var(--text-tertiary);
  border-radius: 5px;
}

::-webkit-scrollbar-thumb:hover {
  background: var(--text-secondary);
}

/* 文本选择样式 */
::selection {
  background: var(--accent-color);
  color: white;
}

/* 过渡动画 */
.app {
  transition: background-color 0.3s ease, color 0.3s ease;
}

/* 图片基础样式 */
img {
  max-width: 100%;
  height: auto;
  display: block;
}

/* 链接样式 */
a {
  color: var(--accent-color);
  text-decoration: none;
  transition: color 0.3s ease;
}

a:hover {
  color: var(--accent-hover);
}

/* 按钮基础样式 */
button {
  font-family: inherit;
  cursor: pointer;
  border: none;
  outline: none;
}

/* 输入框基础样式 */
input {
  font-family: inherit;
  outline: none;
}
  </style>