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
| **Usuários** | Visitante anônimo, cliente logado (Google ou email/senha), admin |
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
| `/admin/portfolio` | Lista de itens do portfólio | Logado (admin) |
| `/admin/portfolio/new` | Criar item do portfólio | Logado (admin) |
| `/admin/portfolio/:id/edit` | Editar item do portfólio | Logado (admin) |

---

## 🎨 5. Design System

### Cores

> Paleta extraída do logo Dimension.Lab3D — dark background com azul elétrico e roxo como acentos.

**Fonte única de verdade:** as cores estão definidas como variáveis CSS em `src/index.css` (`:root`), no formato `R G B` para suportar modificadores de opacidade do Tailwind (ex: `bg-accent-blue/20`). O `tailwind.config.ts` mapeia cada variável para um token Tailwind via `rgb(var(--c-*) / <alpha-value>)`.

Tokens disponíveis: `background`, `surface`, `surface-2`, `border`, `accent-blue`, `accent-purple`, `accent-glow`, `text-primary`, `text-secondary`, `error`.

> **Regra:** nunca use hex fixo para cores do design system — use os tokens Tailwind (`bg-accent-blue`, `text-error`, etc.) ou `rgb(var(--c-*))` em inline styles.

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
- **Toda nova página pública deve incluir `<SEOHead>` com `title`, `description`, `canonical` e `jsonLd` adequado — sem exceção**

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
| react-helmet-async | 3 | Meta tags dinâmicas por página (SEO) |

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
- [x] Página de galeria (`/portfolio`) com filtros por material
- [x] Página de detalhe (`/portfolio/:id`) com viewer 3D (`<model-viewer>`)
- [x] Integração com `GET /api/v1/portfolio-items` e `GET /api/v1/portfolio-items/{id}`
- [x] `PortfolioCard` component, `portfolioService`, testes para card/página/serviço

### Fase 3 — Autenticação e Área do Cliente
- [x] Fluxo de login social (Google) — OAuth2 redirect via `/oauth2/authorization/google`
- [x] Autenticação email/senha — páginas `/login` e `/register`, `POST /api/v1/auth/login` e `/register`
  - Navbar "Entrar" navega para `/login` (opções: email/senha ou Google)
  - Login seta cookie HttpOnly `auth_token` e popula `authStore` via `setUser()`
  - Redirect pós-login: ADMIN → `/admin`, CLIENT → `/my-quotes`
- [x] `useAuth` hook — rehydrata authStore via `GET /api/v1/auth/me` no mount
- [x] `authStore` — `isAuthLoading` para evitar redirect prematuro no PrivateRoute
- [x] Route guards por role — `PrivateRoute` restaurado para `/admin` e `/my-quotes`
- [x] Página `My Quotes` (`/my-quotes`) com histórico e status badges
- [x] Navbar condicional — Entrar/Sair + links por role (CLIENT → Meus Orçamentos, ADMIN → Admin)
- [x] Notificação visual de mudança de status — badge "Atualizado" via localStorage (utils/quoteNotifications + useQuoteNotifications)
- [x] `User` type (`types/user.ts`) — adicionado campo `phone?: string`
- [x] `QuoteRequest` ciente do estado de autenticação:
  - Autenticado: exibe card com nome/email/(telefone se já salvo); pede telefone apenas se perfil não tiver
  - Anônimo: exibe campos nome, e-mail e telefone (telefone obrigatório por validação customizada)
- [x] `authStore.setUser()` chamado imediatamente após submit bem-sucedido com telefone novo — evita re-exibição do campo sem reload de página

### Fase 4 — Painel Admin (concluída)
- [x] Listagem de orçamentos com filtros por status (Dashboard)
- [x] Painel de detalhe do orçamento com atualização de status (QuoteDetail)
- [x] CRUD do portfólio via UI admin (`/admin/portfolio`, `/admin/portfolio/new`, `/admin/portfolio/:id/edit`)
- [x] Toggle de visibilidade e exclusão de itens do portfólio
- [x] Upload real de arquivos (fotos/STL) via `FormData` multipart — backend serve `/uploads/**` via HTTP
- [x] `fileUrl()` utility — prefixa paths `/uploads/*` com `VITE_API_BASE_URL` para uso em `<img src>`
- [x] Catálogo de materiais (`/admin/materials`) — criar + toggle ativo/inativo
- [x] Catálogo de cores (`/admin/colors`) — criar com hex + toggle ativo/inativo
- [x] Editar e remover materiais — `MaterialsAdmin` com edição inline e exclusão (409 → mensagem de erro)
- [x] Editar e remover cores — `ColorsAdmin` com edição inline (nome + hex picker) e exclusão (409 → mensagem de erro)
- [x] Página de configurações do site (`/admin/settings`) — gerencia whatsapp_url, instagram_url, youtube_url, whatsapp_admin_number, bot_number via `GET/PUT /api/v1/settings`

