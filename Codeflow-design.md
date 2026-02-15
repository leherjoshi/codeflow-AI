# Design Document

## Overview

CodeFlow AI is a full-stack web application that transforms unstructured competitive programming practice into personalized, AI-guided learning experiences. The system integrates with LeetCode's public API, leverages AWS AI services (Bedrock Claude 3 Sonnet, Amazon Q Developer), and provides real-time analytics through a responsive React frontend.

The architecture follows a serverless-first approach using AWS Lambda for compute, DynamoDB for data persistence, and Bedrock for AI capabilities. The frontend is built with React + Vite + Tailwind CSS for rapid development and optimal performance. The backend uses FastAPI with Pydantic for type-safe API development.

**Key Design Principles:**

- Serverless architecture for automatic scaling and cost optimization
- API-first design with OpenAPI specification
- Real-time data synchronization with optimistic UI updates
- Graceful degradation when external services are unavailable
- Mobile-first responsive design
- Bilingual support (English/Hindi) at the infrastructure level

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                             │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  React + Vite + Tailwind CSS (Deployed on Vercel)       │  │
│  │  - Dashboard Components                                   │  │
│  │  - Learning Path Viewer                                   │  │
│  │  - Problem Recommendation UI                              │  │
│  │  - Skill Heatmap Visualization                           │  │
│  │  - GitHub OAuth Integration                               │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              │ HTTPS/REST API + JWT
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      API GATEWAY LAYER                           │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  AWS API Gateway                                          │  │
│  │  - Rate Limiting (100 req/min per user, 10 req/min IP)  │  │
│  │  - Request Validation                                     │  │
│  │  - CORS Configuration (React origin)                     │  │
│  │  - JWT Token Validation                                   │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    APPLICATION LAYER                             │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  FastAPI Backend (AWS Lambda + Function URLs)            │  │
│  │  Middleware Stack:                                        │  │
│  │  - CORS Middleware (React frontend)                      │  │
│  │  - Rate Limiting Middleware (SlowAPI)                    │  │
│  │  - JWT Authentication Middleware                          │  │
│  │  - Request Logging Middleware                             │  │
│  │                                                            │  │
│  │  ┌─────────────────┐  ┌──────────────────┐              │  │
│  │  │ Auth Service    │  │ Analysis Service │              │  │
│  │  │ - JWT Auth      │  │ - Profile Parse  │              │  │
│  │  │ - GitHub OAuth  │  │ - Topic Classify │              │  │
│  │  └─────────────────┘  └──────────────────┘              │  │
│  │                                                            │  │
│  │  ┌─────────────────┐  ┌──────────────────┐              │  │
│  │  │ Learning Path   │  │ Recommendation   │              │  │
│  │  │ Service         │  │ Engine           │              │  │
│  │  └─────────────────┘  └──────────────────┘              │  │
│  │                                                            │  │
│  │  ┌─────────────────┐  ┌──────────────────┐              │  │
│  │  │ Hint Service    │  │ Admin Service    │              │  │
│  │  └─────────────────┘  └──────────────────┘              │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
        │                    │                    │
        │                    │                    │
        ▼                    ▼                    ▼
┌──────────────┐  ┌──────────────────┐  ┌──────────────────┐
│ Redis Cache  │  │   DynamoDB       │  │ Celery + SQS     │
│ ElastiCache  │  │   - Users        │  │ Background Jobs: │
│              │  │   - Paths        │  │ - Heavy Analysis │
│ TTL Config:  │  │   - Progress     │  │ - Weekly Emails  │
│ - User: 1h   │  │   - Analytics    │  │ - Bulk Import    │
│ - Path: 24h  │  │   - Sessions     │  │                  │
│ - Hints: 1h  │  │                  │  │                  │
└──────────────┘  └──────────────────┘  └──────────────────┘
        │                    │                    │
        └────────────────────┴────────────────────┘
                              │
        ┌─────────────────────┴─────────────────────┐
        ▼                     ▼                     ▼
┌──────────────┐  ┌──────────────────┐  ┌──────────────────┐
│ AWS Bedrock  │  │ LeetCode Public  │  │ GitHub OAuth API │
│ Claude 3     │  │ API / Scraping   │  │                  │
│ Sonnet       │  │                  │  │                  │
│              │  │                  │  │                  │
│ Amazon Q     │  │                  │  │                  │
│ Developer    │  │                  │  │                  │
└──────────────┘  └──────────────────┘  └──────────────────┘
                           │
                           ▼
                  ┌──────────────────┐
                  │  CloudWatch +    │
                  │  Sentry          │
                  │  (Monitoring)    │
                  └──────────────────┘
```

## Components and Interfaces

### Frontend Components

#### 1. Dashboard Component

**Responsibility:** Display user's skill heatmap, current streak, and quick stats

**Props:**

```typescript
interface DashboardProps {
  userId: string;
  language: 'en' | 'hi';
}
```

**State Management:**

- Uses React Query for server state caching
- Optimistic updates for real-time feel
- Automatic refetch on window focus

#### 2. Learning Path Viewer

**Responsibility:** Display AI-generated problem sequence with progress tracking

**Props:**

```typescript
interface LearningPathProps {
  pathId: string;
  problems: Problem[];
  currentIndex: number;
  onProblemSelect: (problemId: string) => void;
}
```

#### 3. Skill Heatmap Visualization

**Responsibility:** Interactive visualization of topic proficiency using D3.js or Recharts

**Props:**

```typescript
interface SkillHeatmapProps {
  topics: TopicProficiency[];
  onTopicClick: (topic: string) => void;
}
```

#### 4. Problem Recommendation Card

**Responsibility:** Display next recommended problem with hint access

**Props:**

```typescript
interface ProblemCardProps {
  problem: Problem;
  onRequestHint: () => void;
  onMarkComplete: () => void;
}
```

### Backend Services

#### 1. User Service

**Endpoints:**

- `POST /api/v1/users/register` - Register new user with LeetCode username
- `GET /api/v1/users/{user_id}` - Fetch user profile
- `PUT /api/v1/users/{user_id}/preferences` - Update language/settings
- `DELETE /api/v1/users/{user_id}` - Delete user account

**Dependencies:** DynamoDB (Users table), Scraping Service

#### 2. Analysis Service

**Endpoints:**

- `POST /api/v1/analysis/profile` - Analyze LeetCode profile
- `GET /api/v1/analysis/{user_id}/topics` - Get topic proficiency breakdown
- `POST /api/v1/analysis/{user_id}/sync` - Trigger manual sync with LeetCode

**Dependencies:** Scraping Service, DynamoDB (Users, Progress tables)

**Core Logic:**

```python
def calculate_topic_proficiency(submissions: List[Submission]) -> Dict[str, float]:
    """
    Calculate success rate per topic
    Returns: {topic: proficiency_score (0-100)}
    """
    topic_stats = defaultdict(lambda: {"solved": 0, "attempted": 0})
    
    for submission in submissions:
        for topic in submission.topics:
            topic_stats[topic]["attempted"] += 1
            if submission.status == "Accepted":
                topic_stats[topic]["solved"] += 1
    
    return {
        topic: (stats["solved"] / stats["attempted"] * 100)
        for topic, stats in topic_stats.items()
        if stats["attempted"] > 0
    }
