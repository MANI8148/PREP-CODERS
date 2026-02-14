# Syntax Saga Platform - Complete Design Document

## Table of Contents
1. System Architecture
2. Core Components
3. AI Integration (The Adaptive Master)
4. UI/UX Design Specifications
5. Database Schema
6. API Endpoints
7. Technology Stack
8. Security Considerations
9. Performance Optimization
10. Testing Strategy
11. Deployment Pipeline
12. Implementation Examples

---

## 1. System Architecture

### 1.1 High-Level Architecture
```
┌─────────────┐      ┌──────────────┐      ┌─────────────┐
│   Frontend  │ ───> │   Backend    │ ───> │  Code Runner│
│  (React/Vue)│ <─── │  (Node/Flask)│ <─── │  (Judge0)   │
└─────────────┘      └──────────────┘      └─────────────┘
                            │
                            ├──> ┌─────────────┐
                            │    │  AI Service │
                            │    │ (OpenAI API)│
                            │    └─────────────┘
                            │
                            └──> ┌─────────────┐
                                 │  Database   │
                                 │ (PostgreSQL)│
                                 └─────────────┘
```

### 1.2 Component Interaction Flow
1. Student submits code through frontend
2. Backend validates and sanitizes input
3. Code sent to execution engine (Judge0/Piston)
4. Results captured and sent to AI for evaluation
5. AI generates tier-appropriate narrative feedback
6. Response formatted and returned to frontend
7. Frontend displays results with animations

### 1.3 Technology Stack Overview
- **Frontend**: React + TypeScript, Monaco Editor, Canvas API
- **Backend**: Node.js (Express) or Python (FastAPI)
- **Code Execution**: Judge0 API or Piston API
- **AI**: OpenAI GPT-4, Anthropic Claude, Google Gemini
- **Database**: PostgreSQL with Redis caching
- **Real-time**: WebSocket for collaboration
- **Hosting**: Vercel (frontend), Railway/Render (backend)

---

## 2. Core Components

### 2.1 Code Evaluation Pipeline

#### 2.1.1 Evaluation Flow
1. Player submits code
2. Backend validates and sanitizes input
3. Code sent to execution engine with unit tests
4. Results captured (pass/fail, execution time, memory usage)
5. Results + code sent to AI for narrative evaluation
6. AI response formatted and returned to player

#### 2.1.2 Data Structure
```json
{
  "submission": {
    "code": "string",
    "language": "python|javascript|java",
    "challengeId": "uuid",
    "attemptNumber": 1
  },
  "testResults": {
    "passed": 2,
    "failed": 1,
    "errors": ["Expected 5, got -1"],
    "executionTime": "0.023s",
    "memoryUsed": "2.1MB"
  }
}
```


### 2.2 Code Execution Engine Integration

#### Judge0 API Integration
```javascript
async function executeCode(code, language, challengeId) {
  const challenge = await getChallenge(challengeId);
  
  // Prepare test code
  const testCode = generateTestCode(code, challenge.unitTests, language);
  
  // Call Judge0 API
  const response = await fetch('https://judge0-ce.p.rapidapi.com/submissions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'X-RapidAPI-Key': process.env.JUDGE0_API_KEY
    },
    body: JSON.stringify({
      source_code: testCode,
      language_id: getLanguageId(language),
      stdin: '',
      expected_output: challenge.expectedOutput
    })
  });
  
  const submission = await response.json();
  const result = await pollSubmissionResult(submission.token);
  
  return {
    syntaxValid: result.status.id !== 6,
    testResults: parseTestResults(result),
    challenge
  };
}
```

#### Complexity Analysis
```python
def analyze_complexity(code, language):
    analysis = {
        "timeComplexity": "O(n)",
        "spaceComplexity": "O(1)",
        "isOptimal": True,
        "codeSmells": []
    }
    
    # Detect nested loops (O(n²) indicator)
    if has_nested_loops(code):
        analysis["timeComplexity"] = "O(n²)"
        analysis["isOptimal"] = False
        analysis["codeSmells"].append("Nested loops detected")
    
    # Detect unused variables
    if has_unused_variables(code):
        analysis["codeSmells"].append("Unused variables")
    
    return analysis
```

---

## 3. AI Integration (The Adaptive Master)

### 3.1 Tier Detection System
```javascript
const getUserTier = (userProfile) => {
  return userProfile.educationLevel === 'school' ? 'apprentice' : 'grandmaster';
};

const getPersona = (tier) => {
  return tier === 'apprentice' 
    ? 'The Friendly Wizard' 
    : 'The Grumpy Senior Architect';
};
```

### 3.2 System Prompt Structure (Tier-Adaptive)
```
Role: You are {persona} in the Kingdom of Syntax.

Tier: {apprentice|grandmaster}

Vocabulary Rules:
- Apprentice: Use visual metaphors (lists = "train cars", functions = "magic spells")
- Grandmaster: Use technical terms (dynamic arrays, time complexity, code smells)

Tone:
- Apprentice: Encouraging, patient, visual ("You're getting closer!")
- Grandmaster: Critical but constructive, optimization-focused ("This works, but...")

Evaluation Criteria:
- Syntax Pass: Does it run?
- Logic Pass: Does it solve the quest?
- Elite Pass (Grandmaster only): Is it optimal?
```

### 3.3 Context Passed to AI
```json
{
  "systemPrompt": "Adaptive Master role and rules",
  "userTier": "apprentice|grandmaster",
  "persona": "Friendly Wizard|Grumpy Senior Architect",
  "userCode": "submitted code",
  "testResults": "execution results",
  "attemptCount": 2,
  "challengeDescription": "task description",
  "optimalSolution": "reference solution (for comparison only)",
  "vocabularyMode": "visual|technical"
}
```

