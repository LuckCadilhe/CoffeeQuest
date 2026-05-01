# Auditoria Holística — CoffeeQuest
**Framework de 5 Dimensões: Mercado · Segurança · Operações · Financeiro · UX**

**Versão:** 1.0  
**Data:** 2026-04-27  
**Documentos base:** PRD v0.1 · SPEC v1.0 · STACK v1.0 · DESIGN GUIDELINES v1.0  
**Metodologia:** Auditoria estruturada com análise cruzada de interdependências

---

## ① Sumário Executivo

O CoffeeQuest endereça um problema real e pouco explorado — a fragmentação da educação em café especial — com uma proposta de valor B2B2C que é ao mesmo tempo diferenciada e arriscada. A documentação técnica é matura para a fase atual, mas o projeto carrega três tensões estruturais que precisam ser resolvidas antes do desenvolvimento: **o modelo de monetização ainda não foi validado com o mercado-alvo**, **a estratégia de conteúdo (quem cura o quê) permanece indefinida**, e **o escopo do MVP está superdimensionado para o cronograma proposto**. O score médio das cinco dimensões é **6,4/10** — sólido o suficiente para avançar para Discovery com clareza, mas insuficiente para iniciar desenvolvimento sem resolver os bloqueadores identificados.

---

## ② Auditoria por Dimensão

---

### DIMENSÃO 1 — MERCADO

#### Pontos Fortes

**Problema genuíno e mal atendido.** A fragmentação entre conteúdo técnico SCA e experiência casual de consumidor é real. Não existe hoje na América Latina uma plataforma que combine spaced repetition, cupping digital e gamificação num único ecossistema. O gap é legítimo.

**Timing favorável.** O mercado de café especial no Brasil cresce ~15% ao ano (ABIC). A cultura de apps de aprendizagem (Duolingo, Kahoot) está normalizada. O cruzamento desses dois vetores ainda não foi explorado por nenhum player relevante em LatAm.

**Modelo B2B2C inteligente.** Monetizar via cafeteria (B2B) enquanto serve o usuário final (B2C) resolve o problema de aquisição: a cafeteria traz os usuários, o produto retém. Isso reduz drasticamente o CAC para o lado B2C.