```

#### 3. Learning Path Service

**Endpoints:**

- `POST /api/v1/learning-paths/generate` - Generate AI learning path
- `GET /api/v1/learning-paths/{path_id}` - Retrieve learning path
- `PUT /api/v1/learning-paths/{path_id}/regenerate` - Regenerate with updated profile

**Dependencies:** AWS Bedrock, DynamoDB (Paths table)

**Bedrock Integration:**

```python
async def generate_learning_path(
    weak_topics: List[str],
    strong_topics: List[str],
    user_level: str,
    language: str
) -> LearningPath:
    """
    Invoke Claude 3 Sonnet to generate personalized path
    """
    prompt = f"""You are an expert competitive programming mentor.
    
    User Profile:
    - Weak Topics: {', '.join(weak_topics)}
    - Strong Topics: {', '.join(strong_topics)}
    - Current Level: {user_level}
    
    Generate a learning path of 25 problems that:
    1. Prioritizes weak topics (70% of problems)
    2. Includes mixed difficulty (Easy: 30%, Medium: 50%, Hard: 20%)
    3. Follows logical progression (easier concepts first)
    4. Includes 2-3 problems per weak topic
    
    Return JSON format:
    {{
      "problems": [
        {{"title": "...", "difficulty": "...", "topic": "...", "leetcode_id": "..."}}
      ],
      "reasoning": "..."
    }}
    """
    
    response = bedrock_client.invoke_model(
        modelId="anthropic.claude-3-sonnet-20240229-v1:0",
        body=json.dumps({
            "anthropic_version": "bedrock-2023-05-31",
            "max_tokens": 4096,
            "messages": [{"role": "user", "content": prompt}],
            "temperature": 0.7
        })
    )
    
    return parse_learning_path(response)
```

#### 4. Recommendation Engine

**Endpoints:**

- `GET /api/v1/recommendations/{user_id}/next` - Get next Goldilocks problem
- `POST /api/v1/recommendations/{user_id}/feedback` - Record problem completion

**Dependencies:** DynamoDB (Progress, Paths tables)

**Goldilocks Algorithm:**

```python
def select_goldilocks_problem(
    user_id: str,
    learning_path: LearningPath,
    recent_performance: List[Attempt]
) -> Problem:
    """
    Select problem matching current skill level
    """
    # Calculate recent success rate (last 5 attempts)
    recent_success_rate = sum(1 for a in recent_performance[-5:] if a.success) / 5
    
    # Adjust difficulty based on performance
    if recent_success_rate >= 0.8:
        target_difficulty = "Medium" if current_level == "Easy" else "Hard"
    elif recent_success_rate <= 0.4:
        target_difficulty = "Easy" if current_level == "Medium" else "Medium"
    else:
        target_difficulty = current_level
    
    # Find next unsolved problem in path matching difficulty
    for problem in learning_path.problems:
        if not problem.completed and problem.difficulty == target_difficulty:
            return problem
    
    # Fallback to next unsolved problem
    return next(p for p in learning_path.problems if not p.completed)
```

#### 5. Hint Service

**Endpoints:**

- `POST /api/v1/hints/generate` - Generate hint using Amazon Q
- `GET /api/v1/hints/{problem_id}/levels` - Get progressive hints (levels 1-3)

**Dependencies:** Amazon Q Developer, DynamoDB (Hints cache)

**Amazon Q Integration:**

```python
async def generate_hint(
    problem_description: str,
    hint_level: int,
    language: str
) -> str:
    """
    Generate conceptual hint without code spoilers
    """
    system_prompt = """You are a coding mentor. Provide hints that guide thinking 
    without revealing the solution. Focus on:
    - Key observations about the problem
    - Relevant data structures or algorithms
    - Edge cases to consider
    
    DO NOT provide code or explicit step-by-step solutions."""
    
    hint_prompts = {
        1: "What is the key insight needed to solve this problem?",
        2: "What data structure would be most efficient here?",
        3: "Can you outline the high-level approach without code?"
    }
    
    response = await amazon_q_client.generate_hint(
        problem=problem_description,
        hint_level=hint_prompts[hint_level],
        system=system_prompt,
        language=language
    )
    
    return response.hint_text
```

#### 6. Progress Service

**Endpoints:**

- `GET /api/v1/progress/{user_id}` - Get overall progress stats
- `POST /api/v1/progress/{user_id}/update` - Record problem completion
- `GET /api/v1/progress/{user_id}/streak` - Get current streak and badges

**Dependencies:** DynamoDB (Progress, Users tables)

#### 7. Scraping Service

**Endpoints:**

- `POST /api/v1/scrape/leetcode/{username}` - Fetch LeetCode profile data
- `GET /api/v1/scrape/status/{job_id}` - Check scraping job status

**Dependencies:** LeetCode GraphQL API

**Implementation:**

```python
async def scrape_leetcode_profile(username: str) -> LeetCodeProfile:
    """
    Fetch user data from LeetCode GraphQL API
    Implements rate limiting and retry logic
    """
    query = """
    query getUserProfile($username: String!) {
      matchedUser(username: $username) {
        username
        submitStats {
          acSubmissionNum {
            difficulty
            count
          }
        }
        tagProblemCounts {
          advanced {
            tagName
            problemsSolved
          }
        }
        recentSubmissionList(limit: 100) {
          title
          titleSlug
          timestamp
          statusDisplay
          lang
        }
      }
    }
    """
    
    async with httpx.AsyncClient() as client:
        response = await client.post(
            "https://leetcode.com/graphql",
            json={"query": query, "variables": {"username": username}},
            headers={"Content-Type": "application/json"},
            timeout=10.0
        )
        
        if response.status_code == 429:
            raise RateLimitError("LeetCode rate limit exceeded")
        
        data = response.json()
        return parse_leetcode_response(data)