### 3.4 Response Format (Tier-Adaptive)
```json
{
  "narrative": "Epic one-liner (tier-appropriate vocabulary)",
  "result": "Critical Hit|Glancing Blow|Miss",
  "syntaxPass": true,
  "logicPass": false,
  "elitePass": null,
  "insight": "Feedback with RPG metaphors (adapted to tier)",
  "questLog": "Next step hint (visual for apprentice, technical for grandmaster)",
  "xpAwarded": 100,
  "complexityWarning": "Apprentice: 'Your spell is slow!' | Grandmaster: 'O(n²) detected'",
  "bossHealthRemaining": 10,
  "visualFeedback": {
    "spriteAction": "move(10)",
    "animationType": "success|partial|failure"
  }
}
```

### 3.5 AI API Integration Example
```javascript
async function getAIFeedback(context) {
  const { userTier, userCode, testResults, attemptCount } = context;
  
  const userMessage = `
USER TIER: ${userTier}
ATTEMPT COUNT: ${attemptCount}

CHALLENGE: ${context.challengeDescription}

SUBMITTED CODE:
\`\`\`
${userCode}
\`\`\`

TEST RESULTS:
- Syntax Valid: ${testResults.syntaxValid}
- Tests Passed: ${testResults.passed}/${testResults.total}
- Tests Failed: ${testResults.failed}
${testResults.errors.length > 0 ? `- Errors: ${testResults.errors.join(', ')}` : ''}

Please evaluate this submission and provide feedback.
  `.trim();

  const response = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: [
      { role: 'system', content: ADAPTIVE_MASTER_PROMPT },
      { role: 'user', content: userMessage }
    ],
    temperature: 0.7,
    max_tokens: 1000
  });

  return parseAIResponse(response.choices[0].message.content);
}
```

### 3.6 Progressive Hint System

#### Hint Levels
- **Attempt 1**: Conceptual hint (e.g., "Consider the Modulo incantation")
- **Attempt 2**: Structural hint (e.g., "Use a list comprehension with if condition")
- **Attempt 3**: Specific hint (e.g., "Try: [x for x in list if x % 2 == 0]")
- **Attempt 4+**: Reveal solution with explanation

#### Implementation
```python
def get_hint_level(attempt_count):
    if attempt_count == 1:
        return "conceptual"
    elif attempt_count == 2:
        return "structural"
    elif attempt_count == 3:
        return "specific"
    else:
        return "solution"
```

### 3.7 The Judge: Multi-Level Decision Checker

```python
def evaluate_submission(code, tests, user_tier):
    result = {
        "syntaxPass": False,
        "logicPass": False,
        "elitePass": None
    }
    
    # Pass 1: Syntax Check
    try:
        execution_result = execute_code(code)
        result["syntaxPass"] = True
    except SyntaxError as e:
        return generate_riddle(e, user_tier)
    
    # Pass 2: Logic Check
    test_results = run_tests(code, tests)
    if test_results.all_passed:
        result["logicPass"] = True
    else:
        return explain_trap(test_results.failures, user_tier)
    
    # Pass 3: Elite Check (Grandmaster only)
    if user_tier == "grandmaster":
        complexity = analyze_complexity(code)
        style_score = analyze_style(code)
        
        if complexity.is_optimal and style_score > 80:
            result["elitePass"] = True
            return "Critical Hit - Boss defeated!"
        else:
            result["elitePass"] = False
            return "Boss survives with 10% HP - optimize to finish!"
    
    return "Critical Hit - Quest complete!"
```

### 3.8 Tier-Specific Feedback

```python
def generate_feedback(result, user_tier):
    if user_tier == "apprentice":
        return {
            "syntaxFail": "Your spell fizzled! Check line {line} for a missing ingredient.",
            "logicFail": "The door is still locked! Your key doesn't quite fit.",
            "success": "Victory! The monster is defeated! 🎉"
        }
    else:  # grandmaster
        return {
            "syntaxFail": "Compilation error on line {line}: {error_type}",
            "logicFail": "Test case {n} failed: Expected {expected}, got {actual}",
            "suboptimal": "O(n²) complexity detected. Refactor using hash map for O(n).",
            "success": "Optimal solution. Code review: {detailed_analysis}"
        }
