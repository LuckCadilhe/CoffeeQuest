# Design Guidelines — CoffeeQuest
**Sistema de Design e Identidade Visual**

**Versão:** 1.0  
**Data:** 2026-04-27  
**Status:** Base para implementação no Figma

---

## Índice

1. [Princípios de Design](#1-princípios-de-design)
2. [Identidade Visual](#2-identidade-visual)
3. [Paleta de Cores](#3-paleta-de-cores)
4. [Tipografia](#4-tipografia)
5. [Espaçamento e Grid](#5-espaçamento-e-grid)
6. [Iconografia](#6-iconografia)
7. [Componentes](#7-componentes)
8. [Motion e Animações](#8-motion-e-animações)
9. [Tom de Voz e Microcopy](#9-tom-de-voz-e-microcopy)
10. [Acessibilidade](#10-acessibilidade)
11. [Design por Plataforma](#11-design-por-plataforma)
12. [Exemplos de Uso — Do's e Don'ts](#12-exemplos-de-uso--dos-e-donts)

---

## 1. Princípios de Design

O CoffeeQuest vive na interseção entre **aprendizagem séria** e **diversão genuína**. O design deve refletir isso: não é um app corporativo frio nem um jogo infantil — é uma ferramenta que respeita o ofício do café e torna o aprendizado irresistível.

### Os quatro princípios

**1. Orgânico, não clínico**
O café é sensorial, artesanal, humano. Evitamos cantos agressivos, branco esterilizado e grids militares. Preferimos texturas sutis, espaços generosos e formas que lembram a natureza do grão.

**2. Progresso sempre visível**
O usuário deve sentir que está avançando a cada interação. XP, streaks, barras de progresso e badges não são decoração — são o produto principal. Nunca esconda conquistas.

**3. Feedback imediato**
Toda ação tem resposta visual em < 200ms. Acerto, erro, XP ganho, cupping salvo — o app confirma na hora. Silêncio gera ansiedade; feedback gera confiança.

**4. Hierarquia clara, sem poluição**
Baristas usam o app durante o trabalho, entre pedidos. O design precisa ser legível em 3 segundos, numa cozinha com má iluminação, com as mãos ocupadas. Uma ação principal por tela. Sempre.

---

## 2. Identidade Visual

### Conceito
A identidade do CoffeeQuest parte do **grão de café**: orgânico, poroso, cheio de camadas e com uma energia latente que só se revela no processo. O visual traduz isso com texturas sutis de grain, paleta quente terrosa e tipografia que mistura o caráter editorial de uma revista especializada com a leveza de um app moderno.

### Logo
- O logo combina um ícone (grão estilizado ou xícara com seta de "quest") com o wordmark "CoffeeQuest".
- **Versão primária:** wordmark + ícone, horizontal, fundo escuro.
- **Versão secundária:** só ícone (avatar, favicon, app icon).
- **Área de proteção:** mínimo de 16px de espaço livre ao redor do logo em qualquer direção.
- **Tamanho mínimo:** 80px de largura para a versão completa; 24px para o ícone solo.

### Personalidade da marca
| Atributo | É | Não é |
|----------|---|-------|
| Tom | Especializado mas acessível | Elitista ou técnico demais |
| Visual | Quente, orgânico, terroso | Frio, tech, minimalista-branco |
| Energia | Motivador, dinâmico | Agressivo ou ansioso |
| Confiança | Profissional, preciso | Corporativo, burocrático |

---

## 3. Paleta de Cores

### Cores Primárias

| Token | Hex | Uso |
|-------|-----|-----|
| `--color-espresso` | `#2C1A0E` | Texto primário, headings, elementos de peso |
| `--color-roast` | `#6B3F1A` | Cor primária da marca — CTAs, botões, links ativos |
| `--color-roast-hover` | `#8A5230` | Estado hover dos elementos primários |
| `--color-roast-light` | `#C4834A` | Destaques, ícones ativos, selected states |

### Cores de Acento

| Token | Hex | Uso |
|-------|-----|-----|
| `--color-bean` | `#E8A836` | XP, moeda Beans, badges premium, streaks |
| `--color-bean-light` | `#F5CC7A` | Fundo de badges de XP, chips de missão |
| `--color-matcha` | `#3D7A5C` | Sucesso, acerto em quiz, streak ativo, nível up |
| `--color-matcha-light` | `#D4EDE3` | Fundo de estados de sucesso |

### Cores Neutras

| Token | Hex | Uso |
|-------|-----|-----|
| `--color-cream` | `#FAF7F2` | Background principal (app) |
| `--color-parchment` | `#F0EBE1` | Background de cards, superfícies elevadas |
| `--color-latte` | `#E2D9CC` | Bordas, dividers, separadores |
| `--color-stone` | `#A09180` | Texto secundário, labels, placeholders |
| `--color-ash` | `#6B6059` | Texto terciário, hints, metadata |

### Cores Semânticas

| Token | Hex | Uso |
|-------|-----|-----|
| `--color-error` | `#C0392B` | Erro, streak quebrado, resposta incorreta |
| `--color-error-bg` | `#FDECEA` | Fundo de estados de erro |
| `--color-warning` | `#D68910` | Atenção, revisão pendente (SM-2) |
| `--color-warning-bg` | `#FEF9E7` | Fundo de avisos |
| `--color-info` | `#2471A3` | Informação, dicas, tooltips |
| `--color-info-bg` | `#EBF5FB` | Fundo de informações |

### Modo Escuro

| Token Light | Token Dark | Estratégia |
|-------------|------------|------------|
| `--color-cream` (#FAF7F2) | `#1A1208` | Espresso invertido, não preto puro |
| `--color-parchment` (#F0EBE1) | `#241810` | Superfície levemente aquecida |
| `--color-latte` (#E2D9CC) | `#3D2E20` | Bordas sutis no escuro |
| `--color-espresso` (#2C1A0E) | `#F5EEE6` | Texto claro sobre fundo escuro |

> **Regra de ouro:** nunca use preto puro (`#000`) nem branco puro (`#FFF`). O universo do café é quente — até o escuro tem calor.

---

## 4. Tipografia

### Famílias

| Família | Uso | Google Fonts / CDN |
|---------|-----|--------------------|
| **Fraunces** | Headings, títulos de trilha, nomes de badge | `Fraunces` — serif óptico com personalidade |
| **DM Sans** | Body, labels, UI, formulários | `DM_Sans` — geométrico humanista, leitura fácil em telas pequenas |
| **DM Mono** | Scores de cupping, códigos, dados numéricos | `DM_Mono` — alinhamento de colunas numéricas |

```css
/* Importação */
@import url('https://fonts.googleapis.com/css2?family=Fraunces:wght@400;600;700&family=DM+Sans:wght@400;500;600&family=DM+Mono:wght@400;500&display=swap');

--font-display: 'Fraunces', Georgia, serif;
--font-body:    'DM Sans', system-ui, sans-serif;
--font-mono:    'DM Mono', 'Courier New', monospace;
```

### Escala Tipográfica

| Nome | Tamanho | Peso | Font | Line-height | Uso |
|------|---------|------|------|-------------|-----|
| `display-xl` | 48px | 700 | Fraunces | 1.1 | Título de onboarding, screens hero |
| `display-lg` | 36px | 600 | Fraunces | 1.2 | Título de trilha, nome de badge |
| `display-md` | 28px | 600 | Fraunces | 1.25 | Título de módulo, heading de dashboard |
| `heading-lg` | 22px | 600 | DM Sans | 1.3 | Título de card, nome de lição |
| `heading-md` | 18px | 500 | DM Sans | 1.4 | Subtítulo, label de seção |
| `body-lg` | 16px | 400 | DM Sans | 1.6 | Corpo de texto principal |
| `body-md` | 14px | 400 | DM Sans | 1.5 | Labels, descrições, metadata |
| `body-sm` | 12px | 400 | DM Sans | 1.4 | Hints, legenda, timestamps |
| `mono-lg` | 18px | 500 | DM Mono | 1.3 | Score total de cupping |
| `mono-md` | 14px | 400 | DM Mono | 1.4 | Atributos SCA, dados numéricos |

### Regras de Uso

- **Fraunces** é reservada para momentos de destaque. Nunca use em body text longo.
- Peso máximo para **DM Sans** em UI é 600 (semibold). Evite 700+ em corpo de texto.
- Nunca mescle duas famílias serif na mesma tela.
- Tamanho mínimo de texto: **12px** (com contraste adequado para WCAG).
- Comprimento máximo de linha de leitura: **70 caracteres** (~680px em body-lg).

---

## 5. Espaçamento e Grid

### Escala de Espaçamento (base 4px)

| Token | Valor | Uso típico |
|-------|-------|-----------|
| `--space-1` | 4px | Gaps internos mínimos (ícone + label) |
| `--space-2` | 8px | Padding interno de badges, chips |
| `--space-3` | 12px | Gap entre elementos relacionados |
| `--space-4` | 16px | Padding padrão de cards, inputs |
| `--space-5` | 20px | Espaço entre grupos de elementos |
| `--space-6` | 24px | Margin entre seções dentro de uma tela |
| `--space-8` | 32px | Separação entre seções distintas |
| `--space-10` | 40px | Padding vertical de telas |
| `--space-12` | 48px | Espaço generoso entre blocos |
| `--space-16` | 64px | Espaço hero, separação de páginas |

### Grid — Mobile (PWA)

- **Colunas:** 4 colunas
- **Gutter:** 16px
- **Margin lateral:** 16px
- **Max-width do conteúdo:** 428px (iPhone Pro Max)

### Grid — Tablet

- **Colunas:** 8 colunas
- **Gutter:** 20px
- **Margin lateral:** 24px

### Grid — Desktop (Painel B2B)

- **Colunas:** 12 colunas
- **Gutter:** 24px
- **Margin lateral:** 32px
- **Max-width do container:** 1280px
- **Sidebar:** 240px fixa

### Border Radius

| Token | Valor | Uso |
|-------|-------|-----|
| `--radius-xs` | 4px | Tags, chips pequenos |
| `--radius-sm` | 8px | Inputs, botões pequenos |
| `--radius-md` | 12px | Cards, botões padrão, modais |
| `--radius-lg` | 16px | Sheets, painéis, avatares grandes |
| `--radius-xl` | 24px | Bottom sheets, drawers |
| `--radius-full` | 9999px | Avatares circulares, pills |

---

## 6. Iconografia

### Sistema

- **Biblioteca base:** Lucide Icons (open source, MIT, usado no shadcn/ui)
- **Tamanho padrão:** 20px (ações), 16px (inline em texto), 24px (navegação principal)
- **Peso (stroke):** 1.5px — nunca 1px (fraco) nem 2px+ (pesado demais)
- **Cor:** sempre herda do contexto — nunca defina cor de ícone hardcoded

### Ícones de Domínio (customizados)

Alguns elementos do domínio do café precisam de ícones customizados, pois não existem em bibliotecas genéricas:

| Elemento | Ícone customizado necessário |
|----------|------------------------------|
| Cupping / avaliação SCA | Xícara com régua de score |
| Roda de sabores | Círculo segmentado |
| Grão de café | Forma do grão com sulco central |
| Beans (moeda virtual) | Grão estilizado com símbolo de moeda |
| Nível de barista | Estrela com sulco de grão |
| Streak | Chama + número |
| Q-Grader | Diploma/medalha com grão |

> Todos os ícones customizados devem ser entregues como SVG otimizado (SVGO) e como componente React.

### Regras de Uso

- Ícones **nunca** aparecem sem label acessível (`aria-label` ou texto visível adjacente) — exceto em contextos inequívocos (ex: ícone de fechar em modal).
- Em botões: ícone à esquerda do texto, gap de 8px.
- Em navegação: ícone acima do label, gap de 4px.
- Nunca rotacione ícones sem intenção semântica clara.

---

## 7. Componentes

### Botões

```
Hierarquia:
┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│  Primary        │   │  Secondary      │   │  Ghost          │
│  bg: roast      │   │  bg: parchment  │   │  bg: transparent│
│  text: cream    │   │  text: roast    │   │  text: roast    │
│  h: 48px        │   │  border: roast  │   │  h: 48px        │
└─────────────────┘   └─────────────────┘   └─────────────────┘

Tamanhos: sm (36px), md (44px — padrão mobile), lg (52px)
Estados: default → hover → active → disabled → loading
```

- **Tamanho mínimo de toque:** 44×44px (WCAG 2.5.5)
- Botão primário: nunca dois na mesma tela
- Loading state: spinner à esquerda, texto muda para "Aguarde..."
- Disabled: opacity 40%, cursor not-allowed, sem hover effect

### Cards

```
┌───────────────────────────────┐
│ [eyebrow — body-sm, stone]    │
│                               │
│ Título do Card                │  ← heading-lg, espresso
│ Descrição curta do conteúdo   │  ← body-md, ash
│                               │
│ ────────────────────────────  │  ← divider: latte, 1px
│                               │
│ [meta left]        [ação →]   │  ← body-sm / botão ghost
└───────────────────────────────┘

bg: parchment | border: 1px latte | radius: md | padding: 16px
```

- Cards clicáveis têm `cursor: pointer` e scale(0.99) no active
- Cards com destaque: borda left 3px na cor `roast`
- Cards de conquista (badge): fundo gradiente sutil `bean-light → parchment`

### Inputs e Formulários

```
Label (body-md, espresso, weight 500)
┌─────────────────────────────────────┐
│  Placeholder (stone)                │
└─────────────────────────────────────┘
  Hint text (body-sm, ash)

Estados:
- Default:  border latte, bg cream
- Focus:    border roast 2px, bg white, shadow 0 0 0 3px roast/15%
- Error:    border error 2px, bg error-bg
- Success:  border matcha 2px, ícone check à direita
- Disabled: bg parchment, text ash, cursor not-allowed
```

- Altura mínima de input: 48px (mobile)
- Nunca use placeholder como substituto de label
- Mensagem de erro: inline abaixo do campo, cor `error`, ícone de alerta

### CuppingSlider (componente customizado)

```
Aroma
━━━━━━━━━━━●━━━━━━━━━━━━━━  7.5
           ↑
    [Toque e arraste]

Track: 4px, cor latte
Thumb: 24px, cor roast, shadow médio
Fill (esquerda): cor roast
Valor: DM Mono, 18px, à direita
Label: DM Sans 14px, stone, à esquerda
```

- Step: 0.25
- Range: 0.0 – 10.0
- Toque mínimo no thumb: 44px (padding virtual)
- Acessível via teclado: setas ±0.25, Page Up/Down ±1.0

### FlavorWheel (componente customizado)

- SVG interativo, dois anéis concêntricos
- Anel externo: categorias (Frutado, Floral, Doce, Nozes/Cacau, Especiarias, Torrado, Vegetal, Outros, Ácido/Fermentado, Verde/Vegetal)
- Anel interno: subcategorias por categoria selecionada
- Tap: seleciona segmento (destaque cor roast, outros escurecem)
- Selecionados: chips abaixo da roda com "×" para remover
- Máximo de seleções: 10 descritores
- Alternativa acessível: lista de checkboxes em accordion (screen readers)

### XP Toast

```
         ┌──────────────────────┐
         │  +20 XP              │
         │  Cupping registrado  │
         └──────────────────────┘

Posição: bottom-center, 24px acima da navbar
Animação: slide-up 200ms ease-out → hold 2.5s → fade-out 300ms
bg: espresso | text: cream | radius: md | padding: 12px 20px
ícone: grão customizado, cor bean, 16px
```

- Máximo de 1 toast visível por vez (fila FIFO)
- Nunca bloqueia interação (`pointer-events: none`)
- Para conquistas (badge / level-up): modal fullscreen em vez de toast

### BottomSheet

- Puxada de baixo, border-radius top 24px
- Handle visual: 4×32px, cor latte, centrado no topo
- Backdrop: `espresso` a 50% de opacidade
- Animação: spring (stiffness 400, damping 30)
- Fechamento: swipe down, tap no backdrop ou botão X

### Badge

```
┌──────────────────────────────────────┐
│  [ícone 32px]   Nome do Badge        │
│                 Descrição curta      │
│                 Data conquistada     │
└──────────────────────────────────────┘

Conquistado: ícone colorido, texto espresso
Bloqueado: ícone grayscale + blur leve, texto ash, "?" no lugar do ícone
```

### Leaderboard

```
┌───────────────────────────────────────┐
│ #   Avatar  Nome           XP    ↕   │
├───────────────────────────────────────┤
│ 1   [  ]   Lucas Silva    4.280  ▲3  │  ← bg bean-light, texto bold
│ 2   [  ]   Ana Torres     3.910  ─   │
│ 3   [  ]   Pedro Lins     3.750  ▼1  │
│ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │
│ 47  [✓]   Você            980    ▲5  │  ← linha "você" destaca com borda roast
└───────────────────────────────────────┘
```

---

## 8. Motion e Animações

### Princípios

- **Propósito antes de estética:** cada animação comunica algo (progresso, feedback, transição de estado). Animações decorativas são exceção, não regra.
- **Rápido na entrada, suave na saída:** entradas em 150–200ms, saídas em 250–300ms.
- **Respeitar `prefers-reduced-motion`:** todas as animações devem ter versão sem movimento.

### Curvas de Easing

```css
--ease-spring:   cubic-bezier(0.34, 1.56, 0.64, 1);  /* elementos que saltam: badges, XP */
--ease-out:      cubic-bezier(0.0, 0.0, 0.2, 1.0);   /* entradas de conteúdo */
--ease-in-out:   cubic-bezier(0.4, 0.0, 0.2, 1.0);   /* transições de estado */
--ease-sharp:    cubic-bezier(0.4, 0.0, 0.6, 1.0);   /* dismiss, saídas rápidas */
```

### Catálogo de Animações

| Evento | Duração | Easing | Descrição |
|--------|---------|--------|-----------|
| Toque em botão | 80ms | sharp | scale(0.97) + brighten |
| Card hover | 150ms | out | translateY(-2px) + shadow |
| Toast XP | 200ms entrada / 300ms saída | spring / sharp | slide-up + fade |
| Level up | 600ms | spring | burst de partículas + scale do avatar |
| Badge desbloqueado | 800ms | spring | flip 3D da face do badge + glow |
| Streak incremento | 400ms | spring | chama cresce + número salta |
| Barra de progresso | 600ms | out | fill suave da esquerda para a direita |
| BottomSheet | 350ms | spring | slide-up com bounce leve |
| Resposta quiz (acerto) | 250ms | out | flash verde + checkmark draw |
| Resposta quiz (erro) | 300ms | in-out | shake horizontal (3 oscilações) |
| Carregamento de tela | 200ms stagger | out | cards entram em cascata (50ms delay entre cada) |

### `prefers-reduced-motion`

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## 9. Tom de Voz e Microcopy

### Personalidade na escrita

O CoffeeQuest fala como um **barista experiente que também é um bom professor**: direto, entusiasmado com o ofício, nunca condescendente. Celebra o progresso sem exagerar. Diz a verdade quando o usuário erra, mas com gentileza.

### Regras

| Faça | Evite |
|------|-------|
| Frases curtas e ativas | Linguagem passiva e burocrática |
| Vocabulário do café quando contextual | Jargão técnico SCA sem explicação |
| Celebrar conquistas com energia | "Parabéns!" repetitivo e vazio |
| Erro: orientar o próximo passo | Erro: só indicar que errou |
| "Sua próxima missão..." | "Por favor, complete a tarefa..." |
| "Você arrasou!" | "Excelente performance!" |

### Exemplos de Microcopy

| Contexto | Tom errado ❌ | Tom certo ✅ |
|---------|--------------|-------------|
| Streak quebrado | "Você perdeu seu streak." | "Streak pausado. Que tal recomeçar hoje?" |
| Resposta errada | "Resposta incorreta." | "Não dessa vez — a resposta certa é X. Você vai lembrar!" |
| Badge desbloqueado | "Badge adquirido: Primeiro Cupping" | "Primeiro cupping registrado! O paladar começa aqui." |
| Nível alcançado | "Nível 5 atingido." | "Nível 5 — já dá pra sentir a diferença no copo, né?" |
| Tela vazia (sem cuppings) | "Nenhum cupping encontrado." | "Nenhum cupping ainda. Escolhe um café e começa!" |
| Carregando | "Carregando..." | "Preparando seu café..." |
| Erro de rede | "Erro de conexão." | "Sem sinal por aqui — tentando de novo em instantes." |

### Nomenclatura do Produto

| Sempre use | Nunca use |
|-----------|-----------|
| "Beans" (moeda virtual) | "pontos", "créditos", "coins" |
| "Trilha" (track) | "curso", "módulo raiz" |
| "Cupping" | "avaliação", "degustação" (fora do contexto educativo) |
| "Badge" | "conquista", "troféu" |
| "Streak" | "sequência", "combo" |
| "Missão" | "tarefa", "atividade" |

---

## 10. Acessibilidade

### Requisito Base: WCAG 2.1 AA

| Critério | Regra |
|---------|-------|
| Contraste de texto | Mínimo 4.5:1 (texto normal), 3:1 (texto grande ≥18px ou bold ≥14px) |
| Foco visível | Outline de 3px na cor `roast`, offset de 2px — nunca remova o outline sem substituto |
| Área de toque | Mínimo 44×44px para todos os elementos interativos |
| Alternativas textuais | `alt` em todas as imagens, `aria-label` em ícones sem texto adjacente |
| Ordem de leitura | Ordem DOM = ordem visual — nunca use CSS para inverter ordem semanticamente |
| Labels de formulário | Todo input tem `<label>` associado — nunca use só placeholder |

### Componentes Críticos

**CuppingSlider:**
```html
<input
  type="range"
  aria-label="Aroma — valor atual: 7.5 de 10"
  aria-valuemin="0"
  aria-valuemax="10"
  aria-valuenow="7.5"
  role="slider"
/>
```

**FlavorWheel:**
```html
<!-- Alternativa acessível via atributo -->
<div role="group" aria-label="Roda de sabores — selecione os descritores">
  <details>
    <summary>Lista de descritores (alternativa acessível)</summary>
    <!-- checkboxes por categoria -->
  </details>
</div>
```

**Toast de XP:**
```html
<div role="status" aria-live="polite" aria-atomic="true">
  +20 XP — Cupping registrado
</div>
```

### Paleta e Contraste

| Combinação | Ratio | Status |
|------------|-------|--------|
| Espresso (#2C1A0E) sobre Cream (#FAF7F2) | 16.8:1 | ✅ AAA |
| Cream (#FAF7F2) sobre Roast (#6B3F1A) | 7.2:1 | ✅ AAA |
| Stone (#A09180) sobre Cream (#FAF7F2) | 3.8:1 | ✅ AA (texto grande) |
| Bean (#E8A836) sobre Espresso (#2C1A0E) | 8.4:1 | ✅ AAA |
| Error (#C0392B) sobre Cream (#FAF7F2) | 5.1:1 | ✅ AA |

---

## 11. Design por Plataforma

### App Mobile (PWA — Trainee / Barista)

- **Layout:** single-column, scroll vertical, bottom navigation com 5 tabs
- **Gestos:** swipe horizontal para cards de lição, swipe down para fechar sheets
- **Safe areas:** sempre respeitar `env(safe-area-inset-*)` — especialmente para iPhones com notch
- **Orientação:** portrait only no MVP (landscape bloqueado)
- **Teclado:** inputs importantes ficam sempre acima do teclado virtual (KeyboardAvoidingView / scroll-padding-bottom)

### Painel Web (Instrutor / Cafe Manager)

- **Layout:** sidebar fixa à esquerda (240px) + área de conteúdo scrollável
- **Densidade:** maior que o app — tabelas, gráficos e múltiplas métricas por tela são esperados
- **Interações:** hover states ricos, tooltips em dados, drag-and-drop para reordenar conteúdo
- **Breakpoints:**
  - `sm`: 640px — tablet vertical (sidebar colapsada)
  - `md`: 768px — tablet horizontal
  - `lg`: 1024px — desktop mínimo
  - `xl`: 1280px — desktop padrão

### Painel Workshop ao Vivo (Instrutor — Projeção)

- Tela pensada para ser projetada em TV/projetor (16:9)
- Texto mínimo: 24px — legível a 5 metros
- Alto contraste: fundo escuro (espresso), texto claro (cream)
- Remove sidebar — layout fullscreen limpo
- Animações de heatmap visualmente dramáticas (mais impacto = mais engajamento da turma)

---

## 12. Exemplos de Uso — Do's e Don'ts

### Cores

✅ **Certo:** Botão primário com fundo `roast` e texto `cream`  
❌ **Errado:** Botão primário azul ou roxo (fora da paleta de marca)

✅ **Certo:** XP badge com fundo `bean-light` e texto `espresso`  
❌ **Errado:** XP badge com fundo branco puro — sem calor, sem identidade

✅ **Certo:** Fundo do app `cream` (#FAF7F2) — quente e confortável  
❌ **Errado:** Fundo branco puro (#FFFFFF) — frio, genérico

### Tipografia

✅ **Certo:** Título de trilha em Fraunces 600, corpo em DM Sans 400  
❌ **Errado:** Todo o app em Inter ou Roboto — sem personalidade

✅ **Certo:** Score de cupping em DM Mono 500 — alinhamento perfeito de colunas  
❌ **Errado:** Score em DM Sans — números desalinhados, menos legível

### Espaçamento

✅ **Certo:** 16px de padding nos cards, 24px entre seções  
❌ **Errado:** 10px randômico — quebre a grade e quebre a consistência

### Animações

✅ **Certo:** Toast de XP aparece em 200ms (spring), some em 300ms (sharp)  
❌ **Errado:** Toast em 1 segundo — lento demais, parece travado

✅ **Certo:** Badge desbloqueado com flip 3D — momento memorável  
❌ **Errado:** Badge aparece sem animação — desperdício de um momento mágico

### Microcopy

✅ **Certo:** "Streak pausado. Que tal recomeçar hoje?"  
❌ **Errado:** "Você perdeu seu streak de 5 dias."

✅ **Certo:** "Preparando seu café..." (carregamento)  
❌ **Errado:** "Carregando dados do servidor..." — fora do universo da marca

---

## Apêndices

**A.** Tokens CSS completos — exportar do Figma como `design-tokens.json` (formato W3C DTCG)  
**B.** Storybook — todos os componentes documentados em `/packages/ui/stories`  
**C.** Checklist de acessibilidade por tela — a produzir junto ao design no Figma  
**D.** Animações de referência — arquivo Figma com prototype dos momentos chave (badge, level-up, streak)  
**E.** Roda de sabores SCA — aguardando confirmação de licença (D-04 na SPEC)
