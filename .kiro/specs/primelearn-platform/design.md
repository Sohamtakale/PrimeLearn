# Design Document: PrimeLearn Platform

## Overview

PrimeLearn is an AI-powered adaptive learning platform built on AWS serverless architecture. The system uses Bayesian Knowledge Tracing and Item Response Theory to personalize learning paths, AWS Bedrock (Claude models) for dynamic content generation, and a streaming-service UX paradigm to make technology education engaging and efficient.

The platform addresses India's tech education challenges through adaptive learning algorithms, multi-modal content delivery, real-time struggle detection, and India-specific features (Hinglish support, low-bandwidth optimization, placement alignment).

### Design Principles

1. **Adaptive-First**: Every learner gets a personalized path based on their goals, prior knowledge, and real-time performance
2. **Serverless Architecture**: AWS Lambda + DynamoDB + S3 for infinite scalability and cost efficiency
3. **AI-Powered Content**: AWS Bedrock generates Episodes, assessments, and Socratic hints on-demand
4. **Progressive Enhancement**: Text-first rendering with graceful degradation for low-bandwidth users
5. **Data-Driven Decisions**: BKT and IRT algorithms continuously optimize learning paths
6. **India-Optimized**: Hinglish support, university syllabus mapping, placement focus, bandwidth awareness

## Architecture

### High-Level Architecture

The system follows a serverless microservices architecture with clear separation between frontend, API layer, business logic, AI services, and data storage.

```
┌─────────────────────────────────────────────────────────────────┐
│                         Frontend Layer                          │
│  React + Next.js (Vercel/CloudFront)                           │
│  - Season/Episode UI                                            │
│  - Code Editor (Monaco)                                         │
│  - Growth Dashboard (Recharts)                                  │
│  - Skill Visualization (D3.js/Mermaid)                         │
└────────────────────┬────────────────────────────────────────────┘
                     │ HTTPS/REST
┌────────────────────▼────────────────────────────────────────────┐
│                      API Gateway Layer                          │
│  AWS API Gateway                                                │
│  - Authentication (JWT)                                         │
│  - Rate Limiting                                                │
│  - Request Validation                                           │
│  - CORS Policies                                                │
└────────────────────┬────────────────────────────────────────────┘
                     │
        ┌────────────┼────────────┐
        │            │            │
┌───────▼──────┐ ┌──▼──────┐ ┌──▼─────────┐
│   Lambda     │ │ Lambda  │ │  Lambda    │
│   Functions  │ │Functions│ │ Functions  │
│              │ │         │ │            │
│ - Auth       │ │-Content │ │-Assessment │
│ - User Mgmt  │ │ Gen     │ │ Gen        │
│ - Progress   │ │-Adaptive│ │-Analytics  │
│   Tracking   │ │ Engine  │ │-Struggle   │
│              │ │         │ │ Detection  │
└───────┬──────┘ └──┬──────┘ └──┬─────────┘
        │           │           │
        └───────────┼───────────┘
                    │
        ┌───────────┼───────────┐
        │           │           │
┌───────▼──────┐ ┌─▼────────┐ ┌▼──────────┐
│  DynamoDB    │ │AWS Bedrock│ │    S3     │
│              │ │           │ │           │
│ - Users      │ │- Claude   │ │- Generated│
│ - Progress   │ │  Haiku    │ │  Content  │
│ - Seasons    │ │- Claude   │ │- Media    │
│ - Episodes   │ │  Sonnet   │ │  Assets   │
│ - Assessments│ │           │ │           │
└──────────────┘ └───────────┘ └───────────┘
```

### Component Architecture

The system is organized into 8 major components:

1. **Frontend Application**: React/Next.js SPA with streaming-service UX
2. **API Gateway**: Request routing, authentication, rate limiting
3. **Adaptive Engine**: BKT-based skill tracking and path generation
4. **Content Generator**: AWS Bedrock integration for Episode creation
5. **Struggle Detector**: Real-time behavioral analysis and intervention
6. **Assessment Generator**: IRT-based personalized testing
7. **Data Layer**: DynamoDB + S3 for user data and content storage
8. **Analytics Engine**: CloudWatch + custom metrics for insights

## Components and Interfaces

### 1. Frontend Application

**Responsibilities**:
- Render Season/Episode browsing interface
- Display adaptive learning paths with skill visualization
- Provide interactive code editor for Code Labs
- Show Growth Dashboard with analytics
- Handle user authentication and session management

**Key Interfaces**:

```typescript
interface Season {
  id: string;
  title: string;
  description: string;
  skillGraph: SkillNode[];
  totalEpisodes: number;
  estimatedHours: number;
  prerequisites: string[];
  targetAudience: string[];
}

interface Episode {
  id: string;
  seasonId: string;
  nodeId: string;
  title: string;
  format: 'visual_story' | 'code_lab' | 'concept_xray' | 'case_study' | 'quick_byte';
  content: EpisodeContent;
  duration: number;
  learningObjectives: string[];
}

interface SkillNode {
  id: string;
  name: string;
  description: string;
  dependencies: string[];
  masteryProbability: number;
  status: 'locked' | 'available' | 'in_progress' | 'mastered';
}

interface LearnerProgress {
  userId: string;
  seasonId: string;
  completedNodes: string[];
  currentNode: string;
  skillMastery: Map<string, number>;
  totalTimeSpent: number;
  lastAccessed: Date;
}
```

**Technology Stack**:
- React 18 with hooks for state management
- Next.js 14 for SSR and routing
- Monaco Editor for code editing
- D3.js for skill graph visualization
- Recharts for analytics charts
- TailwindCSS for styling
- Deployed on Vercel or AWS CloudFront + S3

### 2. API Gateway Layer

**Responsibilities**:
- Route requests to appropriate Lambda functions
- Validate JWT tokens for authentication
- Enforce rate limits (100 requests/minute per user)
- Apply CORS policies for frontend access
- Log all requests for analytics

**API Endpoints**:

```
Authentication:
POST   /api/auth/register
POST   /api/auth/login
POST   /api/auth/reset-password
GET    /api/auth/verify-email

User Management:
GET    /api/users/{userId}
PUT    /api/users/{userId}
GET    /api/users/{userId}/preferences

Seasons & Episodes:
GET    /api/seasons
GET    /api/seasons/{seasonId}
GET    /api/seasons/{seasonId}/episodes
GET    /api/episodes/{episodeId}
POST   /api/episodes/{episodeId}/complete

Progress Tracking:
GET    /api/progress/{userId}/{seasonId}
PUT    /api/progress/{userId}/{seasonId}
GET    /api/progress/{userId}/dashboard

Adaptive Engine:
POST   /api/adaptive/initialize-season
POST   /api/adaptive/update-mastery
GET    /api/adaptive/next-episode
POST   /api/adaptive/detect-gap

Content Generation:
POST   /api/content/generate-episode
POST   /api/content/generate-bridge-sprint
POST   /api/content/generate-hint

Assessment:
POST   /api/assessment/generate-finale
POST   /api/assessment/submit-answer
GET    /api/assessment/{assessmentId}/results

Socratic Mentor:
POST   /api/mentor/get-hint
POST   /api/mentor/chat

Analytics:
GET    /api/analytics/user-stats
GET    /api/analytics/platform-metrics
```

**Technology Stack**:
- AWS API Gateway (REST API)
- Lambda Authorizer for JWT validation
- Request/Response transformations
- CloudWatch for logging

### 3. Adaptive Engine

**Responsibilities**:
- Generate personalized learning paths using topological sort
- Track skill mastery using Bayesian Knowledge Tracing
- Detect knowledge gaps and trigger Bridge Sprints
- Update learner state in real-time
- Recommend next Episodes based on current mastery

**Core Algorithms**:

**Bayesian Knowledge Tracing (BKT)**:
```
Parameters:
- P(L0): Prior probability of knowing skill (default: 0.1)
- P(T): Probability of learning (default: 0.3)
- P(S): Probability of slip (default: 0.1)
- P(G): Probability of guess (default: 0.2)

Update Formula:
P(L|correct) = P(L) * (1 - P(S)) / [P(L) * (1 - P(S)) + (1 - P(L)) * P(G)]
P(L|incorrect) = P(L) * P(S) / [P(L) * P(S) + (1 - P(L)) * (1 - P(G))]
P(L_next) = P(L) + (1 - P(L)) * P(T)
```

**Topological Sort for Learning Paths**:
```
Algorithm: Kahn's Algorithm
Input: Skill DAG with dependencies
Output: Valid learning sequence

1. Calculate in-degree for all nodes
2. Add nodes with in-degree 0 to queue
3. While queue not empty:
   - Dequeue node
   - Add to learning path
   - Reduce in-degree of dependent nodes
   - Add nodes with in-degree 0 to queue
4. If all nodes processed, return path
5. Else, cycle detected (error)
```

**Gap Detection**:
```
Algorithm: BFS Prerequisite Search
Input: Current node, learner mastery state
Output: Missing prerequisite node

1. If learner fails node after 2 attempts:
2. BFS traverse prerequisites
3. Find nearest node with mastery < 0.6
4. Generate Bridge Sprint for that node
5. Insert into learning path
```

**Interfaces**:

```typescript
interface AdaptiveEngine {
  initializeSeason(userId: string, seasonId: string): LearningPath;
  updateMastery(userId: string, nodeId: string, performance: Performance): void;
  getNextEpisode(userId: string, seasonId: string): Episode | null;
  detectGap(userId: string, nodeId: string): string | null;
  generateBridgeSprint(userId: string, prerequisiteId: string): Episode[];
}

interface LearningPath {
  sequence: string[];
  currentIndex: number;
  unlockedNodes: string[];
  masteryState: Map<string, number>;
}

interface Performance {
  correct: boolean;
  timeSpent: number;
  hintsUsed: number;
  attempts: number;
}
```

**Technology Stack**:
- AWS Lambda (Node.js or Python)
- DynamoDB for state storage
- Custom BKT implementation
- Graph algorithms library

### 4. Content Generator

**Responsibilities**:
- Generate Episodes in 5 formats using AWS Bedrock
- Create Bridge Sprint content for prerequisite gaps
- Generate Season Finale assessment questions
- Produce Socratic hints for struggling learners
- Support Hinglish content generation

**Content Generation Pipeline**:

```
1. Receive generation request (node, format, language)
2. Load node metadata and learning objectives
3. Construct prompt for AWS Bedrock
4. Call Claude Haiku (fast) or Sonnet (complex)
5. Parse and validate generated content
6. Store in S3 with metadata
7. Cache reference in DynamoDB
8. Return content to requester
```

**Prompt Templates**:

```typescript
interface PromptTemplate {
  format: EpisodeFormat;
  systemPrompt: string;
  userPrompt: string;
  temperature: number;
  maxTokens: number;
}

// Example: Visual Story Template
const visualStoryTemplate: PromptTemplate = {
  format: 'visual_story',
  systemPrompt: `You are an expert educator creating engaging narrative-driven 
                 explanations for technology concepts. Use analogies, stories, 
                 and visual descriptions.`,
  userPrompt: `Create a Visual Story Episode for: {concept}
               Learning Objectives: {objectives}
               Target Audience: {audience}
               Language: {language}
               
               Include:
               1. Hook (engaging opening)
               2. Narrative explanation with analogies
               3. Visual diagram descriptions
               4. Key takeaways
               5. Transition to next concept`,
  temperature: 0.7,
  maxTokens: 2000
};

// Example: Code Lab Template
const codeLabTemplate: PromptTemplate = {
  format: 'code_lab',
  systemPrompt: `You are an expert programming instructor creating hands-on 
                 coding exercises with step-by-step guidance.`,
  userPrompt: `Create a Code Lab Episode for: {concept}
               Language: {programmingLanguage}
               Difficulty: {difficulty}
               
               Include:
               1. Learning objectives
               2. Starter code
               3. Step-by-step instructions (3-5 steps)
               4. Test cases
               5. Expected output
               6. Common mistakes to avoid`,
  temperature: 0.3,
  maxTokens: 1500
};
```

**Interfaces**:

```typescript
interface ContentGenerator {
  generateEpisode(nodeId: string, format: EpisodeFormat, language: string): Episode;
  generateBridgeSprint(prerequisiteId: string, language: string): Episode[];
  generateAssessmentQuestion(nodeId: string, difficulty: number): Question;
  generateHint(context: StruggleContext, level: number): string;
}

interface EpisodeContent {
  format: EpisodeFormat;
  sections: ContentSection[];
  codeExamples?: CodeExample[];
  diagrams?: DiagramDescription[];
  exercises?: Exercise[];
}

interface StruggleContext {
  nodeId: string;
  exerciseCode: string;
  errorMessage?: string;
  attemptCount: number;
  timeSpent: number;
}
```

**Technology Stack**:
- AWS Bedrock (Claude 3 Haiku for speed, Sonnet for complexity)
- AWS Lambda for orchestration
- S3 for content storage
- DynamoDB for content metadata and caching

### 5. Struggle Detector

**Responsibilities**:
- Monitor learner behavior in real-time
- Detect struggle signals (time, errors, patterns)
- Trigger Socratic Mentor interventions
- Log struggle events for analytics
- Adapt detection thresholds based on learner history

**Detection Rules**:

```typescript
interface StruggleDetector {
  monitorBehavior(userId: string, episodeId: string, events: BehaviorEvent[]): StruggleSignal | null;
  shouldIntervene(signal: StruggleSignal): boolean;
  getInterventionLevel(signal: StruggleSignal): number;
}

interface BehaviorEvent {
  type: 'code_edit' | 'code_run' | 'error' | 'hint_request' | 'idle';
  timestamp: Date;
  data: any;
}

interface StruggleSignal {
  userId: string;
  episodeId: string;
  indicators: {
    timeWithoutProgress: number;  // seconds
    incorrectAttempts: number;
    errorFrequency: number;
    idleTime: number;
    codeChurn: number;  // lines changed without progress
  };
  severity: 'low' | 'medium' | 'high';
  confidence: number;
}

// Detection Rules
const struggleRules = {
  timeThreshold: 180,  // 3 minutes without progress
  attemptThreshold: 3,  // 3 incorrect attempts
  errorThreshold: 5,    // 5 errors in 2 minutes
  idleThreshold: 120,   // 2 minutes idle
  churnThreshold: 50    // 50 lines changed without progress
};
```

**Intervention Strategy**:

```
Level 1 (Low Severity):
- Subtle UI hint: "Need help? Click for a hint"
- Wait 60 seconds before escalating

Level 2 (Medium Severity):
- Socratic question: "What are you trying to accomplish?"
- Conceptual hint without code
- Wait 90 seconds before escalating

Level 3 (High Severity):
- Specific guidance: "Let's break this down..."
- Code scaffolding or partial solution
- Offer to generate Bridge Sprint if prerequisite gap detected
```

**Technology Stack**:
- AWS Lambda for real-time processing
- DynamoDB Streams for event processing
- CloudWatch for logging struggle events

### 6. Assessment Generator

**Responsibilities**:
- Generate Season Finale assessments using IRT
- Create personalized questions targeting weak areas
- Calibrate question difficulty to learner level
- Score assessments and update BKT estimates
- Provide detailed feedback on performance

**Item Response Theory (IRT)**:

```
3-Parameter Logistic Model:
P(correct) = c + (1 - c) / (1 + e^(-a(θ - b)))

Where:
- θ (theta): Learner ability
- a: Question discrimination (how well it separates abilities)
- b: Question difficulty
- c: Guessing parameter (typically 0.2 for multiple choice)

Question Selection Algorithm:
1. Estimate learner ability θ from BKT mastery scores
2. Select questions where difficulty b ≈ θ (maximum information)
3. Prioritize nodes with mastery < 0.90
4. Include mix of question types
5. Generate 10-15 questions total
```

**Interfaces**:

```typescript
interface AssessmentGenerator {
  generateFinale(userId: string, seasonId: string): Assessment;
  selectQuestions(masteryState: Map<string, number>, count: number): Question[];
  scoreAssessment(assessmentId: string, answers: Answer[]): AssessmentResult;
  updateMasteryFromAssessment(userId: string, result: AssessmentResult): void;
}

interface Assessment {
  id: string;
  userId: string;
  seasonId: string;
  questions: Question[];
  createdAt: Date;
  timeLimit: number;
}

interface Question {
  id: string;
  nodeId: string;
  type: 'multiple_choice' | 'code_completion' | 'debugging' | 'short_answer';
  difficulty: number;  // IRT b parameter
  discrimination: number;  // IRT a parameter
  content: string;
  options?: string[];
  correctAnswer: string;
  explanation: string;
}

interface AssessmentResult {
  assessmentId: string;
  score: number;
  correctCount: number;
  totalQuestions: number;
  timeSpent: number;
  nodeScores: Map<string, number>;
  strengths: string[];
  weaknesses: string[];
  recommendations: string[];
}
```

**Technology Stack**:
- AWS Lambda for generation and scoring
- AWS Bedrock for question content generation
- DynamoDB for assessment storage
- Custom IRT implementation

### 7. Data Layer

**Responsibilities**:
- Store user profiles and authentication data
- Track learner progress and mastery state
- Store Season and Episode metadata
- Cache generated content
- Provide fast read/write access for real-time updates

**DynamoDB Schema Design**:

```typescript
// Users Table
interface UserRecord {
  PK: string;  // USER#{userId}
  SK: string;  // PROFILE
  email: string;
  passwordHash: string;
  name: string;
  university?: string;
  preferences: {
    language: 'english' | 'hinglish';
    darkMode: boolean;
  };
  createdAt: string;
  lastLogin: string;
}

// Progress Table
interface ProgressRecord {
  PK: string;  // USER#{userId}
  SK: string;  // SEASON#{seasonId}
  completedNodes: string[];
  currentNode: string;
  masteryState: Record<string, number>;  // nodeId -> probability
  totalTimeSpent: number;
  lastAccessed: string;
  GSI1PK: string;  // SEASON#{seasonId}
  GSI1SK: string;  // USER#{userId}
}

// Seasons Table
interface SeasonRecord {
  PK: string;  // SEASON#{seasonId}
  SK: string;  // METADATA
  title: string;
  description: string;
  skillGraph: SkillNode[];
  totalEpisodes: number;
  estimatedHours: number;
  prerequisites: string[];
  targetAudience: string[];
  createdAt: string;
}

// Episodes Table
interface EpisodeRecord {
  PK: string;  // EPISODE#{episodeId}
  SK: string;  // METADATA
  seasonId: string;
  nodeId: string;
  title: string;
  format: EpisodeFormat;
  contentS3Key: string;  // Reference to S3
  duration: number;
  learningObjectives: string[];
  GSI1PK: string;  // SEASON#{seasonId}
  GSI1SK: string;  // NODE#{nodeId}
}

// Assessments Table
interface AssessmentRecord {
  PK: string;  // ASSESSMENT#{assessmentId}
  SK: string;  // METADATA
  userId: string;
  seasonId: string;
  questions: Question[];
  answers?: Answer[];
  result?: AssessmentResult;
  status: 'in_progress' | 'completed';
  createdAt: string;
  completedAt?: string;
  GSI1PK: string;  // USER#{userId}
  GSI1SK: string;  // ASSESSMENT#{createdAt}
}

// Analytics Table
interface AnalyticsRecord {
  PK: string;  // ANALYTICS#{date}
  SK: string;  // METRIC#{metricName}
  value: number;
  metadata: Record<string, any>;
  timestamp: string;
}
```