```

---

## 4. UI/UX Design Specifications

### 4.1 Design Guidelines

#### Color Schemes
**Apprentice Tier:**
- Primary: #4A90E2 (Blue)
- Secondary: #7ED321 (Green)
- Accent: #F5A623 (Orange)
- Background: #F8F9FA (Light Gray)

**Grandmaster Tier:**
- Primary: #1E1E1E (Dark Gray)
- Secondary: #00D9FF (Cyan)
- Accent: #FF6B35 (Orange)
- Text: #E0E0E0 (Light Gray)

#### Typography
**Apprentice:**
- Headings: Nunito Bold
- Body: Quicksand Regular
- Code: Comic Mono

**Grandmaster:**
- Headings: Inter Bold
- Body: Inter Regular
- Code: Fira Code

#### Layout Principles
- Apprentice: Large buttons, spacious layout, visual hierarchy
- Grandmaster: Dense information, efficient use of space, multiple panels

### 4.2 Apprentice Tier: "The Visual Quest"

#### Block-to-Text Bridge
```
┌─────────────────────────────────────┐
│  [Drag Block]  →  [Text Preview]    │
│                                     │
│  ┌──────────────┐   def move():    │
│  │ Move Forward │ →   character.x  │
│  │   10 steps   │     += 10        │
│  └──────────────┘                   │
└─────────────────────────────────────┘
```

Features:
- Scratch-style blocks on left panel
- Real-time code generation on right
- Smooth transition animations
- Color-coded block categories (loops, conditions, functions)

#### Animated Sprite System
```javascript
if (code.includes('character.move(10)')) {
  animateSprite({
    action: 'walk',
    distance: 10,
    duration: 500,
    onComplete: showSuccessParticles
  });
}
```

Visual Feedback:
- Character sprite performs actions in real-time
- Success: Green particles, cheerful sound
- Partial: Yellow glow, encouraging message
- Failure: Red shake, gentle error indicator

#### Quest Log (Visual)
```
╔════════════════════════════════════╗
║  🎯 Current Quest                  ║
║  ─────────────────────────────     ║
║  Make the hero collect 5 coins!    ║
║                                    ║
║  💡 Hint: Use a loop spell         ║
║  🎨 [Illustrated diagram]          ║
╚════════════════════════════════════╝
```

### 4.3 Grandmaster Tier: "The Technical Dungeon"

#### The Linter Ghost
```
┌─────────────────────────────────────┐
│  def calculate(x, y):               │
│      result = x + y  👻 "Unused"   │
│      return x * y                   │
│                                     │
│  👻 Code Smell: Dead code detected  │
└─────────────────────────────────────┘
```

Features:
- Real-time floating annotations
- Severity indicators (warning/error/info)
- Click to see detailed explanation
- Auto-fix suggestions

#### Collaboration Mode: "Party Mode"
```
┌──────────────────────────────────────┐
│  Player 1 (You)    │  Player 2       │
│  ─────────────────────────────────   │
│  def merge_sort(): │  # Reviewing... │
│    ...             │  💬 "Try O(n)"  │
│                    │                 │
│  🤝 Merge Conflict Detected!         │
│  ⚔️ Boss: Two-Headed Giant           │
└──────────────────────────────────────┘
```

Features:
- Split-screen code editor
- Real-time cursor tracking
- Chat integration
- Merge conflict visualization
- Shared XP rewards

#### Quest Log (Technical)
```
╔════════════════════════════════════╗
║  ⚙️ Challenge: Optimize Search     ║
║  ─────────────────────────────     ║
║  Current: O(n²) - 2.3s             ║
║  Target:  O(n)  - <0.5s            ║
║                                    ║
║  📊 Performance Metrics             ║
║  Memory: 45MB / 100MB              ║
║  Boss HP: ████████░░ 80%           ║
╚════════════════════════════════════╝
```

### 4.4 Shared UI Components

#### AI Chat Panel
```
┌─────────────────────────────────────┐
│  🤖 AI Assistant                    │
│  ─────────────────────────────────  │
│  You: How do I create a loop?       │
│                                     │
│  AI: [Tier-appropriate response]    │
│  [Code example]                     │
│                                     │
│  Quick Actions:                     │
│  [Explain] [Debug] [Hint] [Example] │
│                                     │
│  [Type your question...]      [Send]│
└─────────────────────────────────────┘
```

#### AI Hint System
```
┌─────────────────────────────────────┐
│                    💡 AI Hint System                    │
├─────────────────────────────────────────────────────────┤
│  Challenge: Create Even Number Filter                   │
│  Attempt: 2 of 3 before solution reveal                │
│                                                         │
│  Hint Difficulty:  ●●○○○                               │
│  [Conceptual] ←────────→ [Specific]                    │
│                                                         │
│  Current Hint (Level 2):                                │
│  "Think about the Modulo spell (%). It helps you       │
│   find remainders! When you divide a number by 2,      │
│   even numbers leave no remainder."                     │
│                                                         │
│  [Show Code Example]  [Ask AI a Question]              │
└─────────────────────────────────────────────────────────┘
```

### 4.5 Animation & Transitions

#### Apprentice Tier Animations
- Sprite movements (walk, jump, celebrate)
- Particle effects (sparkles, confetti)
- Smooth block transitions
- Quest completion celebrations

#### Grandmaster Tier Animations
- Subtle code highlighting
- Performance graph animations
- Boss health bar depletion
- Achievement unlocks

#### Performance Considerations
- 60 FPS target for all animations
- Reduced motion for accessibility
- GPU-accelerated transforms
- Lazy loading for heavy animations

### 4.6 Responsive Design

#### Desktop (1920x1080+)
- Full multi-panel layout
- Side-by-side code editor and output
- Persistent AI chat panel
- Expanded quest log

#### Laptop (1366x768)
- Collapsible side panels
- Tabbed interface for AI features
- Optimized code editor size

#### Tablet (768x1024)
- Single-column layout
- Bottom sheet for AI chat
- Simplified block editor for Apprentice tier
- Touch-optimized controls

#### Mobile (375x667)
- Vertical scrolling layout
- Full-screen code editor
- Floating action button for AI help
- Swipe gestures for navigation

---

## 5. Database Schema

### 5.1 Tables

#### users
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  username VARCHAR(255) UNIQUE NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  tier VARCHAR(20) NOT NULL CHECK (tier IN ('apprentice', 'grandmaster')),
  xp INTEGER DEFAULT 0,
  level INTEGER DEFAULT 1,
  preferences JSONB DEFAULT '{"visualMode": true, "soundEnabled": true}',
  created_at TIMESTAMP DEFAULT NOW()
);
```

#### challenges
```sql
CREATE TABLE challenges (
  id UUID PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  description TEXT NOT NULL,
  difficulty VARCHAR(20) CHECK (difficulty IN ('beginner', 'intermediate', 'advanced')),
  tier_specific VARCHAR(20) CHECK (tier_specific IN ('apprentice', 'grandmaster', 'both')),
  unit_tests JSONB NOT NULL,
  optimal_solution TEXT NOT NULL,
  language VARCHAR(50) NOT NULL,
  visual_assets JSONB DEFAULT '{"spriteActions": [], "animations": []}',
  collaboration_mode BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT NOW()
);
```

