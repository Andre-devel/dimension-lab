# 🧠 CLOUD.md — Dimension.Lab3D (Frontend)

> Documento de especificação viva do frontend. Toda decisão deve ser registrada aqui.
> Nenhuma feature pode ser criada sem teste. Nenhuma modificação sem explicação.

---

## 🎯 1. Classificação

**Build to Learning** — experimento técnico com liberdade para arriscar padrões novos, aplicar TDD e trabalhar com IA como par de programação.

---

## 📌 2. Domínio

| Campo | Descrição |
|---|---|
| **Projeto** | Dimension.Lab3D — site de orçamentos e portfólio para impressão 3D |
| **Repositório** | Separado do backend (multi-repo) |
| **Usuários** | Visitante anônimo, cliente logado (Google), admin |
| **Casos de uso críticos** | Envio de orçamento com anexos, visualização do portfólio, painel admin |

---

## 🏛 3. Arquitetura

### Stack
- **Framework:** React 18 com Vite
- **Linguagem:** TypeScript
- **Estilização:** Tailwind CSS
- **Roteamento:** React Router v6
- **Estado global:** Zustand
- **HTTP:** Axios com interceptors
- **Formulários:** React Hook Form + Zod (validação)
- **Testes:** Vitest + React Testing Library

### Estrutura de Pastas

```
frontend/
├── public/
├── src/
│   ├── assets/              # Imagens, fontes, ícones estáticos
│   ├── components/          # Componentes reutilizáveis
│   │   ├── ui/              # Primitivos: Button, Input, Badge, Card...
│   │   ├── layout/          # Navbar, Footer, Sidebar, PageWrapper
│   │   └── shared/          # PortfolioCard, QuoteCard, StatusBadge, FileUploadZone
│   ├── pages/               # Uma pasta por rota
│   │   ├── Home/
│   │   ├── Portfolio/
│   │   ├── PortfolioDetail/
│   │   ├── QuoteRequest/
│   │   ├── MyQuotes/
│   │   └── Admin/
│   │       ├── Dashboard/
│   │       └── QuoteDetail/
│   ├── services/            # Chamadas à API (axios)
│   │   ├── quoteService.ts
│   │   ├── portfolioService.ts
│   │   └── authService.ts
│   ├── hooks/               # Custom hooks
│   │   ├── useQuotes.ts
│   │   ├── usePortfolio.ts
│   │   └── useAuth.ts
│   ├── store/               # Zustand stores
│   │   ├── authStore.ts
│   │   └── quoteStore.ts
│   ├── types/               # Interfaces e tipos TypeScript
│   │   ├── quote.ts
│   │   ├── portfolio.ts
│   │   └── user.ts
│   ├── utils/               # Funções utilitárias puras
│   ├── constants/           # Enums, valores fixos (status, materiais...)
│   ├── router/              # Definição de rotas e guards
│   └── main.tsx
├── tests/                   # Testes fora do src
│   ├── unit/
│   └── integration/
├── .env.example
├── vite.config.ts
├── tailwind.config.ts
└── tsconfig.json
```

---

## 🗂 4. Páginas e Rotas

| Rota | Página | Acesso |
|---|---|---|
| `/` | Home (Landing) | Público |
| `/portfolio` | Galeria de modelos | Público |
| `/portfolio/:id` | Detalhe do modelo | Público |
| `/quote` | Formulário de orçamento | Público |
| `/my-quotes` | Histórico do cliente | Logado (cliente) |
| `/admin` | Dashboard admin | Logado (admin) |
| `/admin/quotes/:id` | Detalhe do orçamento | Logado (admin) |

---

## 🎨 5. Design System

### Cores (Tailwind custom config)

> Paleta extraída do logo Dimension.Lab3D — dark background com azul elétrico e roxo como acentos.

```js
colors: {
  background:  '#0A0A0F',  // fundo principal — preto azulado profundo
  surface:     '#12121A',  // cards, painéis
  'surface-2': '#1C1C2E',  // hover, elementos elevados
  border:      '#2A2A45',  // bordas e divisores
  'accent-blue':   '#4D9FFF',  // azul elétrico — cor primária do logo
  'accent-purple': '#8B5CF6',  // roxo — cor secundária do logo
  'accent-glow':   '#2563EB',  // azul mais profundo para gradientes e glows
  'text-primary':   '#F0F0FF',  // branco levemente azulado
  'text-secondary': '#7A7A9A',  // texto secundário
}
```

### Gradientes
- **Accent gradient:** `from-[#4D9FFF] to-[#8B5CF6]` — usado em CTAs, badges de destaque, bordas ativas
- **Glow effect:** `box-shadow: 0 0 20px rgba(77, 159, 255, 0.3)` — cards em hover, botão primário
- **Surface gradient:** `from-[#12121A] to-[#1C1C2E]` — cards do portfólio

