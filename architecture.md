# Mudda Platform - AWS Architecture

**Project:** Mudda Civic Social Media Platform  
**Version:** 1.0  
**Last Updated:** 2026-02-15  
**Purpose:** Detailed AWS Architecture Diagrams

---

## Overview

This document provides comprehensive AWS architecture diagrams for the Mudda platform, showing the complete infrastructure, data flow, and service interactions using AWS services.

---

## 1. Complete AWS Infrastructure Architecture

This diagram shows the complete AWS infrastructure with all services, networking, and data flows.

```mermaid
graph TB
    subgraph Internet["🌐 INTERNET"]
        Users["👥 Users<br/>(Mobile & Web)"]
    end
    
    subgraph AWS["☁️ AWS CLOUD"]
        subgraph EdgeLayer["📡 EDGE & CDN LAYER"]
            Route53["🌍 Route 53<br/>DNS Management"]
            CloudFront["⚡ CloudFront<br/>CDN Distribution<br/>Edge Caching"]
            WAF["🛡️ AWS WAF<br/>Web Application<br/>Firewall"]
        end
        
        subgraph APILayer["🚪 API LAYER"]
            APIGateway["🔌 API Gateway<br/>REST APIs<br/>WebSocket APIs<br/>Rate Limiting"]
            Cognito["🔐 Cognito<br/>User Authentication<br/>JWT Tokens"]
        end
        
        subgraph ComputeLayer["⚙️ COMPUTE LAYER"]
            direction TB
            
            subgraph BeanstalkEnv["🌱 Elastic Beanstalk Environment"]
                ALB["⚖️ Application<br/>Load Balancer"]
                ASG["📊 Auto Scaling<br/>Group"]
                EC2Spring["🖥️ EC2 Instances<br/>Spring Boot<br/>Services"]
            end
            
            subgraph LambdaFunctions["⚡ Lambda Functions"]
                LambdaAuth["🔑 Auth Service"]
                LambdaContent["🔍 Content Analysis<br/>• Language Detection<br/>• Hate Speech<br/>• NSFW Filter<br/>• Categorization<br/>• Duplication<br/>• OCR"]
                LambdaNotif["📧 Notification<br/>Service"]
            end
            
            subgraph AICompute["🤖 AI Compute"]
                EC2AI["🖥️ EC2 (AI)<br/>Agentic AI<br/>Service"]
            end
        end
        
        subgraph EventLayer["📨 EVENT & MESSAGING LAYER"]
            EventBridge["📊 EventBridge<br/>Event Bus<br/>Event Routing"]
            MSK["📬 Amazon MSK<br/>Kafka Cluster<br/>3 Brokers<br/>30 Partitions"]
            StepFunctions["🔄 Step Functions<br/>Workflow<br/>Orchestration"]
        end
        
        subgraph DataLayer["💾 DATA LAYER"]
            direction TB
            
            subgraph Databases["🗄️ Databases"]
                RDS["🐘 RDS PostgreSQL<br/>Multi-AZ<br/>Read Replicas<br/>Transactional DB"]
                ElastiCache["⚡ ElastiCache<br/>Redis Cluster<br/>Session Cache"]
            end
            
            subgraph ObjectStorage["📦 Object Storage"]
                S3Media["📸 S3 Bucket<br/>Media Files<br/>Images/Videos"]
                S3Vector["🔢 S3 Bucket<br/>Vector<br/>Embeddings"]
                S3Static["🌐 S3 Bucket<br/>Static Website<br/>Next.js"]
            end
            
            subgraph SearchAnalytics["🔍 Search & Analytics"]
                OpenSearch["🔎 OpenSearch<br/>Full-text Search<br/>Mudda Index"]
                Redshift["📊 Redshift<br/>Data Warehouse<br/>Analytics<br/>ML Feedback"]
            end
        end
        
        subgraph AIMLLayer["🧠 AI/ML LAYER"]
            Bedrock["🤖 Amazon Bedrock<br/>Claude 3<br/>Llama 3<br/>Managed LLMs"]
            SageMaker["🧪 SageMaker<br/>Custom Models<br/>Fine-tuning<br/>Inference"]
        end
        
        subgraph SecurityLayer["🔒 SECURITY LAYER"]
            IAM["👤 IAM<br/>Roles & Policies"]
            SecretsManager["🔑 Secrets Manager<br/>API Keys<br/>DB Credentials"]
            KMS["🔐 KMS<br/>Encryption Keys"]
            ACM["📜 ACM<br/>SSL/TLS<br/>Certificates"]
        end
        
        subgraph MonitoringLayer["📈 MONITORING & OBSERVABILITY"]
            CloudWatch["📊 CloudWatch<br/>Metrics & Logs<br/>Dashboards<br/>Alarms"]
            XRay["🔍 X-Ray<br/>Distributed<br/>Tracing"]
        end
        
        subgraph NotificationLayer["📧 NOTIFICATION SERVICES"]
            SES["📧 SES<br/>Email Service"]
            SNS["📱 SNS<br/>Push<br/>Notifications"]
        end
    end
    
    %% Connections - User Flow
    Users --> Route53
    Route53 --> CloudFront
    CloudFront --> WAF
    WAF --> APIGateway
    APIGateway --> Cognito
    
    %% API to Compute
    APIGateway --> ALB
    APIGateway --> LambdaAuth
    APIGateway --> LambdaNotif
    ALB --> EC2Spring
    
    %% Event Flow
    EC2Spring --> EventBridge
    LambdaAuth --> EventBridge
    EventBridge --> MSK
    MSK --> LambdaContent
    MSK --> StepFunctions
    StepFunctions --> EC2AI
    
    %% Data Connections
    EC2Spring --> RDS
    EC2Spring --> ElastiCache
    EC2Spring --> S3Media
    EC2Spring --> OpenSearch
    LambdaContent --> RDS
    LambdaContent --> S3Vector
    EC2AI --> RDS
    
    %% AI/ML Connections
    EC2AI --> Bedrock
    EC2AI --> SageMaker
    EC2AI --> S3Vector
    LambdaContent --> SageMaker
    
    %% Analytics Flow
    MSK --> Redshift
    EventBridge --> Redshift
    Redshift -.->|Feedback| EC2AI
    
    %% Notification Flow
    EC2AI --> SNS
    EC2AI --> SES
    LambdaNotif --> SNS
    LambdaNotif --> SES
    
    %% Security Connections
    EC2Spring -.-> IAM
    LambdaContent -.-> IAM
    EC2AI -.-> IAM
    EC2Spring -.-> SecretsManager
    RDS -.-> KMS
    S3Media -.-> KMS
    CloudFront -.-> ACM
    APIGateway -.-> ACM
    
    %% Monitoring
    EC2Spring --> CloudWatch
    LambdaContent --> CloudWatch
    EC2AI --> CloudWatch
    EC2Spring --> XRay
    LambdaContent --> XRay
    
    %% CDN for Static Content
    CloudFront --> S3Static
    CloudFront --> S3Media
    
    %% Styling
    classDef userStyle fill:#e1f5ff,stroke:#01579b,stroke-width:3px
    classDef edgeStyle fill:#ff9900,stroke:#232f3e,stroke-width:3px
    classDef apiStyle fill:#4CAF50,stroke:#2E7D32,stroke-width:3px
    classDef computeStyle fill:#2196F3,stroke:#1565C0,stroke-width:3px
    classDef eventStyle fill:#FFC107,stroke:#F57C00,stroke-width:3px
    classDef dataStyle fill:#9C27B0,stroke:#6A1B9A,stroke-width:3px
    classDef aiStyle fill:#F44336,stroke:#C62828,stroke-width:3px
    classDef securityStyle fill:#607D8B,stroke:#37474F,stroke-width:3px
    classDef monitorStyle fill:#00BCD4,stroke:#00838F,stroke-width:3px
    classDef notifStyle fill:#8BC34A,stroke:#558B2F,stroke-width:3px
    
    class Users userStyle
    class Route53,CloudFront,WAF edgeStyle
    class APIGateway,Cognito apiStyle
    class ALB,ASG,EC2Spring,LambdaAuth,LambdaContent,LambdaNotif,EC2AI computeStyle
    class EventBridge,MSK,StepFunctions eventStyle
    class RDS,ElastiCache,S3Media,S3Vector,S3Static,OpenSearch,Redshift dataStyle
    class Bedrock,SageMaker aiStyle
    class IAM,SecretsManager,KMS,ACM securityStyle
    class CloudWatch,XRay monitorStyle
    class SES,SNS notifStyle
```

