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
      
      <div class="instructions-box">
        <h3>How to Create AI-Generated Assessments</h3>
        <ol>
          <li>Download our <a href="/assessment-generation-instructions.md" download class="download-link">Assessment Generation Instructions</a></li>
          <li>Upload the instructions file to your AI chat (ChatGPT, Claude, etc.)</li>
          <li>Use one of the example prompts below (or create your own)</li>
          <li>Copy the generated JSON and paste it below</li>
        </ol>
      </div>

      <div class="input-methods">
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
            <svg class="upload-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M21 15v4a2 2 0 01-2 2H5a2 2 0 01-2-2v-4M17 8l-5-5-5 5M12 3v12"/>
            </svg>
            Upload JSON File
          </label>
        </div>
        
        <div class="divider">
          <span>OR</span>
        </div>
        
        <div class="paste-container">
          <textarea 
            v-model="pastedJson"
            placeholder="Paste your assessment JSON here..."
            class="json-textarea"
            rows="2"
          ></textarea>
          <button @click="handlePastedJson" class="submit-button">
            Create Assessment
          </button>
        </div>
      </div>

      <div class="example-prompts">
        <h3>Example Prompts</h3>
        <p class="prompts-description">Use these prompts with the instructions file for best results:</p>
        
        <div class="prompt-example">
          <div class="prompt-header">
            <h4>Environmental Science</h4>
            <button @click="copyToClipboard(simplePrompt1)" class="copy-button">Copy Prompt</button>
          </div>
          <pre>{{ simplePrompt1 }}</pre>
        </div>
        
        <div class="prompt-example">
          <div class="prompt-header">
            <h4>World History</h4>
            <button @click="copyToClipboard(simplePrompt2)" class="copy-button">Copy Prompt</button>
          </div>
          <pre>{{ simplePrompt2 }}</pre>
        </div>
        
        <div class="prompt-example">
          <div class="prompt-header">
            <h4>Mathematics</h4>
            <button @click="copyToClipboard(simplePrompt3)" class="copy-button">Copy Prompt</button>
          </div>
          <pre>{{ simplePrompt3 }}</pre>
        </div>
        
        <div class="prompt-example">
          <div class="prompt-header">
            <h4>Custom Subject</h4>
            <button @click="copyToClipboard(simplePromptCustom)" class="copy-button">Copy Prompt</button>
          </div>
          <pre>{{ simplePromptCustom }}</pre>
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

// Simplified prompts for use with the instructions file
const simplePrompt1 = `Create an assessment file for Environmental Science suitable for high school students (grades 9-10).

The assessment should:
- Have 20 total questions
- Be divided into 4 subjects: Climate Change, Ecosystems, Pollution, and Conservation
- Include multiple choice questions only
- Cover basic concepts and current environmental issues
- Be at intermediate level

Follow the attached assessment generation instructions.`

const simplePrompt2 = `Create an assessment file for World History focusing on Ancient Civilizations for middle school students.

The assessment should:
- Have 24 total questions
- Cover these civilizations: Ancient Egypt, Ancient Greece, Ancient Rome, and Ancient China
- Include questions about government, culture, achievements, and key historical figures
- Be at beginner to intermediate level

Follow the attached assessment generation instructions.`

const simplePrompt3 = `Create an assessment file for Algebra I suitable for 8th-9th grade students.

The assessment should:
- Have 20 total questions
- Cover 4 topics: Linear Equations, Polynomials, Factoring, and Quadratic Equations
- Focus on problem-solving and application
- Be at intermediate level

Follow the attached assessment generation instructions.`

const simplePromptCustom = `Create an assessment file for [YOUR SUBJECT] suitable for [TARGET AUDIENCE].

The assessment should:
- Have [NUMBER] total questions
- Cover these topics: [LIST TOPICS]
- Include [SPECIFY QUESTION TYPES OR FOCUS]
- Be at [DIFFICULTY] level

Follow the attached assessment generation instructions.`

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
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  padding: 1rem 1.5rem;
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

.instructions-box {
  background: #f0f9ff;
  border: 2px solid #42b883;
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 2rem;
}

.instructions-box h3 {
  margin: 0 0 1rem 0;
  color: #2c3e50;
}

.instructions-box ol {
  margin: 0;
  padding-left: 1.5rem;
  color: #495057;
  line-height: 1.8;
}

.instructions-box li {
  margin-bottom: 0.5rem;
}

.input-methods {
  margin-bottom: 3rem;
}

.upload-container {
  width: 100%;
  text-align: center;
  margin-bottom: 1.5rem;
}

.divider {
  text-align: center;
  margin: 1.5rem 0;
  position: relative;
}

.divider::before {
  content: '';
  position: absolute;
  top: 50%;
  left: 0;
  right: 0;
  height: 1px;
  background: #e0e0e0;
}

.divider span {
  background: white;
  padding: 0 1rem;
  position: relative;
  color: #999;
  font-weight: 600;
  text-transform: uppercase;
  font-size: 0.875rem;
}

.paste-container {
  display: flex;
  gap: 1rem;
  align-items: center;
}

.upload-icon {
  width: 20px;
  height: 20px;
  margin-right: 0.5rem;
  vertical-align: middle;
}

.json-textarea {
  flex: 1;
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-family: 'Courier New', monospace;
  font-size: 0.9rem;
  resize: none;
  line-height: 1.5;
}

.submit-button {
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

.example-prompts {
  margin-top: 3rem;
  padding-top: 2rem;
  border-top: 2px solid #e0e0e0;
}

.example-prompts h3 {
  color: #2c3e50;
  margin-bottom: 0.5rem;
}

.prompts-description {
  color: #666;
  margin-bottom: 2rem;
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

.instructions ol {
  margin: 0.5rem 0;
  padding-left: 1.5rem;
  color: #666;
}

.instructions li {
  margin: 0.5rem 0;
}

.download-link {
  color: #42b883;
  text-decoration: none;
  font-weight: 600;
}

.download-link:hover {
  text-decoration: underline;
}


.prompt-example {
  margin-bottom: 2rem;
  border: 1px solid #e0e0e0;
  border-radius: 4px;
  overflow: hidden;
}


.prompt-header {
  background: #f8f9fa;
  padding: 0.75rem 1rem;
  border-bottom: 1px solid #e0e0e0;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.prompt-example h4 {
  margin: 0;
  color: #2c3e50;
  font-size: 1.1rem;
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

@media (max-width: 768px) {
  .paste-container {
    flex-direction: column;
    gap: 0.5rem;
  }
  
  .json-textarea {
    width: 100%;
  }
  
  .example-prompts {
    margin-top: 2rem;
  }
}
</style>
