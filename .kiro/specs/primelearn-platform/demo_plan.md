# Demo Plan & Development Strategy

## 1. Demo Topics (Pre-Built for Presentation)

We will build 2 complete seasons + 1 bridge sprint for the demo:

### Season 1: "Web Fundamentals" (Conceptual — for non-coders too)
- **EP 1**: How the Internet Works (Visual Story)
- **EP 2**: DNS Resolution (Visual Story)
- **EP 3**: HTTP & REST (Concept X-Ray)
- **EP 4**: How Zomato Handles Scale (Case Study)
- **FINALE**: Explain Challenge — "Explain to a non-tech CEO how their website handles traffic"

### Season 2: "JavaScript Essentials" (Code-heavy)
- **EP 1**: Variables & Scope (Code Lab)
- **EP 2**: Functions & Closures (Code Lab) ← **MAIN DEMO EPISODE**
- **EP 3**: Async/Await (Code Lab)
- **FINALE**: Build Project — "Build an async task queue"

### Bridge Sprint: "Networking Prerequisites"
- **Trigger**: System Design S3
- **Content**:
    - Quick Byte: IP Addresses refresher
    - Visual Story: TCP Handshake
    - Checkpoint

## 2. 3-Minute Video Script

| Time | Screen | Narration | Criteria Hit |
|------|--------|-----------|--------------|
| **0:00-0:15** | Problem stats on screen | "1.5 million engineering graduates every year in India. 55% are unemployable. Not because they lack talent — because learning is broken." | Impact |
| **0:15-0:35** | Home screen | "PrimeLearn fixes this. It structures tech learning like a streaming service — seasons, episodes, personalized to YOU. Tell it your goal, and AI builds your path." | Ideation |
| **0:35-0:55** | Constellation generating | Live demo: type "I want to learn backend development" → watch constellation appear with seasons + episodes. | Creativity |
| **0:55-1:15** | Visual Story episode | Show "How DNS Works" — click through animated steps, show comprehension gate. "Not everything is code. This episode teaches DNS visually." | PS Alignment |
| **1:15-1:40** | Code Lab episode | Show closures Code Lab. Type code with errors. Watch struggle detection trigger. Mentor asks Socratic question (L1), then gives hint (L2). "The AI watches HOW you learn, not just IF you pass." | Technical |
| **1:40-1:55** | Bridge Sprint | Click on System Design → Bridge Sprint alert. "Your caching knowledge is weak. Here's a 15-min sprint." Show bridge generating. | Technical |
| **1:55-2:15** | Season Finale | Show personalized assessment: "Build a task manager. Focus on hooks and effects (your weak areas)." Show AI code review feedback. | Uniqueness |
| **2:15-2:30** | Dashboard | Show skill radar, mastery heatmap, placement readiness score, fading knowledge alert. | Impact |
| **2:30-2:45** | Hinglish + Syllabus | Toggle Hinglish: "API ek waiter hai..." Show SPPU syllabus mapped. "A student in Jalgaon gets the same AI tutor as someone at IIT." | Impact (Bharat) |
| **2:45-3:00** | Architecture + close | Flash architecture diagram. "8 AWS services. ₹500 to run. PrimeLearn. Built for Bharat. Powered by AWS." | Technical + Business |

## 3. Development Plan & Task Division

### Recommended Team Split
- **Frontend Lead**: React + Next.js app, Prime-style UI, Monaco Editor integration, Recharts dashboard, D3.js diagrams (High Priority)
- **Backend Lead**: Lambda functions, API Gateway setup, DynamoDB schemas, all business logic (BKT, struggle scoring, format selection) (High Priority)
- **AI/Bedrock Lead**: Bedrock integration (Haiku + Sonnet), prompt engineering, Knowledge Bases setup, Guardrails config (High Priority)
- **Content & Demo Lead**: Curate knowledge graph for demo topics, write demo script, record video, prepare PPT (High Priority)
- **DevOps + QA**: AWS deployment, S3 setup, CloudWatch, testing, Kiro setup, GitHub repo management (Medium Priority)

### Timeline (7 Days)
- **Day 1**: Foundation (DynamoDB tables, Lambda boilerplate, React scaffold, Bedrock API tests)
- **Day 2**: F1 + F2 Core (Constellation generation, Format selection, 1 Visual Story episode)
- **Day 3**: F2 + F3 (Code Lab with Monaco, Struggle detection signals, Mentor basic responses)
- **Day 4**: F3 + F4 (Socratic hint cascade, Bridge Sprint flow, Guardrails config)
- **Day 5**: F5 + F6 (Season Finale generation, Dashboard with radar/heatmap)
- **Day 6**: F7 + Polish (Hinglish, Syllabus mapper, Curation for demo, UI polish)
- **Day 7**: Demo (Record video, Finalize PPT, Deploy, Write blog, Final testing)

## 4. Kiro Integration Strategy

**As Development Tool**:
- Use Kiro as primary IDE during hackathon.
- "Built WITH Kiro, built ON Bedrock".

**As Feature Reference (Future Scope)**:
- "For advanced users, PrimeLearn's Code Lab sandbox could integrate with Kiro's AI assistance."

**Amazon Q**:
- Used for debugging and AWS service integration.

## 5. Submission Checklist

- [ ] Working prototype on AWS (Backend + AI Lead)
- [ ] GitHub repository (clean, documented) (DevOps)
- [ ] 3-minute video pitch (Content Lead)
- [ ] Presentation deck (10-12 slides) (Content Lead)
- [ ] Architecture diagram (Backend Lead)
- [ ] Demo data (2 seasons + bridge sprint) (Content Lead)
- [ ] Hinglish mode working (AI Lead)
- [ ] Dashboard with real data (Frontend Lead)
- [ ] Kiro mentioned in presentation (Content Lead)
- [ ] Cost estimate documented (Backend Lead)
