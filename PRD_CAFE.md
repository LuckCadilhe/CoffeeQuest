# PRD — CoffeeQuest (nome provisório)
**Plataforma de Gamificação para Educação em Café Especial**

**Versão:** 0.1 (Draft)
**Data:** 2026-04-22
**Autor:** Ricardo Coelho
**Status:** Discovery / Alinhamento

---

## 1. Visão e Contexto

### 1.1 Problema
A educação em café especial hoje é fragmentada: conteúdo denso da SCA de um lado, experiências casuais de consumidor do outro. Cafeterias e escolas investem em workshops presenciais com alto custo de instrutor, mas o engajamento pós-evento cai drasticamente — o aluno não retém o conhecimento e a cafeteria perde o vínculo. Não existe uma ferramenta que combine **aprendizagem estruturada**, **prática sensorial** e **mecânicas sociais de retenção** num mesmo ecossistema.

### 1.2 Visão do Produto
Ser a plataforma de referência para **aprendizagem gamificada de café especial** na América Latina, conectando baristas, clientes curiosos, escolas e cafeterias num ecossistema onde aprender sobre café é tão envolvente quanto jogar.

### 1.3 Objetivos de Negócio
- **B2B2C** — licenciar a plataforma para cafeterias e escolas (SaaS), que a usam para engajar seus clientes/alunos.
- Criar um **acervo de dados sensoriais** (cuppings, preferências, origens) valioso para torrefadores e produtores.
- Tornar-se **canal de distribuição** para campanhas de fazendas, torrefadores e eventos do setor.

### 1.4 Métricas de Sucesso (North Star)
- **North Star:** Cuppings registrados por usuário ativo por mês (proxy de aprendizagem + engajamento).
- **Suportes:**
  - DAU/MAU ≥ 35%
  - Taxa de conclusão de trilhas ≥ 60%
  - Retenção D30 pós-workshop ≥ 40%
  - NPS ≥ 50 (alunos) e ≥ 60 (gestores de cafeteria)
  - Receita recorrente por cafeteria ≥ R$ 400/mês no ano 1

---

## 2. Personas

### 2.1 Lucas, o Barista em Formação (22, primário)
Trabalha há 1 ano numa cafeteria, quer crescer na carreira, faz cursos esporádicos. Usa o app diariamente para estudar, registrar cuppings e competir com colegas.
**Dores:** conteúdo caro, falta de prática estruturada, dificuldade de calibração sensorial.
**Ganhos:** trilhas claras, feedback imediato, reconhecimento social.

### 2.2 Marina, a Cliente Entusiasta (34, secundário)
Frequenta cafeterias especiais, compra grãos para casa, curiosa mas leiga tecnicamente. Usa o app casualmente quando está na cafeteria.
**Dores:** não entende o menu, sente-se intimidada pelo jargão.
**Ganhos:** aprende de forma leve, ganha recompensas reais, vira "embaixadora".

### 2.3 Rafael, o Instrutor/Q-Grader (41, operacional)
Conduz workshops em escolas de café. Precisa de ferramentas para engajar turmas e acompanhar progresso.
**Dores:** alunos dispersos, difícil medir aprendizagem, material repetitivo.
**Ganhos:** sala ao vivo, relatórios de turma, reuso de conteúdo.

### 2.4 Júlia, a Dona da Cafeteria (38, decisor B2B)
Quer fidelizar clientes e diferenciar-se. Considera pagar mensalidade se ver retorno em frequência e ticket médio.
**Dores:** campanhas de fidelidade genéricas, concorrência acirrada.
**Ganhos:** dashboard de engajamento, campanhas prontas, dados do cliente.

### 2.5 Admin Scigeniq (interno)
Gerencia tenants, conteúdo base, moderação, campanhas patrocinadas.

---

## 3. Escopo