### Fase 4.5 — Perfil de Usuário e Segurança (concluída)
- [x] Página `/profile` — editar nome e telefone; e-mail somente leitura; `PATCH /api/v1/auth/profile`
- [x] Navbar com link "Perfil" para todos os autenticados; "Admin" para ADMIN; "Meus Orçamentos" para CLIENT
- [x] `authService.updateProfile(name, phone)` — atualiza `authStore` imediatamente após sucesso
- [x] `authService.checkEmail(email)` e `authService.checkPhone(phone)` — endpoints de verificação
- [x] `QuoteRequest`: backend bloqueia (409) orçamento anônimo com e-mail ou telefone de conta registrada
- [x] `QInput` com `forwardRef` — corrige leitura de valores pelo React Hook Form em componentes customizados
- [x] Campo quantidade usando `Controller` — resolve bug de `NaN` com `type="number"`
- [x] Setas de spin removidas do campo numérico (CSS `appearance: textfield`)
- [x] Refactor `whatsapp` → `phone` em todo o codebase (campo `User.phone`, `Customer.phone`, `CreateQuotePayload.customerPhone`, migration V010)

### Fase 5 — Qualidade e Deploy
- [x] `Dockerfile` do frontend — Node build stage + nginx para servir `/dist`; `VITE_API_BASE_URL=""` (nginx proxy transparente via `/api/`)
- [x] `.env.production` — configurado no VPS via `/opt/dimension-lab/.env`; `VITE_API_BASE_URL` vazio pois Caddy/nginx fazem proxy
- [x] CI/CD — `ci-frontend.yml` (lint + test + build com Vitest coverage ≥ 80%); `cd.yml` dispara deploy ao VPS após ambos CIs passarem; autenticação de submodules privados via `MY_REPO_PAT`
- [x] TypeScript build errors resolvidos — `vite.config.ts` importa de `vitest/config`; `@types/node` adicionado; non-null assertions em `Home`; `description?: string` em `PortfolioItem`; `?? ''` em `PortfolioDetail` e `QuoteRequest`
- [x] Cobertura de testes ≥ 80% — 212 testes passando, 85% de cobertura de funções; novos testes: `Toast`, `CustomSelect`, `Carousel`, `BackButton`, `authService` expandido; `scrollBy` stub no `setup.ts`
- [x] Corrigir typo no `.env`: `VITE_YOUTUBE_URL` tinha espaço — corrigido no VPS
- [x] Substituir `alert()` e `window.confirm()` em `MyQuotes/index.tsx` por confirmação inline no card e banner de erro com mensagem do backend
- [x] Sitemap dinâmico — `GET /sitemap.xml` no backend; `<link rel="sitemap">` no `index.html`
- [x] `BackButton` aplicado em todas as páginas (QuoteDetail, PortfolioDetail, MyQuotes, Profile, QuoteRequest, NewPortfolioItem, EditPortfolioItem)
- [x] `ScrollRestoration` via root layout no `createBrowserRouter` — scroll volta ao topo a cada navegação
- [x] `scroll-behavior: smooth` global no `index.css`
- [x] Shake animation (`field-shake`) nos campos inválidos ao tentar submeter o formulário de orçamento
- [x] Máscara de telefone brasileiro `(XX) XXXXX-XXXX` com validação Zod no formulário de orçamento
- [x] `whatsapp_admin_number` adicionado à página de configurações do admin
- [x] Ícones SVG substituem botões de texto (Editar/Desativar/Excluir) em `ColorsAdmin` e `MaterialsAdmin` — fix layout mobile
- [x] Lighthouse score ≥ 90 — performance 93, accessibility 100, best-practices 96, seo 100
- [x] Remover arquivos de rascunho da raiz do repositório: `form.html`, `cubo-animation.txt`, `index.html`, `img.png`, `logo.jpeg`, `uploads/` — já removidos; `lighthouse-report.json` adicionado ao `.gitignore`
- [x] CI/CD (GitHub Actions): lint + test + build automático — ver acima
- [x] Tratamento de erro global — `ErrorBoundary` (erros de renderização) + interceptor Axios para 5xx e sem conexão → `useToastStore`; `Toast` renderizado em `App.tsx`
- [x] `.env.example` para facilitar onboarding
- [x] `bot_number` em `settingsService` e `SettingsAdmin` — link WhatsApp no `PortfolioDetail` usa `bot_number` da API (antes hardcoded)
- [x] `portfolioItemId` em `Quote` type e exibido no `QuoteDetail` admin — card "Referência do Portfólio" com link para o item
- [x] `QuoteRequest`: `description` não obrigatória quando `portfolioRef` está presente (validação frontend já tratava; backend removeu `@NotBlank` de `QuoteRequest` e `description` no `Quote.builder()` é opcional com `portfolioItemId`)
- [x] `QuoteRequest`: color scroll fix — `id="color-field"` no container das swatches; `triggerShake` usa `scrollIntoView` + `focus`; `focusMap` inclui `color`; `scrollIntoView` stubado no `setup.ts`

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
| ~~1~~ | ~~`src/router/index.tsx`~~ | ~~`PrivateRoute role="ADMIN"` removido~~ | ✅ Restaurado na Fase 3 |

---

*Última atualização: 2026-03-26 — Fase 5 concluída. bot_number em settings; portfolioItemId no QuoteDetail admin; WhatsApp link dinâmico no PortfolioDetail; description opcional com portfolioRef; color scroll fix no QuoteRequest.*
