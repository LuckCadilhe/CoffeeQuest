<div align="center">

<img src="https://img.shields.io/badge/status-em%20desenvolvimento-brown?style=for-the-badge" />
<img src="https://img.shields.io/badge/plataforma-PWA-6B3F1A?style=for-the-badge" />
<img src="https://img.shields.io/badge/versão-0.1%20draft-E8A836?style=for-the-badge" />

# ☕ CoffeeQuest

**A plataforma de gamificação para educação em café especial na América Latina.**

Aprenda sobre café como um jogo. Registre cuppings. Compete com colegas. Evolua como barista.

[Visão Geral](#-visão-geral) · [Funcionalidades](#-funcionalidades) · [Personas](#-para-quem) · [Stack](#-stack) · [Roadmap](#-roadmap) · [Como Rodar](#-como-rodar)

---

</div>

## 🎯 Visão Geral

O mercado de café especial carece de uma ferramenta que combine **aprendizagem estruturada**, **prática sensorial** e **mecânicas sociais de retenção** num mesmo ecossistema.

O CoffeeQuest resolve isso conectando baristas, clientes curiosos, escolas e cafeterias numa plataforma onde aprender sobre café é tão envolvente quanto jogar.

```
┌─────────────────────────────────────────────────────────┐
│  Cafeteria / Escola (tenant)                            │
│                                                         │
│   Baristas ──► Trilhas de Aprendizagem                  │
│   Clientes ──► Gamificação + Recompensas                │
│   Instrutores ► Workshops ao Vivo                       │
│   Gestores ───► Dashboard B2B + Campanhas               │
└─────────────────────────────────────────────────────────┘
```

> **Modelo de negócio:** B2B2C — a plataforma é licenciada para cafeterias e escolas (SaaS), que a usam para engajar seus clientes e alunos.

---

## ✨ Funcionalidades

### 📚 Trilhas de Aprendizagem
- Trilhas temáticas: Origens, Processos, Torra, Extração, Sensorial, Latte Art, Comercial
- Cards com **spaced repetition** (algoritmo SM-2) para retenção de longo prazo
- Quizzes com múltipla escolha, verdadeiro/falso, associação, ordenação e imagem-resposta
- Microvídeos de até 90 segundos com quiz obrigatório ao final
- Dificuldade adaptativa por módulo

### 👅 Cupping SCA Digital
- Formulário completo no padrão SCA com os 10 atributos (sliders precisos de 0 a 10)
- **Roda de sabores interativa** com dois níveis de hierarquia
- Score total calculado em tempo real
- Comparação da sua avaliação com a do instrutor e a média da turma
- Histórico pessoal de cuppings com filtros por origem, processo e data
- Exportação em PDF no template SCA

### 🏆 Gamificação
- Sistema de **XP e níveis** (20 níveis no MVP)
- **Badges** desbloqueáveis por conquistas (configuráveis por cafeteria)
- Moeda virtual **"Beans"** com saldo e extrato
- **Streaks** diários com tolerância configurável
- Missões diárias, semanais e por evento
- Leaderboards por cafeteria, global e por turma

### 🎓 Workshop ao Vivo
- Entrada via **QR code** — aluno cria conta e entra em menos de 60 segundos
- Painel do instrutor com controle de slides, timer e ranking ao vivo
- **Cupping sincronizado**: todos avaliam a mesma amostra ao mesmo tempo
- Heatmap de respostas atualizado em tempo real
- Relatório pós-sessão exportável
- **Certificado digital** verificável com QR code único

### 📊 Dashboard B2B (Cafeteria/Escola)
- Métricas de engajamento: DAU/MAU, top usuários, retenção D1/D7/D30
- Criação de **campanhas locais** com quiz temático e recompensas
- QR codes de campanha para imprimir nas mesas
- Relatório de conversão: engajamento → visita → venda
- Cardápio interativo com QR na mesa

---

## 👤 Para Quem

| Persona | Perfil | Como usa o CoffeeQuest |
|---------|--------|------------------------|
| **Lucas** — Barista em Formação | 22 anos, quer crescer na carreira | Estuda diariamente, registra cuppings, compete no leaderboard |
| **Marina** — Cliente Entusiasta | 34 anos, curiosa sobre café especial | Usa na cafeteria, aprende de forma leve, ganha recompensas |
| **Rafael** — Instrutor / Q-Grader | 41 anos, conduz workshops | Gerencia turmas, acompanha progresso, reutiliza conteúdo |
| **Júlia** — Dona da Cafeteria | 38 anos, quer fidelizar clientes | Acessa o dashboard, cria campanhas, analisa métricas |

---

## 📱 Telas Principais

```
APP (Trainee — mobile)            WEB (Gestor / Instrutor)
┌─────────────────────┐           ┌──────────────────────────┐
│  🔥 Streak: 7 dias  │           │  Dashboard               │
│                     │           │  ├─ DAU/MAU              │
│  Missão do dia      │           │  ├─ Top usuários         │
│  ├─ 1 card          │           │  └─ Campanhas ativas     │
│  └─ 1 quiz          │           │                          │
│                     │           │  Workshops               │
│  Continuar trilha   │           │  ├─ Criar sessão         │
│  "Extração" 60%     │           │  ├─ QR code de entrada   │
│  ████████░░         │           │  └─ Painel ao vivo       │
│                     │           │                          │
│ [Casa][Trilhas][☕]  │           │  Relatórios              │
│      [🏆][Perfil]   │           │  └─ Exportar CSV/PDF     │
└─────────────────────┘           └──────────────────────────┘
```

---

## 🛠 Stack

| Camada | Tecnologia |
|--------|-----------|
| **Frontend** | Next.js 14 + TypeScript + TailwindCSS + shadcn/ui (PWA) |
| **Backend** | NestJS (API REST) + Go (WebSocket / workshop ao vivo) |
| **Banco de Dados** | PostgreSQL 15 (principal) + Redis (cache, leaderboards, sessões) |
| **Mídia** | AWS S3 + Mux / Cloudflare Stream (vídeos) |
| **Real-time** | Socket.io (WebSockets) |
| **Filas** | RabbitMQ + BullMQ |
| **Auth** | Auth0 / Keycloak (OAuth2 + JWT + 2FA) |
| **Infra** | Docker + Kubernetes (AWS/GCP) |
| **Analytics** | PostHog (produto) + Metabase (BI interno) |
| **Pagamentos** | Stripe / Asaas (recorrência B2B) |

### Arquitetura resumida

```
PWA (Next.js)
     │
     ├── REST API (NestJS) ──► PostgreSQL
     │                    ──► Redis
     │                    ──► RabbitMQ ──► Worker (badges, PDFs, notificações)
     │
     └── WebSocket (Go/NestJS) ──► Redis Pub/Sub ──► Sessões de workshop
```

---

## 💰 Planos

| Plano | Preço | Para quem |
|-------|-------|-----------|
| **Café** | R$ 299/mês | Cafeterias pequenas — até 500 usuários |
| **Escola** | R$ 799/mês | Escolas e torrefações — usuários ilimitados + workshops |
| **Enterprise** | Sob consulta | Multi-unidade, API, white-label |
| **Freemium** | Grátis | Alunos individuais (com ads) |
| **Pro** | R$ 19/mês | Alunos — sem ads + trilhas premium |

---

## 🗺 Roadmap

### MVP (em desenvolvimento)
- [x] Especificação técnica e de produto
- [ ] Design system + protótipo Figma
- [ ] Autenticação e perfis (multi-tenant)
- [ ] Módulo de trilhas + spaced repetition
- [ ] Formulário de cupping SCA + roda de sabores
- [ ] Engine de gamificação (XP, badges, leaderboard)
- [ ] Workshop ao vivo (WebSocket)
- [ ] Dashboard B2B
- [ ] PWA mobile-first

### V2
- [ ] App nativo iOS / Android
- [ ] Realidade aumentada (leitura de rótulos)
- [ ] Marketplace de recompensas com parceiros
- [ ] Modo torneio inter-cafeterias
- [ ] Integração com POS da cafeteria

### V3
- [ ] IA para análise de descritores sensoriais em texto livre
- [ ] Recomendação personalizada de grãos
- [ ] Integração com e-commerce
- [ ] Benchmarks de cupping para torrefadores

---

## 🚀 Como Rodar

> **Pré-requisitos:** Node.js 20+, Docker, pnpm

```bash
# Clone o repositório
git clone https://github.com/LuckCadilhe/CoffeeQuest.git
cd CoffeeQuest

# Suba os serviços de infraestrutura
docker compose up -d

# Instale as dependências
pnpm install

# Configure as variáveis de ambiente
cp .env.example .env
# edite o .env com suas credenciais

# Rode as migrations
pnpm db:migrate

# Inicie em modo desenvolvimento
pnpm dev
```

Acesse `http://localhost:3000`

---

## 📋 Métricas de Sucesso (MVP)

| Métrica | Meta |
|---------|------|
| Cuppings registrados por usuário ativo/mês | North Star |
| DAU/MAU | ≥ 35% |
| Taxa de conclusão de trilhas | ≥ 60% |
| Retenção D30 pós-workshop | ≥ 40% |
| NPS (alunos) | ≥ 50 |
| NPS (gestores) | ≥ 60 |
| Receita recorrente por cafeteria (ano 1) | ≥ R$ 400/mês |

---

## 📄 Documentação

- [`SPEC_COFFEEQUEST.md`](./SPEC_COFFEEQUEST.md) — Especificação técnica, de produto e de design completa

---

## 📬 Contato

**Lucas Cadilhe** — Autor e Product Owner  
Projeto em fase de Discovery / Alinhamento — v0.1

---

<div align="center">

Feito com ☕ e muito cupping

</div>