**Access Patterns**:

```
1. Get user profile: Query(PK=USER#{userId}, SK=PROFILE)
2. Get user progress for season: Query(PK=USER#{userId}, SK=SEASON#{seasonId})
3. Get all seasons: Scan(PK begins_with SEASON#)
4. Get episodes for season: Query(GSI1PK=SEASON#{seasonId})
5. Get user assessments: Query(GSI1PK=USER#{userId}, GSI1SK begins_with ASSESSMENT#)
6. Get daily analytics: Query(PK=ANALYTICS#{date})
```

**S3 Bucket Structure**:

```
primelearn-content/
├── episodes/
│   ├── {episodeId}/
│   │   ├── content.json
│   │   ├── diagrams/
│   │   └── code-examples/
├── assessments/
│   ├── {assessmentId}/
│   │   └── questions.json
├── media/
│   ├── images/
│   └── audio/
└── cache/
    └── generated-content/
```

**Technology Stack**:
- DynamoDB with on-demand billing
- S3 with versioning enabled
- DynamoDB Streams for real-time updates
- S3 lifecycle policies for cost optimization

### 8. Analytics Engine

**Responsibilities**:
- Collect user interaction events
- Track platform performance metrics
- Generate engagement reports
- Monitor success KPIs
- Alert on anomalies

**Key Metrics**:

```typescript
interface PlatformMetrics {
  activeUsers: {
    daily: number;
    weekly: number;
    monthly: number;
  };
  engagement: {
    avgSessionDuration: number;
    avgWeeklyLearningTime: number;
    completionRate: number;
  };
  learning: {
    avgTimeReduction: number;  // vs linear courses
    struggleResolutionRate: number;
    bridgeSprintEffectiveness: number;
    masteryRetention30Days: number;
  };
  technical: {
    apiLatencyP95: number;
    errorRate: number;
    bedrockCost: number;
    costPerActiveUser: number;
  };
  geographic: {
    metroVsNonMetro: { metro: number; nonMetro: number };
    topCities: Array<{ city: string; count: number }>;
  };
}
```

**Technology Stack**:
- AWS CloudWatch for logs and metrics
- CloudWatch Dashboards for visualization
- Lambda for custom metric aggregation
- SNS for alerting

## Data Models

### Core Domain Models

```typescript
// User Domain
interface User {
  id: string;
  email: string;
  name: string;
  university?: string;
  preferences: UserPreferences;
  createdAt: Date;
  lastLogin: Date;
}

interface UserPreferences {
  language: 'english' | 'hinglish';
  darkMode: boolean;
  notificationsEnabled: boolean;
}

// Learning Domain
interface Season {
  id: string;
  title: string;
  description: string;
  skillGraph: SkillGraph;
  metadata: SeasonMetadata;
}

interface SkillGraph {
  nodes: SkillNode[];
  edges: SkillEdge[];
}

interface SkillNode {
  id: string;
  name: string;
  description: string;
  type: 'concept' | 'skill' | 'project';
  estimatedMinutes: number;
}

interface SkillEdge {
  from: string;  // prerequisite node
  to: string;    // dependent node
  strength: 'required' | 'recommended';
}

interface SeasonMetadata {
  totalEpisodes: number;
  estimatedHours: number;
  difficulty: 'beginner' | 'intermediate' | 'advanced';
  prerequisites: string[];
  targetAudience: string[];
  tags: string[];
}

interface Episode {
  id: string;
  seasonId: string;
  nodeId: string;
  title: string;
  format: EpisodeFormat;
  content: EpisodeContent;
  metadata: EpisodeMetadata;
}

type EpisodeFormat = 'visual_story' | 'code_lab' | 'concept_xray' | 'case_study' | 'quick_byte';

interface EpisodeMetadata {
  duration: number;
  learningObjectives: string[];
  difficulty: number;
  prerequisites: string[];
  dataBandwidth: number;  // estimated MB
}

// Progress Domain
interface LearnerProgress {
  userId: string;
  seasonId: string;
  state: ProgressState;
  history: ProgressEvent[];
}

interface ProgressState {
  completedNodes: Set<string>;
  currentNode: string;
  masteryScores: Map<string, MasteryScore>;
  totalTimeSpent: number;
  lastAccessed: Date;
}

interface MasteryScore {
  nodeId: string;
  probability: number;  // BKT estimate
  confidence: number;
  lastUpdated: Date;
  decayRate: number;
}

interface ProgressEvent {
  type: 'episode_start' | 'episode_complete' | 'struggle_detected' | 'hint_used' | 'bridge_sprint';
  timestamp: Date;
  nodeId: string;
  metadata: Record<string, any>;
}

// Assessment Domain
interface Assessment {
  id: string;
  userId: string;
  seasonId: string;
  type: 'season_finale' | 'bridge_sprint' | 'practice';
  questions: Question[];
  status: AssessmentStatus;
  result?: AssessmentResult;
  createdAt: Date;
  completedAt?: Date;
}

type AssessmentStatus = 'not_started' | 'in_progress' | 'completed' | 'expired';

interface Question {
  id: string;
  nodeId: string;
  type: QuestionType;
  difficulty: number;
  discrimination: number;
  content: QuestionContent;
  correctAnswer: string;
  explanation: string;
}

type QuestionType = 'multiple_choice' | 'code_completion' | 'debugging' | 'short_answer' | 'true_false';

interface QuestionContent {
  prompt: string;
  options?: string[];
  codeSnippet?: string;
  language?: string;
}

interface Answer {
  questionId: string;
  userAnswer: string;
  isCorrect: boolean;
  timeSpent: number;
  timestamp: Date;
}

// Struggle Detection Domain
interface StruggleEvent {
  id: string;
  userId: string;
  episodeId: string;
  nodeId: string;
  indicators: StruggleIndicators;
  severity: 'low' | 'medium' | 'high';
  interventionProvided: Intervention;
  resolved: boolean;
  timestamp: Date;
}

interface StruggleIndicators {
  timeWithoutProgress: number;
  incorrectAttempts: number;
  errorCount: number;
  idleTime: number;
  codeChurn: number;
  hintsRequested: number;
}

interface Intervention {
  type: 'hint' | 'bridge_sprint' | 'mentor_chat';
  level: number;  // 1-3
  content: string;
  effective: boolean;
}

// Content Generation Domain
interface ContentGenerationRequest {
  nodeId: string;
  format: EpisodeFormat;
  language: 'english' | 'hinglish';
  targetAudience: string;
  difficulty: number;
  context?: string;
}

interface GeneratedContent {
  requestId: string;
  content: EpisodeContent;
  model: string;  // e.g., "claude-3-haiku"
  tokensUsed: number;
  generationTime: number;
  s3Key: string;
  createdAt: Date;
}
```

### Data Relationships

```
User 1:N LearnerProgress (one user, many seasons)
Season 1:N Episode (one season, many episodes)
Season 1:1 SkillGraph (one season, one graph)
SkillNode 1:N Episode (one node, many episodes)
User 1:N Assessment (one user, many assessments)
Assessment N:M Question (many assessments, many questions)
Episode 1:N StruggleEvent (one episode, many struggles)
User 1:N StruggleEvent (one user, many struggles)
```


### Data Flow Diagrams

#### Episode Completion Flow

```
1. Learner completes Episode
   ↓
2. Frontend sends completion event to API Gateway
   ↓
3. Lambda: Progress Tracker
   - Update completion status
   - Calculate performance metrics
   ↓
4. Lambda: Adaptive Engine
   - Update BKT mastery score
   - Check if node mastered (P > 0.85)
   - Unlock dependent nodes
   ↓
5. Lambda: Struggle Detector (if applicable)
   - Analyze completion time
   - Check error patterns
   - Trigger intervention if needed
   ↓
6. DynamoDB: Update progress record
   ↓
7. Response: Next recommended Episode
```

#### Struggle Detection and Intervention Flow

```
1. Learner interacts with Code Lab
   ↓
2. Frontend streams behavior events
   ↓
3. Lambda: Struggle Detector
   - Accumulate events
   - Apply detection rules
   - Calculate severity
   ↓
4. If struggle detected:
   ↓
5. Lambda: Socratic Mentor
   - Load struggle context
   - Generate appropriate hint
   - Call AWS Bedrock (Sonnet)
   ↓
6. Frontend: Display intervention
   - Show hint/question
   - Offer Bridge Sprint if gap detected
   ↓
7. DynamoDB: Log struggle event
   ↓
8. CloudWatch: Track metrics
```

