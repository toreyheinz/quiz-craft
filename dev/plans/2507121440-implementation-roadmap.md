# Implementation Roadmap: Quiz Craft Premium Platform

**Status**: Planning
**Created**: 2025-07-12
**Author**: Claude

## Overview

Transform Quiz Craft from a static quiz application into a full-featured platform with user accounts, database storage, and AI-powered quiz generation for premium users. This roadmap outlines the implementation order and dependencies.

## Project Vision

- **Primary**: AI-powered quiz generation as the main creation method
- **Secondary**: Manual upload/paste functionality for all users
- **Business Model**: Freemium with AI generation limited to premium users
- **Infrastructure**: Built on Cloudflare's edge platform (Workers, D1, AI)

## Implementation Phases

### Phase 1: User Authentication & Accounts (Week 1-2)
**Goal**: Implement secure user authentication system

**Features**:
- User registration and login
- JWT-based authentication
- User profile management
- Free vs Premium tier distinction

**Technical Stack**:
- Cloudflare Workers for auth API
- Cloudflare KV for session storage
- Vue.js auth components
- Pinia for state management

**Key Files**:
- `cloudflare-worker/src/auth/*` - Authentication logic
- `assessment-app/src/stores/auth.js` - Auth state
- `assessment-app/src/components/Auth/*` - Login/Register components

### Phase 2: Database Infrastructure (Week 2-3)
**Goal**: Implement Cloudflare D1 database for persistent storage

**Database Schema**:
```sql
-- Users table
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  tier TEXT DEFAULT 'free', -- 'free' or 'premium'
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  ai_credits INTEGER DEFAULT 0
);

-- Quizzes table
CREATE TABLE quizzes (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER,
  title TEXT NOT NULL,
  description TEXT,
  content JSON NOT NULL, -- Full quiz JSON
  is_public BOOLEAN DEFAULT false,
  source TEXT NOT NULL, -- 'ai', 'upload', 'paste'
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

-- Results table
CREATE TABLE results (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER,
  quiz_id INTEGER,
  score INTEGER NOT NULL,
  total_questions INTEGER NOT NULL,
  answers JSON NOT NULL, -- User's answers
  completed_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id),
  FOREIGN KEY (quiz_id) REFERENCES quizzes(id)
);

-- AI Generation History (for tracking usage)
CREATE TABLE ai_generations (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  prompt TEXT NOT NULL,
  tokens_used INTEGER,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**Features**:
- Store user-created quizzes
- Track quiz results and history
- Public quiz library
- User quiz management

### Phase 3: AI Quiz Generation for Premium Users (Week 3-4)
**Goal**: Implement Cloudflare AI Workers integration with premium tier restrictions

**Features**:
- Interactive chat interface for quiz refinement
- Real-time JSON validation
- Credit-based system for premium users
- Usage tracking and limits

**Architecture Updates**:
```javascript
// Check user tier before AI generation
if (user.tier !== 'premium' || user.ai_credits <= 0) {
  return Response.json({ 
    error: 'AI generation requires premium account' 
  }, { status: 403 });
}
```

**UI Flow**:
1. User clicks "Create with AI" (primary CTA)
2. Free users see upgrade prompt
3. Premium users enter chat interface
4. Iterative refinement through conversation
5. Generate and validate final quiz
6. Save to database

### Phase 4: Premium Features & Monetization (Week 4-5)
**Goal**: Implement premium tier benefits and payment integration

**Premium Features**:
- Unlimited AI quiz generation
- Advanced quiz analytics
- Export options (PDF, CSV)
- Private quiz library
- Custom branding

**Implementation**:
- Stripe integration for payments
- Subscription management
- Usage dashboard
- Billing portal

## Technical Architecture

### Frontend Structure
```
assessment-app/
├── src/
│   ├── views/
│   │   ├── Auth/
│   │   │   ├── Login.vue
│   │   │   └── Register.vue
│   │   ├── Dashboard.vue      # User dashboard
│   │   ├── QuizBuilder.vue    # AI chat interface
│   │   └── Premium.vue        # Upgrade page
│   ├── stores/
│   │   ├── auth.js
│   │   └── quiz.js
│   └── services/
│       ├── api.js
│       └── auth.js
```

### Backend Structure
```
cloudflare-worker/
├── src/
│   ├── index.js           # Main worker
│   ├── auth/
│   │   ├── jwt.js
│   │   └── middleware.js
│   ├── db/
│   │   ├── schema.sql
│   │   └── queries.js
│   ├── ai/
│   │   ├── chat.js
│   │   ├── generation.js
│   │   └── validation.js
│   └── routes/
│       ├── auth.js
│       ├── quiz.js
│       └── user.js
```

## Migration Strategy

1. **Preserve Existing Functionality**: Keep current quiz-taking experience
2. **Progressive Enhancement**: Add features without breaking existing ones
3. **Data Migration**: Import existing JSON quizzes into D1
4. **Backward Compatibility**: Support direct JSON access for legacy links

## Security Considerations

- JWT tokens with short expiration
- Rate limiting on all endpoints
- Input validation and sanitization
- CORS configuration for API
- Secure password hashing (bcrypt)
- SQL injection prevention

## Performance Optimization

- Edge caching for public quizzes
- Lazy loading for dashboard
- Optimistic UI updates
- Background job queue for AI generation
- CDN for static assets

## Success Metrics

- User registration rate
- Free to premium conversion rate
- AI generation usage per premium user
- Quiz completion rates
- User retention (30-day)

## Risk Mitigation

- **AI Cost Overrun**: Implement strict credit system and monitoring
- **Database Performance**: Use appropriate indexes and query optimization
- **User Adoption**: Clear value proposition for premium tier
- **Technical Debt**: Modular architecture for easy updates

## Next Steps

1. Set up Cloudflare Workers project with D1
2. Implement basic authentication system
3. Create database schema and migrations
4. Build user dashboard MVP
5. Integrate AI generation with auth checks
6. Add payment processing
7. Launch beta with limited users

## Dependencies

- Cloudflare Workers account with AI access
- Cloudflare D1 database
- Stripe account for payments
- Domain with Cloudflare DNS
- SSL certificate

This roadmap provides a clear path from the current static application to a full-featured SaaS platform with AI-powered quiz generation as the primary value proposition.