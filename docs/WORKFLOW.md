# Workflow do Projeto — CoffeeQuest
**Fases, Entregas e Responsáveis — Discovery ao GA**

**Versão:** 1.0  
**Data:** 2026-04-27  
**Baseado em:** PRD v0.1 · SPEC v1.0 · AUDIT_HOLISTIC v1.0

---

## Visão Geral da Timeline

```
Sem. 1–4       Sem. 5–8      Sem. 9–20        Sem. 21–28    Sem. 29+
[  Discovery  ][   Design   ][   MVP Dev (12w) ][   Piloto   ][ GA ]
     4 sem          4 sem          12 sem            8 sem
```

**Meta total:** General Availability em ~7 meses a partir do kickoff.

---

## Milestones

| # | Marco | Semana | Critério de Saída |
|---|-------|--------|------------------|
| M1 | Escopo MVP-1 aprovado | 4 | Preço validado · ADRs D-02/D-03 fechados · Squad definido |
| M2 | Protótipo aprovado | 8 | Teste de usabilidade concluído · Infra base funcionando |
| M3 | MVP em staging | 20 | Testes E2E passando · Pentest concluído · SLOs verificados |
| M4 | Piloto concluído | 28 | NPS gestor ≥ 40 · 150 usuários ativos · 500 cuppings · 10 workshops |
| M5 | GA | 29+ | Planos Café e Escola publicados · Cobrança ativa |

---

## Fase 1 — Discovery & Validação (Semanas 1–4)

### Objetivo
Validar as premissas de negócio antes de escrever código. Resolver os bloqueadores críticos identificados no audit.

### Entregas

| Tarefa | Tipo | Responsável | Saída Esperada |
|--------|------|-------------|----------------|
| 10 entrevistas com donos de cafeteria | 🔴 Bloqueador | Ricardo | Decisão de preço validada |
| Definir estratégia de curadoria de conteúdo | 🔴 Bloqueador | Ricardo | Contrato/acordo com Q-Grader ou escola parceira |
| TAM / SAM / SOM | Negócio | Ricardo | Planilha de tamanho de mercado |
| Modelo financeiro (3 cenários) | Negócio | Ricardo + Financeiro | Runway, CAC, LTV, breakeven |
| Definir squad e headcount | Produto | Ricardo | Headcount plan + custo estimado |
| Revisar escopo do MVP | Produto | Ricardo + Tech Lead | MVP-1 redefinido (escopo viável para 12 semanas) |
| Licença FlavorWheel — D-04 | 🔴 Bloqueador | Ricardo + Jurídico | Decisão: licença SCA ou roda própria |
| Spike técnico: Auth0 vs Keycloak — D-02 | Infra | Tech Lead | ADR documentado |
| Spike técnico: NestJS vs Go (ws-service) — D-03 | Infra | Tech Lead | ADR documentado |

### Critério de Saída do M1
- [ ] Preço validado com ≥ 8 entrevistas de cafeteria
- [ ] Squad definido com headcount e modelo de contratação
- [ ] ADRs D-02 e D-03 assinados
- [ ] Escopo MVP-1 reduzido e aprovado
- [ ] Curador de conteúdo identificado

---

## Fase 2 — Design & Prototipagem (Semanas 5–8)

### Objetivo
Produzir o design system completo, o protótipo navegável e a infraestrutura base antes de começar o desenvolvimento.

### Entregas

| Tarefa | Tipo | Responsável | Saída Esperada |
|--------|------|-------------|----------------|
| Design system no Figma | Design | Designer | Todos os tokens + componentes do DESIGN_GUIDELINES.md |
| Protótipo navegável (3 fluxos) | Design | Designer | Onboarding QR · Missão diária · Cupping SCA |
| Teste de usabilidade (5 usuários) | Design + Produto | Designer + Ricardo | Lista de ajustes priorizados |
| CuppingLite para usuários casuais | Design | Designer | Spec do componente (3–4 atributos) |
| Setup do monorepo (Turborepo) | Infra | Tech Lead | Estrutura apps/ + packages/ + infra/ funcionando |
| Infraestrutura base (Docker + CI/CD) | Infra | DevOps | Docker Compose local + GitHub Actions pipeline |
| Automação de onboarding de tenant | Infra | Backend | Script de criação de schema + migrations |