### 3.1 Dentro do escopo (MVP — V1)
- Autenticação e perfis (aluno, instrutor, gestor, admin).
- Trilhas de aprendizagem com cards e quizzes.
- Sistema de XP, níveis e badges.
- Formulário de cupping SCA digital.
- Modo workshop ao vivo (instrutor + alunos em tempo real).
- Leaderboards (por cafeteria e global).
- Dashboard básico para gestor de cafeteria.
- Multi-tenant (cada cafeteria/escola como tenant).
- PWA mobile-first.

### 3.2 Fora do escopo (MVP)
- App nativo iOS/Android (fica para V2).
- Realidade aumentada.
- Marketplace de recompensas com terceiros.
- Integração com sistemas de POS da cafeteria.
- Modo offline completo (apenas cache leve no MVP).
- Certificação SCA oficial (requer parceria institucional).

### 3.3 Roadmap pós-MVP (V2 e além)
- V2: app nativo, AR de rótulos, marketplace, modo torneio inter-cafeterias.
- V3: IA para análise de descritores sensoriais em texto livre, recomendação personalizada de grãos, integração com e-commerce.

---

## 4. Requisitos Funcionais

### 4.1 Módulo de Autenticação e Perfil
- RF-01: Cadastro via e-mail, Google ou Apple.
- RF-02: Papéis: `trainee`, `instructor`, `cafe_manager`, `admin`.
- RF-03: Passaporte do Café público (perfil compartilhável).
- RF-04: Configuração de preferências (objetivos, nível auto-declarado, dietas).

### 4.2 Módulo de Aprendizagem
- RF-10: Trilhas temáticas (Origens, Processos, Torra, Extração, Sensorial, Latte Art, Comercial).
- RF-11: Cards de conhecimento com spaced repetition (algoritmo SM-2 ou similar).
- RF-12: Quizzes com múltipla escolha, V/F, associação, ordenação, imagem-resposta.
- RF-13: Dificuldade adaptativa por módulo.
- RF-14: Microvídeos (≤90s) com quiz obrigatório ao final.
- RF-15: Glossário pesquisável.

### 4.3 Módulo Sensorial
- RF-20: Formulário de cupping no padrão SCA (10 atributos + defeitos).
- RF-21: Roda de sabores interativa (tap-to-select + multi-nível).
- RF-22: Comparação de cuppings (seu vs instrutor vs média da turma).
- RF-23: Histórico e busca de cuppings com filtros (origem, processo, data, score).
- RF-24: Exportação em PDF do formulário de cupping.

### 4.4 Módulo Gamificação
- RF-30: Engine de XP configurável por tipo de evento (quiz, cupping, workshop, streak).
- RF-31: Sistema de badges com triggers configuráveis.
- RF-32: Moeda virtual ("Beans") com saldo e extrato.
- RF-33: Missões diárias, semanais e por evento.
- RF-34: Streaks com tolerância configurável.
- RF-35: Leaderboards filtráveis (global, cafeteria, turma, período).

### 4.5 Módulo Workshop ao Vivo
- RF-40: Criação de sessão com QR code de entrada.
- RF-41: Painel do instrutor com controle de slide, timer, ranking ao vivo.
- RF-42: Cupping sincronizado (todos avaliam a mesma amostra).
- RF-43: Visão em tempo real das respostas (heatmap, curva de acerto).
- RF-44: Relatório pós-sessão exportável.
- RF-45: Certificado digital verificável (QR + URL pública).

### 4.6 Módulo Cafeteria (B2B)
- RF-50: Dashboard com DAU/MAU, top usuários, campanhas ativas.
- RF-51: Cardápio interativo com QR na mesa.
- RF-52: Criação de campanhas locais (ex: "Quiz do café da semana").
- RF-53: Gestão de recompensas físicas (cupons, brindes).
- RF-54: Relatório de conversão (engajamento → visita → venda).

### 4.7 Módulo Admin (Scigeniq)
- RF-60: Gestão de tenants.
- RF-61: Biblioteca central de conteúdo (reutilizável por tenants).
- RF-62: Moderação de conteúdo gerado por usuários.
- RF-63: Configuração de regras globais de gamificação.

