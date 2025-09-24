# DECA AI Studio - System Architecture Diagram

```mermaid
graph TB
    %% External Clients and Triggers
    Client[Web UI Client] --> Auth0{Auth0}
    WebhookSource[External Webhook Sources] --> WHReceiver[Webhook Receiver]
    Scheduler[Cron Scheduler] --> SchDispatcher[Schedule Dispatcher]

    %% Authentication
    Auth0 --> APIServer[API Server<br/>:8080]

    %% Core API and Entry Point
    APIServer --> MongoDB[(MongoDB<br/>Replica Set)]
    APIServer --> Redis[(Redis<br/>Cache)]
    APIServer --> NATS[NATS JetStream<br/>Message Queue]

    %% Service Communication via NATS
    NATS --> FlowManager[Flow Manager<br/>:8081]
    NATS --> NodeRunner[Node Runner<br/>:8082]
    NATS --> EventDispatcher[Event Dispatcher<br/>:8083]
    NATS --> TaskRunner[Task Runner<br/>:8084]

    %% Specialized Services
    APIServer -.->|HTTP| AgentServer[Agent Server<br/>:8000<br/>Python/FastAPI]
    APIServer -.->|HTTP| FunctionServer[Function Server<br/>:8085]
    APIServer -.->|HTTP| PromptServer[Prompt Server<br/>:8001<br/>Python/FastAPI]

    %% MCP Server
    MCPServer[MCP Server<br/>:8002<br/>Python] --> CRM[CRM Integration]

    %% Flow Execution Chain
    FlowManager --> NodeRunner
    NodeRunner --> AgentServer
    NodeRunner --> FunctionServer
    NodeRunner --> PromptServer

    %% Event Flow
    WHReceiver --> EventDispatcher
    SchDispatcher --> EventDispatcher
    EventDispatcher --> FlowManager

    %% Background Processing
    FlowManager --> NATS
    NATS --> TaskRunner

    %% Shared Data Stores
    FlowManager --> MongoDB
    NodeRunner --> MongoDB
    AgentServer --> MongoDB
    FunctionServer --> MongoDB
    TaskRunner --> MongoDB

    FlowManager --> Redis
    AgentServer --> Redis

    %% External Integrations
    AgentServer --> OpenAI[OpenAI API]
    AgentServer --> AnthropicAPI[Anthropic API]
    NodeRunner --> GoogleAPI[Google APIs]
    NodeRunner --> ZendeskAPI[Zendesk API]
    NodeRunner --> S3[AWS S3]

    %% Web UI App
    UIApp[Agent App UI<br/>React/TypeScript<br/>:3000] --> APIServer
    UIApp --> Auth0

    %% Shared Go Components
    subgraph "Go Internals Package"
        Config[Config Management]
        Database[Database Layer]
        Logger[Centralized Logging]
        Models[Shared Models]
        Middleware[HTTP Middleware]
        Utils[Utility Functions]
    end

    %% Dependencies
    FlowManager -.-> Config
    NodeRunner -.-> Database
    APIServer -.-> Logger
    FunctionServer -.-> Models
    TaskRunner -.-> Middleware
    EventDispatcher -.-> Utils

    %% Node Types Available
    subgraph "Node Types (35+)"
        CoreNodes[Agent, Function, Condition<br/>Loop, Transform, Branch]
        TriggerNodes[Webhook, Schedule, Event<br/>Triggers]
        DecaNodes[DECA Chatbot, Pages, Forms<br/>Knowledge Base, CRM, Tables]
        IntegrationNodes[OpenAI, Google, Zendesk<br/>Code Execution, Wait]
    end

    NodeRunner -.-> CoreNodes
    NodeRunner -.-> TriggerNodes
    NodeRunner -.-> DecaNodes
    NodeRunner -.-> IntegrationNodes

    %% Styling
    classDef goService fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef pythonService fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef webService fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef database fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    classDef external fill:#fce4ec,stroke:#880e4f,stroke-width:2px
    classDef messaging fill:#fff8e1,stroke:#ff8f00,stroke-width:2px

    class APIServer,FlowManager,NodeRunner,EventDispatcher,FunctionServer,TaskRunner,SchDispatcher,WHReceiver goService
    class AgentServer,PromptServer,MCPServer pythonService
    class UIApp,Client webService
    class MongoDB,Redis database
    class Auth0,OpenAI,AnthropicAPI,GoogleAPI,ZendeskAPI,S3,CRM external
    class NATS messaging
```

## Data Flow Diagram

