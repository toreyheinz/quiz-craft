# Feature Plan: User Authentication System

**Status**: Planning
**Created**: 2025-07-12
**Author**: Claude
**Priority**: Phase 1 - Must be implemented first

## Overview

Implement a secure user authentication system using Cloudflare Workers, KV storage for sessions, and Vue.js frontend. This system will support both free and premium user tiers, forming the foundation for AI quiz generation and other premium features.

## Requirements

- User registration with email/password
- Secure login with JWT tokens
- Password reset functionality
- User profile management
- Free vs Premium tier distinction
- Session management with Cloudflare KV
- CORS support for SPA
- Email verification (optional for MVP)

## Technical Approach

### Architecture Decisions

1. **JWT Tokens**: Use for stateless authentication
2. **Cloudflare KV**: Store refresh tokens and sessions
3. **D1 Database**: Store user accounts and profiles
4. **bcrypt**: Password hashing (via Web Crypto API)
5. **Pinia**: Frontend state management for auth
6. **Vue Router Guards**: Protect premium routes

### Security Considerations

- JWT with short expiration (15 minutes)
- Refresh tokens in httpOnly cookies
- Rate limiting on auth endpoints
- Input validation and sanitization
- CSRF protection
- Secure password requirements

## Implementation Tasks

- [ ] Set up Cloudflare Workers project with TypeScript
- [ ] Create D1 database and users table
- [ ] Implement password hashing utilities
- [ ] Create JWT generation and validation
- [ ] Build registration endpoint with validation
- [ ] Build login endpoint with rate limiting
- [ ] Implement refresh token mechanism
- [ ] Create logout endpoint
- [ ] Build password reset flow
- [ ] Create Vue auth store with Pinia
- [ ] Build Login/Register components
- [ ] Add route guards for protected pages
- [ ] Create user profile page
- [ ] Add auth interceptor to API client
- [ ] Test security vulnerabilities
- [ ] Document API endpoints

## API Endpoints

### Public Endpoints
```
POST /api/auth/register
  Body: { email, password, name }
  Response: { user, token, refreshToken }

POST /api/auth/login
  Body: { email, password }
  Response: { user, token, refreshToken }

POST /api/auth/refresh
  Body: { refreshToken }
  Response: { token, refreshToken }

POST /api/auth/logout
  Headers: Authorization: Bearer <token>
  Response: { success }

POST /api/auth/forgot-password
  Body: { email }
  Response: { message }

POST /api/auth/reset-password
  Body: { token, newPassword }
  Response: { success }
```

### Protected Endpoints
```
GET /api/auth/me
  Headers: Authorization: Bearer <token>
  Response: { user }

PUT /api/auth/profile
  Headers: Authorization: Bearer <token>
  Body: { name, preferences }
  Response: { user }

POST /api/auth/upgrade
  Headers: Authorization: Bearer <token>
  Response: { paymentUrl }
```

## Database Schema

```sql
-- Users table
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  name TEXT NOT NULL,
  tier TEXT DEFAULT 'free' CHECK(tier IN ('free', 'premium')),
  email_verified BOOLEAN DEFAULT false,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  last_login DATETIME,
  ai_credits INTEGER DEFAULT 0,
  stripe_customer_id TEXT,
  stripe_subscription_id TEXT
);

-- Create indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_tier ON users(tier);

-- Password reset tokens
CREATE TABLE password_resets (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  token TEXT UNIQUE NOT NULL,
  expires_at DATETIME NOT NULL,
  used BOOLEAN DEFAULT false,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);
```

## Frontend Implementation

