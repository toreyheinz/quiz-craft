# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Quiz Craft is a Vue.js application for creating and taking AI-generated assessments. It features pre-built 9th-grade assessments and supports custom quiz creation through file upload or AI generation.

## Architecture

- **Framework**: Vue 3 with Composition API (`<script setup>`)
- **Build Tool**: Vite
- **Router**: Vue Router 4
- **Storage**: LocalStorage for uploaded assessments
- **Deployment**: Configured for Cloudflare Pages

## Key Files

- `src/views/Home.vue` - Landing page with assessment list and upload functionality
- `src/views/Assessment.vue` - Quiz interface with pagination and scoring
- `src/router/index.js` - Route definitions
- `public/assessments/*.json` - Pre-built assessment files

## Development Commands

```bash
# Install dependencies
npm install

# Run development server
npm run dev

# Build for production
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
      "correctAnswer": 0
    }]
  }]
}
```

### Adding New Features
- Components use Vue 3 Composition API with `<script setup>`
- Styling is scoped to components
- Navigation state managed with Vue Router
- Uploaded assessments stored in localStorage with `assessment-${id}` keys

## Deployment

Configured for Cloudflare Pages:
- Build command: `npm run build`
- Output directory: `dist`
- SPA routing handled by `public/_redirects`

## Design System

- Primary color: #42b883 (Vue green)
- Max content width: 800px
- Questions per page: 5
- Logo: 48px height with 5px border radius