---

## 2. Event-Driven Architecture Flow

This diagram shows the detailed event flow through Amazon MSK and how different services interact.

```mermaid
graph LR
    subgraph Producers["📤 EVENT PRODUCERS"]
        SpringBoot["🌱 Spring Boot<br/>Services"]
        LambdaProducers["⚡ Lambda<br/>Functions"]
    end
    
    subgraph MSKCluster["📨 AMAZON MSK CLUSTER"]
        direction TB
        Broker1["🖥️ Broker 1"]
        Broker2["🖥️ Broker 2"]
        Broker3["🖥️ Broker 3"]
        
        subgraph Topics["📋 KAFKA TOPICS"]
            T1["mudda.created"]
            T2["mudda.updated"]
            T3["mudda.status_changed"]
            T4["mudda.analysis_completed"]
            T5["comment.created"]
            T6["media.uploaded"]
            T7["notification.dispatch"]
        end
    end
    
    subgraph Consumers["📥 EVENT CONSUMERS"]
        direction TB
        
        subgraph ContentAnalysis["🔍 Content Analysis (Lambda)"]
            L1["Language<br/>Detection"]
            L2["Hate Speech<br/>Detection"]
            L3["NSFW<br/>Filter"]
            L4["Duplication<br/>Detection"]
            L5["Categorization"]
            L6["OCR<br/>Service"]
        end
        
        StepFunc["🔄 Step Functions<br/>Workflows"]
        SearchIndex["🔎 OpenSearch<br/>Indexer"]
        NotifService["📧 Notification<br/>Service"]
        Analytics["📊 Redshift<br/>Analytics"]
    end
    
    %% Producer to MSK
    SpringBoot -->|Publish| Broker1
    SpringBoot -->|Publish| Broker2
    LambdaProducers -->|Publish| Broker3
    
    %% Brokers to Topics
    Broker1 --> Topics
    Broker2 --> Topics
    Broker3 --> Topics
    
    %% Topics to Consumers
    T1 -->|Subscribe| L1
    T1 -->|Subscribe| L2
    T1 -->|Subscribe| L3
    T1 -->|Subscribe| L4
    T1 -->|Subscribe| L5
    T1 -->|Subscribe| L6
    
    T4 -->|Subscribe| StepFunc
    T1 -->|Subscribe| SearchIndex
    T2 -->|Subscribe| SearchIndex
    T3 -->|Subscribe| SearchIndex
    
    T7 -->|Subscribe| NotifService
    
    Topics -->|Stream All| Analytics
    
    %% Styling
    classDef producerStyle fill:#4CAF50,stroke:#2E7D32,stroke-width:2px
    classDef mskStyle fill:#FFC107,stroke:#F57C00,stroke-width:3px
    classDef consumerStyle fill:#2196F3,stroke:#1565C0,stroke-width:2px
    classDef topicStyle fill:#FFE082,stroke:#F57C00,stroke-width:1px
    
    class SpringBoot,LambdaProducers producerStyle
    class Broker1,Broker2,Broker3 mskStyle
    class T1,T2,T3,T4,T5,T6,T7 topicStyle
    class L1,L2,L3,L4,L5,L6,StepFunc,SearchIndex,NotifService,Analytics consumerStyle
```