```

#### 8. Admin Service

**Endpoints:**

- `GET /api/v1/admin/analytics/dau` - Daily active users
- `GET /api/v1/admin/analytics/retention` - User retention metrics
- `GET /api/v1/admin/health` - System health check
- `GET /api/v1/admin/errors` - Recent error logs

**Dependencies:** DynamoDB (Analytics table), CloudWatch

## Data Models

### DynamoDB Tables

#### Users Table

```python
class User(BaseModel):
    user_id: str  # Partition Key (UUID)
    leetcode_username: str  # GSI
    email: Optional[str]
    language_preference: Literal["en", "hi"] = "en"
    created_at: datetime
    last_login: datetime
    profile_data: LeetCodeProfile
    
class LeetCodeProfile(BaseModel):
    total_solved: int
    easy_solved: int
    medium_solved: int
    hard_solved: int
    topic_proficiency: Dict[str, float]  # {topic: score}
    recent_submissions: List[Submission]
    last_synced: datetime
```

#### LearningPaths Table

```python
class LearningPath(BaseModel):
    path_id: str  # Partition Key (UUID)
    user_id: str  # GSI
    problems: List[PathProblem]
    created_at: datetime
    current_index: int
    completion_rate: float
    
class PathProblem(BaseModel):
    leetcode_id: str
    title: str
    difficulty: Literal["Easy", "Medium", "Hard"]
    topic: str
    completed: bool
    attempts: int
    hints_used: int
```

#### Progress Table

```python
class Progress(BaseModel):
    progress_id: str  # Partition Key (user_id#date)
    user_id: str  # GSI
    date: str  # ISO date
    problems_solved: int
    topics_practiced: List[str]
    streak_count: int
    badges: List[Badge]
    
class Badge(BaseModel):
    badge_id: str
    name: str
    earned_at: datetime
    milestone: int  # e.g., 7, 30, 100 days
```

#### Hints Table (Cache)

```python
class HintCache(BaseModel):
    problem_id: str  # Partition Key
    hint_level: int  # Sort Key (1-3)
    hint_text_en: str
    hint_text_hi: str
    generated_at: datetime
    ttl: int  # DynamoDB TTL for auto-deletion
```

#### Analytics Table

```python
class DailyAnalytics(BaseModel):
    date: str  # Partition Key (ISO date)
    metric_type: str  # Sort Key (DAU, WAU, MAU, etc.)
    value: float
    metadata: Dict[str, Any]