### Auth Store (Pinia)
```javascript
// stores/auth.js
export const useAuthStore = defineStore('auth', {
  state: () => ({
    user: null,
    token: null,
    isAuthenticated: false,
    isPremium: false
  }),
  
  actions: {
    async login(credentials) {
      const response = await api.post('/auth/login', credentials)
      this.setAuth(response.data)
    },
    
    async register(userData) {
      const response = await api.post('/auth/register', userData)
      this.setAuth(response.data)
    },
    
    setAuth({ user, token }) {
      this.user = user
      this.token = token
      this.isAuthenticated = true
      this.isPremium = user.tier === 'premium'
      
      // Store token
      localStorage.setItem('token', token)
      
      // Set axios header
      api.defaults.headers.common['Authorization'] = `Bearer ${token}`
    },
    
    logout() {
      this.user = null
      this.token = null
      this.isAuthenticated = false
      this.isPremium = false
      
      localStorage.removeItem('token')
      delete api.defaults.headers.common['Authorization']
      
      router.push('/login')
    }
  }
})
```

### Route Guards
```javascript
// router/index.js
router.beforeEach((to, from, next) => {
  const authStore = useAuthStore()
  
  if (to.meta.requiresAuth && !authStore.isAuthenticated) {
    next('/login')
  } else if (to.meta.requiresPremium && !authStore.isPremium) {
    next('/premium')
  } else {
    next()
  }
})
```

## Worker Implementation

### JWT Utilities
```javascript
// utils/jwt.js
import { SignJWT, jwtVerify } from 'jose'

const secret = new TextEncoder().encode(env.JWT_SECRET)

export async function createToken(payload) {
  return new SignJWT(payload)
    .setProtectedHeader({ alg: 'HS256' })
    .setExpirationTime('15m')
    .setIssuedAt()
    .sign(secret)
}

export async function verifyToken(token) {
  const { payload } = await jwtVerify(token, secret)
  return payload
}
```

### Password Hashing
```javascript
// utils/crypto.js
export async function hashPassword(password) {
  const encoder = new TextEncoder()
  const data = encoder.encode(password + env.SALT)
  const hash = await crypto.subtle.digest('SHA-256', data)
  return btoa(String.fromCharCode(...new Uint8Array(hash)))
}

export async function verifyPassword(password, hash) {
  const testHash = await hashPassword(password)
  return testHash === hash
}
```

### Middleware
```javascript
// middleware/auth.js
export async function requireAuth(request, env) {
  const token = request.headers.get('Authorization')?.split(' ')[1]
  
  if (!token) {
    return Response.json({ error: 'No token provided' }, { status: 401 })
  }
  
  try {
    const payload = await verifyToken(token)
    const user = await env.DB.prepare(
      'SELECT * FROM users WHERE id = ?'
    ).bind(payload.userId).first()
    
    if (!user) {
      return Response.json({ error: 'User not found' }, { status: 401 })
    }
    
    request.user = user
    return null // Continue
  } catch (error) {
    return Response.json({ error: 'Invalid token' }, { status: 401 })
  }
}

export async function requirePremium(request) {
  if (request.user.tier !== 'premium') {
    return Response.json({ 
      error: 'Premium subscription required',
      upgrade_url: '/premium'
    }, { status: 403 })
  }
  return null // Continue
}
```

## Testing Strategy

### Unit Tests
- Password hashing and verification
- JWT generation and validation
- Input validation functions
- Rate limiting logic

### Integration Tests
- Full registration flow
- Login with valid/invalid credentials
- Token refresh mechanism
- Password reset flow
- Premium upgrade flow

### Security Tests
- SQL injection attempts
- XSS in user inputs
- CSRF protection
- Rate limiting effectiveness
- Token expiration

## Error Handling

- Registration errors: Email already exists, weak password
- Login errors: Invalid credentials, account locked
- Token errors: Expired, invalid, missing
- Database errors: Connection issues, constraints
- Rate limit errors: Too many attempts

## Monitoring

- Track registration conversion rate
- Monitor failed login attempts
- Alert on suspicious activity
- Track token refresh patterns
- Monitor API response times

## Future Enhancements

- Social login (Google, GitHub)
- Two-factor authentication
- Email verification
- Account deletion (GDPR)
- Login history
- Device management
- SSO for enterprise

## Documentation Plan

After implementation, create `dev/docs/2507121445-user-authentication.md` with:
- API endpoint documentation
- Frontend integration guide
- Security best practices
- Troubleshooting guide
- Migration instructions