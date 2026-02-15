# Requirements Document: PrimeLearn Platform

## Introduction

PrimeLearn is an AI-powered adaptive learning platform for technology education in India, designed to address the critical gaps in current tech learning approaches. The platform uses a streaming-service UX paradigm (Seasons and Episodes) combined with adaptive learning algorithms to provide personalized, goal-oriented learning paths. Built for the AWS AI for Bharat Hackathon, PrimeLearn targets India's massive tech education market with features optimized for local needs including Hinglish support, low-bandwidth optimization, and placement-focused learning.

## Glossary

- **Platform**: The PrimeLearn adaptive learning system
- **Learner**: A user consuming educational content on the platform
- **Season**: A goal-oriented learning path (e.g., "Master React", "Backend with Node.js")
- **Episode**: An individual learning unit within a Season (5-15 min duration)
- **Skill_Constellation**: A directed acyclic graph (DAG) representing skill dependencies
- **Knowledge_Node**: A single skill or concept in the Skill_Constellation
- **Adaptive_Engine**: The system component that personalizes learning paths
- **Socratic_Mentor**: The AI-powered conversational assistant that provides progressive hints
- **Struggle_Detector**: The system that monitors learner behavior to identify confusion
- **Bridge_Sprint**: An auto-generated prerequisite module to fill knowledge gaps
- **Season_Finale**: The adaptive assessment at the end of a Season
- **BKT**: Bayesian Knowledge Tracing algorithm for skill mastery estimation
- **IRT**: Item Response Theory for question difficulty calibration
- **Hinglish**: Code-mixed Hindi-English language mode
- **Content_Generator**: AWS Bedrock-powered system for creating learning content
- **Assessment_Generator**: System for creating personalized assessments
- **Growth_Dashboard**: Learner-facing analytics and progress visualization
- **Prerequisite_Intelligence**: System for detecting and filling knowledge gaps
- **Multi_Modal_Episode**: Learning content in one of 5 formats (Visual Story, Code Lab, Concept X-Ray, Case Study, Quick Byte)

## User Personas

### Persona 1: Engineering Student (Tier 2/3 College)
- **Demographics**: 18-22 years old, studying in non-IIT/NIT colleges
- **Population**: ~1.2M students per year
- **Pain Points**:
  - Outdated college curriculum not aligned with industry needs
  - Limited access to quality learning resources
  - Passive lecture-based learning with no personalization
  - Placement preparation requires self-learning
  - Limited bandwidth and expensive data plans
- **Goals**: Build industry-relevant skills, prepare for placements, land first tech job

### Persona 2: Self-Taught Developer
- **Demographics**: 20-30 years old, learning programming independently
- **Population**: ~5M+ active learners
- **Pain Points**:
  - Overwhelmed by unstructured content (YouTube, blogs, courses)
  - No clear learning path or skill progression
  - Difficulty identifying knowledge gaps
  - Lack of personalized guidance
  - Time wasted on irrelevant content
- **Goals**: Build portfolio projects, transition to tech career, learn systematically

### Persona 3: Career Switcher
- **Demographics**: 25-35 years old, non-tech professionals transitioning to tech
- **Population**: ~2M+ annually
- **Pain Points**:
  - Limited time due to current job commitments
  - No prior programming background
  - Difficulty understanding prerequisites
  - One-size-fits-all courses too fast or too slow
  - Expensive bootcamps not affordable
- **Goals**: Learn efficiently, build job-ready skills, minimize learning time

### Persona 4: Working Professional (Upskilling)
- **Demographics**: 25-40 years old, tech professionals learning new skills
- **Population**: ~3M+ active
- **Pain Points**:
  - Limited time for learning
  - Need to learn specific skills quickly
  - Existing knowledge not leveraged
  - Passive video courses ineffective
  - Knowledge decay over time
- **Goals**: Stay relevant, learn new frameworks/tools, advance career

## Requirements

### Requirement 1: Adaptive Skill Constellation Engine

