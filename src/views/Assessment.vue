<template>
  <div class="assessment-container" v-if="assessmentData">
    <div class="assessment-header">
      <h1>{{ assessmentData.title }}</h1>
      <div class="progress-bar">
        <div 
          class="progress-fill" 
          :style="{ width: `${(Math.min(currentQuestionIndex + questionsPerPage, totalQuestions) / totalQuestions) * 100}%` }"
        ></div>
      </div>
      <p class="progress-text">
        Questions {{ currentQuestionIndex + 1 }}-{{ Math.min(currentQuestionIndex + questionsPerPage, totalQuestions) }} of {{ totalQuestions }}
      </p>
    </div>

    <div v-if="!showResults" class="quiz-content">
      <div class="navigation top-nav">
        <button 
          @click="previousPage" 
          :disabled="currentPage === 0"
          class="nav-button"
        >
          ← Previous
        </button>
        <button 
          v-if="!isLastPage"
          @click="nextPage"
          class="nav-button primary"
        >
          Next →
        </button>
        <button 
          v-else
          @click="submitQuiz"
          class="nav-button primary"
        >
          Submit Quiz
        </button>
      </div>

      <div class="question-group">
        <h2>{{ currentSubject.name }}</h2>
        <div 
          v-for="(question, index) in currentPageQuestions" 
          :key="question.id"
          class="question"
        >
          <h3>{{ getQuestionNumber(index) }}. {{ question.question }}</h3>
          <div class="options">
            <label 
              v-for="(option, optionIndex) in question.options" 
              :key="optionIndex"
              class="option"
            >
              <input 
                type="radio" 
                :name="`q${question.id}`"
                :value="optionIndex"
                v-model="answers[question.id]"
              />
              <span>{{ option }}</span>
            </label>
          </div>
        </div>
      </div>

      <div class="navigation">
        <button 
          @click="previousPage" 
          :disabled="currentPage === 0"
          class="nav-button"
        >
          ← Previous
        </button>
        <button 
          v-if="!isLastPage"
          @click="nextPage"
          class="nav-button primary"
        >
          Next →
        </button>
        <button 
          v-else
          @click="submitQuiz"
          class="nav-button primary"
        >
          Submit Quiz
        </button>
      </div>
    </div>

    <div v-else class="results">
      <h2>Quiz Results</h2>
      <div class="score-card">
        <h3>Total Score: {{ score }} / {{ totalQuestions }}</h3>
        <p>{{ answeredQuestions > 0 ? ((score / answeredQuestions) * 100).toFixed(1) : 0 }}% of answered questions</p>
        <div v-if="skippedCount > 0" class="skipped-info">
          <small>{{ skippedCount }} question{{ skippedCount === 1 ? '' : 's' }} skipped</small>
        </div>
      </div>
      
      <div class="subject-scores">
        <h3>Score by Subject:</h3>
        <div v-for="subject in subjectScores" :key="subject.name" class="subject-score">
          <span>{{ subject.name }}:</span>
          <span>
            {{ subject.score }} / {{ subject.total }} 
            ({{ subject.percentage }}%{{ subject.skipped > 0 ? `, ${subject.skipped} skipped` : '' }})
          </span>
        </div>
      </div>

      <router-link to="/" class="nav-button primary">
        Back to Home
      </router-link>
    </div>
  </div>
  
  <div v-else class="loading">
    Loading assessment...
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'

const route = useRoute()
const router = useRouter()

const assessmentData = ref(null)
const answers = ref({})
const currentPage = ref(0)
const showResults = ref(false)
const score = ref(0)
const subjectScores = ref([])
const skippedCount = ref(0)
const answeredQuestions = ref(0)

const questionsPerPage = 5

const allQuestions = computed(() => {
  if (!assessmentData.value) return []
  return assessmentData.value.subjects.flatMap(subject => 
    subject.questions.map(q => ({ ...q, subject: subject.name }))
  )
})

const totalQuestions = computed(() => allQuestions.value.length)

const currentPageQuestions = computed(() => {
  const start = currentPage.value * questionsPerPage
  const end = start + questionsPerPage
  return allQuestions.value.slice(start, end)
})

const currentQuestionIndex = computed(() => currentPage.value * questionsPerPage)

const currentSubject = computed(() => {
  if (currentPageQuestions.value.length === 0) return { name: '' }
  return { name: currentPageQuestions.value[0].subject }
})

const isLastPage = computed(() => {
  return (currentPage.value + 1) * questionsPerPage >= totalQuestions.value
})


const getQuestionNumber = (index) => {
  return currentPage.value * questionsPerPage + index + 1
}

const previousPage = () => {
  if (currentPage.value > 0) {
    currentPage.value--
  }
}

const nextPage = () => {
  if (!isLastPage.value) {
    currentPage.value++
  }
}

const submitQuiz = () => {
  calculateScore()
  showResults.value = true
}

