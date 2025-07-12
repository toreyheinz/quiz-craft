# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Quiz Craft is a Vue.js application for creating and taking AI-generated assessments. It features pre-built 9th and 10th-grade assessments and supports custom quiz creation through file upload, JSON paste, or AI generation.

## Architecture

- **Framework**: Vue 3 with Composition API (`<script setup>`)
- **Build Tool**: Vite 7.0.4
- **Router**: Vue Router 4.5.1
- **Storage**: LocalStorage for uploaded assessments
- **Deployment**: Configured for Cloudflare Pages
- **Testing**: No testing framework currently installed
- **Linting**: No ESLint/Prettier configuration

## Key Files

- `src/views/Home.vue` - Landing page with assessment list and upload functionality
- `src/views/Assessment.vue` - Quiz interface with pagination and scoring
- `src/router/index.js` - Route definitions
- `src/App.vue` - Root component with header/branding
- `public/assessments/*.json` - Pre-built assessment files
- `public/_redirects` - SPA routing configuration for Cloudflare

## Development Commands

```bash
# Install dependencies
npm install

# Run development server (http://localhost:5173)
npm run dev

# Build for production (outputs to dist/)
npm run build

# Preview production build
npm run preview
```

## Key Patterns

### Assessment JSON Structure
```json
{
  "title": "Assessment Title",
  "description": "Description",
  "subjects": [{
    "name": "Subject",
    "questions": [{
      "id": 1,
      "question": "Question?",
      "options": ["A", "B", "C", "D"],
      "correctAnswer": 0,
      "correctAnswerText": "Optional: The correct answer text if different from option",
      "explanation": "Optional: Detailed explanation shown for incorrect answers"
    }]
  }]
}
```
**Note**: `correctAnswer` uses 0-based indexing (0 = first option)

### Component Patterns
- Components use Vue 3 Composition API with `<script setup>`
- Styling is scoped to components using `<style scoped>`
- Navigation state managed with Vue Router
- Uploaded assessments stored in localStorage with `assessment-${id}` keys
- Router views use Suspense for loading states

### Code Organization
```
src/
├── views/          # Page components
├── components/     # Reusable components (empty, components are in views)
├── router/         # Route configuration
├── assets/         # Static assets
└── style.css       # Global styles
```

## Deployment

Configured for Cloudflare Pages:
- Build command: `npm run build`
- Output directory: `dist`
- SPA routing handled by `public/_redirects`
- Repository: https://github.com/toreyheinz/quiz-craft

## Design System

- Primary color: #42b883 (Vue green)
- Secondary: #35495e (dark gray)
- Max content width: 800px
- Questions per page: 5
- Logo: 48px height with 5px border radius
- Mobile breakpoint: 600px
- Button hover: darken by ~10%

## Current Features

1. **Pre-built Assessments**: 9th/10th grade readiness tests
2. **Custom Assessment Methods**:
   - File upload (JSON)
   - Paste JSON content
   - AI generation guide with example prompts
3. **Quiz Interface**:
   - Paginated questions (5 per page)
   - Skip functionality
   - Progress tracking with visual bar
   - Score display with subject breakdown
   - Shows incorrect answers with correct solutions

## Planned Enhancements

Based on `dev/plans/`, the roadmap includes:
1. **User Authentication** - JWT-based with Cloudflare Workers
2. **Database Integration** - Cloudflare D1 for persistent storage
3. **AI Quiz Generation** - Cloudflare AI integration for premium users
4. **Freemium Model** - Free manual upload, premium AI generation