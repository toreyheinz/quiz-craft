# Feature Plan: Cloudflare AI Workers Quiz Generation (Premium Feature)

**Status**: Planning
**Created**: 2025-07-12
**Author**: Claude
**Dependencies**: User Authentication, Cloudflare D1 Database

## Overview

Implement a Cloudflare AI Workers-based system for interactive quiz development as the PRIMARY method for quiz creation. This feature will be exclusive to premium users, with AI generation becoming the main value proposition for paid subscriptions. Free users will retain access to manual upload/paste functionality.

## Requirements
- Interactive chat interface for quiz refinement (premium only)
- Integration with Cloudflare AI Workers API with usage tracking
- Real-time JSON validation
- Credit-based system for premium users
- User authentication checks before AI access
- Store generated quizzes in D1 database
- Track AI usage for billing and analytics
- Clear upgrade prompts for free users

## Technical Approach

### Architecture Decisions
- Use Cloudflare Workers AI for LLM processing to leverage edge computing
- Implement chat interface in Vue.js as primary quiz creation method
- Store chat history in component state (no persistence needed initially)
- Validate JSON structure before allowing import
- Use streaming responses for better UX during generation
- Implement credit system to track and limit AI usage
- Store all generated quizzes in D1 database

### Premium User Flow
1. User clicks "Create Quiz with AI" (primary CTA)
2. System checks authentication and tier:
   - Premium users → Open chat interface
   - Free users → Show upgrade page with benefits
3. Premium users interact with AI to refine quiz
4. Generated quiz is validated and saved to D1
5. User can edit, share, or use quiz immediately

### API Design
The Cloudflare Worker will expose authenticated endpoints:
- `POST /api/chat` - Send message and get AI response (requires premium)
- `POST /api/generate-quiz` - Generate final quiz JSON from conversation (requires premium)
- `POST /api/validate-quiz` - Validate quiz JSON structure (all users)
- `GET /api/usage` - Get user's AI credit usage (premium users)
- `POST /api/save-quiz` - Save generated quiz to D1 (all users)

## Implementation Tasks
- [ ] Wait for auth system and D1 database implementation
- [ ] Research Cloudflare Workers AI models and pricing
- [ ] Add authentication middleware to Worker endpoints
- [ ] Implement premium tier checks in API
- [ ] Create AI chat endpoint with auth validation
- [ ] Implement credit tracking in D1
- [ ] Build JSON validation logic for quiz structure
- [ ] Create Vue.js chat interface component (premium UI)
- [ ] Create upgrade prompt component for free users
- [ ] Update Home.vue with AI as primary creation method
- [ ] Add usage dashboard for premium users
- [ ] Implement streaming response handling
- [ ] Store generated quizzes in D1
- [ ] Test premium vs free user flows
- [ ] Add comprehensive error handling
- [ ] Update documentation with premium features

## Files to Modify
- `assessment-app/src/views/Home.vue` - Add new tab for AI chat interface
- `assessment-app/src/components/QuizChat.vue` - New chat component (to create)
- `cloudflare-worker/src/index.js` - Main worker file (to create)
- `cloudflare-worker/src/prompts.js` - Prompt templates (to create)
- `cloudflare-worker/src/validation.js` - JSON validation logic (to create)
- `cloudflare-worker/wrangler.toml` - Worker configuration (to create)
- `assessment-app/src/services/api.js` - API client for worker (to create)

## Testing Strategy

### Unit Tests
- Test JSON validation logic with various quiz structures
- Test prompt generation with different parameters
- Mock Cloudflare AI responses for chat tests

### Integration Tests
- Test full flow from chat to quiz generation
- Verify generated quizzes load correctly in Assessment.vue
- Test error handling for malformed AI responses

## Edge Cases & Error Handling
- AI generates invalid JSON: Show error and allow retry
- AI response timeout: Implement retry with exponential backoff
- Rate limiting: Show user-friendly message with wait time
- Incomplete quiz structure: Validate and prompt for missing fields
- Network errors: Fallback to manual JSON entry

## Credit System & Cost Management

### Premium User Credits
- Monthly credit allocation based on subscription tier
- 1 credit = 1 quiz generation (regardless of refinements)
- Track token usage for cost analysis
- Show remaining credits in UI
- Option to purchase additional credits

### Cost Tracking
```sql
-- Track usage in ai_generations table
INSERT INTO ai_generations (user_id, prompt, tokens_used, created_at)
VALUES (?, ?, ?, CURRENT_TIMESTAMP);
```

### Rate Limiting
- 10 chat messages per minute per user
- 5 quiz generations per hour per user
- 100 total AI calls per day per premium user

## Open Questions
- [ ] Which Cloudflare AI model should we use? (@cf/meta/llama-3-8b-instruct seems suitable)
- [ ] Should chat history persist between sessions?
- [ ] Monthly credit allocation for premium users?
- [ ] Cost per additional credit package?
- [ ] Should we offer different premium tiers?

## Future Considerations
- Add quiz templates library
- Implement quiz sharing functionality
- Add AI-powered quiz difficulty adjustment
- Support for different question types beyond multiple choice
- Analytics on quiz generation patterns

## Technical Implementation Details

### Cloudflare Workers AI Integration with Auth
```javascript
// Example worker code with premium checks
export default {
  async fetch(request, env) {
    const ai = env.AI;
    const db = env.DB;
    
    // Chat endpoint with auth
    if (url.pathname === '/api/chat') {
      // Verify JWT and get user
      const user = await verifyAuth(request, env);
      if (!user || user.tier !== 'premium') {
        return Response.json({ 
          error: 'Premium subscription required',
          upgrade_url: '/premium'
        }, { status: 403 });
      }
      
      // Check credits
      const credits = await getUserCredits(user.id, db);
      if (credits <= 0) {
        return Response.json({ 
          error: 'No credits remaining',
          purchase_url: '/credits'
        }, { status: 402 });
      }
      
      // Process AI request
      const messages = await request.json();
      const response = await ai.run('@cf/meta/llama-3-8b-instruct', {
        messages: [
          { role: 'system', content: systemPrompt },
          ...messages
        ]
      });
      
      // Track usage
      await trackUsage(user.id, response.usage, db);
      
      return Response.json({ response });
    }
  }
}
```

### Vue.js Chat Component Structure
```vue
<template>
  <div class="quiz-chat">
    <div class="chat-messages">
      <!-- Message history -->
    </div>
    <div class="chat-input">
      <!-- User input -->
    </div>
    <button @click="generateQuiz">Generate Quiz</button>
  </div>
</template>
```

### JSON Validation Schema
The system must validate:
- Required fields: title, description, subjects
- Each subject has name and questions array
- Each question has: id (unique number), question, options (exactly 4), correctAnswer (0-3)
- No duplicate question IDs
- Valid JSON syntax

## Documentation Plan
After implementation, create `dev/docs/2507121434-cloudflare-ai-quiz-generation.md` with:
- Feature overview and purpose
- API endpoint documentation
- Configuration for Cloudflare Workers
- Usage guide for the chat interface
- Troubleshooting common issues
- Example conversations and generated quizzes