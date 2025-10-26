## Arquitetura Proposta

````mermaid
graph TB
    subgraph Frontend["Camada de Apresentação"]
        FE[<strong>Frontend React</strong><br/>- Interface Web<br/>- Dashboard<br/>- Formulários]
    end

    subgraph Gateway["Camada Middle-end - API Gateway"]
        GW[API Gateway<br/>FastAPI]
        AUTH[Auth Middleware<br/>JWT Validation]
        ROUTE[Router<br/>Direcionamento]

        GW --> AUTH
        AUTH --> ROUTE
    end

    subgraph Backend["Camada de Serviços - Backend"]
        direction LR

        BE[Backend API<br/>FastAPI<br/><br/>Autenticação<br/>CRUD Pacientes<br/>CRUD Solicitações<br/>CRUD Exames<br/>CRUD Laudos<br/>Business Logic]

        LLM[AI Service<br/>FastAPI<br/><br/>Análise de Imagens<br/>???<br/>Cache Redis]

        EMAIL[Email Service<br/>FastAPI<br/><br/>SMTP/SendGrid<br/>Templates<br/>Queue<br/>Logs]
    end

    subgraph Data["Camada de Dados"]
        DB[(PostgreSQL<br/><br/>Usuários<br/>Solicitações<br/>Exames<br/>Laudos)]
        CACHE[(Redis<br/><br/>Cache LLM<br/>Sessions<br/>Queue)]
        S3[Object Storage<br/>???<br/><br/>Imagens<br/>PDFs<br/>]
    end

    subgraph External["Serviços Externos"]
        OPENAI[Nome da API da IA<br/>???]
        SMTP[SMTP Provider<br/>SendGrid/AWS SES]
    end

    %% Conexões Frontend -> Gateway
    FE -->|HTTPS| GW

    %% Conexões Gateway -> Serviços
    ROUTE -->|REST| BE
    ROUTE -->|REST| LLM
    ROUTE -->|REST| EMAIL

    %% Conexões Serviços -> Dados
    BE -->|SQL| DB
    BE -->|Upload/Download| S3
    LLM -->|Cache| CACHE
    LLM -->|Read| S3
    EMAIL -->|Queue| CACHE

    %% Conexões Serviços -> Externos
    LLM -->|API| OPENAI
    EMAIL -->|SMTP| SMTP

    %% Estilos
    classDef frontend fill:#61dafb,stroke:#333,stroke-width:2px,color:#000
    classDef gateway fill:#68a063,stroke:#333,stroke-width:2px,color:#fff
    classDef service fill:#ff6b6b,stroke:#333,stroke-width:2px,color:#fff
    classDef database fill:#4ecdc4,stroke:#333,stroke-width:2px,color:#000
    classDef external fill:#95a5a6,stroke:#333,stroke-width:2px,color:#fff
    classDef middleware fill:#f39c12,stroke:#333,stroke-width:2px,color:#000

    class FE frontend
    class GW,ROUTE,AGG gateway
    class AUTH,RATE middleware
    class BE,LLM,EMAIL service
    class DB,CACHE,S3 database
    class OPENAI,SMTP external```
````