### Tipografia
- **Headings:** Orbitron Bold ou Space Grotesk Bold — geométrico, tech, condizente com o logo
- **Body:** Inter Regular
- **Monospace (status, specs):** JetBrains Mono
- **Nome da marca no navbar:** sempre em duas linhas — `DIMENSION` bold + `.LAB3D` em accent-blue

### Responsividade — Mobile First (OBRIGATÓRIO)
- **Toda tela deve ter versão mobile** — o desenvolvimento começa pelo mobile e expande para desktop
- Breakpoints Tailwind utilizados:
    - `default` — mobile (< 640px)
    - `sm:` — 640px+ (tablet pequeno)
    - `md:` — 768px+ (tablet)
    - `lg:` — 1024px+ (desktop)
- Regras:
    - Nunca usar larguras fixas em px — usar `w-full`, `max-w-*`, `%`
    - Grids sempre começam em 1 coluna: `grid-cols-1 md:grid-cols-2 lg:grid-cols-3`
    - Navbar vira menu hamburguer no mobile
    - Sidebar do admin vira drawer deslizante no mobile
    - Formulário de orçamento: coluna única no mobile, duas colunas no desktop
    - Tabela do admin: vira lista de cards no mobile
- **Testes de responsividade:** cada componente deve ser testado nas resoluções 375px (mobile), 768px (tablet) e 1280px (desktop)

### Tokens de Espaçamento
- Grid base: **8px**
- Border radius: cards `8px`, botões `4px`, badges `999px`

### Componentes UI base (src/components/ui/)
| Componente | Variantes |
|---|---|
| `Button` | primary, secondary, ghost, danger |
| `Input` | default, error, disabled |
| `Badge` / `StatusBadge` | por status (cor dinâmica) |
| `Card` | default, hoverable |
| `FileUploadZone` | idle, dragging, uploaded, error |
| `Modal` | — |
| `Spinner` | — |

### Cores dos Status
| Status | Cor | Hex |
|---|---|---|
| RECEIVED | azul elétrico | `#4D9FFF` |
| UNDER_REVIEW | âmbar | `#F59E0B` |
| APPROVED | roxo | `#8B5CF6` |
| PRINTING | azul brilhante | `#2563EB` |
| READY | verde | `#10B981` |
| DELIVERED | cinza | `#4A4A6A` |

---

## 🔐 6. Autenticação

- Login social com Google via redirect para o backend (`/oauth2/authorization/google`)
- Token JWT armazenado em **httpOnly cookie** (gerenciado pelo backend)
- `authStore` guarda: `user`, `role`, `isAuthenticated`
- Route guards:
    - `<PrivateRoute role="CLIENT">` — redireciona para `/` se não logado
    - `<PrivateRoute role="ADMIN">` — redireciona para `/` se não for admin

---

## 🌐 7. Comunicação com o Backend

- Base URL via variável de ambiente: `VITE_API_BASE_URL`
- Axios instance centralizada em `src/services/api.ts`
- Interceptor de request: injeta credenciais (`withCredentials: true`)
- Interceptor de response: trata `401` redirecionando para login
- Padrão de resposta de erro esperado do backend:
```ts
interface ApiError {
  status: number;
  message: string;
  timestamp: string;
  details: string[];
}
```

---

## 🧪 8. TDD — Regras

### Antes de qualquer implementação:
- [ ] Teste escrito e falhando
- [ ] Mocks da API configurados (MSW — Mock Service Worker)
- [ ] Cobertura mínima: **80%** em `components/ui/` e `services/`

### Após implementação:
- [ ] Todos os testes passam
- [ ] Nenhum teste removido
- [ ] Refatoração realizada se necessário

### Ferramentas
- `Vitest` — test runner
- `React Testing Library` — testes de componente
- `MSW` — mock de API nos testes
- `@testing-library/user-event` — simulação de interações do usuário

### O que testar obrigatoriamente
- Todos os componentes `ui/`
- Lógica de validação dos formulários (Zod schemas)
- Custom hooks (`useQuotes`, `usePortfolio`, `useAuth`)
- Services (com MSW mockando o backend)
- Route guards (acesso negado / permitido)

---

## 📘 9. Convenções

