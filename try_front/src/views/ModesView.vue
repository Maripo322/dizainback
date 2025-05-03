<template>
  <div class="modes-view">
    <h1>Выберите режим</h1>
    <div class="mode-buttons">
      <button 
        v-for="mode in modes" 
        :key="mode.id"
        @click="selectMode(mode.id)"
      >
        {{ mode.label }}
      </button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      modes: [
        { id: 'easy', label: 'Легкий' },
        { id: 'medium', label: 'Средний' },
        { id: 'hard', label: 'Сложный' }
      ]
    }
  },
  methods: {
    selectMode(mode) {
      this.$router.push(`/quiz/${mode}`)
    }
  }
}
</script>

<style scoped>
.container {
  height: 100%;
  display: flex;
  flex-direction: column;
  padding: 16px;
}

.content {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.mode-buttons {
  display: grid;
  gap: 16px;
  padding: 16px;
}

.mode-buttons button {
  padding: 20px;
  font-size: 16px;
  border-radius: 16px;
  background: white;
  box-shadow: 0 4px 12px rgba(0,0,0,0.08);
  border: none;
}

.mode-buttons button:hover {
  box-shadow: 0 6px 16px rgba(0,0,0,0.12);
}
</style>