#### Content Generation Flow

```
1. Learner reaches new node
   ↓
2. Check DynamoDB cache for existing content
   ↓
3. If not cached:
   ↓
4. Lambda: Content Generator
   - Load node metadata
   - Select appropriate format
   - Construct Bedrock prompt
   ↓
5. AWS Bedrock: Generate content
   - Claude Haiku (fast, cheap)
   - Claude Sonnet (complex)
   ↓
6. Lambda: Validate and format
   - Parse response
   - Add metadata
   - Generate S3 key
   ↓
7. S3: Store content
   ↓
8. DynamoDB: Cache reference
   ↓
9. Response: Return Episode to frontend
```

#### Season Finale Assessment Flow

```
1. Learner completes all Episodes
   ↓
2. Frontend requests Season Finale
   ↓
3. Lambda: Assessment Generator
   - Load learner mastery state
   - Identify weak nodes (P < 0.90)
   - Estimate learner ability θ
   ↓
4. IRT Question Selection
   - Select questions where b ≈ θ
   - Prioritize weak nodes
   - Mix question types
   ↓
5. AWS Bedrock: Generate questions
   - Create 10-15 questions
   - Include explanations
   ↓
6. DynamoDB: Store assessment
   ↓
7. Frontend: Display assessment
   ↓
8. Learner submits answers
   ↓
9. Lambda: Score assessment
   - Calculate score
   - Update BKT estimates
   - Generate feedback
   ↓
10. DynamoDB: Store results
    ↓
11. Response: Show results and recommendations
```

## API Design

### Authentication Endpoints

```
POST /api/auth/register
Request:
{
  "email": "string",
  "password": "string",
  "name": "string",
  "university": "string?"
}
Response: 201 Created
{
  "userId": "string",
  "token": "string",
  "expiresAt": "ISO8601"
}

POST /api/auth/login
Request:
{
  "email": "string",
  "password": "string"
}
Response: 200 OK
{
  "userId": "string",
  "token": "string",
  "expiresAt": "ISO8601",
  "user": User
}

POST /api/auth/reset-password
Request:
{
  "email": "string"
}
Response: 200 OK
{
  "message": "Reset email sent"
}
```

### Season and Episode Endpoints

```
GET /api/seasons
Query Params:
  - audience: string?
  - difficulty: string?
  - tag: string?
Response: 200 OK
{
  "seasons": Season[],
  "total": number
}

GET /api/seasons/{seasonId}
Response: 200 OK
{
  "season": Season,
  "userProgress": LearnerProgress?
}

GET /api/seasons/{seasonId}/episodes
Response: 200 OK
{
  "episodes": Episode[],
  "skillGraph": SkillGraph,
  "unlockedNodes": string[]
}

GET /api/episodes/{episodeId}
Response: 200 OK
{
  "episode": Episode,
  "content": EpisodeContent,
  "previousEpisode": string?,
  "nextEpisode": string?
}

POST /api/episodes/{episodeId}/complete
Request:
{
  "timeSpent": number,
  "performance": {
    "correct": boolean,
    "attempts": number,
    "hintsUsed": number
  }
}
Response: 200 OK
{
  "masteryUpdated": boolean,
  "newMastery": number,
  "nodeCompleted": boolean,
  "unlockedNodes": string[],
  "nextRecommendation": Episode?
}
```

### Progress Endpoints

```
GET /api/progress/{userId}/{seasonId}
Response: 200 OK
{
  "progress": LearnerProgress,
  "completionPercentage": number,
  "estimatedTimeRemaining": number
}

PUT /api/progress/{userId}/{seasonId}
Request:
{
  "currentNode": "string",
  "completedNodes": string[],
  "masteryScores": Record<string, number>
}
Response: 200 OK
{
  "updated": boolean
}

GET /api/progress/{userId}/dashboard
Response: 200 OK
{
  "activeSeasons": Season[],
  "completedSeasons": Season[],
  "totalTimeSpent": number,
  "currentStreak": number,
  "skillRadar": Record<string, number>,
  "recentActivity": ProgressEvent[],
  "decayAlerts": MasteryScore[]
}
```

### Adaptive Engine Endpoints

```
POST /api/adaptive/initialize-season
Request:
{
  "userId": "string",
  "seasonId": "string",
  "priorKnowledge": string[]?
}
Response: 200 OK
{
  "learningPath": string[],
  "unlockedNodes": string[],
  "estimatedHours": number
}

POST /api/adaptive/update-mastery
Request:
{
  "userId": "string",
  "nodeId": "string",
  "performance": Performance
}
Response: 200 OK
{
  "newMastery": number,
  "nodeCompleted": boolean,
  "unlockedNodes": string[]
}

GET /api/adaptive/next-episode
Query Params:
  - userId: string
  - seasonId: string
Response: 200 OK
{
  "episode": Episode,
  "reasoning": string
}

POST /api/adaptive/detect-gap
Request:
{
  "userId": "string",
  "nodeId": "string",
  "failureCount": number
}
Response: 200 OK
{
  "gapDetected": boolean,
  "prerequisiteNode": string?,
  "bridgeSprint": Episode[]?
}
```

### Content Generation Endpoints

```
POST /api/content/generate-episode
Request:
{
  "nodeId": "string",
  "format": EpisodeFormat,
  "language": "english" | "hinglish",
  "difficulty": number
}
Response: 200 OK
{
  "episode": Episode,
  "generationTime": number,
  "cached": boolean
}

POST /api/content/generate-bridge-sprint
Request:
{
  "userId": "string",
  "prerequisiteId": "string",
  "language": "english" | "hinglish"
}
Response: 200 OK
{
  "episodes": Episode[],
  "estimatedMinutes": number
}

POST /api/content/generate-hint
Request:
{
  "userId": "string",
  "episodeId": "string",
  "context": StruggleContext,
  "level": number
}
Response: 200 OK
{
  "hint": string,
  "type": "conceptual" | "specific" | "scaffolding"
}
```

### Assessment Endpoints

```
POST /api/assessment/generate-finale
Request:
{
  "userId": "string",
  "seasonId": "string"
}
Response: 200 OK
{
  "assessment": Assessment,
  "timeLimit": number
}

POST /api/assessment/submit-answer
Request:
{
  "assessmentId": "string",
  "questionId": "string",
  "answer": string,
  "timeSpent": number
}
Response: 200 OK
{
  "correct": boolean,
  "explanation": string,
  "masteryUpdated": boolean
}

GET /api/assessment/{assessmentId}/results
Response: 200 OK
{
  "result": AssessmentResult,
  "recommendations": string[],
  "reviewEpisodes": Episode[]
}
```

### Socratic Mentor Endpoints

```
POST /api/mentor/get-hint
Request:
{
  "userId": "string",
  "episodeId": "string",
  "context": StruggleContext
}
Response: 200 OK
{
  "hint": string,
  "level": number,
  "followUpQuestion": string?
}

POST /api/mentor/chat
Request:
{
  "userId": "string",
  "episodeId": "string",
  "message": string,
  "conversationHistory": Message[]
}
Response: 200 OK
{
  "response": string,
  "suggestedAction": string?
}
```

### Analytics Endpoints

```
GET /api/analytics/user-stats
Query Params:
  - userId: string
  - period: "week" | "month" | "all"
Response: 200 OK
{
  "totalTimeSpent": number,
  "episodesCompleted": number,
  "averageSessionDuration": number,
  "currentStreak": number,
  "masteryGrowth": Array<{date: string, avgMastery: number}>,
  "struggleRate": number
}

GET /api/analytics/platform-metrics
Query Params:
  - startDate: ISO8601
  - endDate: ISO8601
Response: 200 OK
{
  "metrics": PlatformMetrics
}
```

## Tech Stack Justification

### Frontend: React + Next.js

**Why React?**
- Component-based architecture perfect for reusable Episode formats
- Large ecosystem for code editors (Monaco), charts (Recharts), visualizations (D3.js)
- Strong TypeScript support for type safety
- Excellent performance with virtual DOM

**Why Next.js?**
- Server-side rendering for better SEO and initial load times
- API routes for backend-for-frontend pattern
- Built-in routing and code splitting
- Easy deployment to Vercel or AWS CloudFront
- Image optimization out of the box

**Alternatives Considered**:
- Vue.js: Smaller ecosystem for specialized components
- Angular: Too heavy for hackathon timeline
- Svelte: Less mature ecosystem

### Backend: AWS Lambda + API Gateway

**Why Serverless?**
- Zero infrastructure management (critical for hackathon)
- Auto-scaling from 0 to 10,000+ users
- Pay-per-use pricing (cost-effective for MVP)
- Fast deployment and iteration
- Built-in high availability

**Why Lambda?**
- Sub-second cold starts with Node.js/Python
- Integrates seamlessly with DynamoDB, S3, Bedrock
- Event-driven architecture for real-time features
- 15-minute timeout sufficient for content generation

**Alternatives Considered**:
- EC2: Requires management, over-provisioning
- ECS/Fargate: More complex, slower deployment
- App Runner: Less control over scaling

### Database: DynamoDB