**User Story:** As a learner, I want the platform to create personalized learning paths based on my goals and current knowledge, so that I can learn efficiently without wasting time on content I already know or skipping prerequisites I need.

#### Acceptance Criteria

1. WHEN a learner selects a learning goal, THE Adaptive_Engine SHALL generate a Season with a directed acyclic graph of Knowledge_Nodes representing the skill progression
2. WHEN the Adaptive_Engine generates a Skill_Constellation, THE Platform SHALL use topological sort to determine valid learning sequences
3. WHEN a learner demonstrates mastery of a Knowledge_Node, THE Adaptive_Engine SHALL update the learner's skill state and unlock dependent nodes
4. WHEN a learner starts a Season, THE Adaptive_Engine SHALL assess prior knowledge and skip already-mastered nodes
5. THE Adaptive_Engine SHALL use Bayesian Knowledge Tracing to estimate mastery probability for each Knowledge_Node
6. WHEN a learner's mastery probability for a node exceeds 0.85, THE Platform SHALL mark that node as mastered
7. THE Platform SHALL store the Skill_Constellation as a DAG with nodes containing skill metadata and edges representing dependencies
8. WHEN generating learning paths, THE Adaptive_Engine SHALL ensure no circular dependencies exist in the Skill_Constellation

### Requirement 2: Multi-Modal Adaptive Episodes

**User Story:** As a learner, I want content delivered in different formats based on the concept type and my learning preferences, so that I can understand concepts more effectively than through passive video watching.

#### Acceptance Criteria

1. THE Platform SHALL support five Episode formats: Visual_Story, Code_Lab, Concept_X_Ray, Case_Study, and Quick_Byte
2. WHEN creating an Episode for a procedural skill, THE Content_Generator SHALL prioritize Code_Lab format
3. WHEN creating an Episode for a conceptual topic, THE Content_Generator SHALL prioritize Visual_Story or Concept_X_Ray format
4. WHEN creating an Episode for real-world application, THE Content_Generator SHALL prioritize Case_Study format
5. WHEN creating an Episode for quick reference, THE Content_Generator SHALL use Quick_Byte format with duration under 3 minutes
6. THE Platform SHALL generate Visual_Story Episodes with narrative-driven explanations and visual diagrams
7. THE Platform SHALL generate Code_Lab Episodes with interactive coding environments and step-by-step exercises
8. THE Platform SHALL generate Concept_X_Ray Episodes with deep-dive explanations and multiple perspectives
9. THE Platform SHALL generate Case_Study Episodes with real-world scenarios and problem-solving exercises
10. THE Platform SHALL generate Quick_Byte Episodes with concise summaries and key takeaways
11. WHEN a learner completes an Episode, THE Platform SHALL record completion time and interaction patterns
12. THE Platform SHALL use AWS Bedrock with Claude Haiku for content generation

### Requirement 3: Struggle Detection and Socratic Mentor

**User Story:** As a learner, I want the platform to detect when I'm struggling and provide progressive hints through conversational guidance, so that I can overcome obstacles without getting frustrated or giving up.

#### Acceptance Criteria

1. THE Struggle_Detector SHALL monitor learner behavior including time spent, incorrect attempts, and interaction patterns
2. WHEN a learner spends more than 3 minutes without progress on a Code_Lab exercise, THE Struggle_Detector SHALL flag potential struggle
3. WHEN a learner makes 3 or more incorrect attempts on an exercise, THE Struggle_Detector SHALL flag potential struggle
4. WHEN a learner is flagged as struggling, THE Platform SHALL activate the Socratic_Mentor
5. THE Socratic_Mentor SHALL provide hints in a progressive cascade starting with conceptual guidance
6. WHEN the first hint is insufficient, THE Socratic_Mentor SHALL provide more specific guidance without revealing the solution
7. WHEN multiple hints are insufficient, THE Socratic_Mentor SHALL provide code scaffolding or partial solutions
8. THE Socratic_Mentor SHALL use conversational language and ask guiding questions rather than providing direct answers
9. THE Socratic_Mentor SHALL use AWS Bedrock with Claude Sonnet for conversational interactions
10. WHEN a learner requests help explicitly, THE Socratic_Mentor SHALL respond within 2 seconds
11. THE Platform SHALL log all Socratic_Mentor interactions for learning analytics

