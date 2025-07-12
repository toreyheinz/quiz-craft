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
      <h2>Create Custom Assessment</h2>
      <p>Upload a JSON file or paste JSON content to create a custom assessment</p>
      
      <div class="tab-buttons">
        <button 
          @click="activeTab = 'upload'" 
          :class="{ active: activeTab === 'upload' }"
          class="tab-button"
        >
          Upload File
        </button>
        <button 
          @click="activeTab = 'paste'" 
          :class="{ active: activeTab === 'paste' }"
          class="tab-button"
        >
          Paste JSON
        </button>
        <button 
          @click="activeTab = 'guide'" 
          :class="{ active: activeTab === 'guide' }"
          class="tab-button"
        >
          AI Guide
        </button>
      </div>

      <div v-if="activeTab === 'upload'" class="tab-content">
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
      </div>

      <div v-if="activeTab === 'paste'" class="tab-content">
        <textarea 
          v-model="pastedJson"
          placeholder="Paste your assessment JSON here..."
          class="json-textarea"
          rows="10"
        ></textarea>
        <button @click="handlePastedJson" class="submit-button">
          Create Assessment
        </button>
      </div>

      <div v-if="activeTab === 'guide'" class="tab-content guide-content">
        <h3>How to Create Assessments with AI</h3>
        
        <div class="instructions">
          <p><strong>Step 1:</strong> Copy one of the example prompts below</p>
          <p><strong>Step 2:</strong> Paste it into your AI chat client (ChatGPT, Claude, etc.)</p>
          <p><strong>Step 3:</strong> Copy the generated JSON from the AI's response</p>
          <p><strong>Step 4:</strong> Switch to the "Paste JSON" tab and paste it there</p>
        </div>

        <h4>Example Prompts</h4>
        
        <div class="prompt-example">
          <h5>Example 1: Science Assessment</h5>
          <button @click="copyToClipboard(sciencePrompt)" class="copy-button">Copy Prompt</button>
          <pre>{{ sciencePrompt }}</pre>
        </div>

        <div class="prompt-example">
          <h5>Example 2: Math Assessment</h5>
          <button @click="copyToClipboard(mathPrompt)" class="copy-button">Copy Prompt</button>
          <pre>{{ mathPrompt }}</pre>
        </div>

        <div class="prompt-example">
          <h5>Example 3: Language Assessment</h5>
          <button @click="copyToClipboard(languagePrompt)" class="copy-button">Copy Prompt</button>
          <pre>{{ languagePrompt }}</pre>
        </div>

        <div class="prompt-example">
          <h5>Example 4: Custom Assessment</h5>
          <button @click="copyToClipboard(customPrompt)" class="copy-button">Copy Prompt</button>
          <pre>{{ customPrompt }}</pre>
        </div>

        <div class="tips">
          <h4>Tips for Best Results:</h4>
          <ul>
            <li>Be specific about the topics you want covered</li>
            <li>Specify the difficulty level clearly</li>
            <li>Always verify the AI's answers are correct</li>
            <li>Start with fewer questions to test the format</li>
            <li>Make sure the JSON is valid before pasting</li>
          </ul>
        </div>
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
const activeTab = ref('upload')
const pastedJson = ref('')

const jsonStructure = `{
  "title": "Assessment title",
  "description": "Brief description",
  "subjects": [
    {
      "name": "Subject name",
      "questions": [
        {
          "id": unique number,
          "question": "Question text?",
          "options": ["Option A", "Option B", "Option C", "Option D"],
          "correctAnswer": index (0-3)
        }
      ]
    }
  ]
}`

const importantNotes = `IMPORTANT: 
- Each question must have exactly 4 options
- correctAnswer must be the array index (0-3) of the correct option
- All question IDs must be unique numbers
- Ensure the JSON is valid and properly formatted`

const sciencePrompt = `Create a JSON assessment file for Environmental Science suitable for high school students (grades 9-10).

The assessment should:
- Have 20 total questions
- Be divided into 4 subjects: Climate Change, Ecosystems, Pollution, and Conservation
- Include multiple choice questions only
- Cover basic concepts and current environmental issues
- Be at intermediate level

Format the output as a valid JSON file following this exact structure:
${jsonStructure}

${importantNotes}`

const mathPrompt = `Create a JSON assessment file for Algebra I suitable for 8th-9th grade students.

The assessment should:
- Have 24 total questions
- Be divided into 4 subjects: Linear Equations, Quadratic Functions, Systems of Equations, and Polynomials
- Include multiple choice questions only
- Include both conceptual and computational questions
- Be at intermediate level with some challenging problems

Format the output as a valid JSON file following this exact structure:
${jsonStructure}

${importantNotes}`

const languagePrompt = `Create a JSON assessment file for Spanish Language (Level 2) suitable for intermediate learners.

The assessment should:
- Have 20 total questions
- Be divided into 4 subjects: Vocabulary, Grammar, Reading Comprehension, and Cultural Knowledge
- Include multiple choice questions only
- Focus on present and past tenses, common vocabulary, and Hispanic culture
- Be at intermediate level

Format the output as a valid JSON file following this exact structure:
${jsonStructure}

${importantNotes}`

