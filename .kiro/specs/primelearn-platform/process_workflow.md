# PrimeLearn Process Workflow

This document visualizes the core process workflows of the PrimeLearn platform, detailing the user journey from onboarding to mastery.

```mermaid
graph TD
    %% Nodes
    Start([User Signs Up])
    GoalSelection[Select Goal / Career Path]
    Assessment[Adaptive Initial Assessment\n(IRT Algorithm)]
    GenerateGraph[Generate Skill Constellation\n(Bedrock Haiku)]
    TopoSort[Topological Sort of Curriculum]
    
    HomeScreen[Home Screen / Dashboard]
    SelectEpisode[Select Next Episode]
    
    PreFlight{Prerequisites Met?}
    GenBridge[Generate Bridge Sprint\n(Bedrock Haiku)]
    DoBridge[Complete Bridge Sprint]
    BridgeCheck{Gap Filled?}
    
    CheckCache{Content Cached?}
    GenContent[Generate Episode Content\n(Bedrock Haiku)]
    Retrievecontent[Retrieve from S3/Cache]
    
    StartLearning[Start Episode\n(Visual/Code/X-Ray/Case Study)]
    
    subgraph "Learning Loop"
        Interaction[Learner Interacts]
        Monitor[Struggle Detector\n(Real-time Monitoring)]
        StruggleCheck{Struggle Score > Threshold?}
        Mentor[Socratic Mentor Intervention\n(Bedrock Sonnet + Guardrails)]
        ProgressCheck{Episode Complete?}
    end
    
    UpdateBKT[Update Bayesian Knowledge Tracing\n(Mastery Probabilities)]
    DecayCheck[Check Knowledge Decay\n(Leitner System)]
    
    SeasonComplete{Season Complete?}
    SeasonFinale[Generate Season Finale Assessment\n(Bedrock Sonnet)]
    Portfolio[Generate Portfolio Artifact]
    UnlockNext[Unlock Next Season]
    
    %% Edges
    Start --> GoalSelection
    GoalSelection --> Assessment
    Assessment --> GenerateGraph
    GenerateGraph --> TopoSort
    TopoSort --> HomeScreen
    
    HomeScreen --> SelectEpisode
    SelectEpisode --> PreFlight
    
    PreFlight -- No (Gap Detected) --> GenBridge
    GenBridge --> DoBridge
    DoBridge --> BridgeCheck
    BridgeCheck -- Pass --> PreFlight
    BridgeCheck -- Fail --> GenBridge
    
    PreFlight -- Yes --> CheckCache
    CheckCache -- No --> GenContent
    GenContent --> StartLearning
    CheckCache -- Yes --> Retrievecontent
    Retrievecontent --> StartLearning
    
    StartLearning --> Interaction
    Interaction --> Monitor
    Monitor --> StruggleCheck
    StruggleCheck -- Yes --> Mentor
    Mentor --> Interaction
    StruggleCheck -- No --> ProgressCheck
    ProgressCheck -- No --> Interaction
    
    ProgressCheck -- Yes --> UpdateBKT
    UpdateBKT --> DecayCheck
    DecayCheck --> SeasonComplete
    
    SeasonComplete -- No --> HomeScreen
    SeasonComplete -- Yes --> SeasonFinale
    SeasonFinale --> Portfolio
    Portfolio --> UnlockNext
    UnlockNext --> HomeScreen

    %% Styles
    style Start fill:#d4f1f9,stroke:#0077b6,stroke-width:2px
    style HomeScreen fill:#ffe8d6,stroke:#ff9f1c,stroke-width:2px
    style Mentor fill:#ffccd5,stroke:#e63946,stroke-width:2px,stroke-dasharray: 5 5
    style GenContent fill:#e2ece9,stroke:#2a9d8f,stroke-width:2px
    style Monitor fill:#e2ece9,stroke:#2a9d8f,stroke-width:2px
```

## Workflow Descriptions

1.  **Onboarding & Personalization**:
    *   The user sets a goal (e.g., "Become a Backend Dev").
    *   An IRT-based adaptive assessment estimates their initial skill level (theta).
    *   The AI generates a personalized DAG (Skill Constellation) of seasons and episodes.

2.  **Episode Selection & Prerequisite Intelligence**:
    *   Before an episode starts, the system checks if prerequisites are met (Mastery probability > threshold).
    *   If a gap is found, a **Bridge Sprint** is dynamically generated to fill that specific gap.

3.  **Content Delivery**:
    *   The system checks if the episode content exists in S3 (cache).
    *   If not, AWS Bedrock (Claude Haiku) generates it on-demand based on the user's preferred format (Visual Story, Code Lab, etc.).

4.  **The Learning Loop (Struggle Detection)**:
    *   As the user learns (writes code, answers quizzes), the **Struggle Detector** monitors signals (error rate, idle time).
    *   If the **Struggle Score** exceeds a threshold, the **Socratic Mentor** intervenes with progressive hints, ensuring the learner stays in the "Zone of Proximal Development".

5.  **Completion & Progression**:
    *   Upon completion, **Bayesian Knowledge Tracing (BKT)** updates mastery probabilities for related concepts.
    *   The **Leitner System** checks for knowledge decay and schedules reviews if necessary.
    *   If the season is finished, a **Season Finale** (adaptive assessment) is generated to validate mastery and create a portfolio piece.