```

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property Reflection

After analyzing all acceptance criteria, I've identified the following consolidations to eliminate redundancy:

**Consolidations:**

- Properties 2.3 and 2.4 (weak/strong topic classification) can be combined into a single property about topic classification rules
- Properties 6.1, 6.3, 7.4 (dashboard/profile display requirements) can be consolidated into properties about required UI elements
- Properties 8.1, 8.2, 8.3 (language switching) can be combined into a comprehensive language preference property
- Properties 10.1, 10.2, 10.3 (admin dashboard elements) can be consolidated into a single property about admin metrics display
- Properties 13.1 and 13.2 (encryption) can be combined into a comprehensive security property

**Unique Properties Retained:**

- User registration and validation (1.1-1.4)
- Profile analysis and parsing (2.1, 2.2, 2.5)
- AI learning path generation (3.1-3.5)
- Goldilocks recommendation algorithm (4.1-4.5)
- Hint generation and progression (5.1-5.5)
- Real-time updates (6.5, 9.2)
- Streak and gamification logic (7.1-7.3)
- Sync and retry behavior (9.1, 9.3-9.5)
- Error handling and resilience (12.1-12.5)
- Mobile responsiveness (14.1-14.4)

### Correctness Properties

**Property 1: Valid username creates account**
*For any* valid LeetCode username, when submitted for registration, the system should create a user account with that exact username stored in DynamoDB.
**Validates: Requirements 1.1, 1.4**

**Property 2: Registration performance bound**
*For any* LeetCode username, the scraping service should fetch and return profile statistics within 5 seconds of registration request.
**Validates: Requirements 1.2**

**Property 3: Invalid username rejection**
*For any* invalid or non-existent LeetCode username, the system should return an error response and prevent account creation.
**Validates: Requirements 1.3**

**Property 4: Language preference persistence**
*For any* user who sets language preference to Hindi, all subsequent API responses and UI elements should be in Hindi, and this preference should be stored in DynamoDB.
**Validates: Requirements 1.5, 8.1, 8.2, 8.3**

**Property 5: Submission parsing completeness**
*For any* LeetCode profile data, the system should parse all submissions and correctly categorize each problem by its topic tags.
**Validates: Requirements 2.1**

**Property 6: Topic proficiency calculation**
*For any* set of submissions for a topic, the proficiency score should equal (solved_count / attempted_count) × 100.
**Validates: Requirements 2.2**

**Property 7: Topic classification rules**
*For any* topic with calculated proficiency, topics with score < 40% should be classified as "Weak", topics with score > 70% should be classified as "Strong", and all others as "Moderate".
**Validates: Requirements 2.3, 2.4**

**Property 8: Heatmap completeness**
*For any* user profile analysis, the generated skill heatmap should contain entries for all topics present in the user's submission history with appropriate color codes.
**Validates: Requirements 2.5, 6.2**

**Property 9: Learning path generation trigger**
*For any* completed profile analysis, the system should invoke AWS Bedrock Claude 3 Sonnet with weak topics, strong topics, and user level as parameters.
**Validates: Requirements 3.1**

**Property 10: Learning path structure**
*For any* generated learning path, it should contain between 20-30 problems, with at least 70% targeting weak topics, and difficulty distribution of approximately 30% Easy, 50% Medium, 20% Hard.
**Validates: Requirements 3.2**

**Property 11: Learning path performance**
*For any* learning path generation request, AWS Bedrock should return a response within 10 seconds.
**Validates: Requirements 3.3**

**Property 12: Learning path persistence**
*For any* generated learning path, it should be stored in DynamoDB with user_id association, creation timestamp, and all problem details.
**Validates: Requirements 3.4**

**Property 13: Learning path display completeness**
*For any* learning path displayed to user, each problem should show title, difficulty level, and estimated completion time.
**Validates: Requirements 3.5**

**Property 14: Goldilocks selection from path**
*For any* next problem request, the recommended problem should come from the user's current learning path and match their skill level.
**Validates: Requirements 4.1**

**Property 15: Adaptive difficulty adjustment**
*For any* user with recent success rate ≥ 80%, the next problem should be harder than current level; for success rate ≤ 40%, the next problem should be easier.
**Validates: Requirements 4.2**

**Property 16: Proficiency update on completion**
*For any* successfully completed problem, the user's proficiency score for that problem's topics should increase immediately.
**Validates: Requirements 4.3**

**Property 17: Failure-based difficulty reduction**
*For any* problem failed twice consecutively, the next recommended problem should be easier and on the same topic.
**Validates: Requirements 4.4**

**Property 18: Path regeneration on topic completion**
*For any* weak topic where all problems are completed, the system should trigger learning path regeneration with updated topic priorities.
**Validates: Requirements 4.5**

**Property 19: Hint generation invocation**
*For any* hint request, the system should invoke Amazon Q Developer with the problem description and current hint level.
**Validates: Requirements 5.1**

**Property 20: Hint code-free constraint**
*For any* generated hint, it should not contain code snippets, function implementations, or explicit algorithmic solutions.
**Validates: Requirements 5.2**

**Property 21: Hint response performance**
*For any* hint request, the system should return a response within 3 seconds.
**Validates: Requirements 5.3**

**Property 22: Progressive hint levels**
*For any* problem, requesting hints sequentially should provide increasingly detailed guidance, with a maximum of 3 hint levels available.
**Validates: Requirements 5.4**

**Property 23: Hint service fallback**
*For any* hint request when Amazon Q is unavailable, the system should return a pre-generated hint from the cache database.
**Validates: Requirements 5.5**

**Property 24: Dashboard element completeness**
*For any* user accessing the dashboard, it should display the skill heatmap, all programming topics, problem counts, success rates, and recent attempts.
**Validates: Requirements 6.1, 6.3**

**Property 25: Dashboard performance**
*For any* dashboard load request, the system should fetch and display all data within 2 seconds.
**Validates: Requirements 6.4**

**Property 26: Real-time dashboard updates**
*For any* problem completion, the dashboard should update proficiency scores and statistics without requiring page refresh.
**Validates: Requirements 6.5**

**Property 27: Streak increment on daily solve**
*For any* user who solves at least one problem on a given day, their streak counter should increment by 1.
**Validates: Requirements 7.1**

**Property 28: Streak reset on missed day**
*For any* user who does not solve problems for 24+ hours, their streak counter should reset to 0.
**Validates: Requirements 7.2**

**Property 29: Milestone badge awarding**
*For any* user reaching streak counts of 7, 30, or 100 days, the system should award the corresponding streak badge.
**Validates: Requirements 7.3**

**Property 30: Progress display completeness**
*For any* user viewing progress, the display should show total problems solved, current streak count, and all earned badges.
**Validates: Requirements 7.4, 7.5**

**Property 31: Problem description bilingual display**
*For any* problem displayed in the UI, it should show the original English text with optional Hindi translation available.
**Validates: Requirements 8.4**

**Property 32: Login sync trigger**
*For any* user login event, the system should trigger the scraping service to fetch latest LeetCode statistics.
**Validates: Requirements 9.1**

**Property 33: Profile update propagation**
*For any* new submissions detected by scraping, the system should update the user profile and recalculate all topic proficiency scores.
**Validates: Requirements 9.2**

**Property 34: Rate limit exponential backoff**
*For any* rate limit error from LeetCode, the scraping service should retry with exponentially increasing delays (1s, 2s, 4s).
**Validates: Requirements 9.3**

**Property 35: Scraping failure fallback**
*For any* scraping operation that fails after 3 retries, the system should use cached profile data and display a notification to the user.
**Validates: Requirements 9.4**

**Property 36: Scraping rate limit compliance**
*For any* user, the scraping service should not exceed 10 requests per minute to LeetCode.
**Validates: Requirements 9.5**

**Property 37: Admin metrics completeness**
*For any* admin accessing the analytics dashboard, it should display DAU, WAU, MAU, API response times, error rates, retention rates, and streak distribution.
**Validates: Requirements 10.1, 10.2, 10.3**

**Property 38: Error threshold alerting**
*For any* 5-minute window where error rate exceeds 5%, the system should send alerts to admin users.
**Validates: Requirements 10.4**

**Property 39: Analytics refresh interval**
*For any* analytics dashboard, data should refresh every 5 minutes automatically.
**Validates: Requirements 10.5**

**Property 40: Page load performance**
*For any* page load request, initial content should be delivered within 2 seconds.
**Validates: Requirements 11.1**

**Property 41: API response time percentile**
*For any* 100 consecutive API requests, at least 95 should complete within 500 milliseconds.
**Validates: Requirements 11.2**

**Property 42: Lambda cold start bound**
*For any* AWS Lambda function invocation, cold start time should not exceed 1 second.
**Validates: Requirements 11.4**

**Property 43: Bedrock unavailability handling**
*For any* learning path generation request when AWS Bedrock is unavailable, the system should display a user-friendly error message with retry suggestion.
**Validates: Requirements 12.1**

**Property 44: DynamoDB retry with backoff**
*For any* failed DynamoDB operation, the system should retry up to 3 times with exponential backoff delays.
**Validates: Requirements 12.2**

**Property 45: Invalid HTML fallback**
*For any* scraping operation encountering invalid HTML, the system should log the error to CloudWatch and use cached profile data.
**Validates: Requirements 12.3**

**Property 46: Rate limit request queuing**
*For any* API request when rate limits are exceeded, the system should queue the request and process it when limits reset.
**Validates: Requirements 12.4**

**Property 47: Critical error logging**
*For any* critical error, the system should log to CloudWatch and trigger a Sentry alert with error context.
**Validates: Requirements 12.5**

**Property 48: Data encryption at rest and transit**
*For any* user data stored in DynamoDB, sensitive fields should be encrypted at rest, and all API communications should use HTTPS/TLS.
**Validates: Requirements 13.1, 13.2**

**Property 49: Account deletion completeness**
*For any* user account deletion request, all personal data should be removed from DynamoDB within 24 hours.
**Validates: Requirements 13.3**

**Property 50: Admin authentication requirement**
*For any* admin endpoint access attempt, the system should require valid authentication tokens with admin role.
**Validates: Requirements 13.5**

**Property 51: Mobile responsive layout**
*For any* screen size from 320px to 1920px, the UI should display a properly formatted responsive layout without horizontal scrolling.
**Validates: Requirements 14.1**

**Property 52: Mobile heatmap touch adaptation**
*For any* skill heatmap viewed on mobile devices, touch interactions should be enabled with appropriate gesture handling.
**Validates: Requirements 14.2**

**Property 53: Touch target minimum size**
*For any* interactive button or link on mobile, the tap target should be at least 44px × 44px.
**Validates: Requirements 14.3**

**Property 54: Mobile asset optimization**
*For any* mobile page load, assets should be optimized (compressed images, minified JS/CSS) to reduce data usage.
**Validates: Requirements 14.4**

**Property 55: Payment-free registration**
*For any* user registration flow, no payment information fields should be present or required.
**Validates: Requirements 15.2**

**Property 56: Ad-free experience**
*For any* page in the application, no advertisement elements should be present in the DOM.
**Validates: Requirements 15.4**

## Error Handling

### Error Categories

#### 1. External Service Errors

**LeetCode Scraping Failures:**

- Rate limiting (429): Implement exponential backoff, queue requests
- Invalid username (404): Return user-friendly error, suggest username check
- Network timeout: Retry up to 3 times, fallback to cached data
- Invalid HTML structure: Log error, use cached data, alert admins

**AWS Bedrock Errors:**

- Service unavailable (503): Display retry message, queue request
- Throttling (429): Implement token bucket rate limiting
- Invalid response: Log error, use fallback learning path template
- Timeout (>10s): Cancel request, suggest retry

**Amazon Q Developer Errors:**

- Service unavailable: Use pre-generated hint cache
- Rate limiting: Queue hint requests, process sequentially
- Invalid response: Return generic hint, log for review

#### 2. Database Errors

**DynamoDB Failures:**

- ProvisionedThroughputExceededException: Retry with exponential backoff
- ResourceNotFoundException: Create table/item if missing
- ConditionalCheckFailedException: Handle optimistic locking conflicts
- Network errors: Retry up to 3 times with backoff

#### 3. Validation Errors

**Input Validation:**

- Invalid username format: Return 400 with specific error message
- Missing required fields: Return 422 with field-level errors
- Invalid data types: Return 400 with type mismatch details
- Out-of-range values: Return 400 with acceptable range

#### 4. Authentication/Authorization Errors

- Missing auth token: Return 401 Unauthorized
- Invalid/expired token: Return 401 with refresh instruction
- Insufficient permissions: Return 403 Forbidden with required role
- Rate limit exceeded: Return 429 with retry-after header

### Error Response Format

```python
class ErrorResponse(BaseModel):
    error_code: str  # e.g., "LEETCODE_USER_NOT_FOUND"
    message: str  # User-friendly message
    details: Optional[Dict[str, Any]]  # Additional context
    retry_after: Optional[int]  # Seconds to wait before retry
    support_id: str  # Unique ID for support tracking