---

## 3. Data Architecture & Storage

This diagram shows the data layer architecture with different storage services and their purposes.

```mermaid
graph TB
    subgraph Applications["💻 APPLICATIONS"]
        SpringServices["🌱 Spring Boot<br/>Services"]
        LambdaServices["⚡ Lambda<br/>Functions"]
        AIServices["🤖 AI Services"]
    end
    
    subgraph TransactionalData["🗄️ TRANSACTIONAL DATA"]
        direction TB
        
        subgraph RDSCluster["🐘 RDS PostgreSQL CLUSTER"]
            Primary["📝 Primary<br/>Instance<br/>Write Operations"]
            Replica1["📖 Read Replica 1<br/>Read Operations"]
            Replica2["📖 Read Replica 2<br/>Read Operations"]
        end
        
        Cache["⚡ ElastiCache Redis<br/>Session Cache<br/>Query Cache<br/>Rate Limiting"]
    end
    
    subgraph ObjectStorage["📦 OBJECT STORAGE (S3)"]
        direction LR
        MediaBucket["📸 Media Bucket<br/>Images/Videos<br/>Thumbnails<br/>CDN Origin"]
        VectorBucket["🔢 Vector Bucket<br/>Embeddings<br/>RAG Data"]
        StaticBucket["🌐 Static Bucket<br/>Next.js Build<br/>Assets"]
    end
    
    subgraph SearchLayer["🔍 SEARCH & INDEXING"]
        OpenSearchCluster["🔎 OpenSearch Cluster<br/>Full-text Search<br/>Mudda Index<br/>Autocomplete"]
    end
    
    subgraph AnalyticsData["📊 ANALYTICS & WAREHOUSE"]
        direction TB
        
        RedshiftCluster["📈 Redshift Cluster<br/>Data Warehouse"]
        
        subgraph RedshiftTables["📋 TABLES"]
            FactTables["Fact Tables<br/>• fact_mudda_analysis<br/>• fact_engagement<br/>• fact_resolution_plan"]
            DimTables["Dimension Tables<br/>• dim_mudda<br/>• dim_user<br/>• dim_category<br/>• dim_date"]
        end
    end
    
    subgraph DataPipeline["🔄 DATA PIPELINE"]
        MSKConnect["📨 MSK Connect<br/>Kafka → Redshift"]
        Glue["🔧 AWS Glue<br/>ETL Jobs"]
    end
    
    %% Application to Transactional
    SpringServices -->|Write| Primary
    SpringServices -->|Read| Replica1
    SpringServices -->|Read| Replica2
    SpringServices -->|Cache| Cache
    LambdaServices -->|Read/Write| Primary
    AIServices -->|Read| Replica2
    
    %% Replication
    Primary -.->|Async Replication| Replica1
    Primary -.->|Async Replication| Replica2
    
    %% Object Storage
    SpringServices -->|Upload| MediaBucket
    LambdaServices -->|Store| VectorBucket
    AIServices -->|Read| VectorBucket
    
    %% Search
    SpringServices -->|Index| OpenSearchCluster
    LambdaServices -->|Index| OpenSearchCluster
    SpringServices -->|Query| OpenSearchCluster
    
    %% Analytics Pipeline
    MSKConnect -->|Stream| RedshiftCluster
    Glue -->|Transform| RedshiftCluster
    RedshiftCluster --> FactTables
    RedshiftCluster --> DimTables
    
    %% Feedback Loop
    RedshiftCluster -.->|ML Feedback| AIServices
    
    %% Styling
    classDef appStyle fill:#4CAF50,stroke:#2E7D32,stroke-width:2px
    classDef dbStyle fill:#2196F3,stroke:#1565C0,stroke-width:3px
    classDef storageStyle fill:#9C27B0,stroke:#6A1B9A,stroke-width:2px
    classDef searchStyle fill:#FF9800,stroke:#E65100,stroke-width:2px
    classDef analyticsStyle fill:#00BCD4,stroke:#00838F,stroke-width:3px
    classDef pipelineStyle fill:#FFC107,stroke:#F57C00,stroke-width:2px
    
    class SpringServices,LambdaServices,AIServices appStyle
    class Primary,Replica1,Replica2,Cache dbStyle
    class MediaBucket,VectorBucket,StaticBucket storageStyle
    class OpenSearchCluster searchStyle
    class RedshiftCluster,FactTables,DimTables analyticsStyle
    class MSKConnect,Glue pipelineStyle
```