| Item | Padrão |
|---|---|
| **Idioma do código** | English (variáveis, funções, tipos, commits) |
| **Branches** | `main`, `develop`, `feature/nome-da-feature` |
| **Commits** | Conventional Commits: `feat:`, `fix:`, `test:`, `refactor:`, `style:` |
| **Nomeação de componentes** | PascalCase (`QuoteCard.tsx`) |
| **Nomeação de hooks** | camelCase com prefixo `use` (`useQuotes.ts`) |
| **Nomeação de stores** | camelCase com sufixo `Store` (`authStore.ts`) |
| **Nomeação de services** | camelCase com sufixo `Service` (`quoteService.ts`) |
| **Nomeação de tipos** | PascalCase em `types/` (`Quote`, `PortfolioItem`) |
| **CSS** | Apenas Tailwind — proibido CSS inline e arquivos `.css` avulsos |
| **Imports** | Absolutos via alias `@/` (ex: `@/components/ui/Button`) |

---

## 🔐 10. AI Jail (Regras para IA)

- **Nunca criar componente sem teste**
- **Nunca modificar estrutura sem explicar a razão**
- **Sempre justificar decisões de arquitetura de componentes**
- **Nunca instalar biblioteca sem registrar aqui o motivo**
- **Sempre rodar testes após mudanças**
- **Se errar: não corrigir manualmente — explicar o erro, atualizar este Cloud.md e pedir reexecução**

---

## 📦 11. Bibliotecas Aprovadas

| Biblioteca | Versão | Uso |
|---|---|---|
| react | 18 | Core |
| vite | 5 | Build tool |
| typescript | 5 | Tipagem |
| tailwindcss | 3 | Estilização |
| react-router-dom | 6 | Roteamento |
| zustand | 4 | Estado global |
| axios | 1 | HTTP |
| react-hook-form | 7 | Formulários |
| zod | 3 | Validação |
| vitest | 1 | Testes |
| @testing-library/react | 14 | Testes de componente |
| msw | 2 | Mock de API |
| @google/model-viewer | latest | Viewer 3D (STL/GLB) |
| lucide-react | latest | Ícones |

> Qualquer nova biblioteca deve ser adicionada aqui com justificativa antes de instalar.

---

## 🚀 12. Plano de Fases

### Fase 1 — Base e Formulário de Orçamento
- [ ] Setup do projeto (Vite + TS + Tailwind + Router)
- [ ] Design system: componentes `ui/` com testes
- [ ] Layout base: Navbar, Footer, PageWrapper
- [ ] Página de formulário de orçamento (`/quote`) com upload de arquivos
- [ ] Integração com `POST /api/v1/quotes`

### Fase 2 — Portfólio
- [ ] Página de galeria (`/portfolio`) com filtros
- [ ] Página de detalhe (`/portfolio/:id`) com viewer 3D
- [ ] Integração com `GET /api/v1/portfolio-items`

### Fase 3 — Autenticação e Área do Cliente
- [ ] Fluxo de login social (Google)
- [ ] Route guards por role
- [ ] Página `My Quotes` (`/my-quotes`) com histórico e status
- [ ] Notificação visual de mudança de status

### Fase 4 — Painel Admin
- [ ] Sidebar de navegação
- [ ] Listagem de orçamentos com filtros por status
- [ ] Painel de detalhe do orçamento (resposta + atualização de status)
- [ ] CRUD do portfólio (upload de fotos e STL)

### Fase 5 — Qualidade e Deploy
- [ ] Cobertura de testes ≥ 80%
- [ ] Lighthouse score ≥ 90 (performance, acessibilidade)
- [ ] Build de produção via Docker
- [ ] Variáveis de ambiente separadas por ambiente

---

## 🧠 13. Prompt Padrão para IA

```
Você é meu par de programação no frontend do Dimension.Lab3D.
Stack: React 18 + TypeScript + Tailwind CSS + Vite.
Seguiremos TDD rigoroso — nenhum componente será criado sem teste (Vitest + RTL).
Todas as decisões devem respeitar este Cloud.md.
Explique antes de modificar qualquer estrutura.
Idioma do código: English.
Design system: dark theme azulado (#0A0A0F de fundo), acento azul elétrico (#4D9FFF) e roxo (#8B5CF6) — baseado no logo da marca. Gradiente principal: from-[#4D9FFF] to-[#8B5CF6]. Espaçamento em grid de 8px. Mobile first obrigatório.
```

---

---

## ⚠️ 14. Dívidas Técnicas Registradas

| # | Arquivo | Descrição | Quando restaurar |
|---|---|---|---|
| 1 | `src/router/index.tsx` | `PrivateRoute role="ADMIN"` removido das rotas `/admin` e `/admin/quotes/:id` para visualização local sem backend OAuth | Antes de iniciar a Fase 3 (autenticação) |

---

*Última atualização: 2026-03-12 — Fase 1 em progresso.*