```

### Logging Strategy

**CloudWatch Log Groups:**

- `/codeflow/api/errors` - All API errors with stack traces
- `/codeflow/scraping/failures` - LeetCode scraping failures
- `/codeflow/ai/responses` - AI service responses for debugging
- `/codeflow/performance` - Slow queries and performance issues

**Sentry Integration:**

- Critical errors (5xx, service unavailable)
- Unexpected exceptions with full context
- Performance degradation alerts
- User-impacting errors with session replay

## Testing Strategy

### Unit Testing

**Framework:** pytest with pytest-asyncio for async tests

**Coverage Targets:**

- Core business logic: 90%+ coverage
- API endpoints: 85%+ coverage
- Utility functions: 95%+ coverage

**Key Unit Tests:**

- Topic proficiency calculation with various submission patterns
- Goldilocks algorithm with different success rates
- Streak calculation with edge cases (timezone boundaries, missed days)
- Learning path parsing and validation
- Error handling for each service integration

**Example Unit Test:**

```python
def test_topic_proficiency_calculation():
    """Test proficiency score calculation"""
    submissions = [
        Submission(topic="Arrays", status="Accepted"),
        Submission(topic="Arrays", status="Accepted"),
        Submission(topic="Arrays", status="Wrong Answer"),
        Submission(topic="Arrays", status="Time Limit Exceeded"),
    ]
    
    proficiency = calculate_topic_proficiency(submissions)
    
    assert proficiency["Arrays"] == 50.0  # 2 solved / 4 attempted
```

### Property-Based Testing

**Framework:** Hypothesis (Python property-based testing library)

**Configuration:**

- Minimum 100 iterations per property test
- Deterministic random seed for reproducibility
- Shrinking enabled for minimal failing examples

**Property Test Requirements:**

- Each correctness property from the design document MUST be implemented as a property-based test
- Each test MUST be tagged with: `# Feature: codeflow-ai-platform, Property {N}: {property_text}`
- Tests MUST use Hypothesis strategies to generate diverse inputs
- Tests MUST NOT use mocks for core logic (only for external services)

**Example Property Test:**

```python
from hypothesis import given, strategies as st

# Feature: codeflow-ai-platform, Property 6: Topic proficiency calculation
@given(
    solved=st.integers(min_value=0, max_value=100),
    attempted=st.integers(min_value=1, max_value=100)
)
def test_proficiency_calculation_property(solved, attempted):
    """For any solved/attempted counts, proficiency should equal (solved/attempted)*100"""
    # Ensure solved <= attempted
    solved = min(solved, attempted)
    
    submissions = [
        Submission(topic="Test", status="Accepted" if i < solved else "Wrong Answer")
        for i in range(attempted)
    ]
    
    proficiency = calculate_topic_proficiency(submissions)
    expected = (solved / attempted) * 100
    
    assert abs(proficiency["Test"] - expected) < 0.01  # Float comparison tolerance
```

**Property Test Strategies:**