### Requirement 4: Prerequisite Intelligence and Bridge Sprints

**User Story:** As a learner, I want the platform to automatically detect when I'm missing prerequisite knowledge and provide targeted mini-modules to fill those gaps, so that I don't get stuck or frustrated by concepts I don't understand.

#### Acceptance Criteria

1. THE Prerequisite_Intelligence SHALL analyze learner performance to detect knowledge gaps
2. WHEN a learner fails to progress on a Knowledge_Node after 2 attempts, THE Prerequisite_Intelligence SHALL analyze prerequisite dependencies
3. WHEN a prerequisite gap is detected, THE Platform SHALL generate a Bridge_Sprint targeting the missing knowledge
4. THE Bridge_Sprint SHALL contain 2-4 focused Episodes covering the prerequisite concept
5. WHEN a Bridge_Sprint is generated, THE Platform SHALL insert it into the learner's current learning path
6. THE Platform SHALL use BFS traversal to identify the nearest prerequisite node causing the gap
7. WHEN a learner completes a Bridge_Sprint, THE Platform SHALL reassess readiness for the original Knowledge_Node
8. THE Content_Generator SHALL create Bridge_Sprint content using AWS Bedrock
9. THE Platform SHALL track Bridge_Sprint effectiveness and adjust gap detection thresholds accordingly

### Requirement 5: Season Finale - Adaptive Assessment Generator

**User Story:** As a learner, I want personalized assessments that focus on my weak areas rather than testing everything equally, so that I can efficiently validate my learning and identify remaining gaps.

#### Acceptance Criteria

1. WHEN a learner completes all Episodes in a Season, THE Platform SHALL trigger the Season_Finale assessment
2. THE Assessment_Generator SHALL create personalized questions targeting Knowledge_Nodes with mastery probability below 0.90
3. THE Assessment_Generator SHALL use Item Response Theory to calibrate question difficulty
4. THE Season_Finale SHALL contain 10-15 questions with difficulty matched to learner's skill level
5. WHEN generating questions, THE Assessment_Generator SHALL prioritize nodes where the learner struggled or required Bridge_Sprints
6. THE Platform SHALL include a mix of question types: multiple choice, code completion, debugging, and short answer
7. WHEN a learner answers a question, THE Platform SHALL update BKT estimates in real-time
8. WHEN a learner completes the Season_Finale, THE Platform SHALL provide detailed feedback on strengths and remaining gaps
9. THE Assessment_Generator SHALL use AWS Bedrock to generate question content and variations
10. WHEN a learner scores below 70 percent on the Season_Finale, THE Platform SHALL recommend targeted review Episodes

### Requirement 6: Learner Growth Dashboard

**User Story:** As a learner, I want to visualize my skill progression, mastery levels, and learning patterns over time, so that I can track my growth and stay motivated.

#### Acceptance Criteria

1. THE Growth_Dashboard SHALL display a skill radar chart showing mastery levels across all learned Knowledge_Nodes
2. THE Growth_Dashboard SHALL show a timeline of completed Seasons and Episodes
3. THE Growth_Dashboard SHALL display current learning streak and total learning time
4. THE Growth_Dashboard SHALL show mastery percentage for each active Season
5. WHEN a learner's mastery probability for a node decays below 0.70, THE Platform SHALL display a knowledge decay alert
6. THE Growth_Dashboard SHALL use Leitner spaced repetition algorithm to schedule review reminders
7. THE Growth_Dashboard SHALL display personalized insights such as strongest skills and areas needing review
8. THE Growth_Dashboard SHALL show estimated time to complete active Seasons
9. THE Platform SHALL update dashboard metrics in real-time as learners complete Episodes
10. THE Growth_Dashboard SHALL be accessible on both desktop and mobile devices