---

## 4. AI/ML Services Architecture

This diagram shows the AI and ML services architecture with model hosting and inference.

```mermaid
graph TB
    subgraph Input["📥 INPUT"]
        MuddaData["Mudda Content<br/>Text + Images<br/>Location + Category"]
    end
    
    subgraph ContentAnalysisLambda["⚡ CONTENT ANALYSIS (Lambda)"]
        direction LR
        LanguageDetect["🌐 Language<br/>Detection<br/>Lambda"]
        HateSpeech["🚫 Hate Speech<br/>Detection<br/>Lambda"]
        NSFWFilter["🔞 NSFW<br/>Filter<br/>Lambda"]
        Duplicate["🔄 Duplication<br/>Detection<br/>Lambda"]
        Categorize["🏷️ Categorization<br/>Lambda"]
        OCR["📄 OCR<br/>Service<br/>Lambda"]
    end
    
    subgraph ModelHosting["🧪 MODEL HOSTING"]
        direction TB
        
        subgraph SageMakerEndpoints["SageMaker Endpoints"]
            SM1["Hate Speech<br/>Model<br/>(Multilingual)"]
            SM2["NSFW<br/>Classification<br/>Model"]
            SM3["Categorization<br/>Model"]
            SM4["Embedding<br/>Model"]
        end
        
        subgraph BedrockModels["🤖 Amazon Bedrock"]
            Claude["Claude 3<br/>Opus/Sonnet"]
            Llama["Llama 3<br/>70B/8B"]
        end
    end
    
    subgraph AgenticAI["🧠 AGENTIC AI SERVICE (EC2)"]
        direction TB
        Planner["📋 Planner<br/>DAG Synthesis"]
        LLMEngine["🤖 LLM Engine<br/>Inference"]
        RAGService["📚 RAG Service<br/>Vector Search"]
        ToolRegistry["🔧 Tool Registry<br/>API Calls"]
    end
    
    subgraph VectorDB["🔢 VECTOR DATABASE"]
        S3Vectors["S3 Bucket<br/>Embeddings"]
        OpenSearchVectors["OpenSearch<br/>Vector Index"]
    end
    
    subgraph Analytics["📊 ANALYTICS & FEEDBACK"]
        Redshift["Redshift<br/>Performance Metrics<br/>Bias Detection"]
        CloudWatch["CloudWatch<br/>Model Metrics<br/>Latency"]
    end
    
    %% Input Flow
    MuddaData --> LanguageDetect
    MuddaData --> HateSpeech
    MuddaData --> NSFWFilter
    MuddaData --> Duplicate
    MuddaData --> Categorize
    MuddaData --> OCR
    
    %% Lambda to Models
    LanguageDetect --> SM4
    HateSpeech --> SM1
    NSFWFilter --> SM2
    Categorize --> SM3
    Duplicate --> SM4
    
    %% Agentic AI Flow
    MuddaData --> Planner
    Planner --> RAGService
    RAGService --> S3Vectors
    RAGService --> OpenSearchVectors
    Planner --> LLMEngine
    LLMEngine --> Claude
    LLMEngine --> Llama
    LLMEngine --> ToolRegistry
    
    %% Monitoring
    ContentAnalysisLambda --> CloudWatch
    AgenticAI --> CloudWatch
    SageMakerEndpoints --> CloudWatch
    
    %% Feedback Loop
    Redshift -.->|Threshold Tuning| AgenticAI
    Redshift -.->|Model Retraining| SageMakerEndpoints
    
    %% Styling
    classDef inputStyle fill:#e1f5ff,stroke:#01579b,stroke-width:2px
    classDef lambdaStyle fill:#FF9800,stroke:#E65100,stroke-width:2px
    classDef modelStyle fill:#9C27B0,stroke:#6A1B9A,stroke-width:3px
    classDef agenticStyle fill:#F44336,stroke:#C62828,stroke-width:3px
    classDef vectorStyle fill:#00BCD4,stroke:#00838F,stroke-width:2px
    classDef analyticsStyle fill:#4CAF50,stroke:#2E7D32,stroke-width:2px
    
    class MuddaData inputStyle
    class LanguageDetect,HateSpeech,NSFWFilter,Duplicate,Categorize,OCR lambdaStyle
    class SM1,SM2,SM3,SM4,Claude,Llama modelStyle
    class Planner,LLMEngine,RAGService,ToolRegistry agenticStyle
    class S3Vectors,OpenSearchVectors vectorStyle
    class Redshift,CloudWatch analyticsStyle
```

