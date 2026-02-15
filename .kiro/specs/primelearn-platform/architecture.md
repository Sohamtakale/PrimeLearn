# PrimeLearn System Architecture

This document outlines the high-level architecture of the PrimeLearn platform, connecting the Frontend, Backend, and AWS AI services.

```mermaid
graph TD
    %% Styling
    classDef frontend fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef gateway fill:#fff3e0,stroke:#e65100,stroke-width:2px;
    classDef compute fill:#f3e5f5,stroke:#4a148c,stroke-width:2px;
    classDef storage fill:#e8f5e9,stroke:#1b5e20,stroke-width:2px;
    classDef ai fill:#fff8e1,stroke:#f57f17,stroke-width:2px;
    classDef monitor fill:#eceff1,stroke:#37474f,stroke-width:2px;

    subgraph Client ["Client Side"]
        Frontend["Frontend App<br/>(React + Next.js + Recharts)<br/>Prime UI, Code Sandbox"]:::frontend
    end

    subgraph Cloud ["AWS Cloud"]
        style Cloud fill:#f5faff,stroke:#000,stroke-width:1px,stroke-dasharray: 5 5

        APIG["Amazon API Gateway<br/>(REST API + Auth)"]:::gateway
        
        subgraph Logic ["Business Logic"]
            Lambda["AWS Lambda<br/>(Python 3.12 / Node.js)<br/>Orchestration & Compute"]:::compute
        end
        
        subgraph Data ["Data Layer"]
            DynamoDB[("Amazon DynamoDB<br/>Knowledge Graph<br/>Learner State")]:::storage
            S3[("Amazon S3<br/>Content Cache<br/>Media Assets")]:::storage
        end
        
        subgraph AI_Layer ["AI & ML Services"]
            Bedrock["Amazon Bedrock"]:::ai
            Haiku["Claude Haiku<br/>(Content Gen)"]:::ai
            Sonnet["Claude Sonnet<br/>(Mentor/Assess)"]:::ai
            KB["Knowledge Bases<br/>(RAG)"]:::ai
            Guardrails["Guardrails<br/>(Socratic Safety)"]:::ai
            
            Polly["Amazon Polly<br/>(TTS)"]:::ai
            Comprehend["Amazon Comprehend<br/>(Lang Detect)"]:::ai
        end
        
        CloudWatch["Amazon CloudWatch<br/>(Logs & Monitoring)"]:::monitor
    end

    %% Connections
    Client -->|HTTPS/JSON| APIG
    APIG -->|Event| Lambda
    
    Lambda <-->|Read/Write| DynamoDB
    Lambda <-->|Read/Write| S3
    
    Lambda -->|Invoke| Bedrock
    Bedrock --> Haiku
    Bedrock --> Sonnet
    Bedrock --> KB
    Bedrock --> Guardrails
    
    Lambda -->|Text| Polly
    Polly -->|Audio Stream| Lambda
    
    Lambda -->|Text| Comprehend
    
    Lambda -.->|Logs| CloudWatch
    APIG -.->|Logs| CloudWatch
```

## Service Role Descriptions

| Service | Role | Free Tier Coverage |
| :--- | :--- | :--- |
| **Amazon Bedrock** | **Core AI Engine**. Orchestrates calls to Claude models. | No (Pay-per-token) |
| **Claude Haiku** | **Content Generation**. Fast, cheap model for generating episodes (90% of calls). | N/A |
| **Claude Sonnet** | **Higher Intelligence**. Powering the Socratic Mentor and Final Assessments. | N/A |
| **AWS Lambda** | **Serverless Compute**. Runs BKT algorithms, struggle detection, and API logic. | 1M req/mo (Always Free) |
| **Amazon DynamoDB** | **NoSQL Database**. Stores the Knowledge Graph (DAG), User Progress, and Logs. | 25GB (Always Free) |
| **Amazon S3** | **Object Storage**. Caches generated content and stores static assets/media. | 5GB (12 months) |
| **API Gateway** | **API Front Door**. Manages REST endpoints, auth, and throttling. | 1M calls/mo (12 months) |
| **Amazon Polly** | **Text-to-Speech**. Converts narrative episodes to audio for accessibility. | 5M chars/mo (12 months) |
| **Amazon Comprehend**| **NLP**. Detects language (Hindi/English) to trigger Hinglish mode. | 50K units/mo (12 months) |
