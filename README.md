# Query Enrichment Implementation Diagrams

## Advanced Workflow Diagrams

### 1. Complete Query Processing Pipeline

```mermaid
graph TB
    subgraph "Input Processing"
        A[User Query] --> B[Query Analysis]
        B --> C[Intent Classification]
    end
    
    subgraph "Routing Decision"
        C --> D{Technical vs Process}
        D -->|Technical| E[Technical Path]
        D -->|Process| F[Process Path]
        D -->|Summary| G[Summary Path]
    end
    
    subgraph "Technical Processing"
        E --> H[Context Validation]
        H --> I{Context Available?}
        I -->|Yes| J[Enrich Query]
        I -->|No| K[Request Context]
        J --> L[retrieve_knowledge_passage]
        K --> M[Wait for User Input]
        M --> J
    end
    
    subgraph "Process Processing"
        F --> N[Extract Case Details]
        N --> O[Add Contextual Info]
        O --> P[retrieve]
    end
    
    subgraph "Summary Processing"
        G --> Q{Contains 'ai_case_summary'?}
        Q -->|Yes| R[Format Structured Response]
        Q -->|No| S[Standard Summary]
    end
    
    subgraph "Output Generation"
        L --> T[Technical Response]
        P --> U[Process Response]
        R --> V[Structured Summary]
        S --> W[Standard Summary]
    end
    
    T --> X[Cite Sources]
    U --> Y[Provide References]
    V --> Z[Formatted Output]
    W --> AA[Regular Output]
```

### 2. Context Enrichment State Machine

```mermaid
stateDiagram-v2
    [*] --> QueryReceived
    
    QueryReceived --> ClassifyQuery
    ClassifyQuery --> Technical : Hardware/Software Issue
    ClassifyQuery --> Process : Procedural Question
    ClassifyQuery --> Direct : General/Summary
    
    Technical --> ValidateContext
    ValidateContext --> ContextFound : Context in History
    ValidateContext --> RequestContext : No Context Found
    
    ContextFound --> EnrichQuery
    RequestContext --> WaitingForContext
    WaitingForContext --> ContextProvided : User Provides Info
    WaitingForContext --> ContextTimeout : No Response
    
    ContextProvided --> EnrichQuery
    EnrichQuery --> CallCoveo
    CallCoveo --> ProcessResults
    ProcessResults --> [*]
    
    Process --> ExtractCaseData
    ExtractCaseData --> CallRetriever
    CallRetriever --> ProcessResults
    
    Direct --> GenerateResponse
    GenerateResponse --> [*]
    
    ContextTimeout --> FallbackResponse
    FallbackResponse --> [*]
```

### 3. Context Inheritance Pattern

```mermaid
graph TD
    subgraph "Session Context"
        A[Initial Context Extraction] --> B[Product Names Storage]
        B --> C[Error Codes Storage]
        C --> D[Problem History Storage]
    end
    
    subgraph "Query 1 - Technical"
        E[Technical Query 1] --> F[Use Stored Context]
        F --> G[Call Coveo]
        G --> H[Update Context if New Info]
    end
    
    subgraph "Query 2 - Process"
        I[Process Query] --> J[Context Preserved]
        J --> K[Call Retriever]
    end
    
    subgraph "Query 3 - Technical Follow-up"
        L[Technical Query 3] --> M[Reuse Original Context]
        M --> N[Call Coveo]
        N --> O[Context Still Valid]
    end
    
    A --> E
    H --> I
    O --> P[Session Continues...]
    
    style B fill:#e1f5fe
    style H fill:#f3e5f5
    style J fill:#e8f5e8
    style O fill:#fff3e0
```

### 4. Validation Gate System

```mermaid
graph TD
    A[Technical Query Input] --> B[Gate 1: Query Clarity]
    
    B --> C{Specific & Clear?}
    C -->|Pass| D[Gate 2: Context Check]
    C -->|Fail| E[Request Clarification]
    
    D --> F{Product Name Available?}
    F -->|Pass| G[Gate 3: Conversation Understanding]
    F -->|Fail| H[Search History]
    
    H --> I{Found in History?}
    I -->|Yes| G
    I -->|No| J[Request from User]
    
    G --> K{Understand Problem?}
    K -->|Pass| L[All Gates Passed]
    K -->|Fail| M[Review Conversation]
    
    L --> N[Execute retrieve_knowledge_passage]
    
    E --> O[User Response]
    J --> P[User Provides Context]
    M --> Q[Ask for Clarification]
    
    O --> A
    P --> G
    Q --> A
    
    style L fill:#4CAF50
    style N fill:#2196F3
    style E fill:#FF5722
    style J fill:#FF9800
```

### 5. Multi-Turn Conversation Flow