### Requirement 7: India-Focused Intelligence Features

**User Story:** As a learner in India, I want features tailored to my local context including language support, university syllabus alignment, placement preparation, and low-bandwidth optimization, so that I can learn effectively despite infrastructure and contextual constraints.

#### Acceptance Criteria

1. THE Platform SHALL support Hinglish language mode for content delivery
2. WHEN Hinglish mode is enabled, THE Content_Generator SHALL code-mix Hindi and English naturally
3. THE Platform SHALL map university syllabi from major Indian universities to Seasons
4. WHEN a learner selects their university, THE Platform SHALL highlight Seasons aligned with their curriculum
5. THE Platform SHALL include placement-focused Seasons targeting common interview topics
6. THE Platform SHALL optimize content delivery for low-bandwidth connections under 2 Mbps
7. THE Platform SHALL use text-first rendering with progressive enhancement for images and interactive elements
8. THE Platform SHALL compress and lazy-load media assets to minimize data usage
9. WHEN network conditions are poor, THE Platform SHALL gracefully degrade to text-only mode
10. THE Platform SHALL cache completed Episodes locally for offline review
11. THE Platform SHALL use AWS Polly for text-to-speech in both English and Hinglish
12. THE Platform SHALL provide estimated data usage per Episode to help learners manage costs

### Requirement 8: Interactive Code Sandbox

**User Story:** As a learner, I want to write and execute code directly in the browser for Python, JavaScript, and SQL, so that I can practice programming without setting up local development environments.

#### Acceptance Criteria

1. THE Platform SHALL provide an in-browser code editor for Python, JavaScript, and SQL
2. THE Code_Lab SHALL use Monaco Editor for syntax highlighting and code completion
3. WHEN a learner writes Python code, THE Platform SHALL execute it in a sandboxed environment
4. WHEN a learner writes JavaScript code, THE Platform SHALL execute it in a sandboxed environment
5. WHEN a learner writes SQL code, THE Platform SHALL execute it against a sample database
6. THE Platform SHALL display execution results including output, errors, and execution time
7. THE Platform SHALL limit code execution time to 5 seconds to prevent infinite loops
8. THE Platform SHALL provide starter code and test cases for Code_Lab exercises
9. WHEN a learner's code passes all test cases, THE Platform SHALL mark the exercise as complete
10. THE Platform SHALL save learner code automatically every 10 seconds
11. THE Platform SHALL support code reset to restore starter code

### Requirement 9: Content Generation and Storage

**User Story:** As the platform, I need to generate high-quality educational content dynamically and store it efficiently, so that learners receive personalized Episodes without manual content creation overhead.

#### Acceptance Criteria

1. THE Content_Generator SHALL use AWS Bedrock with Claude models for content generation
2. THE Content_Generator SHALL use Claude Haiku for fast, cost-effective content generation
3. THE Content_Generator SHALL use Claude Sonnet for complex conversational interactions
4. WHEN generating content, THE Content_Generator SHALL include learning objectives, explanations, examples, and exercises
5. THE Platform SHALL store generated content in AWS S3 with versioning enabled
6. THE Platform SHALL cache frequently accessed content in DynamoDB for fast retrieval
7. THE Platform SHALL generate content on-demand when a learner reaches a new Knowledge_Node
8. THE Content_Generator SHALL follow a consistent template structure for each Episode format
9. THE Platform SHALL store learner progress and interaction data in DynamoDB
10. THE Platform SHALL use DynamoDB streams to trigger real-time analytics updates

### Requirement 10: Authentication and User Management

**User Story:** As a learner, I want to create an account, log in securely, and have my progress saved across devices, so that I can access my learning journey from anywhere.

#### Acceptance Criteria

