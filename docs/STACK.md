# Stack Tecnológica — CoffeeQuest

**Versão:** 1.0  
**Data:** 2026-04-27  
**Status:** Proposta aprovada para MVP

---

## Índice

1. [Visão Geral](#1-visão-geral)
2. [Frontend](#2-frontend)
3. [Backend](#3-backend)
4. [Banco de Dados](#4-banco-de-dados)
5. [Real-time](#5-real-time)
6. [Infraestrutura e DevOps](#6-infraestrutura-e-devops)
7. [Autenticação e Segurança](#7-autenticação-e-segurança)
8. [Mídia e Storage](#8-mídia-e-storage)
9. [Filas e Processamento Assíncrono](#9-filas-e-processamento-assíncrono)
10. [Observabilidade](#10-observabilidade)
11. [Integrações Externas](#11-integrações-externas)
12. [Ferramentas de Desenvolvimento](#12-ferramentas-de-desenvolvimento)
13. [Diagrama Completo](#13-diagrama-completo)

---

## 1. Visão Geral

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENTES                                 │
│           PWA (Next.js 14)  ·  iOS Safari  ·  Chrome Android   │
└───────────────────────────────┬─────────────────────────────────┘
                                │ HTTPS / WSS
                     ┌──────────▼──────────┐
                     │   CDN / API Gateway  │
                     │  (Cloudflare / ALB)  │
                     └──┬───────────┬───────┘
                        │           │
             ┌──────────▼──┐   ┌────▼──────────┐
             │  api-service │   │  ws-service   │
             │  (NestJS)    │   │  (NestJS/Go)  │
             └──────┬───────┘   └────┬──────────┘
                    │               │
        ┌───────────▼───────────────▼───────────┐
        │           CAMADA DE DADOS              │
        │  PostgreSQL  │  Redis  │  S3           │
        └───────────────────────────────────────┘
                    │
        ┌───────────▼───────────┐
        │  worker-service       │
        │  (BullMQ + RabbitMQ)  │
        └───────────────────────┘
```

---

## 2. Frontend

### Framework Principal
| Tecnologia | Versão | Justificativa |
|-----------|--------|---------------|
| **Next.js** | 14 (App Router) | SSR/SSG para SEO, RSC para performance, suporte nativo a PWA |
| **TypeScript** | 5.x | Tipagem estática, segurança em refatorações, DX |
| **React** | 18 | Base do Next.js; Concurrent Features para UX fluida |

### Estilização
| Tecnologia | Versão | Justificativa |
|-----------|--------|---------------|
| **TailwindCSS** | 3.x | Utility-first, bundle pequeno, consistência de design tokens |
| **shadcn/ui** | latest | Componentes acessíveis (Radix UI), sem lock-in, customizáveis |
| **Framer Motion** | 10.x | Animações (XP toast, level-up, roda de sabores) |

### PWA
| Tecnologia | Justificativa |
|-----------|---------------|
| **next-pwa** | Service Worker, cache offline de trilhas em andamento |
| **IndexedDB (idb)** | Cache local de cupping e progresso de lição para modo offline |
| **Web Push API** | Notificações push via Firebase Cloud Messaging |

### Estado e Dados
| Tecnologia | Uso |
|-----------|-----|
| **TanStack Query v5** | Cache de dados do servidor, sincronização, revalidação |
| **Zustand** | Estado global leve (sessão do workshop, preferências do usuário) |
| **React Hook Form + Zod** | Formulários (cupping SCA, cadastro, quiz) com validação tipada |

### Qualidade
| Tecnologia | Uso |
|-----------|-----|
| **Vitest** | Testes unitários de componentes e lógica de UI |
| **Playwright** | Testes E2E dos fluxos críticos |
| **Storybook** | Documentação e desenvolvimento isolado de componentes |
| **Lighthouse CI** | Monitoramento de performance (TTI ≤ 3s) no pipeline |

---

## 3. Backend

### api-service (REST API principal)
| Tecnologia | Versão | Justificativa |
|-----------|--------|---------------|
| **NestJS** | 10.x | Estrutura modular, decorators, DI nativa, suporte a TypeScript first-class |
| **Node.js** | 20 LTS | Runtime estável, ecossistema maduro |
| **TypeScript** | 5.x | Tipagem ponta a ponta com o frontend |
| **Fastify Adapter** | — | Performance superior ao Express como adapter HTTP do NestJS |

### ws-service (Workshop ao Vivo)
| Tecnologia | Opção A | Opção B |
|-----------|---------|---------|
| **Runtime** | NestJS + Socket.io | Go (goroutines nativas) |
| **Vantagem** | Reusa stack, menos contexto-switch | Performance superior para 500+ conexões simultâneas |
| **Desvantagem** | Overhead do Node.js em alta concorrência | Curva de aprendizado, outro repositório |
| **Decisão** | ⚠️ **A definir — ver D-03 em SPEC** | |

### worker-service (Processamento Assíncrono)
| Tecnologia | Uso |
|-----------|-----|
| **NestJS** | Base do serviço |
| **BullMQ** | Filas de jobs com Redis: badges, PDFs, push notifications |
| **Puppeteer / Playwright** | Geração de PDFs (cupping, certificados, relatórios) |

### Validação e Contratos
| Tecnologia | Uso |
|-----------|-----|
| **Zod** | Validação de schemas compartilhada entre frontend e backend |
| **class-validator + class-transformer** | Validação de DTOs no NestJS |
| **OpenAPI / Swagger** | Documentação automática da API REST |

---

## 4. Banco de Dados

### Banco Principal
| Tecnologia | Versão | Justificativa |
|-----------|--------|---------------|
| **PostgreSQL** | 15 | ACID, JSONB para conteúdo flexível, suporte a schema-per-tenant |
| **Prisma ORM** | 5.x | Type-safe, migrations versionadas, introspection |

### Estratégia Multi-Tenant
- **Schema-per-tenant** no PostgreSQL
- Middleware NestJS resolve o tenant e injeta o schema correto em cada request
- Schema `public` para biblioteca central de conteúdo (read-only pelos tenants)

### Cache e Tempo Real
| Tecnologia | Versão | Uso |
|-----------|--------|-----|
| **Redis** | 7.x | Cache de sessões, leaderboards (sorted sets), estado de workshop, pub/sub entre instâncias do ws-service |
| **ioredis** | — | Client Redis para Node.js |

### Migrations e Seeds
| Tecnologia | Uso |
|-----------|-----|
| **Prisma Migrate** | Migrations versionadas e reproduzíveis |
| **Prisma Studio** | Inspeção visual do banco em desenvolvimento |

---

## 5. Real-time

| Tecnologia | Justificativa |
|-----------|---------------|
| **Socket.io** | Abstração sobre WebSocket com fallback automático para long-polling; rooms nativas para isolamento de sessões de workshop |
| **Redis Adapter (socket.io-redis)** | Escala horizontal do ws-service: múltiplas instâncias compartilham estado via Redis Pub/Sub |

### Fallback de Conectividade
- Se WebSocket cair → cliente entra em polling automático: `GET /workshops/:id/state` a cada 5 segundos
- Estado da sessão persistido no Redis com TTL de 24h
- Reconexão transparente sem perda de dados

---

## 6. Infraestrutura e DevOps

### Containers e Orquestração
| Tecnologia | Uso |
|-----------|-----|
| **Docker** | Containerização de todos os serviços |
| **Docker Compose** | Ambiente de desenvolvimento local |
| **Kubernetes (EKS/GKE)** | Orquestração em produção; HPA para escala automática |

### Cloud
| Provedor | Serviços utilizados |
|---------|---------------------|
| **AWS** (preferencial) | EKS, RDS (PostgreSQL), ElastiCache (Redis), S3, CloudFront, SES, Secrets Manager |
| **GCP** (alternativo) | GKE, Cloud SQL, Memorystore, GCS, Cloud CDN |

### CI/CD
| Tecnologia | Uso |
|-----------|-----|
| **GitHub Actions** | Pipeline de CI/CD |
| **Docker Hub / ECR** | Registry de imagens |
| **ArgoCD** | GitOps — deploy declarativo no Kubernetes |

### Pipeline CI (por PR)
```
lint → typecheck → test:unit → test:integration → build → lighthouse → deploy:staging
```

### Ambientes
| Ambiente | Propósito |
|---------|-----------|
| `local` | Docker Compose, hot-reload |
| `staging` | Réplica de produção, dados sintéticos, feature flags |
| `production` | Multi-AZ, backups automáticos, alertas |

---

## 7. Autenticação e Segurança

### Auth Provider
| Tecnologia | Opção A | Opção B |
|-----------|---------|---------|
| **Provedor** | Auth0 | Keycloak (self-hosted) |
| **Vantagem** | Managed, SDK maduro, social login fácil | Controle total, sem custo por MAU |
| **Desvantagem** | Custo cresce com usuários | Overhead operacional |
| **Decisão** | ⚠️ **A definir — ver D-02 em SPEC** | |

### Estratégia de Tokens
- **Access Token:** JWT, expira em 1h
- **Refresh Token:** opaque, expira em 30 dias, rotação automática
- **2FA:** obrigatório para `instructor`, `cafe_manager`, `admin`

### Segurança da API
| Medida | Implementação |
|--------|--------------|
| Rate limiting | `@nestjs/throttler` — 100 req/min por IP (público), 1000 req/min por usuário autenticado |
| Helmet | Headers HTTP de segurança |
| CORS | Whitelist de origens por tenant |
| TLS | 1.3 em trânsito; certificados gerenciados pelo ALB/Cloudflare |
| Secrets | AWS Secrets Manager / HashiCorp Vault |
| OWASP | Checklist aplicado em PRs de endpoints críticos |

---

## 8. Mídia e Storage

| Tecnologia | Uso |
|-----------|-----|
| **AWS S3** | Storage de avatares, assets de badges, exports (PDF/CSV) |
| **CloudFront** | CDN para assets estáticos e vídeos |
| **Mux** (preferencial) | Hospedagem e entrega adaptativa de microvídeos (≤90s); analytics de vídeo |
| **Cloudflare Stream** (alternativo) | Entrega de vídeo com menor latência na América Latina |
| **Sharp** | Processamento de imagens server-side (resize, otimização de avatares) |

### Geração de QR Codes
- Geração server-side com `qrcode` (npm)
- Entregue como SVG ou PNG via endpoint REST
- Embutido em PDF para impressão (campanhas, certificados)

---

## 9. Filas e Processamento Assíncrono

| Tecnologia | Uso |
|-----------|-----|
| **BullMQ** | Filas prioritárias com Redis; retry automático; jobs agendados (missões diárias, expiração de streaks) |
| **RabbitMQ** | Mensageria entre serviços (api-service → worker-service) para operações de maior throughput |

### Jobs Principais
| Job | Trigger | Serviço |
|-----|---------|---------|
| `calculate-badges` | Após xp_event | worker-service |
| `generate-pdf` | Após fechar cupping / workshop | worker-service |
| `send-push` | Após badge, streak, missão | worker-service |
| `update-leaderboard` | A cada 5 min | worker-service |
| `reset-daily-missions` | Meia-noite (fuso tenant) | worker-service |
| `expire-streaks` | Meia-noite (fuso tenant) | worker-service |
| `dashboard-metrics` | A cada 1h | worker-service |

---

## 10. Observabilidade

| Tecnologia | Uso |
|-----------|-----|
| **Sentry** | APM, rastreamento de erros, performance por endpoint |
| **PostHog** | Analytics de produto (funis, retenção, feature flags) |
| **Metabase** | BI interno (dashboards para o time CoffeeQuest) |
| **Prometheus + Grafana** | Métricas de infraestrutura (Kubernetes, Redis, PostgreSQL) |
| **Loki** | Agregação de logs estruturados (JSON) de todos os serviços |
| **OpenTelemetry** | Instrumentação padronizada para traces distribuídos |

### Alertas Críticos
- Error rate API > 1% → PagerDuty
- Latência WebSocket p95 > 500ms → Slack #alerts
- Fila de jobs acumulando > 1000 itens → Slack #alerts
- Uptime < 99.5% → PagerDuty

---

## 11. Integrações Externas

| Serviço | Tecnologia | Uso |
|---------|-----------|-----|
| **Pagamentos B2B** | Asaas (preferencial) / Stripe | Recorrência mensal dos planos SaaS; boleto + cartão (Asaas tem melhor suporte BR) |
| **E-mail transacional** | Resend / AWS SES | Confirmação de cadastro, relatórios semanais, certificados |
| **Push Notifications** | Firebase Cloud Messaging (FCM) | Notificações via PWA (badges, streaks, missões) |
| **OAuth Social** | Google, Apple | Login social |

---

## 12. Ferramentas de Desenvolvimento

| Categoria | Ferramenta |
|-----------|-----------|
| **Monorepo** | Turborepo — workspace com `apps/web`, `apps/api`, `apps/worker`, `packages/shared` |
| **Package manager** | pnpm |
| **Linting** | ESLint + Prettier (config compartilhada no monorepo) |
| **Git hooks** | Husky + lint-staged |
| **Commits** | Conventional Commits + Commitlint |
| **Variáveis de ambiente** | dotenv + Zod validation no boot |
| **Documentação API** | Swagger UI (gerado pelo NestJS) |
| **Design** | Figma (design system + protótipo) |
| **Gestão** | GitHub Projects + Issues |

### Estrutura do Monorepo
```
CoffeeQuest/
├── apps/
│   ├── web/          # Next.js 14 (PWA)
│   ├── api/          # NestJS (REST API)
│   ├── ws/           # NestJS/Go (WebSocket)
│   └── worker/       # NestJS (jobs assíncronos)
├── packages/
│   ├── shared/       # Types, schemas Zod, utils compartilhados
│   ├── ui/           # Design system (shadcn/ui customizado)
│   └── config/       # ESLint, TypeScript, Tailwind configs
├── infra/
│   ├── docker/       # Dockerfiles por serviço
│   ├── k8s/          # Manifests Kubernetes
│   └── terraform/    # IaC (VPC, RDS, ElastiCache, S3)
├── docs/
│   ├── README.md
│   ├── SPEC_COFFEEQUEST.md
│   └── STACK.md      # este arquivo
├── docker-compose.yml
└── turbo.json
```

---

## 13. Diagrama Completo

```
                          ┌─────────────────────────────┐
                          │         CLIENTES             │
                          │  PWA · iOS Safari · Android  │
                          └──────────────┬───────────────┘
                                         │ HTTPS / WSS
                          ┌──────────────▼───────────────┐
                          │    Cloudflare / AWS ALB       │
                          │  (CDN + DDoS + TLS 1.3)      │
                          └───────┬──────────┬────────────┘
                                  │          │
                    ┌─────────────▼──┐  ┌────▼──────────────┐
                    │  api-service   │  │   ws-service       │
                    │  NestJS        │  │   NestJS / Go      │
                    │  REST API      │  │   Socket.io        │
                    └──────┬─────────┘  └─────┬─────────────┘
                           │                  │
          ┌────────────────▼──────────────────▼──────────────┐
          │                  DADOS                            │
          │                                                   │
          │  ┌─────────────────┐   ┌────────────────────┐    │
          │  │  PostgreSQL 15  │   │      Redis 7        │    │
          │  │  schema/tenant  │   │  cache · sessions   │    │
          │  │  Prisma ORM     │   │  leaderboards       │    │
          │  └─────────────────┘   │  pub/sub · queues   │    │
          │                        └────────────────────┘    │
          │  ┌─────────────────┐                             │
          │  │    AWS S3        │                             │
          │  │  mídia · PDFs   │                             │
          │  └─────────────────┘                             │
          └───────────────────────────────────────────────────┘
                           │
          ┌────────────────▼──────────────────────────────────┐
          │               worker-service                       │
          │         NestJS · BullMQ · RabbitMQ                │
          │                                                    │
          │  badges · PDFs · push · leaderboard · missões     │
          └───────────────────────────────────────────────────┘
                           │
          ┌────────────────▼──────────────────────────────────┐
          │            INTEGRAÇÕES EXTERNAS                    │
          │                                                    │
          │  Asaas/Stripe  │  Resend/SES  │  FCM  │  Mux     │
          └───────────────────────────────────────────────────┘
                           │
          ┌────────────────▼──────────────────────────────────┐
          │            OBSERVABILIDADE                         │
          │                                                    │
          │  Sentry · PostHog · Prometheus · Grafana · Loki   │
          └───────────────────────────────────────────────────┘
```

---

*Próximo passo: validar D-02 (Auth0 vs Keycloak) e D-03 (NestJS vs Go para ws-service) antes do Sprint 1.*