```python
# Custom Hypothesis strategies for domain objects
@st.composite
def leetcode_username(draw):
    """Generate valid LeetCode usernames"""
    length = draw(st.integers(min_value=3, max_value=20))
    return draw(st.text(
        alphabet=st.characters(whitelist_categories=('Ll', 'Lu', 'Nd'), whitelist_characters='_-'),
        min_size=length,
        max_size=length
    ))

@st.composite
def submission_history(draw):
    """Generate realistic submission histories"""
    num_submissions = draw(st.integers(min_value=1, max_value=200))
    topics = draw(st.lists(st.sampled_from(TOPIC_LIST), min_size=1, max_size=10))
    
    return [
        Submission(
            topic=draw(st.sampled_from(topics)),
            status=draw(st.sampled_from(["Accepted", "Wrong Answer", "Time Limit Exceeded"])),
            timestamp=draw(st.datetimes(min_value=datetime(2020, 1, 1)))
        )
        for _ in range(num_submissions)
    ]
```

### Integration Testing

**Scope:**

- End-to-end user flows (registration → analysis → learning path → problem recommendation)
- AWS service integrations (Bedrock, DynamoDB, CloudWatch)
- LeetCode scraping with mock responses
- Real-time update propagation

**Test Environment:**

- LocalStack for AWS services (DynamoDB, Lambda)
- Mock LeetCode API responses
- Test DynamoDB tables with isolated data

### Performance Testing

**Tools:** Locust for load testing

**Scenarios:**

- 1000 concurrent users accessing dashboard
- 100 simultaneous learning path generations
- Sustained load of 50 req/s for 10 minutes

**Metrics:**

- P50, P95, P99 response times
- Error rate under load
- Database throughput
- Lambda cold start frequency

## Deployment Architecture

### Frontend Deployment (Vercel)

**Build Configuration:**

```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "framework": "vite",
  "env": {
    "VITE_API_BASE_URL": "@api-base-url",
    "VITE_SENTRY_DSN": "@sentry-dsn"
  }
}
```

**Features:**

- Automatic deployments on git push
- Preview deployments for pull requests
- Edge caching for static assets
- Automatic HTTPS with custom domain

### Backend Deployment (AWS Lambda)

**Lambda Configuration:**

```yaml
functions:
  api:
    handler: main.handler
    runtime: python3.11
    memorySize: 1024
    timeout: 30
    environment:
      DYNAMODB_TABLE_PREFIX: ${env:STAGE}
      BEDROCK_MODEL_ID: anthropic.claude-3-sonnet-20240229-v1:0
      LOG_LEVEL: INFO
    layers:
      - arn:aws:lambda:region:account:layer:dependencies:1
```

**Deployment Pipeline:**

1. GitHub Actions trigger on main branch push
2. Run tests (unit + property + integration)
3. Build Lambda deployment package
4. Deploy to staging environment
5. Run smoke tests
6. Deploy to production with gradual rollout (10% → 50% → 100%)

### Database Setup (DynamoDB)

**Table Configurations:**

```python
# Users Table
{
    "TableName": "codeflow-users",
    "KeySchema": [{"AttributeName": "user_id", "KeyType": "HASH"}],
    "GlobalSecondaryIndexes": [
        {
            "IndexName": "leetcode-username-index",
            "KeySchema": [{"AttributeName": "leetcode_username", "KeyType": "HASH"}]
        }
    ],
    "BillingMode": "PAY_PER_REQUEST",
    "StreamSpecification": {"StreamEnabled": True, "StreamViewType": "NEW_AND_OLD_IMAGES"}
}

# LearningPaths Table
{
    "TableName": "codeflow-learning-paths",
    "KeySchema": [{"AttributeName": "path_id", "KeyType": "HASH"}],
    "GlobalSecondaryIndexes": [
        {
            "IndexName": "user-id-index",
            "KeySchema": [{"AttributeName": "user_id", "KeyType": "HASH"}]
        }
    ],
    "BillingMode": "PAY_PER_REQUEST"
}
```

### Monitoring and Observability

**CloudWatch Dashboards:**

- API Gateway metrics (request count, latency, errors)
- Lambda metrics (invocations, duration, errors, cold starts)
- DynamoDB metrics (read/write capacity, throttles)
- Custom business metrics (DAU, problems solved, learning paths generated)

**Alarms:**

- API error rate > 5% for 5 minutes
- Lambda duration > 10s (P95)
- DynamoDB throttling events
- Bedrock API failures > 10% for 5 minutes

**Sentry Configuration:**

```python
sentry_sdk.init(
    dsn=os.getenv("SENTRY_DSN"),
    environment=os.getenv("STAGE"),
    traces_sample_rate=0.1,  # 10% of transactions
    profiles_sample_rate=0.1,
    integrations=[
        FastApiIntegration(),
        HttpxIntegration(),
    ]
)
```

## Scalability Plan

### Current Architecture (MVP - 10K users)

- AWS Lambda with on-demand scaling
- DynamoDB with on-demand billing
- Vercel free tier for frontend
- AWS Bedrock pay-per-use

**Estimated Costs:**

- Lambda: ~$50/month (1M requests)
- DynamoDB: ~$25/month (1M reads, 500K writes)
- Bedrock: ~$100/month (10K learning paths)
- Total: ~$175/month

### Scale to 100K users

**Optimizations:**

- Implement Redis caching layer (ElastiCache) for:
  - User profiles (5-minute TTL)
  - Learning paths (1-hour TTL)
  - Hint cache (24-hour TTL)
- DynamoDB provisioned capacity for predictable workloads
- CloudFront CDN for API responses
- Batch processing for analytics

**Estimated Costs:**

- Lambda: ~$300/month
- DynamoDB: ~$150/month
- ElastiCache: ~$50/month
- Bedrock: ~$800/month
- Total: ~$1,300/month

### Scale to 1M users

**Architecture Changes:**

- Migrate to ECS Fargate for long-running processes
- Implement event-driven architecture with EventBridge
- Add read replicas for DynamoDB global tables
- Implement CDN caching for static learning paths
- Use SQS for async processing (scraping, analytics)

**Estimated Costs:**

- ECS Fargate: ~$500/month
- DynamoDB: ~$1,000/month
- ElastiCache: ~$200/month
- Bedrock: ~$5,000/month
- SQS + EventBridge: ~$100/month
- Total: ~$6,800/month

## Security Considerations

### Authentication

- JWT tokens with 24-hour expiration
- Refresh tokens with 30-day expiration
- Token rotation on security events

### Authorization