1. THE Platform SHALL support user registration with email and password
2. THE Platform SHALL validate email addresses before account activation
3. THE Platform SHALL hash passwords using industry-standard algorithms before storage
4. THE Platform SHALL support secure login with session management
5. THE Platform SHALL allow learners to reset forgotten passwords via email
6. THE Platform SHALL store user profiles including name, email, learning preferences, and university affiliation
7. THE Platform SHALL sync learner progress across all devices in real-time
8. WHEN a learner logs in, THE Platform SHALL restore their active Seasons and progress state
9. THE Platform SHALL support social login with Google and GitHub
10. THE Platform SHALL comply with data privacy regulations for user data storage

### Requirement 11: Analytics and Monitoring

**User Story:** As a platform administrator, I want to monitor system performance, user engagement, and learning outcomes, so that I can optimize the platform and measure success metrics.

#### Acceptance Criteria

1. THE Platform SHALL log all user interactions including Episode views, completions, and time spent
2. THE Platform SHALL track system performance metrics including API latency and error rates
3. THE Platform SHALL use AWS CloudWatch for centralized logging and monitoring
4. THE Platform SHALL track key metrics: active learners, completion rates, average learning time, and struggle frequency
5. THE Platform SHALL generate daily reports on user engagement and content effectiveness
6. THE Platform SHALL monitor AWS Bedrock API usage and costs
7. THE Platform SHALL track geographic distribution of learners across Indian cities
8. THE Platform SHALL measure learning time reduction compared to linear course baselines
9. THE Platform SHALL track placement success rates for learners who complete placement-focused Seasons
10. THE Platform SHALL alert administrators when error rates exceed 5 percent or API latency exceeds 2 seconds

### Requirement 12: API and Backend Architecture

**User Story:** As a developer, I need a scalable, serverless backend architecture that handles content generation, user management, and real-time interactions efficiently, so that the platform can scale to 10,000+ active learners.

#### Acceptance Criteria

1. THE Platform SHALL use AWS Lambda for serverless compute
2. THE Platform SHALL use AWS API Gateway for RESTful API endpoints
3. THE Platform SHALL implement API endpoints for user authentication, content retrieval, progress tracking, and assessment submission
4. THE Platform SHALL use DynamoDB for storing user data, progress state, and content metadata
5. THE Platform SHALL use S3 for storing generated content and media assets
6. THE Platform SHALL implement rate limiting to prevent API abuse
7. THE Platform SHALL return API responses within 500ms for 95 percent of requests
8. THE Platform SHALL handle concurrent requests from 1000+ simultaneous users
9. THE Platform SHALL implement proper error handling and return meaningful error messages
10. THE Platform SHALL use AWS IAM for secure service-to-service authentication
11. THE Platform SHALL implement CORS policies for secure frontend-backend communication

### Requirement 13: Frontend User Interface

**User Story:** As a learner, I want an intuitive, responsive interface that works on desktop and mobile devices, so that I can learn comfortably on any device.

#### Acceptance Criteria

1. THE Platform SHALL use React with Next.js for the frontend application
2. THE Platform SHALL implement a streaming-service inspired UI with Season cards and Episode lists
3. THE Platform SHALL be fully responsive and work on screen sizes from 320px to 2560px width
4. THE Platform SHALL use D3.js or Mermaid for rendering Skill_Constellation visualizations
5. THE Platform SHALL use Recharts for rendering Growth_Dashboard analytics
6. THE Platform SHALL implement smooth transitions and loading states for all user interactions
7. THE Platform SHALL follow accessibility best practices including keyboard navigation and screen reader support
8. THE Platform SHALL use a clean, modern design with consistent color scheme and typography
9. THE Platform SHALL implement dark mode support for reduced eye strain
10. THE Platform SHALL load initial page content within 2 seconds on 3G connections
11. THE Platform SHALL implement progressive web app features for offline capability

## Algorithm Reference