### Critério de Saída do M2
- [ ] Protótipo validado em teste de usabilidade (≥ 5 participantes)
- [ ] Design system completo com light + dark mode
- [ ] Pipeline CI verde (lint → typecheck → test → build)
- [ ] Docker Compose local funcionando com todos os serviços

---

## Fase 3 — Desenvolvimento do MVP (Semanas 9–20)

### Objetivo
Desenvolver os 5 módulos do MVP-1 com qualidade de produção. Ciclos de sprint de 2 semanas com demo interna ao final de cada um.

### Sprints

#### Sprint 1–2 (Semanas 9–12): Autenticação e Perfis
- RF-01 a RF-04
- Cadastro via QR em ≤ 60s, papéis, multi-tenant, 2FA
- JWT + Refresh Token Rotation
- LGPD consent screen + fluxo para menores de 18 anos
- Passaporte público

#### Sprint 3–5 (Semanas 13–18): Módulo de Aprendizagem
- RF-10 a RF-15
- Trilhas + módulos + lições (JSONB com schema documentado)
- Algoritmo SM-2 (spaced repetition)
- 5 tipos de quiz
- Player de microvídeo (Mux/Cloudflare Stream)
- Progresso offline cacheado (IndexedDB + sync)

#### Sprint 4–6 (Semanas 15–20): Módulo Sensorial
- RF-20 a RF-24 (paralelo com Aprendizagem a partir da Sprint 4)
- Formulário SCA completo (10 atributos, step 0.25)
- FlavorWheel interativo (pós-decisão D-04)
- CuppingLite para usuários casuais
- Comparação com instrutor e média da turma
- Exportação PDF (react-pdf ou pdfmake — não Puppeteer)

#### Sprint 5–7 (Semanas 17–22): Gamificação
- RF-30 a RF-35
- Engine de XP configurável por tenant
- Sistema de badges assíncrono (BullMQ)
- Wallet de Beans (transações imutáveis)
- Streaks com tolerância e fuso do tenant
- Missões diárias/semanais com expiração à meia-noite
- Leaderboard (Redis sorted sets + jobs a cada 5min)

#### Sprint 7–10 (Semanas 21–26): Workshop ao Vivo
- RF-40 a RF-45
- ws-service com Socket.io + Redis Pub/Sub
- QR de entrada (gerado em ≤ 2s)
- Painel do instrutor: slides, timer, ranking ao vivo
- Cupping sincronizado (upsert com `ON CONFLICT DO UPDATE`)
- Heatmap em tempo real (atualização ≤ 1s)
- Fallback de polling automático (5s) sem intervenção do usuário
- Certificado digital verificável (QR + URL pública)
- Relatório pós-sessão exportável

#### Sprint 9–11 (Semanas 25–28): Dashboard B2B
- RF-50 a RF-54
- DAU/MAU, top usuários, retenção D1/D7/D30
- Campanhas com QR imprimível e validação por código
- Gestão de recompensas físicas
- Relatório de conversão exportável em CSV
- Jobs: métricas a cada 1h, missões à meia-noite

#### Sprint 10–12 (Semanas 27–30): PWA, Performance e Segurança
- Service Worker, manifest, add-to-homescreen
- Lighthouse CI: TTI ≤ 3s em 4G simulado
- Fallback de push: FCM → in-app feed → e-mail
- Testes em iOS Safari 15+ e Chrome Android 100+
- Rate limiting por tenant (Redis)
- Plano de resposta a incidentes documentado
- Pentest básico de multi-tenant (SQL injection no schema resolver)
- Sentry APM + PostHog feature flags + alertas

#### Testes E2E Contínuos (Playwright — ao longo de todos os sprints)
7 fluxos críticos:
1. Onboarding via QR → workshop → primeiro badge
2. Trilha completa (5 cards + quiz final) → level up
3. Cupping SCA completo → exportar PDF
4. Workshop: instrutor cria → aluno entra → cupping sincronizado → relatório
5. Gestor cria campanha → cliente participa → recompensa validada
6. Streak 7 dias → badge desbloqueado → notificação push
7. Workshop com 50 usuários simultâneos (smoke test)

### Critério de Saída do M3
- [ ] Todos os módulos do MVP-1 funcionando em staging
- [ ] 7 fluxos E2E passando sem falhas
- [ ] Pentest concluído sem incidentes críticos
- [ ] Lighthouse TTI ≤ 3s em 4G simulado
- [ ] Todos os SLOs verificados (API p95 ≤ 500ms, WebSocket p95 ≤ 200ms)