#### submissions
```sql
CREATE TABLE submissions (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  challenge_id UUID REFERENCES challenges(id) ON DELETE CASCADE,
  code TEXT NOT NULL,
  result VARCHAR(20) CHECK (result IN ('critical_hit', 'glancing_blow', 'miss')),
  syntax_pass BOOLEAN NOT NULL,
  logic_pass BOOLEAN NOT NULL,
  elite_pass BOOLEAN,
  attempt_number INTEGER NOT NULL,
  xp_awarded INTEGER DEFAULT 0,
  feedback JSONB NOT NULL,
  complexity_detected VARCHAR(50),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_submissions_user_id ON submissions(user_id);
CREATE INDEX idx_submissions_challenge_id ON submissions(challenge_id);
CREATE INDEX idx_submissions_created_at ON submissions(created_at);
```

---

## 6. API Endpoints

### 6.1 POST /api/submit
Submit code for evaluation

**Request:**
```json
{
  "challengeId": "uuid",
  "code": "string",
  "language": "python"
}
```

**Response:**
```json
{
  "narrative": "You strike the Loop Monster with perfect precision!",
  "result": "critical_hit",
  "syntaxPass": true,
  "logicPass": true,
  "elitePass": null,
  "insight": "Your loop spell worked perfectly! The monster is defeated.",
  "questLog": "Excellent work! Ready for the next challenge?",
  "xpAwarded": 100,
  "visualFeedback": {
    "spriteAction": "victory_dance",
    "animationType": "success"
  }
}
```

### 6.2 GET /api/challenges
Retrieve available challenges

**Query Parameters:**
- `tier`: apprentice|grandmaster|both
- `difficulty`: beginner|intermediate|advanced
- `language`: python|javascript|java

**Response:**
```json
{
  "challenges": [
    {
      "id": "uuid",
      "title": "Create a Loop Spell",
      "description": "Make the hero collect 5 coins using a loop",
      "difficulty": "beginner",
      "tier_specific": "apprentice",
      "language": "python",
      "estimatedTime": "10 minutes",
      "xpReward": 100
    }
  ]
}
```

### 6.3 GET /api/user/progress
Get user's XP, level, and completed challenges

**Response:**
```json
{
  "user": {
    "id": "uuid",
    "username": "student123",
    "tier": "apprentice",
    "xp": 450,
    "level": 5,
    "completedChallenges": 12,
    "strengths": ["loops", "conditionals"],
    "weaknesses": ["recursion"],
    "nextRecommendedChallenge": "uuid"
  }
}
```

---

## 7. Technology Stack (Production-Ready Full Stack)

### 7.1 Frontend Architecture

#### Core Framework & Language
- **React 18.2+** with TypeScript 5.0+
  - Strict mode enabled
  - Concurrent features for better UX
  - Server Components for improved performance
- **Next.js 14+** (App Router)
  - Server-side rendering (SSR)
  - Static site generation (SSG)
  - API routes for BFF pattern
  - Image optimization
  - Built-in SEO support

#### Code Editor
- **Monaco Editor 0.44+** (VS Code engine)
  - Syntax highlighting for Python, JavaScript, Java
  - IntelliSense and autocomplete
  - Custom themes (Apprentice: light, Grandmaster: dark)
  - Minimap and line numbers
- **CodeMirror 6** (fallback/alternative)
  - Lighter weight option
  - Better mobile support

#### UI Components & Styling
- **TailwindCSS 3.4+**
  - Utility-first CSS
  - Custom design system
  - Dark mode support
  - Responsive breakpoints
- **Shadcn/ui** (component library)
  - Accessible components (WCAG 2.1 AA)
  - Customizable with Tailwind
  - Radix UI primitives
- **Framer Motion 10+**
  - Sprite animations
  - Page transitions
  - Gesture animations
  - Spring physics

#### State Management
- **Zustand 4.4+** (global state)
  - Lightweight (1KB)
  - No boilerplate
  - DevTools support
- **React Query (TanStack Query) 5+** (server state)
  - Automatic caching
  - Background refetching
  - Optimistic updates
  - Infinite queries for challenge lists

#### Form Handling & Validation
- **React Hook Form 7.48+**
  - Performant form handling
  - Built-in validation
- **Zod 3.22+**
  - TypeScript-first schema validation
  - Runtime type checking

#### Graphics & Animations
- **Canvas API** (native)
  - Sprite rendering
  - Particle effects
- **PixiJS 7.3+** (alternative for complex animations)
  - WebGL renderer
  - Better performance for heavy animations
- **Lottie React** (JSON animations)
  - Achievement animations
  - Loading states

#### Real-Time Communication
- **Socket.io Client 4.6+**
  - WebSocket for collaboration mode
  - Automatic reconnection
  - Room-based communication

#### Voice & Audio
- **Web Speech API** (native)
  - Voice commands
  - Speech-to-text
- **Howler.js 2.2+**
  - Sound effects
  - Background music
  - Audio sprites

#### Testing
- **Vitest 1.0+** (unit tests)
  - Fast, Vite-powered
  - Jest-compatible API
- **React Testing Library 14+** (component tests)
  - User-centric testing
- **Playwright 1.40+** (E2E tests)
  - Cross-browser testing
  - Visual regression testing
  - Mobile emulation

#### Build & Development Tools
- **Vite 5.0+** (build tool)
  - Lightning-fast HMR
  - Optimized production builds
- **ESLint 8+** with TypeScript plugin
  - Code quality enforcement
- **Prettier 3+**
  - Code formatting
- **Husky 8+** (Git hooks)
  - Pre-commit linting
  - Pre-push testing

---

### 7.2 Backend Architecture

#### Core Framework & Language
- **Node.js 20 LTS** with TypeScript 5.0+
  - **Express.js 4.18+** (REST API)
    - Middleware-based architecture
    - Easy to extend
  - **Fastify 4.24+** (alternative - faster)
    - Schema-based validation
    - Better performance
- **Python 3.11+** (alternative stack)
  - **FastAPI 0.104+**
    - Async support
    - Automatic OpenAPI docs
    - Pydantic validation