| Algorithm | Feature | Purpose | Complexity |
|-----------|---------|---------|------------|
| **Topological Sort (Kahn's)** | F1 | Valid learning order from prerequisite DAG | O(V+E) |
| **Bayesian Knowledge Tracing** | F1, F2, F4, F5 | Per-concept mastery probability tracking | O(1) per update |
| **Item Response Theory (1PL)** | F1, F5 | Adaptive initial assessment placement & finale | O(n) for n questions |
| **Rule-Based Decision Tree** | F3 | Classify learner state from behavioral signals | O(1) |
| **Vygotsky ZPD Threshold** | F3 | Keep learner in 30-70% productive struggle zone | O(1) |
| **Progressive Hint Cascade** | F3 | 4-level escalating Socratic intervention | O(1) |
| **BFS (Breadth-First Search)** | F4 | Find all downstream concepts affected by a gap | O(V+E) |
| **Leitner Spaced Repetition** | F6 | Schedule optimal review timing for decay prevention | O(1) per concept |
| **Format Selection Rules** | F2 | Choose optimal episode format for concept + learner | O(1) |
| **RAG (Retrieval Augmented Generation)** | F3, F7 | Ground AI responses in verified content | O(KB size) |

## Technical Constraints and Assumptions

### Constraints

1. **Hackathon Timeline**: MVP must be completed within 7 days
2. **AWS Services Only**: All infrastructure must use AWS services for hackathon eligibility
3. **Sandbox Limitations**: Code execution limited to Python, JavaScript, and SQL only
4. **Rule-Based Struggle Detection**: ML-based detection requires training data not available for MVP
5. **Default BKT Parameters**: Bayesian Knowledge Tracing uses default parameters until sufficient user data enables calibration
6. **Manual Knowledge Graph**: Initial Skill_Constellation manually curated for demo; auto-generation requires more development time
7. **Cost Budget**: Target cost of ₹2-5 per active user per month
8. **Language Support**: MVP supports English and Hinglish only (no pure Hindi)

### Assumptions

1. Learners have basic internet connectivity (minimum 1 Mbps)
2. Learners have access to modern web browsers (Chrome, Firefox, Safari, Edge)
3. AWS Bedrock API has sufficient rate limits for expected user load
4. Generated content quality from Claude models meets educational standards
5. Learners are motivated to complete Seasons and engage with adaptive features
6. University syllabus mappings can be manually created for top 50 Indian universities
7. Placement preparation content aligns with common interview patterns at Indian tech companies
8. Learners will provide honest self-assessments during initial knowledge evaluation

## Success Metrics and KPIs

### Year 1 Targets

1. **Active Learners**: 10,000 active learners within 12 months of launch
2. **Learning Efficiency**: 40 percent reduction in learning time compared to linear video courses
3. **Geographic Reach**: 60 percent of users from non-metro cities (Tier 2/3)
4. **Employability Impact**: 2,000 students reporting improved placement outcomes or job transitions
5. **Completion Rate**: 50 percent Season completion rate (industry average is 10-15 percent)
6. **User Engagement**: Average 3 hours per week learning time per active user
7. **Cost Efficiency**: Maintain operational cost under ₹5 per active user per month
8. **Content Quality**: 4+ star average rating on Episode quality from learner feedback
9. **Struggle Resolution**: 70 percent of detected struggles resolved through Socratic_Mentor without external help
10. **Knowledge Retention**: 80 percent mastery retention after 30 days for completed Seasons

### Technical Performance Metrics

1. **API Latency**: P95 response time under 500ms
2. **System Uptime**: 99.5 percent availability
3. **Error Rate**: Less than 1 percent of requests result in errors
4. **Content Generation Speed**: Episodes generated within 10 seconds
5. **Bandwidth Usage**: Average data consumption under 50 MB per hour of learning

## Out of Scope for MVP

1. Live instructor sessions or peer collaboration features
2. Mobile native applications (iOS/Android)
3. Advanced ML models for struggle detection (using rule-based approach)
4. Automated knowledge graph generation (manually curated for demo)
5. Payment and subscription management
6. Content creation tools for instructors
7. Advanced gamification features (badges, leaderboards)
8. Video-based Episodes (focusing on text and interactive content)
9. Languages beyond English and Hinglish
10. Integration with external learning management systems