```mermaid
sequenceDiagram
    participant C as Client
    participant A as API Server
    participant N as NATS
    participant FM as Flow Manager
    participant NR as Node Runner
    participant AS as Agent Server
    participant FS as Function Server
    participant DB as MongoDB

    Note over C,DB: Flow Execution Sequence

    C->>A: Execute Flow Request
    A->>DB: Validate & Store Request
    A->>N: Publish FlowExecuteMessage
    N->>FM: Deliver Message

    FM->>DB: Load Flow Definition
    FM->>N: Publish NodeExecuteMessage
    N->>NR: Deliver Message

    alt Agent Node
        NR->>AS: HTTP Request
        AS-->>NR: Response
    else Function Node
        NR->>FS: HTTP Request
        FS-->>NR: Response
    else Condition Node
        NR->>NR: Evaluate Condition
    end

    NR->>N: Publish NodeResultMessage
    N->>FM: Deliver Message
    FM->>DB: Store Node Result

    alt More Nodes
        FM->>N: Next NodeExecuteMessage
    else Flow Complete
        FM->>N: Publish FlowResultMessage
        N->>A: Deliver Message
        A->>C: Return Results
    end
```

## Component Interaction Matrix

| Component | API Server | Flow Manager | Node Runner | Agent Server | Function Server | NATS | MongoDB | Redis |
|-----------|------------|--------------|-------------|--------------|----------------|------|---------|-------|
| **API Server** | - | ✓ (NATS) | ✓ (NATS) | ✓ (HTTP) | ✓ (HTTP) | ✓ | ✓ | ✓ |
| **Flow Manager** | ✓ (NATS) | - | ✓ (NATS) | - | - | ✓ | ✓ | ✓ |
| **Node Runner** | ✓ (NATS) | ✓ (NATS) | - | ✓ (HTTP) | ✓ (HTTP) | ✓ | ✓ | - |
| **Agent Server** | ✓ (HTTP) | - | ✓ (HTTP) | - | ✓ (HTTP) | - | ✓ | ✓ |
| **Function Server** | ✓ (HTTP) | - | ✓ (HTTP) | ✓ (HTTP) | - | - | ✓ | - |
| **Event Dispatcher** | ✓ (NATS) | ✓ (NATS) | - | - | - | ✓ | ✓ | - |
| **Task Runner** | - | ✓ (NATS) | - | - | - | ✓ | ✓ | - |
| **Web UI** | ✓ (HTTP) | - | - | - | - | - | - | - |

## Infrastructure Deployment View

```mermaid
graph TB
    subgraph "AWS Cloud"
        subgraph "ECS Cluster"
            subgraph "API Tier"
                API[API Server<br/>Fargate Service]
                UI[Web UI<br/>Fargate Service]
            end

            subgraph "Processing Tier"
                FM[Flow Manager<br/>Fargate Service]
                NR[Node Runner<br/>Fargate Service]
                ED[Event Dispatcher<br/>Fargate Service]
                TR[Task Runner<br/>Fargate Service]
            end

            subgraph "AI/Function Tier"
                AS[Agent Server<br/>Fargate Service]
                FS[Function Server<br/>Fargate Service]
                PS[Prompt Server<br/>Fargate Service]
            end
        end

        subgraph "Data Services"
            MongoDB[(MongoDB Atlas<br/>Replica Set)]
            Redis[(ElastiCache<br/>Redis)]
            NATS[(NATS JetStream<br/>EC2/ECS)]
            S3[(S3 Bucket<br/>File Storage)]
        end

        subgraph "Security & Monitoring"
            KMS[AWS KMS<br/>Key Management]
            CW[CloudWatch<br/>Monitoring]
            ECR[ECR<br/>Container Registry]
        end
    end

    subgraph "External Services"
        Auth0[Auth0<br/>Authentication]
        OpenAI[OpenAI API]
        Anthropic[Anthropic API]
    end

    subgraph "Load Balancer"
        ALB[Application Load Balancer]
    end

    Internet --> ALB
    ALB --> API
    ALB --> UI

    API --> MongoDB
    API --> Redis
    API --> NATS

    FM --> NATS
    NR --> NATS
    ED --> NATS
    TR --> NATS

    AS --> MongoDB
    AS --> Redis
    AS --> OpenAI
    AS --> Anthropic

    FS --> MongoDB
    PS --> MongoDB

    UI --> Auth0
    API --> Auth0
    API --> S3

    CW --> API
    CW --> FM
    CW --> AS

    ECR --> API
    ECR --> FM
    ECR --> AS

    KMS --> API
    KMS --> FS
```

## Message Flow Patterns

### Flow Execution Pattern
```
Trigger → [FlowExecuteMessage] → Flow Manager
Flow Manager → [NodeExecuteMessage] → Node Runner
Node Runner → [NodeResultMessage] → Flow Manager
Flow Manager → [FlowCompleteMessage] → API Server
```

### Event Processing Pattern
```
Webhook → [WebhookEventMessage] → Event Dispatcher
Schedule → [ScheduleEventMessage] → Event Dispatcher
Event Dispatcher → [EventMessage] → Flow Manager
```

### Task Processing Pattern
```
Flow Manager → [TaskExecuteMessage] → Task Runner
Task Runner → [TaskResultMessage] → Flow Manager
```