#### API Layer
- **tRPC 10+** (type-safe API)
  - End-to-end type safety
  - No code generation
  - Works with Next.js
- **GraphQL** (alternative)
  - **Apollo Server 4+**
  - Flexible data fetching
  - Real-time subscriptions

#### Authentication & Authorization
- **NextAuth.js 4.24+** (primary)
  - OAuth providers (Google, GitHub)
  - JWT sessions
  - Database sessions
- **Clerk** (alternative - managed auth)
  - User management UI
  - Multi-factor authentication
  - Social login
- **Passport.js 0.7+** (for Express)
  - Multiple strategies
  - Session management

#### Database & ORM
- **PostgreSQL 16+** (primary database)
  - ACID compliance
  - JSON support
  - Full-text search
  - Row-level security
- **Prisma 5.6+** (ORM)
  - Type-safe database client
  - Automatic migrations
  - Database introspection
  - Prisma Studio for admin
- **Drizzle ORM** (alternative - lighter)
  - SQL-like syntax
  - Better performance

#### Caching Layer
- **Redis 7.2+** (in-memory cache)
  - Session storage
  - AI response caching
  - Rate limiting
  - Job queue
- **Upstash Redis** (serverless alternative)
  - Pay-per-request
  - Global edge caching

#### Vector Database (AI Embeddings)
- **Pinecone** (managed)
  - Semantic search
  - Plagiarism detection
  - Challenge recommendations
- **Weaviate 1.22+** (self-hosted alternative)
  - Open source
  - GraphQL API
- **pgvector** (PostgreSQL extension)
  - Keep everything in Postgres
  - Simpler architecture

#### Background Jobs & Queue
- **BullMQ 5.0+** (Node.js)
  - Redis-based queue
  - Job scheduling
  - Retry logic
  - Job prioritization
- **Celery 5.3+** (Python)
  - Distributed task queue
  - Periodic tasks
  - Task routing

#### Real-Time Communication
- **Socket.io 4.6+** (WebSocket server)
  - Room-based collaboration
  - Automatic reconnection
  - Binary support
- **Pusher** (managed alternative)
  - Scalable WebSocket
  - Presence channels

#### Code Execution
- **Judge0 CE API** (primary)
  - 60+ languages
  - Sandboxed execution
  - Resource limits
  - Batch submissions
- **Piston API** (fallback)
  - Open source
  - Self-hostable
  - 50+ languages
- **AWS Lambda** (custom solution)
  - Isolated execution
  - Auto-scaling
  - Pay-per-use

#### AI Integration
- **OpenAI SDK 4.20+**
  - GPT-4 Turbo (complex analysis)
  - GPT-3.5 Turbo (fast responses)
  - Whisper (voice recognition)
  - Text Embeddings (semantic search)
- **Anthropic SDK 0.8+**
  - Claude 3 Opus (fallback)
  - Claude 3 Sonnet (balanced)
  - Claude 3 Haiku (fast)
- **Google AI SDK**
  - Gemini Pro (fallback)
- **LangChain 0.1+**
  - AI orchestration
  - Prompt management
  - Chain of thought
  - Memory management
- **Vercel AI SDK 3.0+**
  - Streaming responses
  - Edge runtime support
  - Provider abstraction

#### API Documentation
- **Swagger/OpenAPI 3.0**
  - Auto-generated docs
  - Interactive API explorer
- **Postman Collections**
  - API testing
  - Team collaboration

#### Validation & Security
- **Zod 3.22+** (schema validation)
  - Runtime validation
  - Type inference
- **Helmet.js 7+** (security headers)
  - XSS protection
  - CSRF protection
- **express-rate-limit 7+**
  - Rate limiting
  - DDoS protection
- **bcrypt 5.1+** (password hashing)
  - Secure password storage

#### Testing
- **Jest 29+** (unit tests)
  - Mocking support
  - Coverage reports
- **Supertest 6+** (API tests)
  - HTTP assertions
- **Artillery 2.0+** (load testing)
  - Performance testing
  - Stress testing

---

### 7.3 DevOps & Infrastructure

#### Version Control
- **Git** with **GitHub**
  - Branch protection rules
  - Pull request templates
  - GitHub Actions for CI/CD

#### CI/CD Pipeline
- **GitHub Actions** (primary)
  - Automated testing
  - Build and deploy
  - Environment secrets
- **Vercel** (frontend deployment)
  - Automatic preview deployments
  - Edge network
  - Analytics
- **Railway** (backend deployment)
  - One-click deploy
  - Auto-scaling
  - Database hosting
- **Render** (alternative backend)
  - Free tier available
  - Auto-deploy from Git

#### Containerization
- **Docker 24+**
  - Development containers
  - Production images
- **Docker Compose**
  - Local development stack
  - Service orchestration

#### Container Orchestration (Production Scale)
- **Kubernetes 1.28+** (for scale)
  - Auto-scaling
  - Load balancing
  - Self-healing
- **AWS ECS/Fargate** (managed alternative)
  - Serverless containers
  - AWS integration

#### Infrastructure as Code
- **Terraform 1.6+**
  - Multi-cloud support
  - State management
- **Pulumi** (alternative)
  - TypeScript/Python IaC
  - Better developer experience

#### Cloud Providers

**Primary: AWS**
- **EC2** (compute)
- **RDS PostgreSQL** (managed database)
- **ElastiCache Redis** (managed cache)
- **S3** (file storage)
- **CloudFront** (CDN)
- **Lambda** (serverless functions)
- **API Gateway** (API management)
- **CloudWatch** (monitoring)
- **Secrets Manager** (secrets)

**Alternative: Google Cloud Platform**
- **Cloud Run** (containers)
- **Cloud SQL** (PostgreSQL)
- **Memorystore** (Redis)
- **Cloud Storage** (files)
- **Cloud CDN**

