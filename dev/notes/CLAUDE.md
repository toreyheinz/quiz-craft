# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SelfQuiz is a collection of standalone HTML files for 9th-grade readiness assessments. Each file is a self-contained quiz application with embedded CSS and JavaScript.

## Architecture

The project contains three quiz implementations:
- `index.html` - Compact implementation with 40 questions
- `chatgpt.html` - Identical to index.html  
- `grok.html` - Expanded implementation with traditional form structure

All files follow the same pattern:
1. Single HTML file with inline CSS and JavaScript
2. Client-side scoring (answers visible in source)
3. No external dependencies or build process
4. Questions stored in JavaScript data structures

## Development Commands

- **Run**: Open any HTML file directly in a web browser
- **Test**: Manual browser testing only
- **Deploy**: Copy HTML files to web server

## Key Implementation Details

### Question Format (index.html/chatgpt.html)
Questions use object structure:
```javascript
{ q: "question text", a: correctAnswerIndex, options: ["opt1", "opt2", "opt3", "opt4"] }
```

### Question Format (grok.html)
Questions use traditional HTML forms with answer key object:
```javascript
correctAnswers = { q1: 'b', q2: 'a', ... }
```

### Subject Coverage
- Questions 1-8: History
- Questions 9-16: Geography  
- Questions 17-24: Science
- Questions 25-32: Mathematics
- Questions 33-40: English Language Arts