- Role-based access control (Student, Admin)
- API key authentication for admin endpoints
- Rate limiting per user (100 req/min)

### Data Protection

- DynamoDB encryption at rest (AWS managed keys)
- TLS 1.3 for all API communications
- Secrets stored in AWS Secrets Manager
- No PII in CloudWatch logs

### Compliance

- GDPR-compliant data deletion (24-hour SLA)
- User data export API
- Privacy policy and terms of service
- Audit logging for admin actions

## Future Enhancements

### Phase 2 (Post-MVP)

- Contest calendar integration
- Peer comparison and leaderboards
- Video solution explanations
- Code submission and execution
- Mobile native apps (iOS/Android)

### Phase 3 (Advanced Features)

- AI-powered code review
- Interview preparation mode
- Company-specific problem sets
- Collaborative problem-solving rooms
- Integration with other platforms (Codeforces, HackerRank)

## Skill Roadmap Architecture

### Visual Roadmap Overview

The CodeFlow AI skill roadmap follows a flowchart-style architecture that visualizes the complete learning journey from student entry to successful platform deployment. The roadmap is organized into interconnected phases with clear dependencies and parallel development tracks.

```
                                    CodeFlow AI - Skill Roadmap
                                   Learning Path Architecture

    👤 Student
       |
       |---> 🔐 Auth -----> 📝 Register
       |      (OAuth)        (Profile)
       |
       |---> 🕷️ Scraping (LeetCode)
                |
                |---> 📊 Profile Parse ---> ⚙️ Analysis (Topics)
                         |                        |
                         |                        |
                         v                        v
                    🤖 Bedrock (Claude 3) <-------+
                         |
                         v
                    🎯 Learning Path Gen
                         |
                         v
                    🧠 Recommend Algorithm
                         |
        +----------------+----------------+
        |                |                |
        v                v                v
   💡 Amazon Q      [CENTER]         ⚛️ React
   (Hints)                           (Frontend)
        |                                 |
        v                                 v
   ⚡ FastAPI                        🎨 Tailwind CSS
   (Backend)                              |
        |                                 v
        v                            📈 Charts (D3.js)
   ☁️ Lambda                              |
   (Serverless)                          v
        |                            🗺️ Heatmap (Visual)
        v                                 |
   💾 DynamoDB                            v
   (NoSQL)                           📱 Mobile (Responsive)
        |
        v
   🧪 Testing (Pytest)
        |
        v
   🚀 CI/CD (GitHub)
        |
        +--------> 📊 Monitor (CloudWatch)
        |
        +--------> 🔒 Security (JWT Auth)
        |
        +------------------+------------------+
                           |
                           v
                    🎯 Success Platform
              AI-Powered Learning for 1M+ Students
```

**Legend:**

- 🔴 Red: Authentication & Onboarding
- 🟠 Orange: Data Collection & AI Processing  
- 🟢 Teal: Analysis & Database
- 🟣 Purple: Backend & Infrastructure
- 🔵 Blue: Frontend & Visualization
- 🟪 Magenta: Testing & Quality

### Roadmap Structure

The roadmap follows a tree-based branching structure with the following flow:

```
Student User → Authentication → Data Collection → AI Processing → 
    ├─ Frontend Track (React, Tailwind, Visualization, Mobile)
    ├─ Backend Track (FastAPI, Lambda, Database)
    └─ Quality Track (Testing, CI/CD, Monitoring, Security)
        → Success Platform (1M+ Students)
```

### Phase Breakdown

#### Phase 1: Authentication & Onboarding (Red)

**Skills Required:**

- 🔐 **Authentication**: GitHub OAuth, JWT tokens, session management
- 📝 **Registration**: User profile setup, LeetCode username validation

**Timeline:** Week 1-2  
**Dependencies:** None  
**Deliverables:** Working authentication system with OAuth integration

---

#### Phase 2: Data Collection (Orange)

**Skills Required:**

- 🕷️ **Web Scraping**: LeetCode GraphQL API integration, rate limiting (10 req/min)
- 📊 **Profile Parsing**: Submission history extraction, problem categorization

**Timeline:** Week 2-3  
**Dependencies:** Phase 1 (Authentication)  
**Deliverables:** Automated LeetCode profile scraping service

---

#### Phase 3: Analysis & Database (Teal/Orange)

**Skills Required:**

- ⚙️ **Topic Analysis**: Proficiency calculation, weak/strong topic detection
- 💾 **DynamoDB**: NoSQL data modeling, GSI optimization, caching strategy

**Timeline:** Week 3-4  
**Dependencies:** Phase 2 (Data Collection)  
**Deliverables:** Topic classification engine and database schema

---

#### Phase 4: AI Processing (Orange)

**Skills Required:**

- 🤖 **AWS Bedrock**: Claude 3 Sonnet integration, prompt engineering
- 🎯 **Learning Path Generation**: Weak topic prioritization, difficulty distribution
- 🧠 **Recommendation Engine**: Goldilocks algorithm, adaptive difficulty
- 💡 **Amazon Q**: Hint generation, progressive levels, no code spoilers

**Timeline:** Week 4-7  
**Dependencies:** Phase 3 (Analysis)  
**Deliverables:** AI-powered learning path generator and recommendation system

---

#### Phase 5: Frontend Development (Purple/Blue/Green)

**Skills Required:**

- ⚛️ **React + Vite**: Component architecture, React Query, state management
- 🎨 **Tailwind CSS**: Responsive design, mobile-first approach, accessibility
- 📈 **Data Visualization**: D3.js/Recharts, skill heatmap, progress charts
- 📱 **Mobile Responsive**: Touch interactions, optimized layouts

**Timeline:** Week 5-9 (Parallel with Backend)  
**Dependencies:** Phase 4 (AI Processing)  
**Deliverables:** Complete frontend application with visualizations

---

#### Phase 6: Backend Development (Orange/Purple/Teal)

**Skills Required:**

- ⚡ **FastAPI**: RESTful API design, Pydantic validation, async/await
- ☁️ **AWS Lambda**: Serverless architecture, function URLs, cold start optimization
- 💾 **DynamoDB Integration**: CRUD operations, query optimization
- 🚀 **Background Jobs**: Celery + SQS for async tasks

**Timeline:** Week 5-9 (Parallel with Frontend)  
**Dependencies:** Phase 4 (AI Processing)  
**Deliverables:** Complete backend API with serverless deployment

---

