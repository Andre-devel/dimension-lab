# 🧠 CLOUD.md — 3D Print Shop (Spec Viva)

> Documento de especificação viva do projeto. Toda decisão arquitetural deve ser registrada aqui.
> Nenhuma feature pode ser criada sem teste. Nenhuma modificação sem explicação.

---

## 🎯 1. Classificação

**Build to Learning** — experimento técnico com liberdade para arriscar arquitetura nova, aplicar TDD rigoroso e full-agent com IA.

---

## 📌 2. Domínio

| Campo | Descrição |
|---|---|
| **Problema** | Clientes de impressão 3D não têm canal digital para solicitar orçamentos nem visualizar trabalhos anteriores |
| **Usuários** | Visitantes anônimos, clientes com login (Google OAuth2 ou email/senha), administrador (dono da impressora) |
| **Casos de uso críticos** | Envio de orçamento, notificação WhatsApp, exibição do portfólio |
| **Limitações** | Deploy local (localhost) no início; sem gateway de pagamento no MVP |

### Regras de Negócio Críticas (não podem falhar)

- Cliente deve conseguir enviar orçamento com anexos (imagem, vídeo, STL)
- Admin deve receber notificação no WhatsApp ao chegar novo orçamento
- Portfólio deve estar sempre disponível publicamente

---

## 🏛 3. Arquitetura

### Organização
- **Multi-repo** — frontend e backend em repositórios separados

### Backend
- **Linguagem:** Java 25
- **Framework:** Spring Boot 4.0.3
- **Padrão Arquitetural:** Clean Architecture

```
backend/
├── domain/              # Entidades, Value Objects, interfaces de repositório
│   ├── quote/
│   ├── portfolio/
│   └── notification/
├── application/         # Use Cases, DTOs, ports
│   ├── quote/
│   ├── portfolio/
│   └── notification/
├── infrastructure/      # Implementações: JPA, WhatsApp, Storage, Spring Security
│   ├── persistence/
│   ├── whatsapp/
│   ├── storage/
│   └── security/
└── interfaces/          # Controllers REST, exception handlers
    └── rest/
```

### Frontend
- **Framework:** React (SPA)
- **Separado do backend:** Sim (repositório próprio)

```
frontend/
├── src/
│   ├── pages/           # Home, Portfolio, QuoteForm, Dashboard, Login
│   ├── components/      # Shared UI components
│   ├── services/        # API calls (axios)
│   ├── hooks/           # Custom hooks
│   └── store/           # Estado global (Context ou Zustand)
└── public/
```

### Banco de Dados
- **PostgreSQL**
- Justificativa: relacional, robusto para relacionamentos de orçamento/status/usuário, ótimo suporte com Spring Data JPA

### Storage de Arquivos
- **Sistema de arquivos local** (localhost) com abstração via interface `FileStoragePort`
- Permite trocar para S3/MinIO no futuro sem mudar o domínio

### Comunicação
- **REST** (frontend ↔ backend)
- **Eventos internos** para notificação pós-criação de orçamento (ApplicationEvent do Spring)

---

## 🗂 4. Entidades do Domínio

```
Customer        — id, name, email, phone, oauthProvider, oauthId, passwordHash
Quote           — id, customer, description, files[], material, color, quantity, desiredDeadline, portfolioItemId, status, createdAt
QuoteStatus     — RECEIVED | UNDER_REVIEW | APPROVED | PRINTING | READY | DELIVERED | CANCELLED
QuoteFile       — id, quote, filePath, fileType (IMAGE | VIDEO | MODEL_3D)
PortfolioItem   — id, title, category, material, printTime, complexity, photos[], modelFile, visible
Category        — id, name, slug
```

**Regra de domínio:** `description` é opcional em `Quote` quando `portfolioItemId` estiver presente — o item do portfólio serve como referência suficiente.

---

## 🔔 5. Notificações

| Canal | Serviço | Trigger |
|---|---|---|
| WhatsApp | Evolution API (self-hosted) | Novo orçamento recebido |
| E-mail (futuro) | SMTP / Resend | Mudança de status do pedido |

**Integração Evolution API:**
- Chamada direta via HTTP ao criar `Quote`
- Implementado como `NotificationPort` na camada de domínio
- Injetado via `EvolutionWhatsAppAdapter` na infra
- Envio assíncrono com `@Async` + virtual threads (`spring.threads.virtual.enabled: true`)
- Body da API: `{ "number": "...", "textMessage": { "text": "..." } }`
- Variáveis: `EVOLUTION_ENABLED`, `EVOLUTION_BASE_URL`, `EVOLUTION_INSTANCE`, `EVOLUTION_API_KEY`, `EVOLUTION_ADMIN_NUMBER`
- **Atenção:** número remetente (conectado na Evolution) deve ser diferente do `EVOLUTION_ADMIN_NUMBER` para gerar notificação push

