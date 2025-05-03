<template>
  <div class="quiz-view">
    <h1>Вопрос дня</h1>

    <!-- Кнопка запуска -->
    <div v-if="!quizStarted">
      <button @click="startQuiz">Начать</button>
    </div>

    <!-- Ошибка -->
    <div v-if="loadError">
      <p style="color: red;">{{ loadError }}</p>
    </div>

    <!-- Вопрос -->
    <div v-if="currentWord">
      <p><strong>Английское слово:</strong> {{ currentWord.word_eng }}</p>
      <p><strong>Повторение:</strong> {{ currentWord.was_in_repeat ? 'Да' : 'Нет' }}</p>

      <div class="options">
        <button
          v-for="(option, index) in options"
          :key="index"
          :disabled="answered"
          @click="submitAnswer(option)"
        >
          {{ option }}
        </button>
      </div>

      <p v-if="answered"><strong>{{ feedback }}</strong></p>
    </div>

    <!-- Загрузка -->
    <div v-else-if="loading">
      <p>Загрузка...</p>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  data() {
    return {
      tg_id: null,
      currentWord: null,
      options: [],
      answered: false,
      feedback: '',
      loadError: '',
      loading: false,
      quizStarted: false
    }
  },
  mounted() {
    this.tg_id = localStorage.getItem('tg_id')
    if (!this.tg_id) {
      this.loadError = 'Ошибка: tg_id не найден.'
    }
  },
  methods: {
    async startQuiz() {
      this.quizStarted = true
      this.loading = true
      this.answered = false
      this.feedback = ''
      this.currentWord = null
      this.options = []

      try {
        // Получаем основное слово
        const wordResponse = await axios.get(`/api/quiz/${this.tg_id}`)
        
        if (wordResponse.data.error) {
          this.loadError = wordResponse.data.error
          return
        }

        // Получаем 3 случайных слова для вариантов
        const randomWordsResponse = await axios.get(`/api/random-words/3`)
        const randomOptions = randomWordsResponse.data

        // Формируем данные
        this.currentWord = {
          word_id: wordResponse.data.word_id,
          word_eng: wordResponse.data.word_eng,
          word_rus: wordResponse.data.word_rus,
          was_in_repeat: wordResponse.data.was_in_repeat
        }

        // Создаем варианты ответов
        this.options = [this.currentWord.word_rus, ...randomOptions]
        this.shuffleOptions()

      } catch (e) {
        this.loadError = 'Не удалось загрузить вопрос.'
        console.error(e)
      } finally {
        this.loading = false
      }
    },

    shuffleOptions() {
      // Перемешиваем варианты Fisher-Yates алгоритмом
      for (let i = this.options.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [this.options[i], this.options[j]] = [this.options[j], this.options[i]]
      }
    },

    submitAnswer(selectedOption) {
      this.answered = true
      const isCorrect = selectedOption === this.currentWord.word_rus
      
      this.feedback = isCorrect
        ? 'Правильно! 🎉'
        : `Неправильно 😕. Правильный ответ: ${this.currentWord.word_rus}`

      this.sendAnswerToServer(isCorrect)
      this.prepareNextQuestion()
    },

    async sendAnswerToServer(isCorrect) {
      try {
        await axios.post('/api/answer', {
          tg_id: this.tg_id,
          word_id: this.currentWord.word_id,
          was_in_repeat: this.currentWord.was_in_repeat,
          is_correct: isCorrect
        })
      } catch (e) {
        console.error('Ошибка при отправке ответа:', e)
      }
    },

    prepareNextQuestion() {
      setTimeout(() => {
        this.currentWord = null
        this.feedback = ''
        this.answered = false
        this.startQuiz()
      }, 2000)
    }
  }
}
</script>

<style scoped>
.quiz-view {
  padding: 16px;
}

h1 {
  font-size: 24px;
  color: #2d3436;
  margin-bottom: 24px;
  padding: 0 16px;
}

.options button {
  padding: 16px;
  text-align: left;
  background: white;
  border: 1px solid #e0e0e0;
  box-shadow: 0 2px 6px rgba(0,0,0,0.05);
}

.options button:hover:not(:disabled) {
  border-color: #0984e3;
  background: #f8fbff;
}

p {
  margin: 16px;
  color: #636e72;
}
</style>