**Why DynamoDB?**
- Single-digit millisecond latency at any scale
- Serverless with on-demand billing
- Flexible schema for evolving data models
- DynamoDB Streams for real-time analytics
- Global tables for future multi-region expansion
- Strong consistency for critical operations

**Schema Design Rationale**:
- Single-table design for related entities (User + Progress)
- GSI for reverse lookups (Season → Users)
- Composite sort keys for hierarchical data
- Sparse indexes for optional attributes

**Alternatives Considered**:
- RDS PostgreSQL: Requires provisioning, less scalable
- MongoDB Atlas: Additional vendor, higher cost
- Aurora Serverless: More expensive, overkill for MVP

### Storage: S3

**Why S3?**
- Unlimited storage for generated content
- 99.999999999% durability
- Versioning for content history
- Lifecycle policies for cost optimization
- CloudFront integration for CDN
- Event notifications for processing

**Alternatives Considered**:
- EFS: More expensive, unnecessary for static content
- DynamoDB: 400KB item limit too restrictive

### AI: AWS Bedrock (Claude 3)

**Why Bedrock?**
- Managed service (no infrastructure)
- Multiple models (Haiku for speed, Sonnet for quality)
- Pay-per-token pricing
- Built-in content filtering
- AWS integration (IAM, CloudWatch)

**Why Claude 3?**
- Haiku: 0.25¢/1K tokens, sub-second latency
- Sonnet: 3¢/1K tokens, superior reasoning
- 200K context window for complex prompts
- Strong instruction following
- Good at educational content

**Cost Estimate**:
- Episode generation: ~2K tokens = ₹0.50
- Hint generation: ~500 tokens = ₹0.10
- Assessment questions: ~1K tokens = ₹0.25
- Per user per month: ~₹2-3 for AI

**Alternatives Considered**:
- OpenAI GPT-4: More expensive, external vendor
- Self-hosted LLM: Requires GPU infrastructure
- Cohere: Smaller context window

### Code Execution: Monaco Editor + Sandboxed Runtime

**Why Monaco?**
- Same editor as VS Code
- Excellent syntax highlighting
- IntelliSense and autocomplete
- Lightweight and fast
- TypeScript/JavaScript native

**Sandboxing Strategy**:
- Python: Pyodide (WebAssembly in browser)
- JavaScript: Web Workers with restricted APIs
- SQL: sql.js (SQLite in WebAssembly)

**Why Client-Side Execution?**
- Zero backend cost for code runs
- Instant feedback (no network latency)
- Scales infinitely
- Secure (isolated from backend)

**Alternatives Considered**:
- Server-side execution: Security risks, cost, latency
- Third-party APIs (Judge0): External dependency, cost

### Monitoring: CloudWatch

**Why CloudWatch?**
- Native AWS integration
- Unified logs and metrics
- Custom dashboards
- Alarms and notifications
- Log Insights for querying

**Alternatives Considered**:
- Datadog: Expensive for MVP
- New Relic: Overkill for hackathon
- Self-hosted: Too much overhead

## Security Architecture

### Authentication and Authorization

**JWT-Based Authentication**:
```
1. User logs in with email/password
2. Lambda validates credentials
3. Generate JWT with claims:
   - userId
   - email
   - roles
   - exp (24 hour expiration)
4. Sign with secret key (AWS Secrets Manager)
5. Return token to client
6. Client includes token in Authorization header
7. API Gateway Lambda Authorizer validates token
8. Allow/deny request based on validation
```

**Password Security**:
- Hash with bcrypt (cost factor 12)
- Salt automatically included
- Store only hash in DynamoDB
- Never log passwords

**Session Management**:
- 24-hour token expiration
- Refresh token for extended sessions
- Logout invalidates token (blacklist in DynamoDB)
- Concurrent session limit: 3 devices

### API Security

**Rate Limiting**:
- 100 requests/minute per user
- 1000 requests/minute per IP
- Exponential backoff for violations
- Implemented in API Gateway

**Input Validation**:
- JSON schema validation at API Gateway
- Sanitize all user inputs
- Parameterized queries (no SQL injection)
- Content-Type validation

**CORS Policy**:
```json
{
  "allowedOrigins": ["https://primelearn.app"],
  "allowedMethods": ["GET", "POST", "PUT", "DELETE"],
  "allowedHeaders": ["Content-Type", "Authorization"],
  "maxAge": 3600
}
```

### Data Security

**Encryption**:
- At rest: DynamoDB encryption with AWS KMS
- In transit: TLS 1.3 for all API calls
- S3: Server-side encryption (SSE-S3)
- Secrets: AWS Secrets Manager

**Access Control**:
- IAM roles for Lambda functions (least privilege)
- Resource-based policies for S3 buckets
- VPC endpoints for private communication
- No public database access

**Data Privacy**:
- PII encrypted in DynamoDB
- User data isolated by userId
- No cross-user data leakage
- GDPR-compliant data deletion

### Code Execution Security

**Sandbox Constraints**:
- No network access from code
- No file system access
- 5-second execution timeout
- Memory limit: 50MB
- No eval() or dangerous APIs

**Content Security Policy**:
```
Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'unsafe-eval' (for Monaco);
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  connect-src 'self' https://api.primelearn.app;
```

### AWS Bedrock Security

**Prompt Injection Prevention**:
- Separate system and user prompts
- Input sanitization
- Output validation
- Content filtering enabled

**Cost Protection**:
- Token limits per request
- Daily spending limits
- CloudWatch alarms for anomalies
- Rate limiting on generation endpoints

## Scalability Approach

### Horizontal Scaling

**Serverless Auto-Scaling**:
- Lambda: Automatic concurrency scaling
- DynamoDB: On-demand capacity mode
- API Gateway: Unlimited requests
- S3: Infinite storage

**Scaling Targets**:
- Year 1: 10,000 active users
- Year 2: 50,000 active users
- Year 3: 200,000 active users

### Performance Optimization

**Caching Strategy**:
```
L1: Browser Cache (Service Worker)
  - Completed Episodes: 7 days
  - Static assets: 30 days

L2: CloudFront CDN
  - S3 content: 24 hours
  - API responses: 5 minutes (for public data)

L3: DynamoDB Cache
  - Generated content metadata: 30 days
  - Season/Episode metadata: Until updated

L4: Lambda Memory Cache
  - Frequently accessed data: Function lifetime
```

**Database Optimization**:
- Single-table design reduces joins
- GSI for efficient queries
- Batch operations for bulk updates
- DynamoDB Accelerator (DAX) if needed

**Content Delivery**:
- CloudFront for global distribution
- S3 Transfer Acceleration for uploads
- Lazy loading for images
- Progressive enhancement for low bandwidth

### Cost Optimization

**Estimated Monthly Costs (10,000 active users)**:

```
AWS Lambda:
- 10M requests/month
- 512MB memory, 1s avg duration
- Cost: $20

DynamoDB:
- 50M read units
- 10M write units
- 10GB storage
- Cost: $30

S3:
- 100GB storage
- 1TB transfer
- Cost: $25

AWS Bedrock:
- 50M tokens/month (5K per user)
- Cost: $125 (Haiku) + $150 (Sonnet) = $275

API Gateway:
- 10M requests
- Cost: $35

CloudWatch:
- Logs and metrics
- Cost: $15

Total: ~$400/month = ₹4/user/month ✓
```

**Cost Reduction Strategies**:
- Cache aggressively to reduce Bedrock calls
- Use Haiku for simple content (10x cheaper)
- Compress S3 objects
- Reserved capacity if usage predictable
- Lifecycle policies for old content

### Monitoring and Alerting

**Key Metrics to Track**:
- API latency (P50, P95, P99)
- Error rates by endpoint
- Lambda cold starts
- DynamoDB throttling
- Bedrock token usage
- Active user count
- Cost per active user

**Alerts**:
- Error rate > 5%: Page on-call
- API latency P95 > 2s: Warning
- Daily cost > $50: Alert
- Bedrock errors > 10/hour: Warning
- DynamoDB throttling: Auto-scale

## Deployment Architecture

### CI/CD Pipeline

```
1. Developer pushes to GitHub
   ↓
2. GitHub Actions triggered
   ↓
3. Run tests (unit + integration)
   ↓
4. Build frontend (Next.js)
   ↓
5. Deploy to staging:
   - Lambda functions (SAM/CDK)
   - Frontend to Vercel preview
   - Run smoke tests
   ↓
6. Manual approval for production
   ↓
7. Deploy to production:
   - Blue-green deployment
   - Gradual traffic shift (10% → 50% → 100%)
   - Rollback if errors spike
   ↓
8. Post-deployment validation
```

### Infrastructure as Code

