# 🌐 ArchLens - API Gateway

[![CI](https://github.com/ArchLens-Fiap/archlens-gateway/actions/workflows/ci.yml/badge.svg)](https://github.com/ArchLens-Fiap/archlens-gateway/actions/workflows/ci.yml) [![Quality Gate](https://sonarcloud.io/api/project_badges/measure?project=ArchLens-Fiap_archlens-gateway&metric=alert_status)](https://sonarcloud.io/dashboard?id=ArchLens-Fiap_archlens-gateway) [![Coverage](https://sonarcloud.io/api/project_badges/measure?project=ArchLens-Fiap_archlens-gateway&metric=coverage)](https://sonarcloud.io/dashboard?id=ArchLens-Fiap_archlens-gateway)

> **Reverse Proxy e Gateway de API com YARP**
> Hackathon FIAP - Fase 5 | Pós-Tech Software Architecture + IA para Devs
>
> **Autor:** Rafael Henrique Barbosa Pereira (RM366243)

[![.NET 9.0](https://img.shields.io/badge/.NET-9.0-512BD4?logo=dotnet)](https://dotnet.microsoft.com/)
[![YARP](https://img.shields.io/badge/YARP-Reverse%20Proxy-512BD4)](https://microsoft.github.io/reverse-proxy/)
[![JWT](https://img.shields.io/badge/Auth-JWT%20Bearer-000000?logo=jsonwebtokens)](https://jwt.io/)
[![Docker](https://img.shields.io/badge/Docker-Container-2496ED?logo=docker)](https://www.docker.com/)

---

## 📋 Descrição

O **ArchLens Gateway** é o ponto de entrada único da plataforma ArchLens, implementado com **YARP (Yet Another Reverse Proxy)** sobre .NET 9. Centraliza autenticação JWT, autorização baseada em roles, rate limiting, headers de segurança e correlação de requests para todos os microsserviços do ecossistema.

---

## 🏗️ Arquitetura

```mermaid
flowchart TB
    subgraph Internet
        Client[Cliente / Browser]
        WS[WebSocket Client]
    end

    subgraph Gateway["🌐 ArchLens Gateway :5000"]
        MW[Middlewares]
        AUTH[JWT Authentication]
        RL[Rate Limiting]
        SEC[Security Headers]
        CID[CorrelationId]
        PROXY[YARP Reverse Proxy]
    end

    subgraph Services["Microsserviços"]
        AS[Auth Service<br/>:5120]
        US[Upload Service<br/>:5066]
        OS[Orchestrator Service<br/>:5089]
        RS[Report Service<br/>:5205]
        AI[AI Service<br/>:8000]
        NS[Notification Service<br/>:5150]
    end

    Client --> MW
    WS --> MW
    MW --> AUTH
    AUTH --> RL
    RL --> SEC
    SEC --> CID
    CID --> PROXY

    PROXY --> AS
    PROXY --> US
    PROXY --> OS
    PROXY --> RS
    PROXY --> AI
    PROXY --> NS

    style Gateway fill:#512BD4,color:#fff
    style PROXY fill:#7B68EE,color:#fff
```

---

## 🔄 Fluxo de Request

```mermaid
sequenceDiagram
    participant C as Cliente
    participant GW as Gateway :5000
    participant JWT as JWT Middleware
    participant SVC as Microsserviço

    C->>GW: HTTP Request
    GW->>GW: Add CorrelationId
    GW->>GW: Security Headers
    GW->>GW: Rate Limiting Check
    GW->>JWT: Validate Token
    alt Token Válido
        JWT-->>GW: Authorized
        GW->>SVC: Proxy Request
        SVC-->>GW: Response
        GW-->>C: Response
    else Token Inválido
        JWT-->>GW: 401 Unauthorized
        GW-->>C: 401 Unauthorized
    end
```

---

## 🛠️ Tecnologias

| Tecnologia | Versão | Descrição |
|------------|--------|-----------|
| .NET | 9.0 | Framework principal |
| YARP | 2.x | Reverse Proxy |
| JWT Bearer | - | Autenticação |
| Rate Limiting | Built-in | Controle de taxa |
| OpenTelemetry | 1.x | Traces e métricas |
| Serilog | 4.x | Logs estruturados |

---

## 🔒 Autenticação e Autorização

| Política | Descrição | Roles |
|----------|-----------|-------|
| **Anonymous** | Sem autenticação | - |
| **Authenticated** | JWT Bearer válido | User, Admin |
| **Admin** | JWT Bearer + Role Admin | Admin |

---

## 📡 Rotas e Clusters

### Rotas Configuradas

| Rota | Match | Cluster | Auth |
|------|-------|---------|------|
| `auth-route` | `/api/auth/{**catch-all}` | auth-cluster | ❌ Nenhuma |
| `upload-route` | `/api/upload/{**catch-all}` | upload-cluster | ✅ Authenticated |
| `orchestrator-route` | `/api/orchestrator/{**catch-all}` | orchestrator-cluster | ✅ Authenticated |
| `orchestrator-admin-route` | `/api/orchestrator/admin/{**catch-all}` | orchestrator-cluster | 🔐 Admin |
| `report-route` | `/api/report/{**catch-all}` | report-cluster | ✅ Authenticated |
| `report-admin-route` | `/api/report/admin/{**catch-all}` | report-cluster | 🔐 Admin |
| `ai-health-route` | `/api/ai/health` | ai-cluster | ❌ Nenhuma |
| `ai-route` | `/api/ai/{**catch-all}` | ai-cluster | ✅ Authenticated |
| `notification-hubs-route` | `/notification/hubs/{**catch-all}` | notification-cluster | ❌ WebSocket |

### Clusters Configurados

| Cluster | Destino | Porta | Serviço |
|---------|---------|-------|---------|
| `auth-cluster` | `http://localhost:5120` | 5120 | Auth Service |
| `upload-cluster` | `http://localhost:5066` | 5066 | Upload Service |
| `orchestrator-cluster` | `http://localhost:5089` | 5089 | Orchestrator Service |
| `report-cluster` | `http://localhost:5205` | 5205 | Report Service |
| `ai-cluster` | `http://localhost:8000` | 8000 | AI Service (Python) |
| `notification-cluster` | `http://localhost:5150` | 5150 | Notification Service |

---

## 📁 Estrutura do Projeto

```
archlens-gateway/
├── src/
│   └── ArchLens.Gateway/
│       ├── Program.cs                # Entry point + configuração YARP
│       ├── appsettings.json          # Configuração de rotas e clusters
│       └── Dockerfile
├── .gitignore
└── README.md
```

---

## 🚀 Como Executar

### Pré-requisitos

- .NET 9.0 SDK
- Docker (opcional)

### Executar Local

```bash
cd src/ArchLens.Gateway
dotnet run
```

O gateway estará disponível em: `http://localhost:5000`

---

## 🔧 Variáveis de Ambiente

| Variável | Descrição | Exemplo |
|----------|-----------|---------|
| `Jwt__Key` | Chave secreta para validação JWT | `sua-chave-secreta-256-bits` |
| `Jwt__Issuer` | Emissor do token JWT | `archlens-auth` |
| `Jwt__Audience` | Audiência do token JWT | `archlens-api` |
| `ConnectionStrings__AuthService` | URL do Auth Service | `http://localhost:5120` |
| `ConnectionStrings__UploadService` | URL do Upload Service | `http://localhost:5066` |
| `ConnectionStrings__OrchestratorService` | URL do Orchestrator Service | `http://localhost:5089` |
| `ConnectionStrings__ReportService` | URL do Report Service | `http://localhost:5205` |
| `ConnectionStrings__AiService` | URL do AI Service | `http://localhost:8000` |
| `ConnectionStrings__NotificationService` | URL do Notification Service | `http://localhost:5150` |

---

## 🐳 Docker

```bash
docker build -t archlens-gateway .
docker run -p 5000:5000 archlens-gateway
```

---

## 📈 Observabilidade

### OpenTelemetry (Traces + Métricas)

```mermaid
graph LR
    subgraph "Gateway"
        A[HTTP Request] --> B[YARP Instrumentation]
        B --> C[HttpClient Instrumentation]
    end

    C --> D[OTLP Exporter]
    D --> E[OTel Collector]
    E --> F[Jaeger / Grafana]
```

### Serilog (Logs Estruturados)

| Campo | Descrição |
|-------|-----------|
| `CorrelationId` | ID único de rastreamento por request |
| `ServiceName` | `archlens-gateway` |
| `RouteId` | Rota YARP correspondente |
| `ClusterId` | Cluster de destino |

---

## 📊 Middlewares

| Middleware | Ordem | Descrição |
|------------|-------|-----------|
| CorrelationId | 1 | Adiciona header `X-Correlation-Id` |
| Security Headers | 2 | `X-Content-Type-Options`, `X-Frame-Options`, etc. |
| Rate Limiting | 3 | Controle de requisições por IP/janela |
| Authentication | 4 | Validação JWT Bearer |
| Authorization | 5 | Verificação de roles |
| YARP Proxy | 6 | Encaminhamento para microsserviço |

---

FIAP - Pós-Tech Software Architecture + IA para Devs | Fase 5 - Hackathon (12SOAT + 6IADT)
