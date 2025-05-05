<template>
  <div id="app">
    <div class="status-bar">
      <span :class="{ connected: isConnected, disconnected: !isConnected }">
        {{ isConnected ? '🟢 Бэкенд подключен' : '🔴 Нет подключения к API' }}
      </span>
    </div>

    <router-view />

    <nav class="bottom-nav">
      <button @click="goTo('/')">Вопрос</button>
      <button @click="goTo('/stats')">Статистика</button>
      <button @click="goTo('/modes')">Режимы</button>
    </nav>
  </div>
</template>

<script>
import axios from 'axios'
import { useRouter } from 'vue-router'

export default {
  data() {
    return {
      isConnected: false
    }
  },
  mounted() {
    axios.get('/')
      .then(res => this.isConnected = res.status === 200)
      .catch(err => {
        this.isConnected = false
        console.error('Ошибка подключения к API:', err)
      })
  },
  methods: {
    goTo(path) {
      this.$router.push(path)
    }
  }
}
</script>

<style>
#app {
  font-family: 'Segoe UI', system-ui, sans-serif;
  height: 100vh;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  background: #f8f9fa;
  color: #2d3436;
  padding-bottom: 64px; /* отступ */
}

.status-bar {
  padding: 12px 16px;
  background: #ffffff;
  box-shadow: 0 2px 8px rgba(0,0,0,0.05);
  flex-shrink: 0;
}

router-view {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
}

.bottom-nav {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 64px;
  background: #ffffff;
  display: flex;
  justify-content: space-around;
  align-items: center;
  box-shadow: 0 -2px 12px rgba(0,0,0,0.05);
  z-index: 1000; /* Убедимся что панель поверх контента */
}

.bottom-nav button {
  flex: 1;
  height: 100%;
  border: none;
  background: none;
  font-size: 14px;
  color: #636e72;
  transition: all 0.2s ease;
  position: relative;
}

.bottom-nav button:hover {
  color: #2d3436;
  background: #f8f9fa;
}

.bottom-nav button.router-link-exact-active {
  color: #0984e3;
}

.bottom-nav button.router-link-exact-active::after {
  content: '';
  position: absolute;
  bottom: 4px;
  left: 50%;
  transform: translateX(-50%);
  width: 24px;
  height: 2px;
  background: #0984e3;
  border-radius: 2px;
}

/* Общие стили кнопок */
button {
  padding: 12px 24px;
  font-size: 16px;
  border-radius: 12px;
  border: none;
  background: linear-gradient(145deg, #ffffff, #f8f9fa);
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
  color: #2d3436;
  cursor: pointer;
  transition: all 0.2s ease;
  margin: 8px 0;
}

button:hover {
  box-shadow: 0 4px 12px rgba(0,0,0,0.12);
  transform: translateY(-1px);
}

button:active {
  transform: translateY(0);
  box-shadow: 0 2px 6px rgba(0,0,0,0.1);
}

button:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  box-shadow: none;
}
</style>