**AWS CDK (TypeScript)**:
```typescript
// Example stack definition
export class PrimeLearnStack extends Stack {
  constructor(scope: Construct, id: string) {
    super(scope, id);

    // DynamoDB Tables
    const usersTable = new Table(this, 'Users', {
      partitionKey: { name: 'PK', type: AttributeType.STRING },
      sortKey: { name: 'SK', type: AttributeType.STRING },
      billingMode: BillingMode.PAY_PER_REQUEST,
      encryption: TableEncryption.AWS_MANAGED,
      pointInTimeRecovery: true,
    });

    // Lambda Functions
    const adaptiveEngine = new NodejsFunction(this, 'AdaptiveEngine', {
      entry: 'src/lambda/adaptive-engine.ts',
      handler: 'handler',
      runtime: Runtime.NODEJS_20_X,
      timeout: Duration.seconds(30),
      memorySize: 512,
      environment: {
        USERS_TABLE: usersTable.tableName,
      },
    });

    // API Gateway
    const api = new RestApi(this, 'PrimeLearnAPI', {
      restApiName: 'PrimeLearn API',
      deployOptions: {
        stageName: 'prod',
        throttlingRateLimit: 1000,
        throttlingBurstLimit: 2000,
      },
    });

    // Grant permissions
    usersTable.grantReadWriteData(adaptiveEngine);
  }
}
```

### Environment Strategy

**Three Environments**:

1. **Development** (local)
   - LocalStack for AWS services
   - Mock Bedrock responses
   - SQLite for DynamoDB
   - Fast iteration

2. **Staging** (AWS)
   - Full AWS stack
   - Separate accounts/resources
   - Real Bedrock (limited quota)
   - Integration testing

3. **Production** (AWS)
   - Multi-AZ deployment
   - CloudFront CDN
   - Full monitoring
   - Backup and disaster recovery

### Disaster Recovery

**Backup Strategy**:
- DynamoDB: Point-in-time recovery (35 days)
- S3: Versioning enabled
- Daily snapshots to separate region
- RTO: 1 hour
- RPO: 5 minutes

**Failure Scenarios**:

1. **Lambda failure**: Automatic retry (3 attempts)
2. **DynamoDB throttling**: Exponential backoff
3. **Bedrock unavailable**: Serve cached content
4. **Region outage**: Failover to backup region (manual)
5. **Data corruption**: Restore from point-in-time backup

## Future Improvements

### Phase 2 (Post-Hackathon)

1. **ML-Based Struggle Detection**
   - Train models on user interaction data
   - Predict struggle before it happens
   - Personalized intervention strategies

2. **Automated Knowledge Graph Generation**
   - Extract skill dependencies from content
   - Use LLMs to build skill graphs
   - Community-contributed graphs

3. **Advanced Gamification**
   - Badges and achievements
   - Leaderboards (opt-in)
   - Learning streaks and challenges
   - Social features (study groups)

4. **Mobile Native Apps**
   - iOS and Android apps
   - Offline-first architecture
   - Push notifications for reminders
   - Better mobile code editing

5. **Video-Based Episodes**
   - AI-generated video explanations
   - Interactive video with branching
   - Automatic transcription and translation

### Phase 3 (Scale)

1. **Instructor Platform**
   - Content creation tools
   - Custom Season builder
   - Analytics dashboard for instructors
   - Revenue sharing model

2. **Enterprise Features**
   - Team management
   - Custom learning paths
   - SSO integration
   - Compliance reporting

3. **Advanced Personalization**
   - Learning style detection
   - Optimal time-of-day recommendations
   - Peer matching for collaboration
   - Career path alignment

4. **Multi-Language Support**
   - Pure Hindi, Tamil, Telugu, Bengali
   - Regional language content
   - Voice-based learning for low-literacy users

5. **Placement Integration**
   - Direct company partnerships
   - Interview scheduling
   - Resume building
   - Mock interviews with AI

### Technical Debt to Address

1. **BKT Calibration**: Collect data to tune parameters
2. **IRT Item Bank**: Build validated question database
3. **Performance Testing**: Load test with 10K concurrent users
4. **Security Audit**: Third-party penetration testing
5. **Accessibility**: WCAG 2.1 AA compliance
6. **Internationalization**: i18n framework for all text
7. **Error Handling**: Comprehensive error taxonomy
8. **Documentation**: API docs, architecture diagrams, runbooks


## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: DAG Validity for Skill Constellations

*For any* Season created by the Adaptive Engine, the Skill_Constellation must be a valid directed acyclic graph with no cycles, proper node structure (id, name, dependencies), and at least one node with zero in-degree (starting point).

**Validates: Requirements 1.1, 1.7, 1.8**

### Property 2: Topological Sort Respects Dependencies

*For any* valid Skill_Constellation DAG, the topological sort output must produce an ordering where every node appears after all its prerequisites (for every edge from A to B, A appears before B in the sequence).

**Validates: Requirements 1.2**

### Property 3: Mastery Unlocks Dependent Nodes

*For any* Knowledge_Node that transitions to mastered status (probability > 0.85), all nodes that depend exclusively on mastered nodes should transition from "locked" to "available" status.

**Validates: Requirements 1.3**

### Property 4: Prior Knowledge Filtering

*For any* set of prior knowledge nodes provided at Season initialization, the generated learning path should exclude those nodes and start with the first non-mastered node in the topological order.

**Validates: Requirements 1.4**

### Property 5: Mastery Threshold Classification

*For any* Knowledge_Node with mastery probability exceeding 0.85, the node status must be marked as "mastered"; for any node with probability at or below 0.85, the status must not be "mastered".

**Validates: Requirements 1.6**

### Property 6: Episode Format Selection Rules

*For any* content generation request, the selected Episode format must match the content type: procedural skills → Code_Lab, conceptual topics → Visual_Story or Concept_X_Ray, real-world applications → Case_Study, quick reference → Quick_Byte with duration < 180 seconds.

**Validates: Requirements 2.2, 2.3, 2.4, 2.5**

### Property 7: Episode Content Completeness

*For any* generated Episode, the content structure must include all required elements for its format: learning objectives, explanations, and format-specific components (code examples for Code_Lab, diagrams for Visual_Story, test cases for exercises).

**Validates: Requirements 2.6, 2.7, 2.8, 2.9, 2.10, 9.4**

### Property 8: Episode Completion Persistence

*For any* Episode marked as complete by a learner, the database must contain a record with completion timestamp, time spent, and interaction patterns within 1 second of completion.

**Validates: Requirements 2.11**

### Property 9: Struggle Detection Triggers

*For any* Code_Lab session where time without progress exceeds 180 seconds OR incorrect attempts reach 3 or more, the Struggle_Detector must flag the session and activate the Socratic_Mentor.

**Validates: Requirements 3.2, 3.3, 3.4**

### Property 10: Socratic Mentor Response Time

*For any* explicit help request from a learner, the Socratic_Mentor must return a response within 2000 milliseconds.

**Validates: Requirements 3.10**

### Property 11: Mentor Interaction Logging

*For any* Socratic_Mentor interaction (hint request, chat message), a log entry must be created in the analytics system with timestamp, user ID, episode ID, and interaction content.

**Validates: Requirements 3.11**

### Property 12: Gap Detection After Failures

*For any* Knowledge_Node where a learner fails to progress after 2 attempts, the Prerequisite_Intelligence must analyze prerequisite dependencies and either identify a gap or confirm no prerequisite issues exist.

**Validates: Requirements 4.2**

### Property 13: Bridge Sprint Generation and Insertion

*For any* detected prerequisite gap, the Platform must generate a Bridge_Sprint containing 2-4 Episodes and insert it into the learner's current learning path before the blocked node.

**Validates: Requirements 4.3, 4.4, 4.5**

### Property 14: Bridge Sprint Completion Triggers Reassessment

*For any* completed Bridge_Sprint, the Platform must reassess the learner's readiness for the originally blocked Knowledge_Node by re-evaluating mastery scores for the prerequisite.

**Validates: Requirements 4.7**

### Property 15: Season Completion Triggers Finale

*For any* learner who completes all Episodes in a Season (all nodes in completed state), the Platform must automatically trigger the Season_Finale assessment generation.

**Validates: Requirements 5.1**

### Property 16: Finale Targets Weak Areas

*For any* Season_Finale assessment, at least 70% of questions must target Knowledge_Nodes with mastery probability below 0.90, prioritizing nodes where the learner struggled or required Bridge_Sprints.

**Validates: Requirements 5.2, 5.5**

### Property 17: Finale Question Count and Difficulty

*For any* generated Season_Finale, the assessment must contain between 10 and 15 questions (inclusive), with question difficulty calibrated to within 0.5 of the learner's estimated ability level (theta).

**Validates: Requirements 5.4**

### Property 18: Finale Question Type Diversity

*For any* Season_Finale assessment, the questions must include at least 3 different question types from the set: multiple_choice, code_completion, debugging, short_answer.

**Validates: Requirements 5.6**

### Property 19: Real-Time BKT Updates During Assessment

*For any* question answered during an assessment, the Platform must update the BKT mastery estimate for the associated Knowledge_Node before presenting the next question.

**Validates: Requirements 5.7**

### Property 20: Finale Completion Generates Feedback

*For any* completed Season_Finale, the Platform must generate a results object containing score, node-level performance, identified strengths (nodes with score > 80%), weaknesses (nodes with score < 60%), and recommendations.

**Validates: Requirements 5.8**

### Property 21: Low Score Triggers Review Recommendations

*For any* Season_Finale with overall score below 70%, the Platform must recommend at least 3 targeted review Episodes focusing on the weakest Knowledge_Nodes.

**Validates: Requirements 5.10**

### Property 22: Knowledge Decay Alerts

*For any* Knowledge_Node where mastery probability decays below 0.70 (from previously mastered state), the Growth_Dashboard must display a decay alert with the node name and recommended review action.

**Validates: Requirements 6.5**