---

## 5. Security & Compliance Architecture

This diagram shows the security layers and compliance controls.

```mermaid
graph TB
    subgraph External["🌐 EXTERNAL"]
        Users["Users"]
        Attackers["⚠️ Potential<br/>Threats"]
    end
    
    subgraph EdgeSecurity["🛡️ EDGE SECURITY"]
        WAF["AWS WAF<br/>• SQL Injection<br/>• XSS Protection<br/>• Rate Limiting<br/>• Geo Blocking"]
        Shield["AWS Shield<br/>DDoS Protection"]
        CloudFront["CloudFront<br/>• HTTPS Only<br/>• Edge Caching<br/>• Origin Protection"]
    end
    
    subgraph NetworkSecurity["🔒 NETWORK SECURITY"]
        direction TB
        
        VPC["🏢 VPC<br/>10.0.0.0/16"]
        
        subgraph PublicSubnets["Public Subnets"]
            ALB["Application<br/>Load Balancer"]
            NAT["NAT Gateway"]
        end
        
        subgraph PrivateSubnets["Private Subnets"]
            EC2Private["EC2 Instances<br/>Spring Boot"]
            LambdaVPC["Lambda<br/>in VPC"]
            RDSPrivate["RDS<br/>PostgreSQL"]
        end
        
        SecurityGroups["🔐 Security Groups<br/>Firewall Rules"]
        NACL["Network ACLs<br/>Subnet Rules"]
    end
    
    subgraph IdentitySecurity["👤 IDENTITY & ACCESS"]
        Cognito["Amazon Cognito<br/>• User Pools<br/>• JWT Tokens<br/>• MFA Support"]
        IAM["AWS IAM<br/>• Roles<br/>• Policies<br/>• Least Privilege"]
    end
    
    subgraph DataSecurity["🔐 DATA SECURITY"]
        KMS["AWS KMS<br/>• Encryption Keys<br/>• Key Rotation<br/>• Audit Logs"]
        SecretsManager["Secrets Manager<br/>• API Keys<br/>• DB Credentials<br/>• Auto Rotation"]
        ACM["AWS ACM<br/>SSL/TLS<br/>Certificates"]
    end
    
    subgraph ComplianceMonitoring["📋 COMPLIANCE & MONITORING"]
        CloudTrail["CloudTrail<br/>API Audit Logs<br/>Compliance Trail"]
        Config["AWS Config<br/>Resource<br/>Compliance"]
        GuardDuty["GuardDuty<br/>Threat<br/>Detection"]
        SecurityHub["Security Hub<br/>Security<br/>Posture"]
    end
    
    subgraph DataProtection["🔒 DATA PROTECTION"]
        Encryption["Encryption<br/>• At Rest (KMS)<br/>• In Transit (TLS)<br/>• End-to-End"]
        Backup["AWS Backup<br/>• RDS Snapshots<br/>• S3 Versioning<br/>• Point-in-Time"]
        PIISanitization["PII Sanitization<br/>• Pattern Detection<br/>• Redaction<br/>• Compliance"]
    end
    
    %% Threat Flow
    Attackers -.->|Blocked| WAF
    Attackers -.->|Blocked| Shield
    
    %% User Flow
    Users --> CloudFront
    CloudFront --> WAF
    WAF --> Shield
    Shield --> ALB
    
    %% Network Flow
    ALB --> SecurityGroups
    SecurityGroups --> EC2Private
    SecurityGroups --> LambdaVPC
    SecurityGroups --> RDSPrivate
    NACL -.->|Subnet Rules| PrivateSubnets
    
    %% Identity
    Users --> Cognito
    EC2Private --> IAM
    LambdaVPC --> IAM
    
    %% Data Security
    RDSPrivate --> KMS
    EC2Private --> SecretsManager
    CloudFront --> ACM
    ALB --> ACM
    
    %% Monitoring
    VPC --> CloudTrail
    IAM --> CloudTrail
    KMS --> CloudTrail
    VPC --> Config
    VPC --> GuardDuty
    CloudTrail --> SecurityHub
    Config --> SecurityHub
    GuardDuty --> SecurityHub
    
    %% Data Protection
    RDSPrivate --> Encryption
    RDSPrivate --> Backup
    EC2Private --> PIISanitization
    
    %% Styling
    classDef threatStyle fill:#f44336,stroke:#c62828,stroke-width:2px
    classDef edgeStyle fill:#ff9800,stroke:#e65100,stroke-width:3px
    classDef networkStyle fill:#2196f3,stroke:#1565c0,stroke-width:2px
    classDef identityStyle fill:#4caf50,stroke:#2e7d32,stroke-width:2px
    classDef dataStyle fill:#9c27b0,stroke:#6a1b9a,stroke-width:3px
    classDef complianceStyle fill:#00bcd4,stroke:#00838f,stroke-width:2px
    classDef protectionStyle fill:#607d8b,stroke:#37474f,stroke-width:2px
    
    class Attackers threatStyle
    class WAF,Shield,CloudFront edgeStyle
    class VPC,ALB,NAT,EC2Private,LambdaVPC,RDSPrivate,SecurityGroups,NACL networkStyle
    class Cognito,IAM identityStyle
    class KMS,SecretsManager,ACM dataStyle
    class CloudTrail,Config,GuardDuty,SecurityHub complianceStyle
    class Encryption,Backup,PIISanitization protectionStyle
```