---

## 🔐 6. Autenticação

- **Spring Security + OAuth2** (Google como provider)
- Login opcional — visitante pode enviar orçamento sem conta
- Se logado, `Quote` fica vinculado ao `Customer`
- Admin identificado por role: `ROLE_ADMIN`

---

## 🧪 7. TDD — Regras

### Antes de qualquer implementação:
- [ ] Teste escrito e falhando
- [ ] Mocks configurados (Mockito)
- [ ] Cobertura mínima: **80%** nas camadas de `domain` e `application`

### Após implementação:
- [ ] Todos os testes passam
- [ ] Nenhum teste removido
- [ ] Refatoração realizada se necessário
- [ ] Testes de regressão executados

### Ferramentas
- `JUnit 5` + `Mockito` (unit)
- `Spring Boot Test` + `Testcontainers` (integração com PostgreSQL real)
- `RestAssured` ou `MockMvc` (testes de controller)

---

## 📘 8. Convenções

| Item | Padrão |
|---|---|
| **Idioma do código** | English (variáveis, métodos, classes, commits) |
| **Branches** | `main`, `develop`, `feature/nome-da-feature` |
| **Commits** | Conventional Commits: `feat:`, `fix:`, `test:`, `refactor:`, `docs:` |
| **Nomeação de classes** | PascalCase |
| **Nomeação de métodos/variáveis** | camelCase |
| **Endpoints REST** | kebab-case: `/api/portfolio-items`, `/api/quotes` |
| **Padrão de logs** | SLF4J com níveis: INFO para operações normais, ERROR para falhas |
| **Padrão de erro** | `ApiError { status, message, timestamp, details[] }` |
| **Versionamento API** | `/api/v1/...` desde o início |

---

## 🔐 9. AI Jail (Regras para IA)

- **Nunca criar feature sem teste**
- **Nunca modificar código sem explicar a razão**
- **Sempre justificar decisões arquiteturais**
- **Nunca acessar fora do workspace do projeto**
- **Sempre rodar testes após mudanças**

---

## 🚀 10. Plano de Fases

### Fase 1 — MVP (Core do negócio)
- [x] Configurar projeto Spring Boot com Clean Architecture
- [x] Entidades: `Quote`, `QuoteFile`, `Customer`
- [x] Endpoint `POST /api/v1/quotes` com upload de arquivos
- [x] Integração Evolution API (notificação WhatsApp)
- [x] Painel admin básico: listar e atualizar status de orçamentos (`GET /api/v1/quotes`, `PATCH /api/v1/quotes/:id/status`)
- [x] **Testes obrigatórios para todos os use cases**

### Fase 2 — Portfólio
- [x] Entidades: `PortfolioItem`, `Category`
- [x] `GET /api/v1/portfolio-items` — galeria pública (apenas visíveis)
- [x] `GET /api/v1/portfolio-items/{id}` — detalhe público
- [x] `GET /api/v1/portfolio-items/all` — listagem completa incluindo ocultos (ADMIN)
- [x] `POST /api/v1/portfolio-items` — criar item com upload de arquivos via multipart (ADMIN)
- [x] `PUT /api/v1/portfolio-items/{id}` — atualizar item com upload de arquivos via multipart (ADMIN)
- [x] `DELETE /api/v1/portfolio-items/{id}` — deletar item (ADMIN)
- [x] `PATCH /api/v1/portfolio-items/{id}/visibility` — toggle visibilidade (ADMIN)
- [x] `LocalFileStorageAdapter` — retorna paths `/uploads/{filename}` (URL relativa, não caminho absoluto)
- [x] `WebMvcConfig` — serve `/uploads/**` do filesystem via HTTP
- [x] `SecurityConfig` — `/uploads/**` permitAll()

### Fase 3 — Área do Cliente
- [x] OAuth2 Google no backend (`OAuth2SuccessHandler`, `JwtService`, `JwtAuthFilter`)
- [x] Autenticação email/senha — `POST /api/v1/auth/register` e `POST /api/v1/auth/login`
  - `V008` migration: `oauth_provider`/`oauth_id` anuláveis, coluna `password_hash` adicionada
  - `BCryptPasswordEncoder` bean em `SecurityConfig`; hash salvo no registro, verificado no login
  - Cookie `auth_token` HttpOnly emitido nos dois fluxos (Google e email/senha)