### Property 23: Real-Time Dashboard Updates

*For any* Episode completion event, the Growth_Dashboard metrics (completion percentage, mastery scores, time spent) must update within 2 seconds without requiring page refresh.

**Validates: Requirements 6.9**

### Property 24: Hinglish Content Code-Mixing

*For any* content generated with language mode set to "hinglish", the output text must contain both Hindi (Devanagari or romanized) and English words, with natural code-mixing patterns.

**Validates: Requirements 7.2**

### Property 25: University Syllabus Filtering

*For any* learner who selects a university affiliation, the Season browsing interface must highlight or filter Seasons that map to that university's curriculum.

**Validates: Requirements 7.4**

### Property 26: Network Degradation Handling

*For any* detected poor network condition (bandwidth < 1 Mbps or high latency), the Platform must gracefully degrade to text-only mode by disabling image loading and interactive visualizations.

**Validates: Requirements 7.9**

### Property 27: Offline Episode Caching

*For any* Episode marked as completed, the Episode content must be stored in the browser's local cache for offline access.

**Validates: Requirements 7.10**

### Property 28: Episode Data Usage Metadata

*For any* Episode, the metadata must include an estimated data usage value (in MB) calculated from content size, images, and interactive elements.

**Validates: Requirements 7.12**

### Property 29: Multi-Language Code Execution

*For any* valid code snippet in Python, JavaScript, or SQL, the Code_Lab sandbox must execute the code and return results (output, errors, execution time) within the 5-second timeout limit.

**Validates: Requirements 8.3, 8.4, 8.5, 8.6, 8.7**

### Property 30: Test Case Validation for Completion

*For any* Code_Lab exercise with defined test cases, the exercise must only be marked as complete when the learner's code passes all test cases (100% pass rate).

**Validates: Requirements 8.9**

### Property 31: Auto-Save Code Persistence

*For any* code editor session, the learner's code must be automatically saved to the database at intervals not exceeding 10 seconds, ensuring no more than 10 seconds of work can be lost.

**Validates: Requirements 8.10**

### Property 32: Code Reset Restores Starter Code

*For any* Code_Lab exercise, invoking the reset function must restore the exact starter code provided in the Episode definition, discarding all learner modifications.

**Validates: Requirements 8.11**

### Property 33: On-Demand Content Generation

*For any* learner reaching a new Knowledge_Node without cached content, the Platform must trigger content generation and return a complete Episode within 15 seconds.

**Validates: Requirements 9.7**

### Property 34: Consistent Episode Template Structure

*For any* two Episodes of the same format, the content structure must follow the same template schema (same top-level sections, same field types).

**Validates: Requirements 9.8**

### Property 35: Email Validation for Registration

*For any* registration attempt with an invalid email format (missing @, invalid domain, etc.), the Platform must reject the registration and return a validation error.

**Validates: Requirements 10.2**

### Property 36: Password Hashing Before Storage

*For any* user registration or password change, the password must be hashed using bcrypt (or equivalent) before storage, and the plaintext password must never be stored in the database.

**Validates: Requirements 10.3**

### Property 37: User Profile Completeness

*For any* created user account, the user record in the database must contain all required fields: userId, email, passwordHash, name, preferences object, createdAt timestamp.

**Validates: Requirements 10.6**

### Property 38: Cross-Device Progress Synchronization

*For any* progress update made on one device, the same progress state must be retrievable from a different device within 5 seconds (eventual consistency).

**Validates: Requirements 10.7**

### Property 39: Login Restores User State

*For any* successful login, the API response must include the user's active Seasons, current progress state for each Season, and last accessed timestamps.

**Validates: Requirements 10.8**

### Property 40: Comprehensive Interaction Logging

*For any* user interaction (Episode view, completion, hint request, assessment submission), a log entry must be created with timestamp, userId, action type, and relevant metadata.

**Validates: Requirements 11.1, 11.2, 11.4, 11.6, 11.7**

### Property 41: Daily Analytics Report Generation

*For any* calendar day, the Platform must generate a daily analytics report by 00:30 UTC containing user engagement metrics, content effectiveness scores, and system performance data.

**Validates: Requirements 11.5**

### Property 42: Learning Time Reduction Measurement

*For any* completed Season, the Platform must calculate the learner's total time spent and compare it to a baseline linear course duration, storing the percentage reduction in the analytics database.

**Validates: Requirements 11.8**

### Property 43: Placement Outcome Tracking

*For any* learner who completes a placement-focused Season, the Platform must track whether they report a successful placement outcome within 90 days.

**Validates: Requirements 11.9**

### Property 44: Performance Threshold Alerts

*For any* 5-minute window where API error rate exceeds 5% OR P95 latency exceeds 2000ms, the Platform must send an alert notification to administrators.

**Validates: Requirements 11.10**

### Property 45: API Rate Limiting Enforcement

*For any* user making more than 100 requests per minute, the API Gateway must return 429 (Too Many Requests) status and reject subsequent requests until the rate drops below the threshold.

**Validates: Requirements 12.6**

### Property 46: API Response Time SLA

*For any* 100 consecutive API requests under normal load, at least 95 of them must return responses within 500 milliseconds (P95 < 500ms).

**Validates: Requirements 12.7**

### Property 47: Concurrent User Handling

*For any* load test with 1000 simultaneous users making requests, the Platform must successfully process all requests without errors or timeouts.

**Validates: Requirements 12.8**

### Property 48: Structured Error Responses

*For any* API error (4xx or 5xx status), the response body must include a structured error object with fields: errorCode, message, timestamp, and optionally details.

**Validates: Requirements 12.9**

### Property 49: CORS Header Presence

*For any* API response to a cross-origin request from the frontend domain, the response must include appropriate CORS headers (Access-Control-Allow-Origin, Access-Control-Allow-Methods).

**Validates: Requirements 12.11**

### Property 50: Responsive UI Rendering

*For any* screen width between 320px and 2560px, the UI must render without horizontal scrolling, with all interactive elements accessible and readable.

**Validates: Requirements 13.3**

### Property 51: Accessibility Compliance

*For any* interactive UI element, keyboard navigation must be supported (tab order, enter/space activation), and screen reader labels must be present for all non-text elements.

**Validates: Requirements 13.7**

### Property 52: Initial Page Load Performance

*For any* initial page load on a simulated 3G connection (750 Kbps), the first contentful paint must occur within 2000 milliseconds.

**Validates: Requirements 13.10**

## Error Handling

### Error Categories

The platform implements a comprehensive error taxonomy with specific handling strategies:

**1. User Input Errors (4xx)**
- Invalid email format → Return validation error with specific field
- Weak password → Return requirements (min length, complexity)
- Duplicate email → Return "Email already registered"
- Invalid credentials → Return "Invalid email or password" (no user enumeration)
- Missing required fields → Return list of missing fields

**2. Authentication Errors (401, 403)**
- Expired token → Return "Session expired, please login"
- Invalid token → Return "Invalid authentication token"
- Insufficient permissions → Return "Access denied"
- Rate limit exceeded → Return "Too many requests, try again in X seconds"

**3. Resource Errors (404, 409)**
- Season not found → Return "Season does not exist"
- Episode not found → Return "Episode does not exist"
- User not found → Return "User does not exist"
- Conflict (concurrent update) → Return "Resource modified, please refresh"

**4. System Errors (500, 503)**
- DynamoDB throttling → Exponential backoff, retry 3 times
- S3 unavailable → Serve cached content if available
- Bedrock API error → Fallback to cached content or generic response
- Lambda timeout → Return "Request timeout, please try again"
- Out of memory → Log error, return "Service temporarily unavailable"

**5. External Service Errors**
- Bedrock rate limit → Queue request, retry with backoff
- Bedrock content filter → Return "Content generation failed, trying alternative"
- Network timeout → Retry with exponential backoff (max 3 attempts)

### Error Recovery Strategies

**Graceful Degradation**:
```
1. Content Generation Failure:
   - Try Haiku model first
   - If fails, try Sonnet
   - If both fail, serve cached similar content
   - If no cache, serve generic template
   - Log error for manual review

2. Database Unavailability:
   - Serve stale data from cache (if < 5 minutes old)
   - Queue writes for later processing
   - Display "Some features temporarily unavailable"
   - Retry in background

3. Code Execution Failure:
   - Catch sandbox errors
   - Display error message to learner
   - Offer to reset code
   - Log error for debugging
```

**Retry Logic**:
```typescript
async function retryWithBackoff<T>(
  operation: () => Promise<T>,
  maxRetries: number = 3,
  baseDelay: number = 1000
): Promise<T> {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      if (attempt === maxRetries - 1) throw error;
      
      const delay = baseDelay * Math.pow(2, attempt);
      await sleep(delay);
    }
  }
  throw new Error('Max retries exceeded');
}
```

**Circuit Breaker Pattern**:
```typescript
class CircuitBreaker {
  private failureCount = 0;
  private lastFailureTime = 0;
  private state: 'closed' | 'open' | 'half-open' = 'closed';
  
  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.state === 'open') {
      if (Date.now() - this.lastFailureTime > 60000) {
        this.state = 'half-open';
      } else {
        throw new Error('Circuit breaker open');
      }
    }
    
    try {
      const result = await operation();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  private onSuccess() {
    this.failureCount = 0;
    this.state = 'closed';
  }
  
  private onFailure() {
    this.failureCount++;
    this.lastFailureTime = Date.now();
    
    if (this.failureCount >= 5) {
      this.state = 'open';
    }
  }
}
```

