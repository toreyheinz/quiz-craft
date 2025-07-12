# Quiz Craft

> Create and take AI-generated assessments with ease

Quiz Craft is a Vue.js web application that enables users to create, upload, and take custom assessments. It features built-in 9th-grade readiness assessments and supports AI-generated quizzes through a simple copy-paste workflow.

## 🌟 Features

- **Pre-built Assessments**: Ready-to-use 9th-grade assessments covering History, Geography, Science, Math, and Language Arts
- **AI Integration**: Built-in prompts for generating assessments with ChatGPT, Claude, or other AI assistants
- **Multiple Input Methods**: Upload JSON files or paste JSON directly
- **Paginated Interface**: Questions displayed 5 per page for better user experience
- **Progress Tracking**: Visual progress bar and question counter
- **Comprehensive Scoring**: Total score plus subject-by-subject breakdown
- **Local Storage**: Uploaded assessments persist across sessions
- **Responsive Design**: Works on desktop and mobile devices

## 🚀 Live Demo

Visit [quiz-craft.example.com](https://quiz-craft.example.com) (update with your deployed URL)

## 🛠️ Technology Stack

- **Vue 3** with Composition API
- **Vite** for fast development and building
- **Vue Router** for navigation
- **LocalStorage** for data persistence
- **Cloudflare Pages** ready

## 📦 Installation

```bash
# Clone the repository
git clone https://github.com/toreyheinz/quiz-craft.git
cd quiz-craft

# Install dependencies
npm install

# Start development server
npm run dev
```

## 🏗️ Building for Production

```bash
# Build for production
npm run build

# Preview production build
npm run preview
```

## ☁️ Deployment

### Cloudflare Pages

1. Push your code to GitHub
2. Connect your GitHub repository to Cloudflare Pages
3. Set build configuration:
   - Build command: `npm run build`
   - Build output directory: `dist`
   - Root directory: `/`

### Other Platforms

The built files in `dist/` can be deployed to any static hosting service.

## 📝 Creating Custom Assessments

### Method 1: Using AI

1. Go to the "AI Guide" tab
2. Copy one of the example prompts
3. Paste into your favorite AI chat (ChatGPT, Claude, etc.)
4. Copy the generated JSON
5. Paste into the "Paste JSON" tab

### Method 2: Manual Creation

Create a JSON file following this structure:

```json
{
  "title": "Your Assessment Title",
  "description": "Brief description",
  "subjects": [
    {
      "name": "Subject Name",
      "questions": [
        {
          "id": 1,
          "question": "Question text?",
          "options": ["Option A", "Option B", "Option C", "Option D"],
          "correctAnswer": 0
        }
      ]
    }
  ]
}
```

**Important Notes:**
- `correctAnswer` uses 0-based indexing (0 = first option, 1 = second, etc.)
- Each question must have exactly 4 options
- Question IDs must be unique numbers

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Built with [Vue.js](https://vuejs.org/) and [Vite](https://vitejs.dev/)
- Inspired by the need for easy assessment creation
- Thanks to all contributors

## 📧 Contact

Torey Heinz - [GitHub](https://github.com/toreyheinz)

Project Link: [https://github.com/toreyheinz/quiz-craft](https://github.com/toreyheinz/quiz-craft)