**Posicionamento de nicho com potencial de expansão.** Começar com café especial é um nicho defensável — os concorrentes (Angel's Cup, Cropster) não têm foco em LatAm nem em educação gamificada. O nicho pode ser escalonado para outros domínios de bebidas (vinho, cerveja artesanal) no V3.

#### Riscos

**Willingness-to-pay não validada.** O PRD assume que cafeterias pagarão R$ 299–799/mês, mas não há evidência de validação desse preço com nenhum cliente real. O risco de "problem existe, mas ninguém paga por isso" é alto em mercados de PMEs (cafeterias independentes têm margens apertadas).

**Dupla dependência de adoção.** O B2B2C só funciona se a cafeteria adota E os clientes engajam. Se uma das pontas falhar, o produto não gera valor para a outra. Isso cria um problema de ovo-e-galinha no piloto.

**Sazonalidade e alta rotatividade de baristas.** Baristas têm alta rotatividade no Brasil (~40% ao ano no setor). O Lucas (persona primária) pode sair da cafeteria antes de completar qualquer trilha, levando XP e histórico para uma conta desvinculada do tenant — gerando frustração para gestor e usuário.

**Concorrência indireta subestimada.** O PRD foca em Angel's Cup e Cropster, mas ignora concorrentes indiretos mais perigosos: YouTube (conteúdo de café gratuito, massivo), grupos de WhatsApp de baristas (comunidade gratuita), e apps genéricos de quiz (Kahoot, Quizlet) que podem ser adaptados por um instrutor sem custo.

**TAM/SAM/SOM ausentes.** O PRD não quantifica o mercado. Sem isso, é impossível avaliar se o negócio pode ser venture-backable ou se é melhor como bootstrapped.

#### Oportunidades

- **Parceria com ABRABE ou BSCA como canal de distribuição** pode validar e acelerar a entrada sem custo de vendas direto.
- **Certificação própria** (não-SCA) pode criar uma âncora de valor para baristas que não têm acesso aos cursos SCA formais (caros, em inglês).
- **Dados sensoriais agregados** como produto secundário para torrefadores e produtores (feature V3) podem representar uma receita recorrente de alto valor que transforma o unit economics do negócio.

---

### DIMENSÃO 2 — SEGURANÇA

#### Pontos Fortes

**LGPD na fundação, não no afterthought.** O PRD e a SPEC mencionam opt-in explícito, anonimização, DPA e direito de exclusão desde o início. A maioria dos projetos trata compliance como dívida técnica — aqui está no design.

**TLS 1.3 + AES-256 + 2FA para roles sensíveis.** O básico de segurança em trânsito e repouso está especificado, com 2FA obrigatório para instructors e managers.

**Segmentação de serviços.** A arquitetura com múltiplos serviços (`api-service`, `ws-service`, `worker-service`) limita o raio de explosão de um incidente — uma vulnerabilidade no WebSocket não compromete automaticamente a API REST.

#### Riscos

**Ausência de plano de resposta a incidentes.** Nenhum dos documentos menciona o que acontece quando há uma brecha. Sem runbook de incidente (quem aciona, como isolar, como comunicar usuários afetados), o tempo de resposta será caótico.

**Schema-per-tenant é isolamento lógico, não físico.** Se houver uma injeção SQL que escapa do middleware de resolução de tenant, um atacante pode acessar dados de outros tenants. O isolamento por schema é mais frágil que row-level security ou bancos separados. A SPEC não menciona testes de penetração de multi-tenancy.

**Refresh token sem rotação documentada** (achado A-03 do audit técnico). Um token roubado tem janela de 30 dias. Em produtos com dados sensoriais de usuário (historicamente privados), isso é risco relevante.

**Dependência de Auth0/Keycloak ainda indefinida.** Auth0 tem histórico de incidentes (breach em 2022). Keycloak self-hosted exige maturidade operacional que o time pode não ter no MVP. A decisão adiada aumenta o risco.

**Sem menção a SBOM (Software Bill of Materials) ou policy de dependências.** A stack usa dezenas de pacotes npm. Uma dependência comprometida (supply chain attack, como o incidente `xz utils` de 2024) pode comprometer todo o sistema sem que o time perceba.

**Dados de menores de 18 anos sem fluxo de consentimento.** O produto será usado em workshops com estudantes, potencialmente menores. A LGPD exige consentimento parental para menores de 18. Esse fluxo está ausente (achado M-06).

#### Oportunidades

- Implementar **SOC 2 Type I** desde o MVP simplifica futuras vendas enterprise e diferencia da concorrência no pitch para redes de cafeterias maiores.
- **Bug bounty program** (mesmo que informal com pesquisadores convidados) pode identificar vulnerabilidades antes do lançamento a custo baixo.

---

### DIMENSÃO 3 — OPERAÇÕES

#### Pontos Fortes

**Documentação técnica acima da média para o estágio.** Ter SPEC, STACK e DESIGN GUIDELINES produzidos antes do primeiro commit é incomum e valioso — reduz drasticamente o risco de decisões inconsistentes no calor do desenvolvimento.

**Stack moderna e com ecossistema maduro.** Next.js + NestJS + PostgreSQL + Redis é uma combinação com vasta literatura, comunidade e disponibilidade de talentos no Brasil. Não há apostas tecnológicas arriscadas.

**Monorepo com Turborepo bem planejado.** A estrutura `apps/` + `packages/shared` + `infra/` favorece consistência entre serviços e reutilização de tipos — o que reduz bugs de integração.

#### Riscos

**Time não está definido.** Os documentos não mencionam o tamanho do squad, as especialidades presentes nem o modelo de trabalho (CLT, PJ, parceiros). Para um MVP com 5 módulos complexos em 12 semanas, a composição do time é o risco operacional número 1.

**12 semanas para o MVP é insuficiente para o escopo definido.** A SPEC documenta 5 módulos funcionais completos, multi-tenancy, real-time, gamification engine e dashboard B2B. Mesmo com um time de 5-6 pessoas, 12 semanas entrega um produto funcional em 1-2 módulos — não em 5. Esse desalinhamento entre cronograma e escopo é uma bomba-relógio.

**Curadoria de conteúdo é um gargalo operacional não endereçado.** O produto depende de trilhas de aprendizagem de qualidade. Sem definir quem cura (Q-Graders contratados? equipe interna?), como são revisadas e com que cadência, o conteúdo se torna o gargalo que paralisa a retenção do usuário independentemente da qualidade técnica.

**Dependência crítica de Q-Graders para conteúdo.** Se o único Q-Grader disponível sair, o pipeline de conteúdo para. Não há menção a redundância editorial nem a processos de onboarding de novos curadores.

**Ausência de processo de vendas B2B.** O modelo depende de vender para cafeterias, mas não há menção a time de vendas, processo de onboarding de tenant, SLA de suporte ou customer success. A Júlia (persona) precisará de mão na mão para ver valor nas primeiras semanas.

**Sem plano de continuidade para o ws-service.** O workshop ao vivo é a feature de maior risco operacional (alta concorrência, tempo real, falha visível para 500 pessoas). Não há documentação de disaster recovery para esse serviço além do fallback de polling.

#### Oportunidades

- **Piloto com 3 cafeterias** é a decisão operacional mais inteligente do PRD — permite iterar sem escala antes de comprometer recursos. Preserve isso a qualquer custo.
- **Feature flags com PostHog** (já na stack) permitem entregar MVPs incrementais para cada cafeteria piloto em ritmos diferentes — reduzindo risco de lançamento.

---

### DIMENSÃO 4 — FINANCEIRO

#### Pontos Fortes

**Modelo de receita diversificado.** O PRD prevê três fontes: SaaS B2B (recorrente, previsível), freemium B2C (escala), e campanhas patrocinadas (ad revenue). A combinação reduz dependência de uma única fonte.

**Plano Café a R$ 299/mês é acessível para o mercado-alvo.** Cafeterias especiais em capitais brasileiras têm ticket médio de produto entre R$ 15-35. Um único cliente fidelizado pela plataforma compra 2-3 produtos extras por mês para amortizar o custo da assinatura — o ROI para a cafeteria é narrativamente simples.

**Piloto gratuito de 60 dias reduz barreira de entrada.** Permite validar willingness-to-pay sem exigir compromisso financeiro inicial da cafeteria.

#### Riscos

**Ausência total de projeções financeiras.** O PRD não tem: custo estimado de desenvolvimento, custo de infraestrutura por tenant, CAC, LTV, churn esperado, breakeven ou runway. Sem esses números, é impossível saber se o projeto precisa de investimento, bootstrapping ou um co-fundador técnico.

**Unit economics provavelmente negativos no MVP.** O custo de infraestrutura de uma sessão de workshop com 500 usuários (ws-service + Redis + PostgreSQL + worker) pode facilmente superar R$ 50-100 por sessão em escala pequena. Com plano Café a R$ 299/mês e 2-3 workshops/mês, a margem bruta por tenant pode ser negativa no início.

**R$ 400/mês de MRR por cafeteria no ano 1 é uma meta, não um dado.** O PRD lista isso como North Star financeiro mas não há cálculo de como chegar lá nem qual percentual de upsell do Plano Café para Escola é necessário.

**Campanhas patrocinadas requerem escala de audiência que o MVP não tem.** Para torrefadores e fazendas pagarem por campanhas, precisam de audiência relevante. Com 200 usuários no piloto, esse modelo não gera receita — mas cria custo de desenvolvimento (infraestrutura de campanhas).

**Custo de Q-Graders como curadores não estimado.** Q-Graders certificados no Brasil cobram R$ 200-500/hora por consultoria. Sem essa estimativa, o custo de conteúdo pode ser o item mais caro do P&L após infraestrutura.

**Risco de churn alto no B2B.** Cafeterias têm ciclos de vida curtos no Brasil (~30% fecham nos primeiros 2 anos). Churn estrutural do cliente B2B pode ser muito mais alto do que o esperado — e o PRD não modela esse cenário.

#### Oportunidades

- **Dados sensoriais agregados como produto** (questão aberta 6 do PRD) podem gerar uma linha de receita B2B de alto valor para torrefadores e exportadores — possivelmente mais rentável que o SaaS de cafeteria no longo prazo.
- **Modelo de revenue sharing com cafeterias** (ex.: CoffeeQuest recebe % do upsell gerado por campanha) alinha incentivos e pode substituir o modelo de assinatura pura em clientes mais resistentes.
- **Grant de inovação** (FINEP, EMBRAPII, Sebrae Inovação) para plataformas de educação tech são acessíveis e podem financiar parte do desenvolvimento sem diluição.

---

### DIMENSÃO 5 — EXPERIÊNCIA DO USUÁRIO (UX)

#### Pontos Fortes

**Onboarding via QR code em ≤ 60 segundos é uma decisão excepcional.** Elimina a maior barreira de adoção em contexto de workshop (ninguém quer criar conta antes de entrar na sala). É uma vantagem competitiva real que precisa ser protegida a qualquer custo de complexidade.

**Gamificação bem estruturada e contextual.** XP, streaks, leaderboard e badges não são decoração — estão conectados a ações de aprendizagem real (quiz, cupping, workshop). O risco de gamificação vazia (pontos sem significado) está mitigado pelo design.

**Design system com identidade forte.** A paleta terrosa, a tipografia Fraunces + DM Sans e os tokens semânticos (Espresso, Roast, Bean) criam uma identidade coerente com o domínio. Evita o "SaaS genérico azul-e-branco" que destrói credibilidade em nichos especializados.

**Princípios de design claros e acionáveis.** Os 4 princípios ("Orgânico, não clínico"; "Progresso sempre visível"; "Feedback imediato"; "Hierarquia clara") são suficientemente específicos para tomar decisões de design sem ambiguidade.

**Fluxos principais detalhados e testáveis.** Os 4 fluxos na SPEC (primeiro acesso, cupping sincronizado, missão diária, campanha do gestor) são suficientemente granulares para escrever testes E2E sem interpretação adicional.

#### Riscos

**Complexidade do formulário SCA pode intimidar o usuário casual (Marina).** O formulário de cupping com 10 atributos, sliders de precisão 0.25 e roda de sabores de dois níveis é adequado para o Lucas (barista). Para a Marina (cliente entusiasta), é esmagador. O produto ainda não tem uma versão simplificada do cupping para usuários casuais — o que contradiz a persona secundária.

**Ausência de estado de erro de rede no fluxo de workshop ao vivo.** O fluxo de fallback para polling está especificado tecnicamente, mas a experiência do usuário durante a degradação (o que o aluno vê quando perde conexão) não está documentada no Design Guidelines nem na SPEC. Em redes 4G instáveis de cafeterias, isso vai acontecer com frequência.

**Leaderboard pode criar experiência negativa para a Marina.** Uma cliente casual que entra pelo QR code da mesa e aparece em último lugar no ranking pode sentir vergonha e abandonar o produto. O PRD não diferencia a experiência de leaderboard por contexto (workshop vs. uso casual).

**Microcopy em PT-BR não cobre estados de erro técnico.** O Design Guidelines define microcopy para estados de negócio (streak quebrado, resposta errada), mas não cobre erros técnicos: timeout de API, erro de validação de formulário, falha no upload de avatar. Esses estados são frequentemente deixados como mensagens técnicas que quebram o tom de voz.

**Ausência de empty states para o gestor de cafeteria.** O Dashboard B2B é uma tela crítica de retenção do cliente pagante. Se uma cafeteria nova abre o dashboard e vê tabelas vazias sem orientação, a percepção de valor desaparece. O Design Guidelines documenta empty states para o app, mas não para o painel B2B.

**PWA em iOS Safari tem limitações não documentadas.** Push notifications via PWA são extremamente limitadas no iOS 16 e anteriores (exige adicionar à tela inicial, opt-in trabalhoso). O produto depende de push para reengajamento (streaks, missões), mas a experiência no iPhone — o dispositivo dominante no público de cafeterias especiais brasileiras — pode ser significativamente pior do que no Android.

#### Oportunidades

- **Versão "lite" do cupping** para usuários casuais (3-5 atributos + descritores simples) pode aumentar o engajamento da Marina sem comprometer a experiência do Lucas.
- **Onboarding adaptativo por persona** (barista vs. cliente casual) na avaliação inicial (RF-04) pode personalizar a experiência desde o primeiro acesso e aumentar retenção.
- **Tutorial contextual** (coach marks) no primeiro cupping e no primeiro quiz pode reduzir abandono sem aumentar fricção de onboarding.

---

## ③ Matriz de Interdependências

| Dimensão A | Dimensão B | Interdependência | Severidade |
|-----------|-----------|-----------------|-----------|
| **Mercado** | **Financeiro** | Willingness-to-pay não validada + unit economics possivelmente negativos no MVP = risco de pivotar o modelo antes do break-even | 🔴 Crítico |
| **Operações** | **Mercado** | Time indefinido + cronograma de 12 semanas subdimensionado = produto incompleto no piloto = cafeterias piloto não veem valor = validação de mercado comprometida | 🔴 Crítico |
| **Segurança** | **UX** | Ausência de fluxo de consentimento para menores + dados sensoriais potencialmente sensíveis = incidente de LGPD em workshop escolar pode destruir a confiança do produto antes do PMF | 🔴 Crítico |
| **Operações** | **Mercado** | Curadoria de conteúdo indefinida = trilhas de baixa qualidade = retenção D30 abaixo da meta (40%) = North Star de cuppings/usuário não atingido = produto não escala | 🟠 Alto |
| **Financeiro** | **Operações** | Custo de Q-Graders não estimado + custo de infra de workshop não modelado = burn rate real pode ser 2-3x o estimado = runway insuficiente para o piloto | 🟠 Alto |
| **Mercado** | **UX** | Alta rotatividade de baristas + XP vinculado ao tenant = usuário perde histórico ao trocar de cafeteria = proposta de valor do "Passaporte do Café" (público, portável) entra em conflito com o modelo multi-tenant | 🟠 Alto |
| **Segurança** | **Financeiro** | Schema-per-tenant como isolamento lógico = potencial vazamento de dados entre tenants = um incidente de segurança pode gerar litígio com múltiplos clientes B2B simultaneamente | 🟠 Alto |
| **UX** | **Mercado** | Formulário SCA complexo para usuários casuais = Marina abandona = cafeteria não vê engajamento de clientes = churn do plano B2B | 🟡 Médio |
| **Operações** | **UX** | Ausência de processo de customer success = gestor Júlia não aprende a usar o dashboard = não cria campanhas = não vê ROI = churn | 🟡 Médio |
| **Financeiro** | **Mercado** | Campanhas patrocinadas como receita prevista no MVP = distrai o roadmap com feature de monetização que só funciona com escala | 🟡 Médio |
| **Segurança** | **Operações** | Ausência de plano de resposta a incidentes = primeiro incidente de segurança paralisa operações por dias = SLA com tenants piloto quebrado | 🟡 Médio |
| **UX** | **Operações** | PWA com limitações em iOS = experiência degradada no dispositivo dominante = suporte aumentado = custo operacional não planejado | 🟡 Médio |

---

## ④ Scorecard por Dimensão

| Dimensão | Score | Justificativa |
|----------|-------|---------------|
| **Mercado** | 7/10 | Problema real, timing favorável, posicionamento inteligente. Perde pontos por willingness-to-pay não validada, TAM/SAM/SOM ausentes e concorrência indireta subestimada. |
| **Segurança** | 5/10 | Base sólida (LGPD na fundação, TLS, 2FA), mas sem plano de resposta a incidentes, isolamento de tenant frágil, decisão de auth adiada e ausência de SBOM. Muitos itens "planejados" sem implementação especificada. |
| **Operações** | 4/10 | Documentação acima da média, stack sólida. Perde pontos críticos por time indefinido, cronograma irreal para o escopo, curadoria de conteúdo sem dono e ausência de processo de vendas/CS. |
| **Financeiro** | 3/10 | Modelo de receita diversificado e interessante, mas sem projeções, sem unit economics, sem estimativa de custos de infra ou conteúdo. É impossível avaliar viabilidade financeira com o que está documentado. |
| **UX** | 8/10 | Onboarding via QR, gamificação contextual, design system forte e fluxos detalhados. Perde pontos por complexidade do cupping para usuários casuais, limitações de PWA no iOS não planejadas e ausência de empty states no B2B. |

**Score médio: 5,4/10**

> O score financeiro e operacional puxam a média para baixo de forma relevante. O produto tem alma — a visão, o design e a arquitetura são de qualidade. O risco real é de execução e de viabilidade econômica, não de produto.

---

## ⑤ Top 5 Recomendações Priorizadas

### #1 — Valide o preço antes de escrever código
**Impacto:** Crítico | **Urgência:** Imediata

Antes de qualquer sprint, execute 8-10 entrevistas com donos de cafeteria usando um protótipo em Figma (não um produto funcional). Teste o preço de R$ 299/mês com a pergunta direta: "Se isso resolvesse [problema X], você pagaria R$ 299/mês? E R$ 199? E R$ 99?". O modelo inteiro — stack, equipe, cronograma — depende de confirmar que existe willingness-to-pay real. Descobrir que o preço precisa ser R$ 99 depois de 3 meses de desenvolvimento redefine o negócio inteiro.

**Como fazer:** Use o framework de Sean Ellis ("produto must-have") + teste de preço por ancoragem. 2 semanas de Discovery com 10 entrevistas é suficiente para uma primeira validação.

---

### #2 — Reduza o escopo do MVP pela metade
**Impacto:** Crítico | **Urgência:** Antes do kickoff

O MVP atual tem 5 módulos complexos. Um MVP real deveria ter 2. Recomendação concreta de corte:

**MVP-1 (manter):** Autenticação via QR + Trilhas (spaced repetition) + Cupping SCA básico + XP/Streak/Badge simples.

**Mover para MVP-2:** Workshop ao vivo (alto custo técnico, alto risco operacional), Dashboard B2B completo (complexidade de analytics), Campanhas e recompensas físicas, Leaderboard global.

O risco de entregar tudo pela metade é maior do que o risco de entregar metade com excelência. O piloto com 3 cafeterias precisa de um produto que funciona, não de um produto que tem tudo.

---

### #3 — Resolva a curadoria de conteúdo antes da stack
**Impacto:** Alto | **Urgência:** Imediata

Definir quem cura o conteúdo das trilhas é mais urgente do que decidir entre Auth0 e Keycloak. Sem conteúdo de qualidade, nenhuma feature técnica retém usuário. Opções concretas: (a) contratar 1 Q-Grader part-time como curador editorial, (b) fazer parceria com escola de café que cede conteúdo em troca de licença gratuita, (c) criar um board editorial de 3-5 Q-Graders voluntários com incentivo de visibilidade na plataforma. Defina qual caminho, estime o custo e coloque no plano financeiro.

---

### #4 — Monte um modelo financeiro básico antes do Sprint 1
**Impacto:** Alto | **Urgência:** Antes do kickoff

Uma planilha com 3 cenários (pessimista, base, otimista) cobrindo: custo de desenvolvimento (horas × taxa), custo de infra por tenant, CAC estimado, LTV assumido (duração média do contrato × MRR), churn mensal esperado e runway. Sem isso, o projeto pode chegar ao piloto sem dinheiro para o lançamento público — ou pode descobrir que o modelo só funciona com 50+ tenants, o que muda o go-to-market completamente.

**Insumos necessários:** tamanho do time, regime de contratação, custo de cloud estimado (AWS Calculator para o workload do MVP), e preço de referência de Q-Graders.

---

### #5 — Crie uma versão "lite" do cupping para usuários casuais
**Impacto:** Médio | **Urgência:** Sprint 1

O formulário SCA completo (10 atributos + roda de sabores) é perfeito para o Lucas (barista). Para a Marina (cliente entusiasta), é uma barreira de abandono. Uma versão com 3-4 atributos principais (aroma, sabor, acidez, impressão geral) + 5-8 descritores pré-selecionados resolve a experiência casual sem comprometer a profundidade para baristas. Isso também aumenta o número de cuppings registrados (North Star), pois usuários casuais representam a maioria dos usuários de uma cafeteria.

---

## ⑥ Próximos Passos Sugeridos

| Semana | Ação | Responsável | Output Esperado |
|--------|------|-------------|----------------|
| 0–1 | 10 entrevistas com donos de cafeteria (validação de preço e problema) | Ricardo | Decisão: continuar com preço atual ou pivotar modelo |
| 0–1 | Definir composição do squad (quantas pessoas, quais especialidades) | Ricardo | Headcount plan + estimativa de custo de desenvolvimento |
| 0–1 | Montar modelo financeiro básico (3 cenários) | Ricardo + Financeiro | Planilha com runway, CAC, LTV, breakeven |
| 1–2 | Definir estratégia de curadoria de conteúdo e assinar primeiro curador | Ricardo | Contrato ou acordo com Q-Grader ou escola parceira |
| 1–2 | Revisar escopo do MVP com base nas entrevistas e no modelo financeiro | Ricardo + Tech Lead | MVP-1 redefinido com escopo viável para 12 semanas |
| 2–3 | Resolver D-02 (Auth) e D-03 (ws-service) via spike técnico | Tech Lead | ADR documentado para cada decisão |
| 2–3 | Criar protótipo navegável no Figma dos 3 fluxos principais | Design | Protótipo para teste de usabilidade com 5 baristas |
| 3–4 | Teste de usabilidade do protótipo com Lucas e Marina reais | Design + Ricardo | Lista de ajustes de UX antes do desenvolvimento |
| 4 | Kickoff do Sprint 1 com escopo validado, time definido e decisões técnicas tomadas | Todo o time | Início do desenvolvimento com confiança |

---

*Este audit deve ser revisado após a rodada de entrevistas de Discovery (semanas 0–1). As dimensões de Mercado e Financeiro terão scores revisados com base nos dados coletados.*
