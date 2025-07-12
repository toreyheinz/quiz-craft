# Feature Plan: Cloudflare D1 Database Implementation

**Status**: Planning
**Created**: 2025-07-12
**Author**: Claude
**Priority**: Phase 2 - Implement alongside authentication

## Overview

Implement Cloudflare D1 database to provide persistent storage for users, quizzes, results, and AI generation tracking. D1 is Cloudflare's serverless SQL database built on SQLite, providing global read replicas and seamless integration with Workers.

## Requirements

- Store user accounts with tier information
- Store quiz definitions (both AI-generated and uploaded)
- Track quiz results and user progress
- Monitor AI usage for billing
- Support public/private quiz visibility
- Enable quiz sharing and collaboration
- Maintain data integrity with foreign keys
- Support efficient querying and pagination

## Database Schema

### Core Tables

```sql
-- Users table (extended from auth plan)
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  name TEXT NOT NULL,
  tier TEXT DEFAULT 'free' CHECK(tier IN ('free', 'premium', 'enterprise')),
  email_verified BOOLEAN DEFAULT false,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  last_login DATETIME,
  ai_credits INTEGER DEFAULT 0,
  stripe_customer_id TEXT,
  stripe_subscription_id TEXT,
  preferences JSON DEFAULT '{}',
  avatar_url TEXT
);

-- Quizzes table
CREATE TABLE quizzes (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  title TEXT NOT NULL,
  description TEXT,
  content JSON NOT NULL, -- Full quiz JSON structure
  thumbnail_url TEXT,
  is_public BOOLEAN DEFAULT false,
  is_published BOOLEAN DEFAULT false,
  source TEXT NOT NULL CHECK(source IN ('ai', 'upload', 'paste', 'template')),
  ai_generation_id INTEGER,
  question_count INTEGER GENERATED ALWAYS AS (json_array_length(json_extract(content, '$.subjects[0].questions'))) STORED,
  difficulty TEXT CHECK(difficulty IN ('beginner', 'intermediate', 'advanced')),
  tags JSON DEFAULT '[]',
  views INTEGER DEFAULT 0,
  likes INTEGER DEFAULT 0,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (ai_generation_id) REFERENCES ai_generations(id)
);

-- Quiz results/attempts
CREATE TABLE results (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER,
  quiz_id INTEGER NOT NULL,
  score INTEGER NOT NULL,
  total_questions INTEGER NOT NULL,
  percentage REAL GENERATED ALWAYS AS (CAST(score AS REAL) / total_questions * 100) STORED,
  answers JSON NOT NULL, -- {questionId: selectedAnswer, ...}
  time_taken INTEGER, -- seconds
  ip_address TEXT,
  user_agent TEXT,
  started_at DATETIME NOT NULL,
  completed_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL,
  FOREIGN KEY (quiz_id) REFERENCES quizzes(id) ON DELETE CASCADE
);

-- AI Generation tracking
CREATE TABLE ai_generations (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  conversation JSON NOT NULL, -- Full chat history
  final_prompt TEXT NOT NULL,
  model TEXT NOT NULL,
  tokens_used INTEGER NOT NULL,
  cost_cents INTEGER, -- Track cost in cents
  status TEXT CHECK(status IN ('pending', 'completed', 'failed')),
  error_message TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  completed_at DATETIME,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Quiz sharing
CREATE TABLE quiz_shares (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  quiz_id INTEGER NOT NULL,
  shared_by INTEGER NOT NULL,
  share_code TEXT UNIQUE NOT NULL,
  access_level TEXT DEFAULT 'view' CHECK(access_level IN ('view', 'copy', 'edit')),
  expires_at DATETIME,
  max_uses INTEGER,
  use_count INTEGER DEFAULT 0,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (quiz_id) REFERENCES quizzes(id) ON DELETE CASCADE,
  FOREIGN KEY (shared_by) REFERENCES users(id) ON DELETE CASCADE
);

-- User favorites
CREATE TABLE favorites (
  user_id INTEGER NOT NULL,
  quiz_id INTEGER NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (user_id, quiz_id),
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (quiz_id) REFERENCES quizzes(id) ON DELETE CASCADE
);

-- Analytics events
CREATE TABLE analytics_events (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER,
  event_type TEXT NOT NULL,
  event_data JSON,
  ip_address TEXT,
  user_agent TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL
);
```

### Indexes for Performance

```sql
-- User queries
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_tier ON users(tier);
CREATE INDEX idx_users_created_at ON users(created_at);

-- Quiz queries
CREATE INDEX idx_quizzes_user_id ON quizzes(user_id);
CREATE INDEX idx_quizzes_public ON quizzes(is_public, is_published);
CREATE INDEX idx_quizzes_created_at ON quizzes(created_at);
CREATE INDEX idx_quizzes_source ON quizzes(source);

-- Results queries
CREATE INDEX idx_results_user_id ON results(user_id);
CREATE INDEX idx_results_quiz_id ON results(quiz_id);
CREATE INDEX idx_results_completed_at ON results(completed_at);

-- AI usage queries
CREATE INDEX idx_ai_generations_user_id ON ai_generations(user_id);
CREATE INDEX idx_ai_generations_created_at ON ai_generations(created_at);

-- Share code lookup
CREATE INDEX idx_quiz_shares_code ON quiz_shares(share_code);
```

## Implementation Tasks