- [x] `GET /api/v1/auth/me` e `POST /api/v1/auth/logout`
- [x] `GET /api/v1/quotes/my` — histórico do cliente autenticado (`ListMyQuotesUseCase`)
- [x] `SecurityConfig` — endpoints admin protegidos com `ROLE_ADMIN`, `/my` com `ROLE_CLIENT`
- [x] `Customer` convertido para Java **record** — accessors: `.id()`, `.name()`, `.email()`, `.phone()`, `.oauthProvider()`, `.oauthId()`
- [x] `V004` migration — `quotes.customer_id` nullable + FK `REFERENCES customers(id) ON DELETE SET NULL`
- [x] `JwtService.generateToken()` aceita 4 args (`subject`, `name`, `email`, `role`); adicionado `extractName()`
- [x] `GET /api/v1/auth/me` — carrega `name` e `phone` do banco via `CustomerRepository.findById()`
- [x] `CustomerRepository` — adicionado `findById(UUID id)`
- [x] `POST /api/v1/quotes` — suporta cliente autenticado via `authenticatedCustomerId` em `CreateQuoteRequest`; vincula orçamento ao `Customer` real no banco
- [x] Telefone obrigatório para todos os orçamentos (autenticado ou anônimo)
- [x] Ao submeter orçamento com autenticado sem telefone no perfil: telefone é salvo em `customers` automaticamente
- [x] Login social no React (frontend — concluído na Fase 3 frontend)
- [ ] Acompanhamento de status com notificação por e-mail (não implementado)

### Fase 3.5 — Catálogo, Configurações e Perfil (concluída)
- [x] `GET /api/v1/materials` — listar ativos (público)
- [x] `GET /api/v1/materials/all` — listar todos incluindo inativos (ADMIN)
- [x] `POST /api/v1/materials` — criar material (ADMIN)
- [x] `PATCH /api/v1/materials/{id}/toggle` — ativar/desativar (ADMIN)
- [x] `PUT /api/v1/materials/{id}` — editar nome do material (ADMIN)
- [x] `DELETE /api/v1/materials/{id}` — remover material (ADMIN); valida quotes vinculadas (409)
- [x] `GET /api/v1/colors` — listar ativos (público)
- [x] `GET /api/v1/colors/all` — listar todos incluindo inativos (ADMIN)
- [x] `POST /api/v1/colors` — criar cor com hex (ADMIN)
- [x] `PATCH /api/v1/colors/{id}/toggle` — ativar/desativar (ADMIN)
- [x] `PUT /api/v1/colors/{id}` — editar nome e hex da cor (ADMIN)
- [x] `DELETE /api/v1/colors/{id}` — remover cor (ADMIN); valida quotes vinculadas (409)
- [x] **Configurações do site** — tabela `site_settings` (key/value) — V009 migration com seeds
  - `GET /api/v1/settings` — retorna todas as configurações públicas (whatsapp_url, instagram_url, youtube_url, whatsapp_admin_number, bot_number)
  - `PUT /api/v1/settings/{key}` — atualiza valor (ADMIN)
  - V014 migration: `bot_number` — número do WhatsApp bot usado no link "Solicitar via WhatsApp" no PortfolioDetail
- [x] `PATCH /api/v1/auth/profile` — atualiza `name` e `phone` do cliente autenticado
- [x] `GET /api/v1/auth/check-email?email=` — verifica se e-mail pertence a conta registrada (público)
- [x] `GET /api/v1/auth/check-phone?phone=` — verifica se telefone pertence a conta registrada (público)
- [x] `CustomerRepository.findByPhone(String)` — busca por número de telefone
- [x] `V010` migration — renomeia `whatsapp` → `phone` em `customers` e `customer_whatsapp` → `customer_phone` em `quotes`
- [x] `CreateQuoteUseCase` — bloqueia orçamento anônimo se e-mail ou telefone pertencer a conta registrada (409)