#### Phase 7: Quality & Operations (Purple/Orange/Blue)

**Skills Required:**

- 🧪 **Testing**: Pytest, Hypothesis (property-based testing), 90%+ coverage
- 🚀 **CI/CD**: GitHub Actions, automated testing, gradual rollout
- 📊 **Monitoring**: CloudWatch dashboards, Sentry error tracking
- 🔒 **Security**: JWT authentication, rate limiting, CORS, input validation

**Timeline:** Week 10-12 (Parallel with Phases 5-6)  
**Dependencies:** Phases 5 & 6  
**Deliverables:** Production-ready system with comprehensive testing and monitoring

---

#### Phase 8: Integration & Launch

**Skills Required:**

- Integration testing across all components
- Performance optimization and load testing
- Security audit and penetration testing
- Documentation and deployment guides

**Timeline:** Week 13-16  
**Dependencies:** All previous phases  
**Deliverables:** Fully integrated platform ready for 10K users (MVP)

---

### Color-Coded Skill Categories

The roadmap uses color coding to represent different skill domains:

| Color | Category | Technologies |
|-------|----------|-------------|
| 🔴 **Red** | Authentication & Onboarding | OAuth, JWT, Session Management |
| 🟠 **Orange** | Data Collection & AI | LeetCode API, AWS Bedrock, Amazon Q, FastAPI |
| 🟢 **Teal** | Analysis & Database | Topic Classification, DynamoDB, Proficiency Tracking |
| 🟣 **Purple** | Backend & Infrastructure | AWS Lambda, Serverless, Monitoring |
| 🔵 **Blue** | Frontend & Visualization | React, Tailwind, D3.js, Mobile |
| 🟪 **Magenta** | Testing & Quality | Pytest, Hypothesis, CI/CD |

### Parallel Development Tracks

The roadmap supports parallel development across three main tracks:

**Track 1: Frontend Development (Weeks 5-9)**

- React component development
- Tailwind CSS styling
- Data visualization implementation
- Mobile responsiveness

**Track 2: Backend Development (Weeks 5-9)**

- FastAPI endpoint creation
- AWS Lambda deployment
- Database integration
- Background job processing

**Track 3: Quality Assurance (Weeks 10-12)**

- Test suite development
- CI/CD pipeline setup
- Monitoring configuration
- Security hardening

### Critical Path

The critical path through the roadmap (longest sequence of dependent tasks):

1. Authentication (2 weeks)
2. Data Collection (1 week)
3. Analysis & Database (1 week)
4. AI Processing (3 weeks)
5. Frontend/Backend Development (4 weeks, parallel)
6. Quality & Operations (2 weeks)
7. Integration & Launch (4 weeks)

**Total Critical Path Duration:** 16 weeks

### Skill Prerequisites by Role

#### Frontend Developer

- React.js (intermediate to advanced)
- Tailwind CSS (intermediate)
- D3.js or Recharts (basic to intermediate)
- Responsive design principles
- State management (React Query)

#### Backend Developer

- Python (advanced)
- FastAPI (intermediate)
- AWS services (Lambda, DynamoDB, Bedrock)
- Async programming
- RESTful API design

#### AI/ML Engineer

- AWS Bedrock and Claude 3 Sonnet
- Prompt engineering
- Amazon Q Developer
- Algorithm design (recommendation systems)
- Data analysis

#### DevOps Engineer

- AWS infrastructure (Lambda, DynamoDB, CloudWatch)
- CI/CD pipelines (GitHub Actions)
- Monitoring and alerting (Sentry, CloudWatch)
- Security best practices
- Performance optimization

#### QA Engineer

- Pytest framework
- Hypothesis (property-based testing)
- Integration testing
- Load testing (Locust)
- Test automation

### Learning Resources

For each skill category, recommended learning resources:

**AWS Services:**

- AWS Bedrock Documentation: <https://docs.aws.amazon.com/bedrock/>
- AWS Lambda Best Practices: <https://docs.aws.amazon.com/lambda/>
- DynamoDB Developer Guide: <https://docs.aws.amazon.com/dynamodb/>

**Frontend Development:**

- React Official Documentation: <https://react.dev/>
- Tailwind CSS Documentation: <https://tailwindcss.com/docs>
- D3.js Tutorials: <https://d3js.org/>

**Backend Development:**

- FastAPI Documentation: <https://fastapi.tiangolo.com/>
- Python Async/Await Guide: <https://docs.python.org/3/library/asyncio.html>

**Testing:**

- Pytest Documentation: <https://docs.pytest.org/>
- Hypothesis Documentation: <https://hypothesis.readthedocs.io/>

### Success Metrics

**MVP Success Criteria (10K Users):**

- ✅ 90%+ user authentication success rate
- ✅ <5 second LeetCode profile analysis time
- ✅ <10 second AI learning path generation
- ✅ 90%+ test coverage
- ✅ <500ms API response time (P95)
- ✅ 99.9% uptime
- ✅ <$175/month operational cost

**Scale Success Criteria (1M Users):**

- ✅ Support 1000+ concurrent users
- ✅ <2 second page load time
- ✅ 99.99% uptime
- ✅ <$6,800/month operational cost
- ✅ 95%+ user satisfaction rate

### Risk Mitigation

**Technical Risks:**

1. **LeetCode Rate Limiting**: Implement exponential backoff and request queuing
2. **AWS Bedrock Costs**: Monitor usage, implement caching, optimize prompts
3. **Cold Start Latency**: Use Lambda provisioned concurrency for critical functions
4. **Data Consistency**: Implement eventual consistency patterns with DynamoDB

**Timeline Risks:**

1. **AI Integration Complexity**: Allocate buffer time (3 weeks → 4 weeks)
2. **Frontend-Backend Integration**: Weekly sync meetings, shared API contracts
3. **Testing Delays**: Start testing early, parallel with development

### Roadmap Visualization

The complete visual roadmap is available in:

- **SVG Format**: `skill-roadmap-flowchart.svg` (scalable, editable)
- **HTML Viewer**: `view-flowchart.html` (interactive, can export to PNG)

To view the roadmap:

1. Open `view-flowchart.html` in any modern web browser
2. Use the "Download as PNG" button to save a high-resolution image
3. Or right-click the image and select "Save image as..."

The roadmap provides a comprehensive visual guide for the entire development journey, showing skill dependencies, parallel tracks, and the convergence to the final success platform.