- [ ] Set up D1 database in Cloudflare dashboard
- [ ] Create migration system for schema updates
- [ ] Write initial schema migration
- [ ] Create database connection utilities
- [ ] Implement query builder helpers
- [ ] Build user CRUD operations
- [ ] Build quiz CRUD operations
- [ ] Implement result tracking
- [ ] Create AI usage tracking
- [ ] Add quiz sharing functionality
- [ ] Implement analytics tracking
- [ ] Create backup strategy
- [ ] Add data validation layer
- [ ] Implement pagination utilities
- [ ] Test database performance
- [ ] Document query patterns

## API Integration

### Database Connection
```javascript
// db/connection.js
export class Database {
  constructor(env) {
    this.db = env.DB
  }
  
  async query(sql, params = []) {
    const stmt = this.db.prepare(sql)
    return params.length > 0 ? stmt.bind(...params) : stmt
  }
  
  async transaction(callback) {
    // D1 doesn't support transactions yet
    // Implement optimistic locking pattern
    return callback(this)
  }
}
```

### Query Patterns

#### Create Quiz
```javascript
async function createQuiz(userId, quizData) {
  const { title, description, content, source } = quizData
  
  const result = await db.query(`
    INSERT INTO quizzes (user_id, title, description, content, source)
    VALUES (?, ?, ?, ?, ?)
    RETURNING *
  `, [userId, title, description, JSON.stringify(content), source]).first()
  
  return result
}
```

#### Get User's Quizzes with Pagination
```javascript
async function getUserQuizzes(userId, page = 1, limit = 10) {
  const offset = (page - 1) * limit
  
  const quizzes = await db.query(`
    SELECT q.*, 
           COUNT(r.id) as attempt_count,
           AVG(r.percentage) as avg_score
    FROM quizzes q
    LEFT JOIN results r ON q.id = r.quiz_id
    WHERE q.user_id = ?
    GROUP BY q.id
    ORDER BY q.created_at DESC
    LIMIT ? OFFSET ?
  `, [userId, limit, offset]).all()
  
  const total = await db.query(
    'SELECT COUNT(*) as count FROM quizzes WHERE user_id = ?',
    [userId]
  ).first()
  
  return {
    quizzes,
    pagination: {
      page,
      limit,
      total: total.count,
      pages: Math.ceil(total.count / limit)
    }
  }
}
```

#### Track AI Usage
```javascript
async function trackAIUsage(userId, conversation, tokensUsed) {
  // Start tracking
  const generation = await db.query(`
    INSERT INTO ai_generations (user_id, conversation, tokens_used, status)
    VALUES (?, ?, ?, 'pending')
    RETURNING id
  `, [userId, JSON.stringify(conversation), tokensUsed]).first()
  
  // Update user credits
  await db.query(`
    UPDATE users 
    SET ai_credits = ai_credits - 1
    WHERE id = ? AND ai_credits > 0
  `, [userId]).run()
  
  return generation.id
}
```

## Migration System

```javascript
// migrations/001_initial_schema.js
export const up = async (db) => {
  await db.exec(`
    CREATE TABLE IF NOT EXISTS migrations (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT UNIQUE NOT NULL,
      applied_at DATETIME DEFAULT CURRENT_TIMESTAMP
    )
  `)
  
  // Create all tables
  await db.exec(INITIAL_SCHEMA)
}

export const down = async (db) => {
  // Drop all tables
  const tables = ['users', 'quizzes', 'results', ...]
  for (const table of tables) {
    await db.exec(`DROP TABLE IF EXISTS ${table}`)
  }
}
```

## Data Validation

```javascript
// validation/quiz.js
import { z } from 'zod'

export const QuizSchema = z.object({
  title: z.string().min(3).max(200),
  description: z.string().max(1000).optional(),
  subjects: z.array(z.object({
    name: z.string(),
    questions: z.array(z.object({
      id: z.number(),
      question: z.string(),
      options: z.array(z.string()).length(4),
      correctAnswer: z.number().min(0).max(3)
    }))
  }))
})

export function validateQuiz(data) {
  return QuizSchema.parse(data)
}
```

## Performance Considerations

1. **Query Optimization**
   - Use prepared statements
   - Implement query result caching
   - Batch operations where possible
   - Use appropriate indexes

2. **Data Limits**
   - Quiz content: 1MB max (JSON)
   - Results per query: 100 max
   - Batch inserts: 1000 records max

3. **Caching Strategy**
   - Cache public quizzes in KV
   - Cache user sessions
   - Cache aggregated statistics

## Backup & Recovery

1. **Automated Backups**
   - Daily exports to R2 storage
   - Point-in-time recovery
   - Cross-region replication

2. **Data Export**
   ```javascript
   // Export user data for GDPR
   async function exportUserData(userId) {
     const data = {
       user: await db.query('SELECT * FROM users WHERE id = ?', [userId]).first(),
       quizzes: await db.query('SELECT * FROM quizzes WHERE user_id = ?', [userId]).all(),
       results: await db.query('SELECT * FROM results WHERE user_id = ?', [userId]).all()
     }
     return data
   }
   ```

## Monitoring

- Query performance metrics
- Database size tracking
- Error rate monitoring
- Usage patterns analysis
- Cost tracking

## Security

- SQL injection prevention via parameterized queries
- Input sanitization
- Row-level security for multi-tenancy
- Encrypted sensitive data
- Audit logging for compliance

## Future Enhancements

- Full-text search on quiz content
- Real-time collaboration
- Version history for quizzes
- Advanced analytics
- Data warehousing for insights
- GraphQL API layer

## Documentation Plan

After implementation, create `dev/docs/2507121448-d1-database.md` with:
- Database schema documentation
- Query optimization guide
- Migration instructions
- Backup procedures
- Performance tuning tips