### Error Logging and Monitoring

**Structured Logging**:
```typescript
interface ErrorLog {
  timestamp: string;
  level: 'error' | 'warning' | 'critical';
  errorCode: string;
  message: string;
  userId?: string;
  requestId: string;
  service: string;
  stackTrace?: string;
  context: Record<string, any>;
}
```

**Alert Thresholds**:
- Error rate > 5% for 5 minutes → Page on-call
- Bedrock errors > 10/minute → Warning
- Database throttling → Auto-scale alert
- Lambda cold starts > 1000ms → Warning
- Any 500 error → Log to Slack channel

## Testing Strategy

### Dual Testing Approach

The PrimeLearn platform requires both unit testing and property-based testing for comprehensive coverage. These approaches are complementary:

- **Unit tests** validate specific examples, edge cases, and integration points
- **Property tests** verify universal properties across all inputs through randomization

### Unit Testing

**Focus Areas**:
1. **Specific Examples**: Concrete test cases demonstrating correct behavior
2. **Edge Cases**: Boundary conditions, empty inputs, maximum values
3. **Error Conditions**: Invalid inputs, missing data, constraint violations
4. **Integration Points**: Component interactions, API contracts, database operations

**Example Unit Tests**:
```typescript
describe('Adaptive Engine', () => {
  it('should generate valid DAG for "Learn React" season', () => {
    const season = adaptiveEngine.initializeSeason('user123', 'react-basics');
    expect(season.skillGraph.nodes).toHaveLength(12);
    expect(hasNoCycles(season.skillGraph)).toBe(true);
  });
  
  it('should handle empty prior knowledge', () => {
    const path = adaptiveEngine.generatePath('user123', 'python-basics', []);
    expect(path.sequence[0]).toBe('variables-and-types');
  });
  
  it('should reject invalid email formats', () => {
    expect(() => validateEmail('notanemail')).toThrow('Invalid email');
    expect(() => validateEmail('missing@domain')).toThrow('Invalid email');
  });
});
```

**Unit Test Coverage Targets**:
- Core business logic: 90%
- API endpoints: 85%
- Utility functions: 95%
- Error handling: 80%

### Property-Based Testing

**Configuration**:
- Minimum 100 iterations per property test (due to randomization)
- Each test must reference its design document property
- Tag format: `Feature: primelearn-platform, Property {number}: {property_text}`

**Property Test Framework**:
- **JavaScript/TypeScript**: fast-check
- **Python**: Hypothesis

**Example Property Tests**:

```typescript
import fc from 'fast-check';

describe('Property Tests: Adaptive Engine', () => {
  it('Property 1: DAG Validity for Skill Constellations', () => {
    // Feature: primelearn-platform, Property 1: DAG Validity
    fc.assert(
      fc.property(
        fc.array(fc.record({
          id: fc.string(),
          name: fc.string(),
          dependencies: fc.array(fc.string())
        }), { minLength: 1, maxLength: 20 }),
        (nodes) => {
          const season = adaptiveEngine.createSeason(nodes);
          const graph = season.skillGraph;
          
          // Must be valid DAG
          expect(hasNoCycles(graph)).toBe(true);
          // Must have at least one starting node
          expect(graph.nodes.some(n => n.dependencies.length === 0)).toBe(true);
          // All nodes must have required fields
          graph.nodes.forEach(node => {
            expect(node).toHaveProperty('id');
            expect(node).toHaveProperty('name');
            expect(node).toHaveProperty('dependencies');
          });
        }
      ),
      { numRuns: 100 }
    );
  });
  
  it('Property 2: Topological Sort Respects Dependencies', () => {
    // Feature: primelearn-platform, Property 2: Topological Sort
    fc.assert(
      fc.property(
        generateValidDAG(),
        (dag) => {
          const sorted = topologicalSort(dag);
          
          // For every edge A -> B, A must appear before B
          dag.edges.forEach(edge => {
            const indexA = sorted.indexOf(edge.from);
            const indexB = sorted.indexOf(edge.to);
            expect(indexA).toBeLessThan(indexB);
          });
        }
      ),
      { numRuns: 100 }
    );
  });
  
  it('Property 29: Multi-Language Code Execution', () => {
    // Feature: primelearn-platform, Property 29: Code Execution
    fc.assert(
      fc.property(
        fc.oneof(
          generateValidPythonCode(),
          generateValidJavaScriptCode(),
          generateValidSQLCode()
        ),
        async (code) => {
          const result = await sandbox.execute(code);
          
          // Must return within 5 seconds
          expect(result.executionTime).toBeLessThan(5000);
          // Must include output, errors, and time
          expect(result).toHaveProperty('output');
          expect(result).toHaveProperty('errors');
          expect(result).toHaveProperty('executionTime');
        }
      ),
      { numRuns: 100 }
    );
  });
});
```

**Custom Generators**:
```typescript
// Generator for valid DAGs
function generateValidDAG() {
  return fc.record({
    nodes: fc.array(fc.string(), { minLength: 1, maxLength: 10 }),
    edges: fc.array(fc.tuple(fc.nat(), fc.nat()))
  }).map(({ nodes, edges }) => {
    // Ensure edges only reference valid nodes and create no cycles
    const validEdges = edges
      .filter(([from, to]) => from < nodes.length && to < nodes.length && from !== to)
      .filter(([from, to]) => from < to); // Ensures no cycles in simple case
    
    return {
      nodes: nodes.map((name, id) => ({ id: String(id), name, dependencies: [] })),
      edges: validEdges.map(([from, to]) => ({ from: String(from), to: String(to) }))
    };
  });
}

// Generator for valid Python code
function generateValidPythonCode() {
  return fc.oneof(
    fc.constant('print("Hello, World!")'),
    fc.tuple(fc.integer(), fc.integer()).map(([a, b]) => `print(${a} + ${b})`),
    fc.array(fc.integer()).map(arr => `print(sum([${arr.join(', ')}]))`),
    fc.string().map(s => `print(len("${s}"))`)
  );
}
```

### Integration Testing

**API Integration Tests**:
```typescript
describe('API Integration Tests', () => {
  it('should complete full learning flow', async () => {
    // Register user
    const user = await api.post('/auth/register', {
      email: 'test@example.com',
      password: 'SecurePass123!',
      name: 'Test User'
    });
    
    // Initialize season
    const season = await api.post('/adaptive/initialize-season', {
      userId: user.userId,
      seasonId: 'python-basics'
    });
    
    // Get first episode
    const episode = await api.get('/adaptive/next-episode', {
      params: { userId: user.userId, seasonId: 'python-basics' }
    });
    
    // Complete episode
    const completion = await api.post(`/episodes/${episode.id}/complete`, {
      timeSpent: 300,
      performance: { correct: true, attempts: 1, hintsUsed: 0 }
    });
    
    expect(completion.masteryUpdated).toBe(true);
    expect(completion.unlockedNodes.length).toBeGreaterThan(0);
  });
});
```

### Performance Testing

**Load Testing with Artillery**:
```yaml
config:
  target: 'https://api.primelearn.app'
  phases:
    - duration: 60
      arrivalRate: 10
      name: "Warm up"
    - duration: 300
      arrivalRate: 100
      name: "Sustained load"
    - duration: 60
      arrivalRate: 200
      name: "Spike test"
      
scenarios:
  - name: "Complete learning flow"
    flow:
      - post:
          url: "/auth/login"
          json:
            email: "{{ $randomEmail() }}"
            password: "TestPass123!"
      - get:
          url: "/seasons"
      - get:
          url: "/adaptive/next-episode"
          qs:
            userId: "{{ userId }}"
            seasonId: "python-basics"
      - think: 30
      - post:
          url: "/episodes/{{ episodeId }}/complete"
          json:
            timeSpent: 300
            performance:
              correct: true
              attempts: 1
```

### Test Execution Strategy

**CI/CD Pipeline**:
```
1. On Pull Request:
   - Run unit tests (must pass)
   - Run property tests (100 iterations)
   - Run linting and type checking
   - Estimated time: 5 minutes

2. On Merge to Main:
   - Run full test suite
   - Run integration tests against staging
   - Run property tests (1000 iterations)
   - Deploy to staging
   - Run smoke tests
   - Estimated time: 15 minutes

3. Before Production Deploy:
   - Run load tests (1000 concurrent users)
   - Run security scans
   - Manual approval required
   - Estimated time: 30 minutes
```

**Test Data Management**:
- Use factories for generating test data
- Seed staging database with realistic data
- Anonymize production data for testing
- Clean up test data after each run

### Monitoring and Observability in Tests

**Test Metrics to Track**:
- Test execution time trends
- Flaky test identification
- Property test failure rates
- Coverage trends over time
- Performance regression detection

**Property Test Failure Analysis**:
When a property test fails, the framework provides:
- Minimal failing example (shrinking)
- Seed for reproduction
- Full input that caused failure
- Stack trace and assertion details

Example failure output:
```
Property 29: Multi-Language Code Execution
Failed after 47 iterations
Counterexample: {
  language: "python",
  code: "while True: pass"
}
Error: Execution timeout exceeded 5000ms
Seed: 1234567890
```