const calculateScore = () => {
  let totalScore = 0
  let totalSkipped = 0
  let totalAnswered = 0
  const subjectData = {}

  assessmentData.value.subjects.forEach(subject => {
    subjectData[subject.name] = { correct: 0, total: 0, skipped: 0, answered: 0 }
    
    subject.questions.forEach(question => {
      subjectData[subject.name].total++
      
      if (answers.value[question.id] === undefined) {
        // Question was skipped
        totalSkipped++
        subjectData[subject.name].skipped++
      } else {
        // Question was answered
        totalAnswered++
        subjectData[subject.name].answered++
        
        if (answers.value[question.id] === question.correctAnswer) {
          totalScore++
          subjectData[subject.name].correct++
        }
      }
    })
  })

  score.value = totalScore
  skippedCount.value = totalSkipped
  answeredQuestions.value = totalAnswered
  
  subjectScores.value = Object.entries(subjectData).map(([name, data]) => ({
    name,
    score: data.correct,
    total: data.total,
    skipped: data.skipped,
    percentage: data.answered > 0 
      ? ((data.correct / data.answered) * 100).toFixed(1) 
      : '0.0'
  }))
}

const handleKeydown = (event) => {
  // Only handle arrow keys when not in results view
  if (showResults.value) return
  
  // Don't interfere with form inputs
  if (event.target.tagName === 'INPUT' || event.target.tagName === 'TEXTAREA') return
  
  if (event.key === 'ArrowLeft' && currentPage.value > 0) {
    previousPage()
  } else if (event.key === 'ArrowRight') {
    if (!isLastPage.value) {
      nextPage()
    }
  }
}

onMounted(async () => {
  // Add keyboard event listener
  window.addEventListener('keydown', handleKeydown)
  const assessmentId = route.params.assessment
  
  try {
    // Check if it's an uploaded assessment first
    if (assessmentId.startsWith('uploaded-')) {
      const storedData = localStorage.getItem(`assessment-${assessmentId}`)
      if (storedData) {
        assessmentData.value = JSON.parse(storedData)
        return
      }
    }
    
    // Otherwise, try to fetch from public folder
    const response = await fetch(`/assessments/${assessmentId}.json`)
    if (!response.ok) {
      throw new Error('Assessment not found')
    }
    assessmentData.value = await response.json()
  } catch (error) {
    console.error('Error loading assessment:', error)
    router.push('/')
  }
})

onUnmounted(() => {
  // Remove keyboard event listener
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<style scoped>
.assessment-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
}

.assessment-header {
  text-align: center;
  margin-bottom: 2rem;
}

.assessment-header h1 {
  color: #2c3e50;
  margin-bottom: 1rem;
}

.progress-bar {
  width: 100%;
  height: 8px;
  background-color: #e0e0e0;
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 0.5rem;
}

.progress-fill {
  height: 100%;
  background-color: #42b883;
  transition: width 0.3s ease;
}

.progress-text {
  color: #666;
  font-size: 0.9rem;
}

.quiz-content {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.question-group h2 {
  color: #2c3e50;
  margin-bottom: 1.5rem;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid #e0e0e0;
}

.question {
  margin-bottom: 2rem;
}

.question:last-child {
  margin-bottom: 0;
}

.question h3 {
  color: #333;
  margin-bottom: 1rem;
  font-size: 1.1rem;
}

.options {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.option {
  display: flex;
  align-items: center;
  padding: 0.75rem 1rem;
  background: #f8f9fa;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.option:hover {
  background: #e9ecef;
}

.option input[type="radio"] {
  margin-right: 0.75rem;
}

.navigation {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 2rem;
  padding-top: 2rem;
  border-top: 1px solid #e0e0e0;
}

.top-nav {
  margin-top: 0;
  margin-bottom: 1rem;
  padding-top: 0;
  padding-bottom: 0;
  border-top: none;
  border-bottom: none;
}

.nav-button {
  padding: 0.75rem 1.5rem;
  border: none;
  border-radius: 4px;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.2s;
  background: #e0e0e0;
  color: #333;
}

.nav-button:hover:not(:disabled) {
  background: #d0d0d0;
}

.nav-button.primary {
  background: #42b883;
  color: white;
}

.nav-button.primary:hover:not(:disabled) {
  background: #35a372;
}

.nav-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.results {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  text-align: center;
}

.results h2 {
  color: #2c3e50;
  margin-bottom: 2rem;
}

.score-card {
  background: #f0f9ff;
  padding: 2rem;
  border-radius: 8px;
  margin-bottom: 2rem;
}

.score-card h3 {
  color: #2c3e50;
  margin-bottom: 0.5rem;
  font-size: 1.5rem;
}

.score-card p {
  color: #42b883;
  font-size: 2rem;
  font-weight: bold;
  margin: 0;
}

.skipped-info {
  margin-top: 0.5rem;
  color: #ff9800;
  font-size: 1rem;
}

.subject-scores {
  text-align: left;
  margin-bottom: 2rem;
}

.subject-scores h3 {
  color: #2c3e50;
  margin-bottom: 1rem;
}

.subject-score {
  display: flex;
  justify-content: space-between;
  padding: 0.5rem 0;
  border-bottom: 1px solid #e0e0e0;
}

.loading {
  text-align: center;
  padding: 4rem;
  color: #666;
}
</style>