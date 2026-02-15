# Business Context & Strategy

## 1. Business Feasibility

### Value Proposition
- **Learners**: "Stop watching tutorials you'll forget. Start a learning journey that adapts to you, tests you, and builds your portfolio."
- **Institutions**: "Give every student a personal AI tutor. See exactly where each student struggles. Intervene before they fail."

### Revenue Model

**B2C (Freemium)**:
- **Free**: 2 seasons, 20 mentor queries/day, basic dashboard.
- **Pro (₹299/month)**: Unlimited seasons + mentor, full dashboard, portfolio, placement readiness.
- **Pro Annual (₹2,499/year)**: Same as Pro, 30% discount.

**B2B (Institutional)**:
- **College License**: ₹50K/year/500 students (Admin dashboard, syllabus mapper, batch analytics).
- **Bootcamp API**: ₹20K/month (Embeddable episodes, progress API).
- **Corporate Training**: ₹1L/year/200 employees (Custom skill paths, compliance tracking).

### Unit Economics
- **Cost per active user**: ~₹2-5/month (Haiku inference + caching)
- **Revenue per Pro user**: ₹299/month
- **Gross margin**: ~95%
- **LTV:CAC**: ~5:1 (Target CAC <₹500)

### Go-To-Market Strategy
1.  **Phase 1 (0-6 months) - Community-Led Growth**: Free tier for engineering students, partnerships with college coding clubs.
2.  **Phase 2 (6-12 months) - Monetize**: Launch Pro tier, pilot institutional licenses with 5-10 colleges.
3.  **Phase 3 (12-24 months) - Scale**: B2B corporate training, regional language expansion, placement portal integrations.

### Competitive Differentiation
| Competitor | Their Model | Our Edge |
|------------|-------------|----------|
| **Unacademy** | Passive video, expensive | Adaptive, interactive, 20x cheaper |
| **Scaler** | Mentor-heavy, very expensive | AI-first, 100x cheaper, scalable |
| **LeetCode** | Practice only, no teaching | We teach AND practice, adapt to gaps |
| **ChatGPT** | No structure/progression | Structured seasons, portfolio, tracking |

## 2. Impact Statement

### Quantifiable Impact (Year 1 Targets)
- **Active Learners**: 10,000
- ** employability**: 2,000 students reporting better outcomes.
- **Learning Time Saved**: 40% reduction vs linear courses.
- **Tier 2/3 Reach**: 60% of users from non-metro cities.
- **Bridge Sprints**: 5,000 gaps detected and filled.

### Social Impact
- **Democratization**: Tier 3 students get same quality AI tutor as IIT students.
- **Reduced Stigma**: Adaptive pace means no one is labeled "slow".
- **Language Bridge**: Hinglish breaks English proficiency barriers.
- **Proof of Skills**: Portfolio-based validation over certificates.

## 3. Cost Estimate (Hackathon Scale)

| Service | Usage Estimate | Cost (Approx) |
|---------|----------------|---------------|
| **Bedrock (Haiku)** | ~800K input + 300K output | ~₹200 |
| **Bedrock (Sonnet)** | ~100K input + 50K output | ~₹100 |
| **DynamoDB** | <1GB, <25 RCU/WCU | ₹0 (Free Tier) |
| **S3** | <1GB | ₹0 (Free Tier) |
| **Lambda** | <100K invocations | ₹0 (Free Tier) |
| **API Gateway** | <100K calls | ₹0 (Free Tier) |
| **Polly** | <500K chars | ₹0 (Free Tier) |
| **Comprehend** | <10K units | ₹0 (Free Tier) |
| **TOTAL** | | **₹200-400 (~$2.50-5)** |

**Optimization Strategies**:
- Use Haiku for 90% of calls.
- Aggressive S3 caching of generated content.
- Pre-generate demo content.
- Use fresh AWS account for Free Tier.
