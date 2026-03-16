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
| **Usuários** | Visitantes anônimos, clientes com login social, administrador (dono da impressora) |
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
Customer        — id, name, email, whatsapp, oauthProvider, oauthId
Quote           — id, customer, description, files[], material, color, quantity, finish, desiredDeadline, status, createdAt
QuoteStatus     — RECEIVED | UNDER_REVIEW | APPROVED | PRINTING | READY | DELIVERED
QuoteFile       — id, quote, filePath, fileType (IMAGE | VIDEO | MODEL_3D)
PortfolioItem   — id, title, category, material, printTime, complexity, photos[], modelFile, visible
Category        — id, name, slug
```

---

## 🔔 5. Notificações

| Canal | Serviço | Trigger |
|---|---|---|
| WhatsApp | Evolution API (self-hosted) | Novo orçamento recebido |
| E-mail (futuro) | SMTP / Resend | Mudança de status do pedido |

**Integração Evolution API:**
- Webhook ou chamada direta via HTTP ao criar `Quote`
- Implementado como `NotificationPort` na camada de domínio
- Injetado via `EvolutionWhatsAppAdapter` na infra

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
- [x] `GET /api/v1/auth/me` e `POST /api/v1/auth/logout`
- [x] `GET /api/v1/quotes/my` — histórico do cliente autenticado (`ListMyQuotesUseCase`)
- [x] `SecurityConfig` — endpoints admin protegidos com `ROLE_ADMIN`, `/my` com `ROLE_CLIENT`
- [x] `Customer` convertido para Java **record** — accessors: `.id()`, `.name()`, `.email()`, `.whatsapp()`, `.oauthProvider()`, `.oauthId()`
- [x] `V004` migration — `quotes.customer_id` nullable + FK `REFERENCES customers(id) ON DELETE SET NULL`
- [x] `JwtService.generateToken()` aceita 4 args (`subject`, `name`, `email`, `role`); adicionado `extractName()`
- [x] `GET /api/v1/auth/me` — carrega `name` e `whatsapp` do banco via `CustomerRepository.findById()`
- [x] `CustomerRepository` — adicionado `findById(UUID id)`
- [x] `POST /api/v1/quotes` — suporta cliente autenticado via `authenticatedCustomerId` em `CreateQuoteRequest`; vincula orçamento ao `Customer` real no banco
- [x] WhatsApp obrigatório para todos os orçamentos (autenticado ou anônimo)
- [x] Ao submeter orçamento com autenticado sem WhatsApp no perfil: WhatsApp é salvo em `customers` automaticamente
- [ ] Login social no React (frontend — concluído na Fase 3 frontend)
- [ ] Acompanhamento de status com notificação por e-mail

### Fase 4 — Produção
- [ ] Docker Compose (backend + PostgreSQL + Evolution API)
- [ ] CI/CD básico (GitHub Actions: lint + test + build)
- [ ] Deploy em VPS ou Raspberry Pi
- [ ] Variáveis de ambiente seguras (.env fora do repo)
- [ ] Monitoramento básico (Actuator + logs estruturados)

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

*Última atualização: 2026-03-16 — Fase 3 backend completa: Customer como record, V004 FK nullable, JWT com name, /me carrega do DB, orçamento vincula cliente autenticado, WhatsApp obrigatório e persistido no perfil na primeira submissão.*