### Fase 4 — Produção
- [x] `Dockerfile` do backend — multi-stage (eclipse-temurin:25-jdk builder + 25-jre runtime); usuário não-root; volume `/app/uploads`; healthcheck em `/actuator/health`
- [x] `docker-compose.yml` — orquestra postgres + backend + frontend + volumes persistentes; portas ligadas a `127.0.0.1` (Caddy como proxy); healthcheck do backend via `/dev/tcp` (sem curl/wget na imagem); frontend na `3000:80` (Caddy ocupa a 80); postgres expõe `5432` localmente para acesso via SSH tunnel (DBeaver)
- [x] `application-prod.yml` — secrets via env vars no `.env` do VPS; `COOKIE_SECURE=true`; `STORAGE_PATH=/app/uploads` mapeado em volume Docker
- [x] Rate limiting — `RateLimitFilter` (Bucket4j 8.10.1) em `POST /auth/register` (5/15min), `POST /auth/login` (10/15min), `POST /quotes` (5/15min); por IP com suporte a `X-Forwarded-For`; retorna `429` com JSON `ApiError`
- [x] `cookie.setSecure(cookieSecure)` — controlado via `COOKIE_SECURE` env var (default `false`); aplicado em `AuthController` e `OAuth2SuccessHandler`; setar `true` em produção (HTTPS)
- [x] `GET /sitemap.xml` — `SitemapController` retorna XML com páginas estáticas + portfolio items visíveis; usa `FRONTEND_URL`; público no `SecurityConfig`
- [x] CI/CD — GitHub Actions: `ci-backend.yml` (lint + test), `cd.yml` (deploy ao VPS via SSH após ambos CIs passarem); autenticação de submodules privados via secret `MY_REPO_PAT` com `token:` no checkout action
- [x] Deploy em VPS — site no ar em `dimensionlab.tech` com HTTPS via Caddy (Let's Encrypt automático); Caddy faz proxy `/api/*` → backend:8080 e `/` → frontend:3000; subdomínio `evolution.dimensionlab.tech` → porta 8081
- [x] `portfolioItemId` adicionado ao `QuoteResponse` — admin vê referência do portfólio no detalhe do orçamento
- [x] `description` opcional em `Quote.builder()` quando `portfolioItemId` presente — regra validada no domínio; `@NotBlank` removido de `QuoteRequest` no layer de interfaces
- [x] `CANCELLED` adicionado ao `QuoteStatus` — cancelamento pelo cliente via `PATCH /api/v1/quotes/{id}/cancel` (`CancelQuoteUseCase`)
- [x] `EVOLUTION_API_KEY` mapeado no `docker-compose.yml` via `${EVOLUTION_API_KEY:-}`
- [x] Compressão de imagem — `LocalFileStorageAdapter` comprime automaticamente `FileType.IMAGE` via Thumbnailator (`net.coobird:thumbnailator:0.4.20`); busca binária de qualidade (0.40–0.90) para atingir `STORAGE_IMAGE_MAX_SIZE_KB` (default 500KB); resize para `STORAGE_IMAGE_MAX_WIDTH/HEIGHT` (default 1920×1920) sem upscale; PNG > limite convertido automaticamente para JPEG; fallback silencioso para bytes originais se imagem não puder ser parseada
- [x] Upload organizado em subpastas — `FileStoragePort.store()` recebe `subfolder`; portfólio salva em `/uploads/portfolio/`, orçamentos em `/uploads/quotes/`; delete e serve `/uploads/**` continuam funcionando
- [x] Delete de arquivos antigos no update do portfólio — `PortfolioController.update()` deleta fotos e modelFile anteriores do disco antes de salvar os novos
- [x] `RestClientConfig` — bean explícito `RestClient.Builder` via `@Bean` em `infrastructure/config/RestClientConfig.java`; necessário porque Spring Boot 4.x não auto-configura mais esse bean (ao contrário do 3.x); injetado no `GeminiImageStandardizationAdapter`
- [x] `GlobalExceptionHandler` — handler `HttpClientErrorException` para erros de APIs externas: 429 → 503 com mensagem clara sobre spending cap; outros → 502; evita NPE e expõe mensagem útil para o frontend
- [x] `docker-compose.yml` — `GEMINI_API_KEY` e `GEMINI_IMAGE_MODEL` mapeados para o container backend via `${GEMINI_API_KEY:-}` e `${GEMINI_IMAGE_MODEL:-gemini-3.1-flash-image-preview}`; regra: sempre mapear nova env var no docker-compose ao adicioná-la no `.env.example`
- [ ] Paginação em `GET /api/v1/quotes` (admin) — sem paginação a query cresce sem limite
- [ ] Monitoramento: Actuator já expõe `/health` e `/info`; métricas (Prometheus/Grafana) são opcionais

---

## 📊 11. Métricas

| Métrica | Meta |
|---|---|
| Cobertura de testes (domain + application) | ≥ 80% |
| Bugs detectados antes do deploy | Máximo possível via TDD |
| Dívida técnica | Revisada a cada fase |

---

## 🧠 12. Prompt Padrão para IA

```
Você é meu par de programação neste projeto de impressão 3D.
Seguiremos TDD rigoroso — nenhuma feature será criada sem teste.
Todas as decisões arquiteturais devem respeitar este Cloud.md.
Explique antes de modificar qualquer estrutura.
O padrão arquitetural é Clean Architecture com Java 25 + Spring Boot 4.0.3 .
Idioma do código: English.
```

---

*Última atualização: 2026-03-27 — RestClientConfig bean explícito (Spring Boot 4.x compat); GlobalExceptionHandler para HttpClientErrorException (429→503, outros→502); GEMINI_API_KEY/MODEL mapeados no docker-compose. Pendente: paginação de quotes admin.*