---

## Fase 4 — Piloto Fechado (Semanas 21–28)

### Objetivo
Validar o produto com usuários reais. Iterar semanalmente até atingir os critérios de aceite do MVP.

### Estrutura do Piloto
- **Parceiros:** 3 cafeterias + 2 escolas de café
- **Usuários esperados:** ~200 (meta mínima: 150 ativos/semana)
- **Duração:** 8 semanas consecutivas
- **Cadência de iteração:** Sprint de 1 semana (feedback → priorização → fix → deploy)

### Entregas

| Tarefa | Tipo | Responsável | Saída Esperada |
|--------|------|-------------|----------------|
| Onboarding das 3 cafeterias piloto | Negócio | Ricardo + CS | Tenant configurado + QR nas mesas |
| NPS semanal (gestores e alunos) | Produto | Ricardo | Score NPS por semana |
| Entrevistas quinzenais de follow-up | Produto | Ricardo | Backlog de ajustes priorizados |
| Monitoramento de SLOs | Infra | DevOps | Dashboard de uptime e performance |
| Acompanhar North Star | Produto | Ricardo | Cuppings/usuário ativo/mês |
| Preparar go-to-market | Negócio | Ricardo | Canais, materiais de venda, deck |
| Plano de escala (k6 load testing) | Infra | DevOps | 500 usuários em workshop sem degradação |

### Critério de Saída do M4 (Critérios de Aceite do MVP)
- [ ] 3 cafeterias piloto rodando por 60 dias consecutivos
- [ ] ≥ 150 usuários ativos semanais entre os pilotos
- [ ] ≥ 500 cuppings registrados
- [ ] ≥ 10 workshops ao vivo com taxa de sucesso técnico ≥ 95%
- [ ] NPS de gestores ≥ 40
- [ ] Nenhum incidente crítico de segurança ou LGPD

---

## Fase 5 — General Availability (Semana 29+)

### Objetivo
Lançamento público com planos Café e Escola. Expansão gradual.

### Entregas

| Tarefa | Tipo | Responsável | Saída Esperada |
|--------|------|-------------|----------------|
| Publicar Planos Café e Escola | Negócio | Ricardo | Página de pricing + cobrança recorrente ativa |
| Ativar cobrança (Asaas/Stripe) | Infra | Backend | Recorrência mensal funcionando |
| Comunicado para lista de espera e parceiros | Marketing | Ricardo | E-mail + post em canais especializados |
| Presença em eventos do setor | Negócio | Ricardo | SCA Expo · Semana Internacional do Café |
| Parceria ABIC / BSCA | Negócio | Ricardo | Acordo formal assinado |
| Kickoff roadmap V2 | Produto | Ricardo + Tech Lead | Backlog priorizado para V2 |

---

## Definition of Done (por feature)

Uma feature só é considerada pronta quando:

- [ ] Código revisado por 1 peer (PR aprovado)
- [ ] Testes unitários com ≥ 80% de cobertura da lógica de negócio nova
- [ ] Testes de integração cobrindo happy path e principais error paths
- [ ] Critérios de aceite da user story verificados manualmente em staging
- [ ] Acessibilidade verificada (contraste, foco, labels ARIA)
- [ ] Sem erros críticos no Sentry (staging)
- [ ] Feature flag ativa em staging antes de ir para produção
- [ ] Dark mode verificado no componente

---

## Responsáveis por Área

| Área | Responsável | Escopo |
|------|-------------|--------|
| Product Owner | Ricardo Coelho | Visão, priorização, validação com clientes |
| Tech Lead | A definir | Arquitetura, ADRs, revisão de PRs críticos |
| Backend | A definir | api-service, ws-service, worker-service |
| Frontend | A definir | PWA (Next.js), design system, componentes |
| Design | A definir | Figma, protótipos, testes de usabilidade |
| DevOps | A definir | Infra, CI/CD, monitoramento, escala |
| Customer Success | A definir | Onboarding de tenants, NPS, suporte piloto |
| Curador Editorial | A definir (Q-Grader) | Trilhas de conteúdo, revisão técnica |

---

*Próxima atualização: após M1 (Semana 4), quando o escopo do MVP-1 e o squad estiverem definidos.*
