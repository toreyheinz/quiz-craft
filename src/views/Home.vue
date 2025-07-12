<template>
  <div class="home">
    <h1>Student Assessment Portal</h1>
    <p>Select an assessment to begin:</p>

    <div class="assessment-list">
      <router-link
        v-for="assessment in allAssessments"
        :key="assessment.id"
        :to="`/${assessment.id}`"
        class="assessment-card"
      >
        <h3>{{ assessment.title }}</h3>
        <p>{{ assessment.description }}</p>
        <span v-if="assessment.isUploaded" class="uploaded-badge">Uploaded</span>
      </router-link>
    </div>

    <div class="upload-section">
      <h2>Upload Custom Assessment</h2>
      <p>Upload a JSON file to create a custom assessment</p>
      <div class="upload-container">
        <input 
          type="file" 
          ref="fileInput"
          @change="handleFileUpload"
          accept=".json"
          id="file-upload"
          class="file-input"
        />
        <label for="file-upload" class="file-label">
          Choose JSON File
        </label>
      </div>
      <div v-if="uploadError" class="error-message">
        {{ uploadError }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const availableAssessments = ref([
  {
    id: '9th-grade-assessment-chatgpt',
    title: '9th Grade Readiness Assessment (ChatGPT)',
    description: 'Test your knowledge in History, Geography, Science, Math, and Language Arts'
  },
  {
    id: '9th-grade-assessment-grok',
    title: '9th Grade Readiness Assessment (Grok)',
    description: 'Alternative 9th grade assessment with different questions'
  },
  {
    id: '10th-grade-assessment',
    title: '10th Grade Readiness Assessment',
    description: 'Sample assessment for 10th grade students'
  }
])

const uploadedAssessments = ref([])
const uploadError = ref('')
const fileInput = ref(null)

const allAssessments = computed(() => {
  return [...availableAssessments.value, ...uploadedAssessments.value]
})

const loadUploadedAssessments = () => {
  const stored = localStorage.getItem('uploadedAssessments')
  if (stored) {
    try {
      uploadedAssessments.value = JSON.parse(stored)
    } catch (e) {
      console.error('Failed to load uploaded assessments:', e)
    }
  }
}

const validateAssessment = (data) => {
  if (!data.title || !data.description || !data.subjects) {
    throw new Error('Assessment must have title, description, and subjects')
  }
  
  if (!Array.isArray(data.subjects) || data.subjects.length === 0) {
    throw new Error('Assessment must have at least one subject')
  }
  
  for (const subject of data.subjects) {
    if (!subject.name || !subject.questions || !Array.isArray(subject.questions)) {
      throw new Error('Each subject must have a name and questions array')
    }
    
    for (const question of subject.questions) {
      if (!question.id || !question.question || !question.options || typeof question.correctAnswer !== 'number') {
        throw new Error('Each question must have id, question text, options, and correctAnswer')
      }
      
      if (!Array.isArray(question.options) || question.options.length < 2) {
        throw new Error('Each question must have at least 2 options')
      }
      
      if (question.correctAnswer < 0 || question.correctAnswer >= question.options.length) {
        throw new Error('correctAnswer must be a valid index for the options array')
      }
    }
  }
  
  return true
}

const handleFileUpload = async (event) => {
  uploadError.value = ''
  const file = event.target.files[0]
  
  if (!file) return
  
  try {
    const text = await file.text()
    const data = JSON.parse(text)
    
    validateAssessment(data)
    
    const assessmentId = `uploaded-${Date.now()}`
    const assessment = {
      id: assessmentId,
      title: data.title,
      description: data.description,
      isUploaded: true
    }
    
    // Store the full assessment data in localStorage
    localStorage.setItem(`assessment-${assessmentId}`, JSON.stringify(data))
    
    // Add to uploaded assessments list
    uploadedAssessments.value.push(assessment)
    
    // Save the list of uploaded assessments
    localStorage.setItem('uploadedAssessments', JSON.stringify(uploadedAssessments.value))
    
    // Clear the file input
    if (fileInput.value) {
      fileInput.value.value = ''
    }
    
  } catch (error) {
    uploadError.value = error.message || 'Failed to parse JSON file'
    console.error('Upload error:', error)
  }
}

onMounted(() => {
  loadUploadedAssessments()
})
</script>

<style scoped>
.home {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
}

h1 {
  color: #2c3e50;
  margin-bottom: 1rem;
  text-align: center;
}

p {
  text-align: center;
  margin-bottom: 2rem;
  color: #666;
}

.assessment-list {
  display: grid;
  gap: 1rem;
}

.assessment-card {
  display: block;
  padding: 1.5rem;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  text-decoration: none;
  color: inherit;
  transition: transform 0.2s, box-shadow 0.2s;
}

.assessment-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.15);
}

.assessment-card h3 {
  color: #2c3e50;
  margin-bottom: 0.5rem;
}

.assessment-card p {
  color: #666;
  margin: 0;
  text-align: left;
}

.assessment-card {
  position: relative;
}

.uploaded-badge {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: #42b883;
  color: white;
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: bold;
}

.upload-section {
  margin-top: 3rem;
  padding: 2rem;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.upload-section h2 {
  color: #2c3e50;
  margin-bottom: 0.5rem;
}

.upload-section p {
  color: #666;
  margin-bottom: 1.5rem;
}

.upload-container {
  margin-bottom: 1rem;
}

.file-input {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0,0,0,0);
  white-space: nowrap;
  border: 0;
}

.file-label {
  display: inline-block;
  padding: 0.75rem 1.5rem;
  background: #42b883;
  color: white;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
  font-weight: 500;
}

.file-label:hover {
  background: #35a372;
}

.error-message {
  color: #dc3545;
  padding: 0.75rem;
  background: #f8d7da;
  border: 1px solid #f5c6cb;
  border-radius: 4px;
}
</style>