**Alternative: Vercel + Railway**
- **Vercel** (frontend + edge functions)
- **Railway** (backend + database)
- **Upstash** (Redis)
- **Cloudflare R2** (S3-compatible storage)

#### Monitoring & Observability
- **Sentry 7.80+** (error tracking)
  - Error monitoring
  - Performance monitoring
  - Release tracking
- **LogRocket** (session replay)
  - User session recording
  - Console logs
  - Network requests
- **Datadog** (APM)
  - Application performance
  - Infrastructure monitoring
  - Log aggregation
- **Grafana + Prometheus** (self-hosted alternative)
  - Custom dashboards
  - Alerting
- **Vercel Analytics** (frontend metrics)
  - Web vitals
  - User analytics

#### Logging
- **Winston 3.11+** (Node.js)
  - Structured logging
  - Multiple transports
- **Pino 8.16+** (faster alternative)
  - JSON logging
  - Low overhead
- **CloudWatch Logs** (AWS)
  - Centralized logging
  - Log insights

#### Security & Secrets Management
- **AWS Secrets Manager**
  - API key rotation
  - Encrypted storage
- **HashiCorp Vault** (self-hosted)
  - Dynamic secrets
  - Encryption as a service
- **Doppler** (managed alternative)
  - Secret syncing
  - Team collaboration

#### CDN & Edge
- **Cloudflare** (primary)
  - DDoS protection
  - WAF
  - Edge caching
  - Workers (edge compute)
- **AWS CloudFront** (alternative)
  - Global edge network
  - Lambda@Edge

#### Database Backups
- **AWS RDS Automated Backups**
  - Point-in-time recovery
  - Cross-region replication
- **pg_dump** (manual backups)
  - Full database dumps
  - Scheduled via cron

---

### 7.4 Development Tools

#### Code Editor
- **VS Code** with extensions:
  - ESLint
  - Prettier
  - TypeScript
  - Prisma
  - Tailwind CSS IntelliSense
  - GitLens

#### API Testing
- **Postman** (API development)
- **Insomnia** (alternative)
- **Thunder Client** (VS Code extension)

#### Database Management
- **Prisma Studio** (built-in)
- **pgAdmin 4** (PostgreSQL GUI)
- **TablePlus** (multi-database GUI)
- **DBeaver** (free alternative)

#### Design & Prototyping
- **Figma** (UI/UX design)
  - Component library
  - Design system
  - Prototyping
- **Excalidraw** (diagrams)
  - Architecture diagrams
  - Flowcharts

#### Documentation
- **Notion** (team docs)
- **Confluence** (enterprise)
- **Docusaurus** (technical docs)
  - Versioned docs
  - Search
  - MDX support

#### Project Management
- **Linear** (issue tracking)
  - Sprint planning
  - Roadmap
- **Jira** (enterprise alternative)
- **GitHub Projects** (simple alternative)

#### Communication
- **Slack** (team chat)
  - GitHub integration
  - Sentry alerts
- **Discord** (alternative)

---

### 7.5 Third-Party Services

#### Email
- **Resend** (transactional email)
  - React email templates
  - High deliverability
- **SendGrid** (alternative)
- **AWS SES** (cost-effective)

#### SMS/Notifications
- **Twilio** (SMS)
- **Firebase Cloud Messaging** (push notifications)
- **OneSignal** (alternative)

#### Analytics
- **Vercel Analytics** (web vitals)
- **Google Analytics 4** (user analytics)
- **Mixpanel** (product analytics)
  - Event tracking
  - Funnel analysis
- **PostHog** (open source alternative)
  - Self-hostable
  - Feature flags

#### Payment Processing
- **Stripe** (primary)
  - Subscriptions
  - Invoicing
  - Tax calculation
- **Paddle** (alternative - handles VAT)

#### Customer Support
- **Intercom** (chat support)
  - In-app messaging
  - Knowledge base
- **Zendesk** (ticketing)
- **Crisp** (free alternative)

#### Feature Flags
- **LaunchDarkly** (managed)
  - Gradual rollouts
  - A/B testing
- **Unleash** (open source)
  - Self-hosted
  - Free tier

---

### 7.6 Recommended Tech Stack by Scale

#### Startup/MVP (0-1,000 users)
**Frontend:**
- Next.js 14 + TypeScript
- TailwindCSS + Shadcn/ui
- Zustand + React Query
- Monaco Editor

**Backend:**
- Next.js API Routes (BFF)
- Prisma + PostgreSQL (Supabase)
- Redis (Upstash)
- Judge0 API

**AI:**
- OpenAI GPT-4 + GPT-3.5
- Vercel AI SDK

**Hosting:**
- Vercel (all-in-one)
- Supabase (database + auth)

**Cost:** ~$200/month

---

#### Growth (1,000-10,000 users)
**Frontend:**
- Next.js 14 + TypeScript
- TailwindCSS + Shadcn/ui
- Zustand + React Query
- Monaco Editor

**Backend:**
- Node.js + Express + TypeScript
- Prisma + PostgreSQL (AWS RDS)
- Redis (AWS ElastiCache)
- BullMQ (job queue)
- Socket.io (WebSocket)

**AI:**
- OpenAI (GPT-4, GPT-3.5)
- Anthropic Claude (fallback)
- LangChain (orchestration)

**Hosting:**
- Vercel (frontend)
- Railway/Render (backend)
- AWS RDS (database)
- Cloudflare (CDN)

**Cost:** ~$1,500/month

---

#### Scale (10,000+ users)
**Frontend:**
- Next.js 14 + TypeScript
- TailwindCSS + Shadcn/ui
- Zustand + React Query
- Monaco Editor

**Backend:**
- Node.js + Fastify + TypeScript
- Prisma + PostgreSQL (AWS RDS Multi-AZ)
- Redis Cluster (AWS ElastiCache)
- BullMQ (job queue)
- Socket.io with Redis adapter