---

## 5. Requisitos Não-Funcionais

| Categoria | Requisito |
|---|---|
| **Performance** | TTI ≤ 3s em 4G; resposta de quiz ≤ 500ms p95 |
| **Escala** | Suportar 500 usuários simultâneos em workshop ao vivo por sessão |
| **Disponibilidade** | 99.5% uptime (fora de janelas de manutenção) |
| **Segurança** | LGPD-compliant, criptografia em trânsito (TLS 1.3) e em repouso, 2FA para instrutores/admins |
| **Privacidade** | Opt-in explícito para dados sensoriais agregados; anonimização para dashboards de terceiros |
| **Acessibilidade** | WCAG 2.1 AA, suporte a screen readers, contraste mínimo 4.5:1 |
| **Internacionalização** | PT-BR (default), EN, ES no MVP |
| **Mobile** | PWA funcional em iOS Safari 15+, Chrome Android 100+ |
| **Offline** | Cache de trilha em andamento; sincronização ao reconectar |
| **Observabilidade** | Logs estruturados, métricas de produto (Amplitude/PostHog), APM (Sentry) |

---

## 6. Arquitetura Técnica (Proposta)

### 6.1 Stack sugerida
- **Frontend:** Next.js 14 + TypeScript + TailwindCSS + shadcn/ui. PWA com Service Worker.
- **Backend:** Node.js (NestJS) ou Go para serviços de alta concorrência (workshop ao vivo).
- **Banco:** PostgreSQL 15 (dados relacionais) + Redis (cache, sessões, leaderboards) + S3 (mídia).
- **Real-time:** WebSocket (Socket.io) ou Phoenix Channels para sala ao vivo.
- **Filas:** RabbitMQ para processamento assíncrono (badges, notificações, relatórios).
- **Infra:** Docker + Kubernetes (ou ECS) na AWS/GCP.
- **Auth:** Auth0 ou Keycloak.
- **Analytics:** PostHog (produto) + Metabase (BI interno).

### 6.2 Modelo de dados (entidades principais)
- `users`, `tenants`, `memberships`
- `tracks`, `modules`, `lessons`, `cards`, `quizzes`, `questions`
- `cupping_sessions`, `cupping_entries`, `samples`
- `xp_events`, `badges`, `user_badges`, `wallets`, `transactions`
- `workshops`, `workshop_participants`, `live_events`
- `campaigns`, `rewards`, `redemptions`
- `leaderboards` (materialized view + Redis sorted sets)

### 6.3 Integrações
- **Pagamentos B2B:** Stripe ou Asaas (recorrência).
- **E-mail transacional:** Resend ou SES.
- **Push:** Firebase Cloud Messaging (via PWA).
- **Vídeo:** Mux ou Cloudflare Stream.
- **QR Code:** geração server-side.

---

## 7. Modelo de Negócio

### 7.1 Monetização
- **Plano Café** (cafeteria pequena): R$ 299/mês — até 500 usuários, branding básico.
- **Plano Escola** (escola/torrefação): R$ 799/mês — usuários ilimitados, conteúdo customizado, modo workshop.
- **Plano Enterprise:** sob consulta — multi-unidade, API, white-label.
- **Campanhas patrocinadas:** fazendas/torrefadores pagam por campanha no feed.
- **Freemium para alunos:** grátis com anúncios leves; Pro (R$ 19/mês) remove ads + trilhas premium.

### 7.2 Go-to-market
- Piloto com 3 cafeterias parceiras em São Paulo e Belo Horizonte.
- Parceria com associação (ABIC, BSCA) para credibilidade.
- Presença em eventos (Semana Internacional do Café, SCA Expo).

---

## 8. Jornadas Principais (resumo)