```mermaid
sequenceDiagram
    participant User
    participant Agent
    participant ContextManager
    participant Coveo
    participant Retriever
    
    Note over User,Retriever: Initial Technical Query
    User->>Agent: "Server won't boot"
    Agent->>ContextManager: Check for context
    ContextManager-->>Agent: No context found
    Agent->>User: "Need Product Name"
    
    User->>Agent: "HPE ProLiant ML350"
    Agent->>ContextManager: Store context
    ContextManager-->>Agent: Context stored
    Agent->>Coveo: Query + Context
    Coveo-->>Agent: Technical results
    Agent->>User: Boot troubleshooting steps
    
    Note over User,Retriever: Process Question
    User->>Agent: "How to order replacement part?"
    Agent->>ContextManager: Context preserved
    Agent->>Retriever: Process query
    Retriever-->>Agent: Process results
    Agent->>User: Part ordering process
    
    Note over User,Retriever: Follow-up Technical
    User->>Agent: "What's next step?"
    Agent->>ContextManager: Retrieve stored context
    ContextManager-->>Agent: HPE ProLiant ML350 + boot issue
    Agent->>Coveo: Follow-up query + context
    Coveo-->>Agent: Next troubleshooting steps
    Agent->>User: Advanced boot diagnostics
```

### 6. Error Handling and Fallbacks

```mermaid
graph TD
    A[Query Processing] --> B{Processing Error?}
    
    B -->|No Error| C[Normal Flow]
    B -->|Context Error| D[Context Fallback]
    B -->|Tool Error| E[Tool Fallback]
    B -->|Response Error| F[Response Fallback]
    
    D --> G[Use Basic Query]
    D --> H[Request Context Again]
    D --> I[Use Previous Context]
    
    E --> J[Try Alternative Tool]
    E --> K[Use Direct Response]
    E --> L[Escalate to Human]
    
    F --> M[Provide Generic Response]
    F --> N[Request Query Rephrase]
    F --> O[Offer Alternative Help]
    
    G --> P[Execute with Limited Context]
    H --> Q[Wait for User Input]
    I --> R[Proceed with Cached Data]
    
    style D fill:#ffeb3b
    style E fill:#ff9800
    style F fill:#f44336
```

### 7. Context Quality Assessment

```mermaid
graph LR
    A[Context Input] --> B[Quality Assessment]
    
    B --> C{Product Name?}
    B --> D{Error Codes?}
    B --> E{Problem Description?}
    B --> F{Previous Steps?}
    
    C -->|Present| G[High Quality +3]
    C -->|Missing| H[Low Quality +0]
    
    D -->|Present| I[Medium Quality +2]
    D -->|Missing| J[Low Quality +0]
    
    E -->|Present| K[Medium Quality +2]
    E -->|Missing| L[Low Quality +0]
    
    F -->|Present| M[Low Quality +1]
    F -->|Missing| N[No Quality +0]
    
    G --> O[Calculate Total Score]
    I --> O
    K --> O
    M --> O
    H --> O
    J --> O
    L --> O
    N --> O
    
    O --> P{Score >= 5?}
    P -->|Yes| Q[High Quality Context]
    P -->|No| R{Score >= 3?}
    R -->|Yes| S[Acceptable Context]
    R -->|No| T[Insufficient Context]
    
    style Q fill:#4caf50
    style S fill:#ffeb3b
    style T fill:#f44336
```

### 8. Tool Integration Architecture

```mermaid
graph TB
    subgraph "Agent Core"
        A[Query Router] --> B[Context Manager]
        B --> C[Validation Engine]
        C --> D[Tool Selector]
    end
    
    subgraph "Technical Tools"
        D --> E[retrieve_knowledge_passage]
        E --> F[Coveo API]
        F --> G[Knowledge Base]
    end
    
    subgraph "Process Tools"
        D --> H[retrieve]
        H --> I[Internal Retriever]
        I --> J[Documentation DB]
    end
    
    subgraph "Utility Tools"
        D --> K[kcs_tool]
        K --> L[Draft Article Creator]
        L --> M[KCS System]
    end
    
    subgraph "Response Processing"
        G --> N[Result Processor]
        J --> N
        M --> N
        N --> O[Response Formatter]
        O --> P[Citation Generator]
        P --> Q[Final Response]
    end
    
    style A fill:#2196f3
    style F fill:#ff9800
    style I fill:#4caf50
    style L fill:#9c27b0
```

## Implementation Guide

### Context Structure Implementation

```typescript
interface QueryContext {
  productNames: string[];
  errorCodes: string[];
  problemDescription: string;
  stepsAttempted: string[];
  timestamp: Date;
  quality: ContextQuality;
}

interface ContextQuality {
  score: number;
  hasProductName: boolean;
  hasErrorDetails: boolean;
  hasProblemDescription: boolean;
  level: 'high' | 'medium' | 'low' | 'insufficient';
}
```

### Validation Rules Implementation

```python
def validate_technical_query(query, context, conversation_history):
    """
    Implement three-gate validation system
    """
    # Gate 1: Query Clarity
    if not is_specific_and_clear(query, conversation_history):
        return ValidationResult.NEEDS_CLARIFICATION
    
    # Gate 2: Context Check
    if not has_sufficient_context(context, conversation_history):
        return ValidationResult.NEEDS_CONTEXT
    
    # Gate 3: Conversation Understanding
    if not understands_problem_context(query, conversation_history):
        return ValidationResult.NEEDS_REVIEW
    
    return ValidationResult.PASSED
```

This comprehensive implementation guide provides the technical foundation for implementing the query enrichment system with proper error handling, context management, and validation processes.
