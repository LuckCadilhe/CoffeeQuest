# SPEC — CoffeeQuest
**Especificação Técnica, de Produto e de Design**

**Versão:** 1.0  
**Baseado no PRD:** v0.1 (2026-04-22)  
**Autor:** Lucas Cadilhe  
**Status:** Em revisão

---

## Índice

1. [Visão Geral](#1-visão-geral)
2. [Arquitetura do Sistema](#2-arquitetura-do-sistema)
3. [Modelo de Dados](#3-modelo-de-dados)
4. [APIs e Contratos](#4-apis-e-contratos)
5. [Especificação de Produto — User Stories](#5-especificação-de-produto--user-stories)
6. [Especificação de Design](#6-especificação-de-design)
7. [Fluxos Principais](#7-fluxos-principais)
8. [Critérios de Aceite por Módulo](#8-critérios-de-aceite-por-módulo)
9. [Requisitos Não-Funcionais Detalhados](#9-requisitos-não-funcionais-detalhados)
10. [Plano de Testes](#10-plano-de-testes)
11. [Decisões em Aberto](#11-decisões-em-aberto)

---

## 1. Visão Geral

### 1.1 Resumo do Produto

CoffeeQuest é uma plataforma B2B2C de gamificação para educação em café especial. O sistema é multi-tenant — cada cafeteria ou escola opera como um tenant isolado — e serve quatro perfis de usuário: `trainee`, `instructor`, `cafe_manager` e `admin`.

O MVP entrega cinco módulos funcionais: **Aprendizagem** (trilhas + quizzes com spaced repetition), **Sensorial** (cupping SCA digital), **Gamificação** (XP, badges, leaderboards), **Workshop ao Vivo** (sessão em tempo real com instrutor) e **Dashboard B2B** (métricas e campanhas para o gestor da cafeteria).

### 1.2 Premissas Técnicas

- A unidade organizacional principal é o `Tenant`. Todo dado de usuário, conteúdo e métricas pertence a um tenant.
- A biblioteca central de conteúdo (gerenciada pelo Admin Scigeniq) é compartilhável entre tenants, mas instâncias de progresso e cupping são sempre isoladas por tenant.
- O frontend é uma PWA (Next.js 14, mobile-first). Não há app nativo no MVP.
- O real-time do Workshop usa WebSockets (Socket.io). Fallback assíncrono é obrigatório.
- Toda a stack roda em containers (Docker) orquestrados por Kubernetes ou ECS.

### 1.3 Personas de Referência

| ID | Persona | Papel no sistema | Frequência de uso |
|----|---------|-----------------|------------------|
| P1 | Lucas, Barista em Formação | `trainee` | Diário |
| P2 | Marina, Cliente Entusiasta | `trainee` | Semanal/esporádico |
| P3 | Rafael, Instrutor/Q-Grader | `instructor` | Por sessão de workshop |
| P4 | Júlia, Dona da Cafeteria | `cafe_manager` | Semanal |
| P5 | Admin Scigeniq | `admin` | Contínuo |

---

## 2. Arquitetura do Sistema

### 2.1 Diagrama de Alto Nível

```
┌─────────────────────────────────────────────────────────┐
│                     CLIENTES                            │
│   PWA (Next.js 14)        QR Code Scanner               │
│   iOS Safari / Chrome Android                           │
└───────────────────────┬─────────────────────────────────┘
                        │ HTTPS / WSS
┌───────────────────────▼─────────────────────────────────┐
│                  API GATEWAY / CDN                      │
│         (CloudFront ou Cloudflare Workers)              │
└───┬───────────────┬──────────────┬───────────────────────┘
    │               │              │
┌───▼───┐    ┌──────▼─────┐  ┌────▼──────┐
│ REST  │    │ WebSocket  │  │  Static   │
│ API   │    │  Server    │  │  Assets   │
│(NestJS│    │  (NestJS/  │  │  (S3/CDN) │
│  Go)  │    │   Go)      │  └───────────┘
└───┬───┘    └──────┬─────┘
    │               │
┌───▼───────────────▼─────────────────────────────────────┐
│                  CAMADA DE SERVIÇOS                     │
│  Auth  │  Learning  │  Sensorial  │  Gamification  │ B2B│
└───┬────┴─────┬──────┴──────┬──────┴────────┬───────┴────┘
    │          │             │               │
┌───▼──────────▼─────────────▼───────────────▼────────────┐
│                  INFRAESTRUTURA DE DADOS                │
│   PostgreSQL 15    │    Redis       │    S3 (mídia)     │
│   (principal)      │ (cache/sessões │                   │
│                    │  /leaderboards)│                   │
└────────────────────┴────────────────┴───────────────────┘
         │                    │
┌────────▼────────┐  ┌────────▼────────┐
│   RabbitMQ      │  │    PostHog      │
│ (filas assíncr) │  │  (analytics)    │
└─────────────────┘  └─────────────────┘
```

### 2.2 Serviços e Responsabilidades

| Serviço | Tecnologia | Responsabilidade |
|---------|-----------|-----------------|
| `api-service` | NestJS + TypeScript | REST API principal (auth, learning, cupping, gamification, B2B) |
| `ws-service` | NestJS (Socket.io) ou Go | WebSocket do workshop ao vivo; alta concorrência (500 usuários/sessão) |
| `worker-service` | NestJS + BullMQ | Processamento assíncrono: cálculo de badges, notificações push, relatórios PDF |
| `cdn` | CloudFront/Cloudflare | Entrega de assets estáticos e vídeos |
| `auth` | Auth0 ou Keycloak | JWT, OAuth2, RBAC por tenant |

### 2.3 Multi-Tenancy

- Estratégia: **schema-per-tenant** no PostgreSQL para isolamento forte com custo operacional moderado.
- Tenant resolvido via subdomínio (`minhacafeteria.coffeequest.app`) ou header `X-Tenant-ID`.
- Middleware de resolução de tenant injeta o schema correto em todas as queries.
- Conteúdo da biblioteca central (`public` schema) é readonly por todos os tenants.

### 2.4 Real-Time (Workshop)

```
Instrutor                     Servidor WS                    Alunos
   │                               │                            │
   │── JOIN room:{workshopId} ────►│                            │
   │                               │◄─ JOIN room:{workshopId} ──│
   │── SLIDE_CHANGE {slideIdx} ───►│── broadcast ──────────────►│
   │── START_CUPPING {sampleId} ──►│── broadcast ──────────────►│
   │                               │◄─ CUPPING_ENTRY {data} ────│
   │                               │── aggregate ──────────────►│
   │◄─ LIVE_STATS {heatmap, ...} ──│                            │
   │── END_SESSION ───────────────►│── broadcast ──────────────►│
```

**Fallback assíncrono:** se a conexão WebSocket cair, o cliente entra em modo polling (REST `GET /workshops/:id/state` a cada 5s). O servidor mantém o estado da sessão no Redis com TTL de 24h.

---

## 3. Modelo de Dados

### 3.1 Entidades Principais

#### `tenants`
```sql
CREATE TABLE tenants (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  slug        VARCHAR(63) UNIQUE NOT NULL,  -- subdomínio
  name        VARCHAR(255) NOT NULL,
  plan        VARCHAR(50) NOT NULL,         -- 'cafe' | 'escola' | 'enterprise'
  settings    JSONB DEFAULT '{}',
  created_at  TIMESTAMPTZ DEFAULT NOW(),
  deleted_at  TIMESTAMPTZ
);
```

#### `users`
```sql
CREATE TABLE users (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id     UUID REFERENCES tenants(id),
  external_id   VARCHAR(255),               -- ID do Auth0/Keycloak
  email         VARCHAR(255) NOT NULL,
  role          VARCHAR(50) NOT NULL,       -- 'trainee' | 'instructor' | 'cafe_manager' | 'admin'
  display_name  VARCHAR(100),
  avatar_url    TEXT,
  preferences   JSONB DEFAULT '{}',        -- objetivos, nível, dieta
  xp_total      INTEGER DEFAULT 0,
  level         INTEGER DEFAULT 1,
  streak_days   INTEGER DEFAULT 0,
  last_active   TIMESTAMPTZ,
  created_at    TIMESTAMPTZ DEFAULT NOW()
);
CREATE INDEX idx_users_tenant ON users(tenant_id);
CREATE INDEX idx_users_email ON users(email);
```

#### `tracks` (Trilhas de Aprendizagem)
```sql
CREATE TABLE tracks (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id   UUID REFERENCES tenants(id),  -- NULL = biblioteca central
  title       VARCHAR(255) NOT NULL,
  slug        VARCHAR(100) NOT NULL,
  description TEXT,
  difficulty  VARCHAR(20),                  -- 'beginner' | 'intermediate' | 'advanced'
  tags        TEXT[],
  is_public   BOOLEAN DEFAULT FALSE,
  position    INTEGER DEFAULT 0,
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE modules (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  track_id    UUID REFERENCES tracks(id),
  title       VARCHAR(255) NOT NULL,
  position    INTEGER NOT NULL,
  xp_reward   INTEGER DEFAULT 10
);

CREATE TABLE lessons (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  module_id   UUID REFERENCES modules(id),
  type        VARCHAR(20) NOT NULL,         -- 'card' | 'quiz' | 'video'
  title       VARCHAR(255) NOT NULL,
  content     JSONB NOT NULL,               -- estrutura varia por tipo
  position    INTEGER NOT NULL,
  xp_reward   INTEGER DEFAULT 5
);
```

#### `user_progress`
```sql
CREATE TABLE user_progress (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id       UUID REFERENCES users(id),
  lesson_id     UUID REFERENCES lessons(id),
  status        VARCHAR(20) DEFAULT 'not_started', -- 'not_started' | 'in_progress' | 'completed'
  score         NUMERIC(5,2),
  attempts      INTEGER DEFAULT 0,
  next_review   TIMESTAMPTZ,               -- SM-2: próxima revisão
  ease_factor   NUMERIC(4,2) DEFAULT 2.5,  -- SM-2
  interval_days INTEGER DEFAULT 1,         -- SM-2
  completed_at  TIMESTAMPTZ,
  UNIQUE(user_id, lesson_id)
);
```

#### `cupping_sessions` e `cupping_entries`
```sql
CREATE TABLE cupping_sessions (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id     UUID REFERENCES tenants(id),
  workshop_id   UUID REFERENCES workshops(id),  -- NULL se standalone
  instructor_id UUID REFERENCES users(id),
  title         VARCHAR(255),
  status        VARCHAR(20) DEFAULT 'open',     -- 'open' | 'closed' | 'archived'
  created_at    TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE samples (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  session_id    UUID REFERENCES cupping_sessions(id),
  label         VARCHAR(100) NOT NULL,           -- ex: "Amostra A"
  origin        VARCHAR(100),
  process       VARCHAR(100),
  roast_level   VARCHAR(50),
  metadata      JSONB DEFAULT '{}'
);

CREATE TABLE cupping_entries (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  sample_id       UUID REFERENCES samples(id),
  user_id         UUID REFERENCES users(id),
  -- Atributos SCA (0–10 cada)
  fragrance       NUMERIC(4,2),
  aroma           NUMERIC(4,2),
  flavor          NUMERIC(4,2),
  aftertaste      NUMERIC(4,2),
  acidity         NUMERIC(4,2),
  body            NUMERIC(4,2),
  balance         NUMERIC(4,2),
  uniformity      NUMERIC(4,2),
  clean_cup       NUMERIC(4,2),
  sweetness       NUMERIC(4,2),
  overall         NUMERIC(4,2),
  defects         INTEGER DEFAULT 0,
  descriptors     TEXT[],                        -- roda de sabores selecionados
  notes           TEXT,
  total_score     NUMERIC(5,2) GENERATED ALWAYS AS (
    fragrance + aroma + flavor + aftertaste + acidity + body +
    balance + uniformity + clean_cup + sweetness + overall - defects
  ) STORED,
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(sample_id, user_id)
);
```

#### `xp_events` e `badges`
```sql
CREATE TABLE xp_events (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id     UUID REFERENCES users(id),
  source_type VARCHAR(50) NOT NULL,   -- 'quiz' | 'cupping' | 'workshop' | 'streak' | 'mission'
  source_id   UUID,
  xp_amount   INTEGER NOT NULL,
  metadata    JSONB DEFAULT '{}',
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE badges (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id   UUID REFERENCES tenants(id),  -- NULL = global
  name        VARCHAR(100) NOT NULL,
  description TEXT,
  icon_url    TEXT,
  trigger     JSONB NOT NULL               -- ex: {"type": "xp_threshold", "value": 1000}
);

CREATE TABLE user_badges (
  user_id     UUID REFERENCES users(id),
  badge_id    UUID REFERENCES badges(id),
  earned_at   TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (user_id, badge_id)
);
```

#### `workshops`
```sql
CREATE TABLE workshops (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id       UUID REFERENCES tenants(id),
  instructor_id   UUID REFERENCES users(id),
  title           VARCHAR(255) NOT NULL,
  status          VARCHAR(20) DEFAULT 'draft',  -- 'draft' | 'live' | 'ended'
  join_code       VARCHAR(8) UNIQUE NOT NULL,   -- código curto para QR
  slides          JSONB DEFAULT '[]',
  current_slide   INTEGER DEFAULT 0,
  started_at      TIMESTAMPTZ,
  ended_at        TIMESTAMPTZ,
  created_at      TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE workshop_participants (
  workshop_id   UUID REFERENCES workshops(id),
  user_id       UUID REFERENCES users(id),
  joined_at     TIMESTAMPTZ DEFAULT NOW(),
  xp_earned     INTEGER DEFAULT 0,
  PRIMARY KEY (workshop_id, user_id)
);
```

#### `wallets` e `transactions` (Beans)
```sql
CREATE TABLE wallets (
  user_id   UUID PRIMARY KEY REFERENCES users(id),
  balance   INTEGER DEFAULT 0
);

CREATE TABLE transactions (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  wallet_id   UUID REFERENCES wallets(user_id),
  amount      INTEGER NOT NULL,               -- positivo = crédito, negativo = débito
  reason      VARCHAR(100) NOT NULL,
  metadata    JSONB DEFAULT '{}',
  created_at  TIMESTAMPTZ DEFAULT NOW()
);
```

### 3.2 Índices Críticos de Performance

```sql
-- Leaderboard por tenant
CREATE INDEX idx_users_xp_tenant ON users(tenant_id, xp_total DESC);

-- Histórico de cupping por usuário
CREATE INDEX idx_cupping_user ON cupping_entries(user_id, created_at DESC);

-- Progresso de trilha
CREATE INDEX idx_progress_user ON user_progress(user_id, status);

-- XP events para cálculo de streak
CREATE INDEX idx_xp_user_date ON xp_events(user_id, created_at DESC);
```

---

## 4. APIs e Contratos

> Base URL: `https://api.coffeequest.app/v1`  
> Autenticação: `Authorization: Bearer <jwt>`  
> Tenant: resolvido via subdomínio ou header `X-Tenant-ID`

### 4.1 Autenticação

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| `POST` | `/auth/register` | Cadastro com e-mail/senha |
| `POST` | `/auth/login` | Login e-mail/senha |
| `POST` | `/auth/oauth/callback` | Callback OAuth (Google/Apple) |
| `POST` | `/auth/refresh` | Renovar JWT |
| `DELETE` | `/auth/sessions` | Logout (revogar token) |

**POST /auth/register — Request:**
```json
{
  "email": "lucas@cafeteria.com",
  "password": "s3nh@F0rte",
  "display_name": "Lucas",
  "tenant_slug": "cafeteria-exemplo",
  "role": "trainee"
}
```
**Response 201:**
```json
{
  "user": { "id": "uuid", "email": "...", "role": "trainee" },
  "access_token": "eyJ...",
  "refresh_token": "eyJ..."
}
```

---

### 4.2 Aprendizagem

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| `GET` | `/tracks` | Lista trilhas disponíveis (tenant + global) |
| `GET` | `/tracks/:id` | Detalhe de uma trilha com módulos |
| `GET` | `/tracks/:id/progress` | Progresso do usuário autenticado na trilha |
| `GET` | `/lessons/:id` | Conteúdo de uma lição |
| `POST` | `/lessons/:id/complete` | Marcar lição como concluída e registrar score |
| `GET` | `/lessons/review` | Próximas lições para revisão (SM-2) |

**POST /lessons/:id/complete — Request:**
```json
{
  "score": 85.5,
  "time_spent_seconds": 120,
  "answers": [
    { "question_id": "uuid", "answer": "B", "correct": true }
  ]
}
```
**Response 200:**
```json
{
  "xp_earned": 15,
  "new_level": null,
  "badges_earned": [],
  "next_review": "2026-04-25T08:00:00Z"
}
```

---

### 4.3 Cupping

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| `POST` | `/cupping/sessions` | Criar sessão de cupping |
| `GET` | `/cupping/sessions` | Listar sessões (filtros: data, origem) |
| `GET` | `/cupping/sessions/:id` | Detalhe com amostras e entradas |
| `POST` | `/cupping/sessions/:id/samples` | Adicionar amostra |
| `POST` | `/cupping/samples/:id/entries` | Registrar avaliação SCA |
| `GET` | `/cupping/samples/:id/comparison` | Comparar entradas (usuário vs instrutor vs média) |
| `GET` | `/cupping/sessions/:id/export` | Exportar PDF do formulário |

**POST /cupping/samples/:id/entries — Request:**
```json
{
  "fragrance": 8.0,
  "aroma": 7.5,
  "flavor": 8.25,
  "aftertaste": 7.75,
  "acidity": 8.0,
  "body": 7.5,
  "balance": 8.0,
  "uniformity": 10.0,
  "clean_cup": 10.0,
  "sweetness": 10.0,
  "overall": 8.0,
  "defects": 0,
  "descriptors": ["chocolate", "caramel", "citrus"],
  "notes": "Acidez brilhante, corpo médio"
}
```
**Response 201:**
```json
{
  "entry_id": "uuid",
  "total_score": 93.0,
  "xp_earned": 20,
  "comparison_available": true
}
```

---

### 4.4 Gamificação

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| `GET` | `/me/xp` | Saldo de XP e histórico |
| `GET` | `/me/badges` | Badges conquistados |
| `GET` | `/me/wallet` | Saldo e extrato de Beans |
| `GET` | `/leaderboards` | Leaderboard (params: `scope=global\|tenant\|workshop`, `period=all\|week\|month`) |
| `GET` | `/missions` | Missões ativas (diárias, semanais, por evento) |
| `POST` | `/missions/:id/claim` | Resgatar recompensa de missão completada |

---

### 4.5 Workshop ao Vivo

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| `POST` | `/workshops` | Criar workshop |
| `GET` | `/workshops/:id` | Detalhe e estado atual |
| `POST` | `/workshops/join` | Entrar via código (`{ "code": "ABCD1234" }`) |
| `PATCH` | `/workshops/:id/state` | Atualizar estado (start, end, slide) — instructor only |
| `GET` | `/workshops/:id/report` | Relatório pós-sessão (exportável) |
| `GET` | `/workshops/:id/certificate/:userId` | Certificado digital verificável |

**WebSocket — Namespace `/workshops`:**

```
// Cliente entra na sala
socket.emit('join', { workshopId: 'uuid', token: 'jwt' })

// Instrutor muda slide
socket.emit('slide_change', { slideIndex: 2 })

// Servidor broadcasteia para alunos
socket.on('slide_changed', ({ slideIndex }) => { ... })

// Instrutor inicia cupping sincronizado
socket.emit('start_cupping', { sampleId: 'uuid' })

// Aluno envia entrada de cupping
socket.emit('cupping_entry', { /* campos SCA */ })

// Servidor envia heatmap em tempo real
socket.on('live_stats', ({ heatmap, accuracy_curve }) => { ... })
```

---

### 4.6 Dashboard B2B (Cafe Manager)

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| `GET` | `/dashboard/overview` | DAU/MAU, top usuários, campanhas ativas |
| `POST` | `/campaigns` | Criar campanha local |
| `GET` | `/campaigns` | Listar campanhas do tenant |
| `PATCH` | `/campaigns/:id` | Editar campanha |
| `POST` | `/rewards` | Criar recompensa (cupom, brinde) |
| `GET` | `/dashboard/conversion` | Relatório engajamento → visita → venda |
| `GET` | `/menu/qr` | Gerar QR code do cardápio interativo |

---

### 4.7 Padrões de Erro

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Campo 'fragrance' deve estar entre 0 e 10",
    "field": "fragrance",
    "status": 422
  }
}
```

| Código HTTP | Código de Erro | Situação |
|------------|---------------|----------|
| 400 | `BAD_REQUEST` | Payload malformado |
| 401 | `UNAUTHORIZED` | Token ausente ou expirado |
| 403 | `FORBIDDEN` | Papel insuficiente ou tenant errado |
| 404 | `NOT_FOUND` | Recurso não existe |
| 409 | `CONFLICT` | Duplicidade (ex: cupping já registrado) |
| 422 | `VALIDATION_ERROR` | Regra de negócio violada |
| 429 | `RATE_LIMITED` | Limite de requisições excedido |
| 500 | `INTERNAL_ERROR` | Erro inesperado |

---

## 5. Especificação de Produto — User Stories

### Épico 1 — Autenticação e Perfil

**US-01** — Como barista em formação, quero me cadastrar via QR code de um workshop para criar minha conta em menos de 60 segundos, para que eu não perca o início da sessão.

**Critérios de Aceite:**
- O QR code pré-preenche o `tenant_slug` e o `workshop_id`.
- O formulário de cadastro exige apenas nome, e-mail e senha.
- Após o cadastro, o usuário é automaticamente incluído no workshop.
- Tempo da tela de QR ao primeiro badge: ≤ 60 segundos (meta de UX).

---

**US-02** — Como usuário, quero um "Passaporte do Café" público para compartilhar meu histórico de aprendizagem e cuppings com colegas e recrutadores.

**Critérios de Aceite:**
- URL pública no formato `coffeequest.app/u/:username`.
- Exibe: nível atual, badges conquistados, total de cuppings, trilhas concluídas.
- Privacidade: scores de cupping individuais são privados por padrão (opt-in para exibir).
- Link pode ser copiado e compartilhado sem login.

---

### Épico 2 — Aprendizagem

**US-10** — Como barista, quero estudar por trilhas temáticas divididas em cards diários para não me sentir sobrecarregado.

**Critérios de Aceite:**
- Cada sessão diária sugere no máximo 5 cards + 1 quiz.
- Cards com spaced repetition (SM-2): o sistema agenda a próxima revisão com base no score.
- Progresso visual por módulo (barra de progresso 0–100%).
- Após completar uma trilha, o usuário recebe XP bônus e badge de conclusão.

---

**US-11** — Como instrutor, quero criar quizzes customizados para meu tenant para adaptar o conteúdo ao perfil dos meus alunos.

**Critérios de Aceite:**
- Tipos suportados: múltipla escolha, verdadeiro/falso, associação, ordenação, imagem-resposta.
- Dificuldade configurável por questão (1–3).
- Questões podem ser reusadas da biblioteca central (fork local).
- Quiz publicado fica disponível para todos os trainees do tenant.

---

### Épico 3 — Sensorial (Cupping)

**US-20** — Como barista, quero registrar um cupping no padrão SCA pelo celular durante o serviço para manter histórico preciso das amostras.

**Critérios de Aceite:**
- Formulário contém todos os 10 atributos SCA com slider (0–10, precisão 0.25).
- Roda de sabores interativa com dois níveis de hierarquia (tap para selecionar).
- Score total calculado automaticamente na tela.
- Cupping salvo mesmo com conexão instável (cache local + sync ao reconectar).
- Exportação em PDF disponível após salvar.

---

**US-22** — Como barista, quero comparar minha avaliação com a do instrutor e a média da turma para calibrar meu paladar.

**Critérios de Aceite:**
- Comparação disponível somente após o instrutor "fechar" a amostra.
- Exibe: diferença por atributo (delta colorido: verde/amarelo/vermelho).
- Sugere trilha de melhoria se delta de algum atributo > 1.0.

---

### Épico 4 — Gamificação

**US-30** — Como barista, quero ganhar XP e subir de nível para sentir progresso constante na minha formação.

**Critérios de Aceite:**
- Regras de XP configuráveis por tenant (defaults: quiz = 10 XP, cupping = 20 XP, workshop = 50 XP, streak diário = 5 XP).
- Nível calculado por faixas de XP (tabela de progressão pré-definida, 20 níveis no MVP).
- Animação de level-up exibida inline na tela.
- Notificação push ao ganhar novo badge.

---

**US-34** — Como barista, quero manter uma sequência (streak) de estudo diário para me sentir motivado a acessar o app todos os dias.

**Critérios de Aceite:**
- Streak incrementa se o usuário completar ao menos 1 ação pontuável no dia.
- Tolerância configurável por tenant (default: 1 dia de "freezeDay" por semana).
- Streak quebrado exibe tela de "recomeço" com missão especial de recuperação.
- Leaderboard de streaks disponível por tenant.

---

### Épico 5 — Workshop ao Vivo

**US-40** — Como instrutor, quero criar uma sessão de workshop e compartilhar um QR code de entrada para que os alunos entrem facilmente sem precisar de cadastro prévio.

**Critérios de Aceite:**
- QR code gerado em até 2 segundos após criar o workshop.
- Alunos sem conta podem criar uma durante o ingresso (fluxo US-01).
- Painel do instrutor exibe lista de participantes em tempo real.
- Número máximo de participantes configurável (default: 500).

---

**US-43** — Como instrutor, quero ver em tempo real como minha turma está respondendo às perguntas para ajustar minha didática na hora.

**Critérios de Aceite:**
- Heatmap de respostas atualizado em ≤ 1 segundo após cada submissão.
- Curva de acerto por questão exibida no painel.
- Opção de exibir resultado anonimizado para a turma (projetor).
- Dados persistidos para o relatório pós-sessão.

---

### Épico 6 — Dashboard B2B

**US-50** — Como gestora de cafeteria, quero ver um dashboard semanal de engajamento para saber se meus clientes estão usando a plataforma.

**Critérios de Aceite:**
- Métricas exibidas: DAU/MAU (7 dias), top 5 usuários mais ativos, trilhas mais iniciadas, cuppings registrados.
- Gráfico de retenção (D1, D7, D30).
- Exportação em CSV disponível para qualquer relatório.
- Dados atualizados a cada hora (não real-time para evitar custo).

---

**US-52** — Como gestora, quero criar uma campanha de quiz temático com desconto no café para engajar meus clientes nas mesas.

**Critérios de Aceite:**
- Campanha tem: nome, descrição, quiz vinculado, recompensa (cupom/brinde), data início/fim.
- QR code da campanha pode ser impresso para colocar nas mesas.
- Métricas da campanha: scans, participações, resgates, conversão.
- Recompensa pode ser validada pelo operador via código no app do gestor.

---

## 6. Especificação de Design

### 6.1 Design System

**Paleta de cores:**
| Token | Valor | Uso |
|-------|-------|-----|
| `--color-primary` | `#6B3F1A` | Marrom café — ações primárias, CTAs |
| `--color-primary-light` | `#A0622D` | Hover, estados ativos |
| `--color-accent` | `#E8A836` | Destaque, badges, XP |
| `--color-success` | `#3D9970` | Acerto, conquista |
| `--color-error` | `#D9534F` | Erro, streak quebrado |
| `--color-background` | `#FAFAF8` | Fundo geral |
| `--color-surface` | `#FFFFFF` | Cards, modais |
| `--color-text-primary` | `#1A1A1A` | Corpo de texto |
| `--color-text-secondary` | `#6B6B6B` | Labels, hints |

**Tipografia:**
- Família: `Inter` (primária), `system-ui` (fallback)
- Scale: 12 / 14 / 16 / 18 / 24 / 32 / 48px
- Peso: Regular (400), Medium (500), Semibold (600), Bold (700)

**Espaçamento:** escala de 4px (4, 8, 12, 16, 24, 32, 48, 64)

**Raio de borda:** 4px (inputs), 8px (cards), 16px (bottom sheets), 50% (avatares)

---

### 6.2 Componentes Principais

#### `LessonCard`
```
┌─────────────────────────────────┐
│ [ícone tipo]  Título da lição   │
│               ──────────────    │
│               Subtítulo / hint  │
│                                 │
│ [badge dificuldade]  +10 XP →  │
└─────────────────────────────────┘
```
- Estados: `not_started`, `in_progress`, `completed`, `due_review`
- `due_review` exibe chip amarelo "Revisar hoje"

#### `CuppingSlider`
```
Aroma
[──────●────────────────] 7.5
       ↑
   Tap to score
```
- Range: 0–10, step 0.25
- Taps grandes (min 44px hit area)
- Valor exibido em tempo real

#### `FlavorWheel` (Roda de Sabores)
- SVG interativo com dois anéis (categoria → subcategoria)
- Primeiro toque seleciona categoria (anel externo iluminado)
- Segundo toque seleciona subcategoria
- Chips com descritores selecionados aparecem abaixo da roda
- Baseado na roda oficial SCA (licença a confirmar)

#### `LiveStatsPanel` (Instrutor)
```
┌──────────────────────────────────────────────┐
│ Questão 3/10          ⏱ 00:45               │
│                                              │
│ Respostas recebidas: 18/24                   │
│                                              │
│ A ████████████████ 67%                       │
│ B ████ 17%                                   │
│ C ████ 17%                                   │
│ D ░░░░  0%                                   │
│                                              │
│ [Revelar resposta]  [Próxima questão →]      │
└──────────────────────────────────────────────┘
```

#### `XPToast`
```
         ┌────────────────┐
         │  +20 XP 🎉     │
         │  Cupping salvo │
         └────────────────┘
```
- Aparece na base da tela por 3 segundos
- Animação slide-up + fade-out
- Não bloqueia interação (pointer-events: none)

---

### 6.3 Fluxos de Navegação

#### App (Trainee) — Bottom Navigation
```
[Casa]  [Trilhas]  [Cupping]  [Ranking]  [Perfil]
```

- **Casa:** feed de missões diárias, streak atual, resumo de progresso
- **Trilhas:** lista de trilhas + progresso, busca
- **Cupping:** histórico + botão "Novo cupping"
- **Ranking:** leaderboard (tenant/global, semanal/mensal)
- **Perfil:** Passaporte, badges, configurações

#### Web (Instrutor / Cafe Manager) — Sidebar Navigation
```
[Dashboard]
[Workshops]
[Conteúdo]
[Alunos / Clientes]
[Campanhas]
[Relatórios]
[Configurações]
```

---

### 6.4 Estados de UI Críticos

| Tela | Estado vazio | Estado de erro | Estado de loading |
|------|-------------|----------------|-------------------|
| Feed de missões | "Nenhuma missão ativa. Volte amanhã!" | Toast de erro + botão "Tentar novamente" | Skeleton 3 cards |
| Lista de trilhas | "Nenhuma trilha disponível ainda." | Tela de erro com ilustração | Skeleton grid |
| Workshop ao vivo (aluno) | — | "Conexão perdida. Reconectando..." (polling) | Spinner com mensagem |
| Leaderboard | "Seja o primeiro a pontuar!" | Toast de erro | Skeleton tabela |

---

## 7. Fluxos Principais

### 7.1 Primeiro Acesso via Workshop

```
1. Instrutor cria workshop → sistema gera QR code (join_code = 8 chars)
2. Aluno escaneia QR code → abre PWA no browser
3. PWA detecta: usuário não autenticado → redireciona para /onboarding?code=ABCD1234
4. Aluno preenche nome, e-mail, senha (3 campos) → POST /auth/register
5. Sistema cria user, wallet, vincula ao workshop (workshop_participants)
6. Redirecionamento para /workshop/:id → socket.emit('join', ...)
7. Aluno vê sala de espera com lista de participantes
8. Instrutor inicia sessão → todos recebem evento 'session_started'
9. Fim do workshop → aluno recebe badge "Primeira Sessão" + XP de participação
10. Push notification: "Continue sua trilha de Fundamentos!"
```

### 7.2 Sessão de Cupping Sincronizado no Workshop

```
1. Instrutor no painel: clica "Iniciar Cupping" → seleciona amostra da sessão
2. socket.emit('start_cupping', { sampleId })
3. Todos os alunos recebem 'cupping_started' → tela de formulário SCA abre
4. Alunos preenchem atributos e descritores (offline-safe: dados em memória)
5. Aluno confirma → socket.emit('cupping_entry', { dados }) + POST /cupping/samples/:id/entries
6. Servidor agrega: calcula média, atualiza heatmap
7. Instrutor vê heatmap atualizado em tempo real
8. Instrutor clica "Fechar amostra" → socket.emit('close_sample')
9. Alunos recebem 'sample_closed' → botão "Ver comparação" ativado
10. Aluno toca "Ver comparação" → tela de delta (meu score vs instrutor vs média)
```

### 7.3 Missão Diária (Barista)

```
1. Barista abre app → sistema verifica missões do dia (GET /missions)
2. Feed exibe: "Complete 1 card + 1 quiz" (missão diária) e "Registre 1 cupping" (missão semanal)
3. Barista completa card → POST /lessons/:id/complete → +5 XP
4. Barista completa quiz → POST /lessons/:id/complete → +10 XP
5. Worker detecta missão completada → POST /missions/:id/claim (automático)
6. +5 Beans creditados na wallet → XPToast aparece
7. Streak incrementa → animação de streak na tela home
8. Se streak múltiplo de 7 → badge "Dedicação Semanal" desbloqueado → push notification
```

### 7.4 Criação de Campanha pelo Gestor

```
1. Gestor acessa /dashboard → clica "Nova Campanha"
2. Preenche: nome, quiz vinculado, recompensa (R$ 5 de desconto), datas
3. POST /campaigns → sistema gera QR code da campanha
4. Gestor faz download do QR code (PNG/PDF) para imprimir nas mesas
5. Cliente escaneia → abre PWA, faz o quiz
6. Quiz completado → cupom gerado (código único) + XP para o cliente
7. Cliente apresenta cupom no caixa → operador valida pelo app (tela simplificada)
8. Conversão registrada → visível no dashboard do gestor
```

---

## 8. Critérios de Aceite por Módulo

### Módulo de Autenticação e Perfil
- [ ] Cadastro completo (e-mail, Google, Apple) funciona em iOS Safari 15+ e Chrome Android 100+
- [ ] Cadastro via QR code de workshop em ≤ 60 segundos (fluxo end-to-end)
- [ ] JWT expira em 1h; refresh token válido por 30d
- [ ] 2FA obrigatório para `instructor`, `cafe_manager` e `admin`
- [ ] Passaporte público acessível sem autenticação, dados privados protegidos
- [ ] LGPD: tela de consentimento exibida no primeiro acesso, opt-in granular para dados sensoriais

### Módulo de Aprendizagem
- [ ] Trilhas da biblioteca central visíveis para todos os tenants
- [ ] Algoritmo SM-2 implementado: ease_factor e interval_days atualizados corretamente após cada resposta
- [ ] Todos os 5 tipos de quiz renderizados sem bugs em mobile
- [ ] Microvídeos ≤ 90s entregues via Mux/Cloudflare Stream com fallback de qualidade (480p mínimo em 3G)
- [ ] Quiz obrigatório ao final de cada vídeo antes de prosseguir
- [ ] Progresso cacheado localmente (IndexedDB) e sincronizado ao reconectar

### Módulo Sensorial
- [ ] Todos os 10 atributos SCA com slider 0–10 (step 0.25) funcional em touch
- [ ] Roda de sabores: seleção de até 10 descritores, dois níveis de hierarquia
- [ ] Score total calculado em tempo real (fórmula SCA)
- [ ] Comparação disponível apenas após instrutor fechar amostra
- [ ] Exportação PDF gerada em ≤ 5 segundos, formatada conforme template SCA
- [ ] Cupping salvo offline e sincronizado sem perda de dados

### Módulo Gamificação
- [ ] XP atualizado em ≤ 2 segundos após ação pontuável
- [ ] Leaderboard Redis atualizado em ≤ 5 segundos
- [ ] Engine de badges processa triggers de forma assíncrona sem bloquear a UX
- [ ] Streak calculado corretamente considerando fuso horário do tenant
- [ ] Wallet nunca entra em saldo negativo (transação rejeitada se saldo insuficiente)
- [ ] Missões expiram à meia-noite no fuso do tenant

### Módulo Workshop ao Vivo
- [ ] 500 usuários simultâneos em uma sala sem degradação perceptível de latência (<200ms p95)
- [ ] Fallback para polling automático se WebSocket desconectar (sem intervenção do usuário)
- [ ] QR code de entrada gerado em ≤ 2 segundos
- [ ] Certificado digital com URL pública verificável e QR code único por participante
- [ ] Relatório pós-sessão com heatmap, curva de acerto e lista de participantes exportável em PDF

### Dashboard B2B
- [ ] DAU/MAU calculado corretamente (usuários únicos por dia/mês no tenant)
- [ ] Dados do dashboard atualizados a cada 1 hora (job agendado)
- [ ] Exportação CSV funcional para todos os relatórios
- [ ] QR code de campanha gerado e baixável em PNG e PDF
- [ ] Recompensa validável pelo operador via código de 8 dígitos no app

---

## 9. Requisitos Não-Funcionais Detalhados

### 9.1 Performance

| Métrica | Target | Medição |
|---------|--------|---------|
| TTI (Time to Interactive) | ≤ 3s em 4G simulado | Lighthouse CI no pipeline |
| Resposta de quiz | ≤ 500ms p95 | APM (Sentry) |
| Latência WebSocket (workshop) | ≤ 200ms p95 | Métricas Socket.io |
| Geração de PDF | ≤ 5s | Logs do worker |
| Carregamento do leaderboard | ≤ 300ms | Redis sorted set |

### 9.2 Escalabilidade

- Workshop ao vivo: 500 usuários/sessão simultâneos.
- Escala horizontal: `ws-service` stateless com Redis como broker de pub/sub entre instâncias.
- `api-service`: horizontal scaling via HPA no Kubernetes (CPU > 70% → +1 pod).
- Banco de dados: read replicas para queries de leitura intensiva (leaderboards, relatórios).

### 9.3 Segurança

- TLS 1.3 em trânsito, AES-256 em repouso.
- Secrets gerenciados via AWS Secrets Manager ou Vault.
- Rate limiting: 100 req/min por IP para endpoints públicos; 1000 req/min por user autenticado.
- OWASP Top 10: revisão no checklist de PR para endpoints críticos.
- 2FA obrigatório para roles com acesso a dados de terceiros (instructor, cafe_manager, admin).
- Auditoria de acesso a dados sensoriais (log imutável).

### 9.4 Privacidade (LGPD)

- Opt-in explícito para dados sensoriais agregados (não default).
- Anonimização de cuppings antes de agregação para dashboards de terceiros (hash do user_id).
- Direito de exclusão: endpoint `DELETE /me` remove dados pessoais, anonimiza cuppings históricos.
- DPA (Data Processing Agreement) assinado com cada tenant antes do acesso.
- Auditoria anual de conformidade.

### 9.5 Acessibilidade (WCAG 2.1 AA)

- Contraste mínimo 4.5:1 para texto normal, 3:1 para texto grande.
- Todos os sliders (cupping) acessíveis via teclado (setas) e screen reader.
- Roda de sabores com alternativa textual (lista de checkboxes para screen readers).
- Foco visível em todos os elementos interativos.
- Testes com VoiceOver (iOS) e TalkBack (Android) antes do launch.

### 9.6 Internacionalização

- Strings armazenadas em arquivos i18n (`pt-BR`, `en`, `es`).
- Datas, números e moedas formatados pelo locale do usuário.
- Conteúdo de trilhas localizável por tenant (campo `locale` na lição).
- PT-BR é o locale padrão; fallback para EN se tradução ausente.

---

## 10. Plano de Testes

### 10.1 Pirâmide de Testes

```
         /\
        /  \     E2E (Playwright)
       /────\    ~20 fluxos críticos
      /      \
     /────────\  Integration (Supertest + TestContainers)
    /          \  ~100 casos por serviço
   /────────────\ Unit (Jest/Vitest)
  /              \ ~80% cobertura de negócio
```

### 10.2 Fluxos Críticos para E2E

1. Cadastro via QR code de workshop → participação → primeiro badge
2. Completar trilha completa (5 cards + quiz final) → level up
3. Registrar cupping SCA completo → exportar PDF
4. Instrutor cria workshop → aluno entra → cupping sincronizado → relatório
5. Gestor cria campanha → cliente participa → recompensa validada
6. Streak 7 dias → badge desbloqueado → notificação push recebida
7. Workshop com 50 usuários simultâneos (smoke test de carga)

### 10.3 Testes de Carga

| Cenário | Ferramenta | Critério de Aceite |
|---------|-----------|-------------------|
| 500 usuários em workshop | k6 | Latência WS p95 ≤ 200ms, 0 mensagens perdidas |
| 1000 req/s na API REST | k6 | p99 ≤ 500ms, error rate < 0.1% |
| Geração de 100 PDFs simultâneos | k6 | Todos entregues em ≤ 30s |

### 10.4 Definition of Done

Uma feature só é considerada pronta quando:
- [ ] Código revisado por 1 peer (PR aprovado)
- [ ] Testes unitários escritos (cobertura ≥ 80% da lógica de negócio nova)
- [ ] Testes de integração cobrindo happy path e principais error paths
- [ ] Critérios de aceite da user story verificados manualmente em staging
- [ ] Acessibilidade verificada (contraste, foco, labels)
- [ ] Sem erros críticos no Sentry (staging)
- [ ] Feature flag ativa em staging antes de ir para produção

---

## 11. Decisões em Aberto

| ID | Decisão | Prazo | Responsável | Impacto |
|----|---------|-------|-------------|---------|
| D-01 | Nome definitivo do produto (CoffeeQuest é placeholder) | Antes do design system | Ricardo + Marketing | Alto — afeta domínio, branding |
| D-02 | Auth0 vs Keycloak — custo vs controle | Sprint 1 | Tech Lead | Alto — bloqueia RF-01/02 |
| D-03 | NestJS vs Go para `ws-service` — velocidade de dev vs performance | Sprint 1 | Tech Lead | Médio — Go é mais performático mas NestJS reutiliza a stack |
| D-04 | Licença da roda de sabores SCA — uso comercial permitido? | Discovery | Ricardo + Jurídico | Alto — bloqueia RF-21 |
| D-05 | Curadoria do conteúdo base — equipe interna ou rede de Q-Graders? | Discovery | Ricardo | Alto — bloqueia RF-10/11 |
| D-06 | Parceria SCA/BSCA viável no MVP para credibilidade institucional? | Discovery | Ricardo | Médio — afeta Go-to-market |
| D-07 | Dados de cupping agregados como produto para torrefadores — modelo de negócio e consentimento | Antes do piloto | Ricardo + Jurídico | Alto — LGPD e receita |
| D-08 | Stripe vs Asaas para recorrência B2B | Sprint 2 | Tech Lead + Financeiro | Médio — Asaas tem melhor suporte a boleto BR |

---

*Próximos passos: validar D-01 a D-03 até o kickoff do Sprint 1; agendar entrevistas de Discovery conforme cronograma do PRD.*
