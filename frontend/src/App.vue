<template>
  <div id="app">
    <router-view />
  </div>
</template>

<script setup>
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body {
  height: 100%;
}

#app {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Noto Sans', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  min-height: 100vh;
  background-color: #f6f8fa;
  color: #1f2328;
  font-size: 14px;
  line-height: 1.5;
}

/* 滚动条 */
::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background: transparent;
}

::-webkit-scrollbar-thumb {
  background: #d0d7de;
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: #afb8c1;
}

/* Element Plus 覆盖 - GitHub风格 */
.el-card {
  border-radius: 6px;
  border: 1px solid #d0d7de;
  box-shadow: none;
}

.el-button {
  border-radius: 6px;
  font-weight: 500;
  font-size: 14px;
}

.el-button--primary {
  background-color: #2da44e;
  border-color: #2da44e;
}

.el-button--primary:hover {
  background-color: #2c974b;
  border-color: #2c974b;
}

.el-input__wrapper {
  border-radius: 6px;
  box-shadow: inset 0 0 0 1px #d0d7de;
}

.el-input__wrapper:hover {
  box-shadow: inset 0 0 0 1px #0969da;
}

.el-input__wrapper.is-focus {
  box-shadow: inset 0 0 0 1px #0969da, 0 0 0 3px rgba(9, 105, 218, 0.3);
}

.el-tag {
  border-radius: 20px;
}

.el-table {
  border-radius: 6px;
  border: 1px solid #d0d7de;
}

.el-dialog {
  border-radius: 12px;
}

/* 链接 */
a {
  color: #0969da;
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}
</style>