const customPrompt = `Create a JSON assessment file for [YOUR SUBJECT HERE] suitable for [YOUR AUDIENCE HERE].

The assessment should:
- Have [NUMBER] total questions
- Be divided into [NUMBER] subjects: [LIST YOUR TOPICS HERE]
- Include multiple choice questions only
- Cover [DESCRIBE WHAT TO COVER]
- Be at [DIFFICULTY LEVEL] level

Format the output as a valid JSON file following this exact structure:
${jsonStructure}

${importantNotes}`

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

const createAssessment = (data) => {
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
  
  return assessment
}

const handleFileUpload = async (event) => {
  uploadError.value = ''
  const file = event.target.files[0]
  
  if (!file) return
  
  try {
    const text = await file.text()
    const data = JSON.parse(text)
    
    validateAssessment(data)
    createAssessment(data)
    
    // Clear the file input
    if (fileInput.value) {
      fileInput.value.value = ''
    }
    
    uploadError.value = ''
    
  } catch (error) {
    uploadError.value = error.message || 'Failed to parse JSON file'
    console.error('Upload error:', error)
  }
}

const handlePastedJson = () => {
  uploadError.value = ''
  
  if (!pastedJson.value.trim()) {
    uploadError.value = 'Please paste JSON content'
    return
  }
  
  try {
    const data = JSON.parse(pastedJson.value)
    validateAssessment(data)
    createAssessment(data)
    
    // Clear the textarea
    pastedJson.value = ''
    uploadError.value = ''
    
    // Switch to upload tab to show success
    activeTab.value = 'upload'
    
  } catch (error) {
    uploadError.value = error.message || 'Failed to parse JSON content'
    console.error('Paste error:', error)
  }
}

const copyToClipboard = async (text) => {
  try {
    await navigator.clipboard.writeText(text)
    // You could add a success notification here
  } catch (err) {
    console.error('Failed to copy:', err)
    // Fallback for older browsers
    const textArea = document.createElement('textarea')
    textArea.value = text
    textArea.style.position = 'fixed'
    textArea.style.left = '-999999px'
    document.body.appendChild(textArea)
    textArea.select()
    try {
      document.execCommand('copy')
    } catch (err) {
      console.error('Fallback copy failed:', err)
    }
    document.body.removeChild(textArea)
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
  margin-top: 1rem;
}

.tab-buttons {
  display: flex;
  gap: 1rem;
  margin-bottom: 1.5rem;
  border-bottom: 2px solid #e0e0e0;
}

.tab-button {
  padding: 0.75rem 1.5rem;
  background: none;
  border: none;
  border-bottom: 2px solid transparent;
  color: #666;
  cursor: pointer;
  font-size: 1rem;
  transition: all 0.2s;
}

.tab-button:hover {
  color: #333;
}

.tab-button.active {
  color: #42b883;
  border-bottom-color: #42b883;
}

.tab-content {
  padding: 1rem 0;
}

.json-textarea {
  width: 100%;
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-family: 'Courier New', monospace;
  font-size: 0.9rem;
  resize: vertical;
  min-height: 200px;
}

.submit-button {
  margin-top: 1rem;
  padding: 0.75rem 1.5rem;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
  transition: background-color 0.2s;
}

.submit-button:hover {
  background: #35a372;
}

.guide-content h3 {
  color: #2c3e50;
  margin-bottom: 1rem;
}

.guide-content h4 {
  color: #2c3e50;
  margin: 1.5rem 0 1rem;
}

.guide-content h5 {
  color: #333;
  margin-bottom: 0.5rem;
}

.instructions {
  background: #f0f9ff;
  padding: 1rem;
  border-radius: 4px;
  margin-bottom: 1.5rem;
}

.instructions p {
  margin: 0.5rem 0;
  color: #666;
}

.prompt-example {
  margin-bottom: 2rem;
  border: 1px solid #e0e0e0;
  border-radius: 4px;
  overflow: hidden;
}

.prompt-example h5 {
  background: #f8f9fa;
  padding: 0.75rem 1rem;
  margin: 0;
  border-bottom: 1px solid #e0e0e0;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.copy-button {
  padding: 0.5rem 1rem;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.875rem;
  transition: background-color 0.2s;
  float: right;
  margin-left: 1rem;
}

.copy-button:hover {
  background: #35a372;
}

.prompt-example pre {
  margin: 0;
  padding: 1rem;
  background: #f8f9fa;
  overflow-x: auto;
  font-family: 'Courier New', monospace;
  font-size: 0.875rem;
  line-height: 1.5;
  white-space: pre-wrap;
  word-wrap: break-word;
}

.tips {
  background: #f0f9ff;
  padding: 1rem;
  border-radius: 4px;
  margin-top: 1.5rem;
}

.tips h4 {
  color: #2c3e50;
  margin-bottom: 0.5rem;
}

.tips ul {
  margin: 0;
  padding-left: 1.5rem;
  color: #666;
}

.tips li {
  margin-bottom: 0.25rem;
}
</style>