---

## 6. Deployment & CI/CD Architecture

This diagram shows the deployment pipeline and infrastructure as code setup.

```mermaid
graph LR
    subgraph Development["👨‍💻 DEVELOPMENT"]
        Developer["Developer<br/>Local Dev"]
        GitRepo["📦 Git Repository<br/>GitHub/CodeCommit"]
    end
    
    subgraph CICD["🔄 CI/CD PIPELINE"]
        direction TB
        
        CodePipeline["AWS CodePipeline<br/>Orchestration"]
        
        subgraph BuildStage["🔨 BUILD STAGE"]
            CodeBuild["CodeBuild<br/>• Maven/Gradle<br/>• npm build<br/>• Docker build<br/>• Unit Tests"]
        end
        
        subgraph TestStage["🧪 TEST STAGE"]
            IntegrationTests["Integration Tests<br/>Test Environment"]
            SecurityScan["Security Scan<br/>• SAST<br/>• Dependency Check"]
        end
        
        subgraph DeployStage["🚀 DEPLOY STAGE"]
            DeployDev["Deploy to Dev"]
            DeployStaging["Deploy to Staging"]
            DeployProd["Deploy to Prod<br/>Manual Approval"]
        end
    end
    
    subgraph IaC["📜 INFRASTRUCTURE AS CODE"]
        CloudFormation["CloudFormation<br/>Templates"]
        CDK["AWS CDK<br/>TypeScript/Python"]
        Terraform["Terraform<br/>(Optional)"]
    end
    
    subgraph ArtifactStorage["📦 ARTIFACT STORAGE"]
        ECR["Amazon ECR<br/>Docker Images"]
        S3Artifacts["S3 Bucket<br/>Build Artifacts"]
        CodeArtifact["CodeArtifact<br/>Maven/npm<br/>Packages"]
    end
    
    subgraph Environments["🌍 ENVIRONMENTS"]
        direction TB
        
        Dev["🔧 Development<br/>• Single AZ<br/>• Smaller Instances<br/>• Test Data"]
        Staging["🎭 Staging<br/>• Multi-AZ<br/>• Prod-like Config<br/>• Sanitized Data"]
        Prod["🚀 Production<br/>• Multi-AZ<br/>• Auto Scaling<br/>• Real Data<br/>• High Availability"]
    end
    
    %% Development Flow
    Developer --> GitRepo
    GitRepo -->|Webhook| CodePipeline
    
    %% Pipeline Flow
    CodePipeline --> CodeBuild
    CodeBuild --> IntegrationTests
    IntegrationTests --> SecurityScan
    SecurityScan --> DeployDev
    DeployDev --> DeployStaging
    DeployStaging --> DeployProd
    
    %% IaC Flow
    Developer --> CloudFormation
    Developer --> CDK
    CDK --> CloudFormation
    CloudFormation --> Environments
    
    %% Artifact Flow
    CodeBuild --> ECR
    CodeBuild --> S3Artifacts
    CodeBuild --> CodeArtifact
    
    %% Deployment to Environments
    DeployDev --> Dev
    DeployStaging --> Staging
    DeployProd --> Prod
    
    ECR --> Dev
    ECR --> Staging
    ECR --> Prod
    
    %% Styling
    classDef devStyle fill:#4caf50,stroke:#2e7d32,stroke-width:2px
    classDef cicdStyle fill:#2196f3,stroke:#1565c0,stroke-width:3px
    classDef iacStyle fill:#ff9800,stroke:#e65100,stroke-width:2px
    classDef artifactStyle fill:#9c27b0,stroke:#6a1b9a,stroke-width:2px
    classDef envStyle fill:#00bcd4,stroke:#00838f,stroke-width:3px
    
    class Developer,GitRepo devStyle
    class CodePipeline,CodeBuild,IntegrationTests,SecurityScan,DeployDev,DeployStaging,DeployProd cicdStyle
    class CloudFormation,CDK,Terraform iacStyle
    class ECR,S3Artifacts,CodeArtifact artifactStyle
    class Dev,Staging,Prod envStyle
```