**AI:**
- OpenAI (GPT-4, GPT-3.5)
- Anthropic Claude (fallback)
- Google Gemini (fallback)
- LangChain (orchestration)
- Pinecone (vector DB)

**Infrastructure:**
- AWS ECS/Fargate (containers)
- AWS RDS (database)
- AWS ElastiCache (Redis)
- AWS S3 + CloudFront (CDN)
- Kubernetes (if needed)

**Monitoring:**
- Datadog (full stack)
- Sentry (errors)
- LogRocket (sessions)

**Cost:** ~$5,000-10,000/month

---

### 7.7 Technology Decision Matrix

| Requirement | Technology | Alternative | Reason |
|-------------|-----------|-------------|---------|
| Frontend Framework | Next.js 14 | Remix | SSR, SEO, API routes |
| Language | TypeScript | - | Type safety, better DX |
| Styling | TailwindCSS | CSS Modules | Utility-first, fast |
| State Management | Zustand | Redux Toolkit | Lightweight, simple |
| Server State | React Query | SWR | Caching, refetching |
| Code Editor | Monaco | CodeMirror | VS Code engine, features |
| Backend Framework | Express | Fastify | Mature, ecosystem |
| ORM | Prisma | Drizzle | Type-safe, migrations |
| Database | PostgreSQL | MySQL | JSON, full-text search |
| Cache | Redis | Memcached | Data structures, pub/sub |
| Queue | BullMQ | RabbitMQ | Redis-based, simple |
| Auth | NextAuth | Clerk | Open source, flexible |
| AI Primary | OpenAI | Anthropic | Best performance |
| Code Execution | Judge0 | Piston | More languages, stable |
| Hosting (Frontend) | Vercel | Netlify | Best Next.js support |
| Hosting (Backend) | Railway | Render | Easy deploy, scaling |
| Monitoring | Sentry | Rollbar | Best React support |
| Analytics | Mixpanel | Amplitude | Product analytics |

---

### 7.8 Development Environment Setup

```bash
# Prerequisites
node --version  # v20+
npm --version   # v10+
docker --version # v24+
git --version   # v2.40+

# Clone repository
git clone https://github.com/your-org/syntax-saga.git
cd syntax-saga

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env.local
# Edit .env.local with your API keys

# Start development database
docker-compose up -d postgres redis

# Run database migrations
npx prisma migrate dev

# Start development server
npm run dev

# Open browser
# Frontend: http://localhost:3000
# Backend API: http://localhost:3000/api
# Prisma Studio: http://localhost:5555
```

---

This comprehensive tech stack provides a clear path from MVP to scale, with specific recommendations for each growth stage.

---

## 8. Security Considerations

### 8.1 Security Measures

**Code Execution:**
- Always use sandboxed environment (Judge0/Piston)
- Resource limits: 5s timeout, 256MB memory
- No network access from executed code
- Automatic termination of infinite loops

**Input Validation:**
- Sanitize all user inputs
- Validate code length (max 10,000 characters)
- Check for malicious patterns
- Rate limiting per user

**Authentication:**
- JWT-based auth for user sessions
- OAuth2 for social login (Google, GitHub)
- Password hashing with bcrypt
- Session expiration and refresh tokens

**AI Prompt Injection:**
- Sanitize user code before sending to AI
- Separate system prompts from user content
- Validate AI responses before displaying
- Content filtering for inappropriate responses

**Data Protection:**
- Encryption at rest (database)
- Encryption in transit (HTTPS/TLS)
- GDPR compliance for user data
- AI training data opt-out option
- Secure API key management (vault/secrets manager)

**DDoS Protection:**
- Rate limiting (10 submissions/minute per user)
- WAF (Web Application Firewall)
- CDN for static assets
- Load balancing

---

## 9. Performance Optimization

### 9.1 Optimization Strategies

**Caching:**
- Cache common challenge results (Redis, 24hr TTL)
- Cache AI responses for similar code (semantic caching)
- Cache challenge descriptions and hints
- CDN for static assets

**Code Execution:**
- Request queuing for code execution
- Parallel test execution where possible
- Timeout optimization (5s max)

**AI Optimization:**
- Token usage optimization (target <1000 tokens per request)
- Model tiering (GPT-3.5 for simple, GPT-4 for complex)
- Batch processing for non-urgent tasks
- Streaming responses for better perceived latency

**Database:**
- Indexed queries on user_id, challenge_id, created_at
- Connection pooling
- Query optimization
- Read replicas for analytics

**Frontend:**
- Code splitting and lazy loading
- Image optimization
- Service worker for offline support
- Virtual scrolling for long lists

---

## 10. Testing Strategy

### 10.1 Testing Approach

**Unit Tests:**
- Evaluation logic
- AI response parsing
- Complexity analysis
- XP calculation

**Integration Tests:**
- API endpoints
- Database operations
- Code execution pipeline
- AI integration

**E2E Tests:**
- Complete submission flow
- User authentication
- Challenge completion
- Collaboration mode

**Load Testing:**
- Concurrent submissions (100+ users)
- AI API rate limits
- Database performance
- WebSocket connections

**AI Response Validation:**
- Ensure narrative quality
- Verify tier-appropriate language
- Check for hallucinations
- Validate code suggestions

---

## 11. Deployment Pipeline

### 11.1 Deployment Process

**Environments:**
1. Development (local)
2. Staging (pre-production)
3. Production

**CI/CD Pipeline:**
1. Code push to GitHub
2. Automated tests run
3. Build and deploy to staging
4. Manual approval for production
5. Deploy to production
6. Health checks and monitoring

**Database Migrations:**
- Managed via Prisma/Alembic
- Automated in staging
- Manual approval for production
- Rollback plan for failures

**Environment Variables:**
- API keys (OpenAI, Judge0, etc.)
- Database connection strings
- JWT secrets
- Feature flags

