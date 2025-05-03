<template>
    <div>
      <h1>Режим: {{ modeLabel }}</h1>
      
      <div v-if="!quizStarted">
        <button @click="startQuiz">Начать квиз</button>
      </div>
  
      <div v-if="loading">Загрузка...</div>
      
      <div v-if="currentQuiz">
        <div class="quiz-content">
          <p><strong>Слово:</strong> {{ currentQuiz.word_eng }}</p>
          <div class="options">
            <button
              v-for="(option, index) in currentQuiz.options"
              :key="index"
              @click="submitAnswer(option)"
              :disabled="answered"
            >
              {{ option }}
            </button>
          </div>
          <p v-if="feedback">{{ feedback }}</p>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import axios from 'axios'
  
  export default {
    data() {
      return {
        tg_id: null,
        mode: null,
        currentQuiz: null,
        quizStarted: false,
        loading: false,
        answered: false,
        feedback: ''
      }
    },
    computed: {
      modeLabel() {
        const labels = {
          easy: 'Легкий',
          medium: 'Средний',
          hard: 'Сложный'
        }
        return labels[this.mode] || ''
      }
    },
    mounted() {
      this.tg_id = localStorage.getItem('tg_id')
      this.mode = this.$route.params.mode
    },
    methods: {
      async startQuiz() {
        this.quizStarted = true
        this.loading = true
        try {
          const response = await axios.get(`/api/quiz/${this.mode}/${this.tg_id}`)
          this.currentQuiz = response.data
        } catch (error) {
          console.error('Ошибка загрузки квиза:', error)
        }
        this.loading = false
      },
  
      async submitAnswer(selectedOption) {
        this.answered = true
        try {
          const response = await axios.post(`/api/quiz/${this.mode}/answer`, {
            tg_id: this.tg_id,
            word_id: this.currentQuiz.word_id,
            selected_option: selectedOption
          })
          
          this.feedback = response.data.correct ? '✅ Правильно!' : '❌ Неправильно'
          
          if (response.data.next) {
            this.currentQuiz = response.data.next
          } else {
            this.quizStarted = false
          }
        } catch (error) {
          console.error('Ошибка отправки ответа:', error)
        }
        this.answered = false
      }
    }
  }
  </script>
  
<style scoped>
.quiz-content {
  background: white;
  padding: 24px;
  margin: 16px;
  border-radius: 16px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.08);
}

.options {
  display: grid;
  gap: 12px;
  margin: 24px 0;
}

.options button {
  padding: 16px;
  text-align: left;
  background: #ffffff;
  border: 1px solid #e0e0e0;
  box-shadow: 0 2px 6px rgba(0,0,0,0.05);
}

.options button:hover:not(:disabled) {
  border-color: #0984e3;
  background: #f8fbff;
}

.options button:disabled[data-correct="true"] {
  background: #d4edda;
  border-color: #28a745;
}

.options button:disabled[data-correct="false"] {
  background: #f8d7da;
  border-color: #dc3545;
}

h1 {
  font-size: 24px;
  color: #2d3436;
  margin: 24px 16px;
}
</style>