---

## 7. Monitoring & Observability Architecture

This diagram shows the complete monitoring and observability stack.

```mermaid
graph TB
    subgraph Services["⚙️ SERVICES"]
        SpringBoot["Spring Boot<br/>Services"]
        Lambda["Lambda<br/>Functions"]
        AIService["AI Services"]
        RDS["RDS"]
        MSK["MSK"]
    end
    
    subgraph Metrics["📊 METRICS COLLECTION"]
        CloudWatchMetrics["CloudWatch Metrics<br/>• CPU/Memory<br/>• Request Count<br/>• Latency<br/>• Error Rate"]
        CustomMetrics["Custom Metrics<br/>• Business KPIs<br/>• AI Confidence<br/>• User Engagement"]
    end
    
    subgraph Logs["📝 LOGGING"]
        CloudWatchLogs["CloudWatch Logs<br/>• Application Logs<br/>• Access Logs<br/>• Error Logs"]
        LogGroups["Log Groups<br/>• /aws/lambda/*<br/>• /aws/elasticbeanstalk/*<br/>• /aws/rds/*"]
        LogInsights["CloudWatch Logs Insights<br/>Query & Analysis"]
    end
    
    subgraph Tracing["🔍 DISTRIBUTED TRACING"]
        XRay["AWS X-Ray<br/>• Request Tracing<br/>• Service Map<br/>• Latency Analysis<br/>• Error Detection"]
    end
    
    subgraph Dashboards["📈 DASHBOARDS & VISUALIZATION"]
        CWDashboards["CloudWatch Dashboards<br/>• System Health<br/>• AI Performance<br/>• Business Metrics"]
        Grafana["Grafana<br/>(Optional)<br/>Advanced<br/>Visualization"]
    end
    
    subgraph Alerting["🚨 ALERTING & NOTIFICATIONS"]
        CloudWatchAlarms["CloudWatch Alarms<br/>• Threshold Alerts<br/>• Anomaly Detection<br/>• Composite Alarms"]
        SNSAlerts["SNS Topics<br/>• Email<br/>• Slack<br/>• PagerDuty"]
        EventBridge["EventBridge Rules<br/>Automated<br/>Remediation"]
    end
    
    subgraph AIMonitoring["🤖 AI-SPECIFIC MONITORING"]
        ModelMetrics["Model Metrics<br/>• Accuracy<br/>• Latency<br/>• Confidence Scores"]
        BiasDetection["Bias Detection<br/>• Regional Fairness<br/>• Language Parity"]
        DriftDetection["Drift Detection<br/>• Performance Decay<br/>• Distribution Shift"]
    end
    
    %% Services to Metrics
    SpringBoot --> CloudWatchMetrics
    Lambda --> CloudWatchMetrics
    AIService --> CloudWatchMetrics
    RDS --> CloudWatchMetrics
    MSK --> CloudWatchMetrics
    
    SpringBoot --> CustomMetrics
    AIService --> CustomMetrics
    
    %% Services to Logs
    SpringBoot --> CloudWatchLogs
    Lambda --> CloudWatchLogs
    AIService --> CloudWatchLogs
    RDS --> CloudWatchLogs
    CloudWatchLogs --> LogGroups
    LogGroups --> LogInsights
    
    %% Services to Tracing
    SpringBoot --> XRay
    Lambda --> XRay
    AIService --> XRay
    
    %% Metrics to Dashboards
    CloudWatchMetrics --> CWDashboards
    CustomMetrics --> CWDashboards
    CloudWatchMetrics --> Grafana
    XRay --> CWDashboards
    
    %% Alerting Flow
    CloudWatchMetrics --> CloudWatchAlarms
    CloudWatchLogs --> CloudWatchAlarms
    CloudWatchAlarms --> SNSAlerts
    CloudWatchAlarms --> EventBridge
    
    %% AI Monitoring
    AIService --> ModelMetrics
    AIService --> BiasDetection
    AIService --> DriftDetection
    ModelMetrics --> CWDashboards
    BiasDetection --> CloudWatchAlarms
    DriftDetection --> CloudWatchAlarms
    
    %% Styling
    classDef serviceStyle fill:#4caf50,stroke:#2e7d32,stroke-width:2px
    classDef metricsStyle fill:#2196f3,stroke:#1565c0,stroke-width:2px
    classDef logsStyle fill:#ff9800,stroke:#e65100,stroke-width:2px
    classDef traceStyle fill:#9c27b0,stroke:#6a1b9a,stroke-width:2px
    classDef dashStyle fill:#00bcd4,stroke:#00838f,stroke-width:3px
    classDef alertStyle fill:#f44336,stroke:#c62828,stroke-width:3px
    classDef aiMonitorStyle fill:#673ab7,stroke:#4527a0,stroke-width:2px
    
    class SpringBoot,Lambda,AIService,RDS,MSK serviceStyle
    class CloudWatchMetrics,CustomMetrics metricsStyle
    class CloudWatchLogs,LogGroups,LogInsights logsStyle
    class XRay traceStyle
    class CWDashboards,Grafana dashStyle
    class CloudWatchAlarms,SNSAlerts,EventBridge alertStyle
    class ModelMetrics,BiasDetection,DriftDetection aiMonitorStyle
```

---

## Summary

This architecture document provides comprehensive AWS infrastructure diagrams for the Mudda platform covering:

1. **Complete Infrastructure** - All AWS services, networking, and data flows
2. **Event-Driven Architecture** - Amazon MSK cluster with Kafka topics and consumers
3. **Data Architecture** - RDS, S3, OpenSearch, Redshift with data pipelines
4. **AI/ML Services** - Content analysis Lambda functions, SageMaker models, Bedrock LLMs, and Agentic AI
5. **Security & Compliance** - WAF, VPC, encryption, identity management, and monitoring
6. **CI/CD Pipeline** - CodePipeline, build/test/deploy stages, and multi-environment setup
7. **Monitoring & Observability** - CloudWatch metrics/logs, X-Ray tracing, dashboards, and AI-specific monitoring

All diagrams use Mermaid syntax and can be rendered in any Markdown viewer that supports Mermaid diagrams.