**Monitoring:**
- Sentry for error tracking
- LogRocket for session replay
- Datadog for performance monitoring
- Custom dashboards for AI costs

---

## 12. Implementation Examples

### 12.1 Complete Code Evaluation Function

```javascript
async function evaluateCodeSubmission(submission) {
  const { userId, challengeId, code, language, attemptCount } = submission;

  // Step 1: Get user profile to determine tier
  const user = await getUserProfile(userId);
  const userTier = user.educationLevel === 'school' ? 'apprentice' : 'grandmaster';
  
  // Step 2: Execute code and run tests
  const executionResult = await executeCode(code, language, challengeId);
  
  // Step 3: Analyze code complexity (for grandmaster tier)
  const complexityAnalysis = userTier === 'grandmaster' 
    ? await analyzeComplexity(code, language)
    : null;
  
  // Step 4: Prepare context for AI
  const aiContext = {
    userTier,
    userCode: code,
    testResults: executionResult.testResults,
    attemptCount,
    challengeDescription: executionResult.challenge.description,
    optimalSolution: executionResult.challenge.optimalSolution,
    previousHints: await getPreviousHints(userId, challengeId),
    complexityAnalysis
  };
  
  // Step 5: Call AI for narrative feedback
  const aiFeedback = await getAIFeedback(aiContext);
  
  // Step 6: Determine evaluation passes
  const evaluation = {
    syntaxPass: executionResult.syntaxValid,
    logicPass: executionResult.testResults.allPassed,
    elitePass: userTier === 'grandmaster' 
      ? (complexityAnalysis.isOptimal && executionResult.testResults.allPassed)
      : null
  };
  
  // Step 7: Calculate XP and determine result tier
  const result = determineResult(evaluation, userTier);
  const xpAwarded = calculateXP(result, userTier, attemptCount);
  
  // Step 8: Save submission to database
  await saveSubmission({
    userId,
    challengeId,
    code,
    result: result.tier,
    ...evaluation,
    xpAwarded,
    feedback: aiFeedback,
    complexityDetected: complexityAnalysis?.complexity || null
  });
  
  // Step 9: Return response to frontend
  return {
    ...aiFeedback,
    ...evaluation,
    result: result.tier,
    xpAwarded,
    visualFeedback: generateVisualFeedback(result, userTier)
  };
}
```

### 12.2 AI System Prompt (Complete)

```
You are The Adaptive Master, an expert AI tutor and narrator for the Syntax Saga RPG coding platform.

CRITICAL: Detect User Tier First
- Tier = Apprentice (School): Ages 10-17, learning fundamentals
- Tier = Grandmaster (College): Ages 18+, optimizing for production

Apprentice Tier: "The Friendly Wizard"
- Tone: Encouraging, patient, enthusiastic
- Vocabulary: Visual metaphors (lists = "train cars", functions = "magic spells")
- Feedback: "You're getting closer! Try thinking about..."
- Focus: Correctness and understanding over optimization

Grandmaster Tier: "The Grumpy Senior Architect"
- Tone: Critical but constructive, professional, direct
- Vocabulary: Technical terms (dynamic arrays, time complexity, code smells)
- Feedback: "This works, but you're leaving performance on the table."
- Focus: Optimization, scalability, and professional standards

Three-Pass Evaluation System:

Pass 1: Syntax Check
- Question: Does the code run without errors?
- Fail (Apprentice): "Your spell fizzled! Check line {line} for a missing ingredient."
- Fail (Grandmaster): "SyntaxError on line {line}: Missing closing parenthesis."

Pass 2: Logic Check
- Question: Does it solve the quest correctly?
- Fail (Apprentice): "The door is still locked! Your key doesn't quite fit."
- Fail (Grandmaster): "Test case {n} failed: Expected {expected}, got {actual}."

Pass 3: Elite Check (Grandmaster ONLY)
- Question: Is this the optimal solution?
- Fail: "Boss survives with 10% HP! Optimize to O(n) using a hash map."
- Pass: "Critical Hit! Boss defeated! Optimal solution."

Response Format:
🎭 NARRATIVE HEADER
[One epic sentence describing the action in RPG terms]

⚔️ THE RESULT
[Success / Partial Success / Failure]

🔮 THE ORACLE'S INSIGHT
[Technical feedback wrapped in tier-appropriate language]

📜 THE QUEST LOG
[Clear next step or hint - never give full solution unless 3+ attempts]

✨ REWARDS
XP Awarded: [amount]
[Additional tier-specific feedback]

Progressive Hint System:
- Attempt 1: Conceptual hint
- Attempt 2: Structural hint
- Attempt 3: Specific hint
- Attempt 4+: Reveal solution with explanation

Special Rules:
- Never provide full corrected code unless attempt count ≥ 3
- Guide with questions: "What happens if you...?"
- Use analogies appropriate to tier
- Celebrate small wins for Apprentice tier
- Focus on growth areas for Grandmaster tier

Your Mission:
Transform every code submission into an epic learning moment. Make students WANT to fix their code because they're invested in defeating the boss, completing the quest, and becoming a master of the Kingdom of Syntax.
```

---

## Conclusion

This design document provides a complete technical blueprint for building Syntax Saga, an AI-powered RPG coding education platform. The architecture is designed to:

- **Scale efficiently**: AI costs remain constant at $5.90/user regardless of scale
- **Deliver personalized learning**: Tier-adaptive AI provides appropriate feedback
- **Ensure security**: Sandboxed code execution and comprehensive security measures
- **Optimize performance**: Aggressive caching and smart routing reduce costs by 68%
- **Maintain quality**: Comprehensive testing and monitoring ensure reliability

The platform leverages modern technologies and AI to create a transformative learning experience that is both engaging and effective, with measurable improvements in learning outcomes compared to traditional coding platforms.