### 8.1 Primeira sessão do aluno
1. Recebe QR code no workshop → escaneia → cria conta em ≤60s.
2. Faz avaliação inicial (5 perguntas) → define trilha sugerida.
3. Participa do workshop ao vivo, ganha primeiro badge.
4. Pós-workshop: recebe push com "continue sua trilha".

### 8.2 Uso contínuo (barista)
1. Abre app pela manhã → missão diária (1 card + 1 quiz).
2. Durante turno → registra cupping de café novo da casa.
3. Noite → revê leaderboard, desafia colega para duelo.

### 8.3 Gestor de cafeteria
1. Login no dashboard → vê DAU da semana, top 5 clientes.
2. Cria campanha "Quiz do grão etíope" com recompensa de R$ 5 de desconto.
3. Imprime QR codes para mesas.

---

## 9. Riscos e Premissas

| Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|
| Cafeterias não enxergam ROI no SaaS | Média | Alto | Piloto gratuito de 60d + dashboard de conversão robusto |
| Conteúdo técnico SCA sem curadoria vira ruído | Média | Alto | Board editorial com Q-Graders; biblioteca central |
| Workshop ao vivo falha com rede instável | Alta | Médio | Fallback para modo assíncrono; testes de carga |
| Baixa retenção pós-workshop | Alta | Alto | Push contextual + missões diárias + streaks |
| LGPD — consentimento para dados sensoriais | Baixa | Alto | Opt-in granular, DPA, auditoria anual |
| Concorrência (Angel's Cup, Cropster) | Média | Médio | Foco no B2B2C + mercado LatAm (eles não atendem bem) |

### Premissas
- Cafeterias especiais têm disposição de pagar por engajamento (validar em discovery).
- Usuários finais usam celular durante a experiência (validar em workshop piloto).
- Existe base suficiente de Q-Graders dispostos a contribuir com conteúdo.

---

## 10. Cronograma de Alto Nível

| Fase | Duração | Entregas |
|---|---|---|
| **Discovery** | 4 semanas | Entrevistas (15 alunos + 8 cafeterias + 4 instrutores), wireframes, validação de preço |
| **Design** | 4 semanas | Design system, protótipo navegável em Figma, testes de usabilidade |
| **MVP Dev** | 12 semanas | Backend + frontend + infra; foco em trilhas, cupping, workshop e gamificação básica |
| **Piloto fechado** | 8 semanas | 3 cafeterias, 2 escolas, ~200 usuários, iteração semanal |
| **Lançamento público** | — | GA com plano Café e Escola |

**Meta:** GA em ~7 meses a partir do kickoff.

---

## 11. Critérios de Aceite do MVP

O MVP será considerado pronto quando:
- [ ] 3 cafeterias piloto rodando por 60 dias consecutivos.
- [ ] ≥ 150 usuários ativos semanais entre os pilotos.
- [ ] ≥ 500 cuppings registrados.
- [ ] ≥ 10 workshops ao vivo realizados com taxa de sucesso técnico ≥ 95%.
- [ ] NPS de gestores ≥ 40.
- [ ] Nenhum incidente crítico de segurança ou LGPD.

---

## 12. Questões em Aberto

1. Nome definitivo do produto (CoffeeQuest é placeholder).
2. Parceria institucional com SCA/BSCA é viável no curto prazo?
3. Estratégia de moderação de conteúdo gerado pelo usuário (reviews, notas públicas).
4. Quem é o curador editorial do conteúdo base — interno ou rede de Q-Graders?
5. Integração com balanças/termômetros Bluetooth no V2 — faz sentido?
6. Política de dados: os cuppings agregados podem virar produto (benchmarks para torrefadores)?

---

## 13. Apêndices

- **A.** Glossário (SCA, Q-Grader, cupping, TDS, agtron, etc.)
- **B.** Referências de concorrentes (Angel's Cup, Cropster Hub, SCA Coffee Skills Program, Duolingo, Kahoot)
- **C.** Mockups e wireframes — a produzir na fase de Design
- **D.** Matriz RACI do time do projeto — a definir
