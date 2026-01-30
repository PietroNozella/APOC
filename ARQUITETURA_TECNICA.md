# 🏗️ Arquitetura Técnica - Plataforma de Compliance
## Stack: Vercel + Supabase

---

## 📋 Sumário

1. [Visão Geral da Arquitetura](#1-visão-geral-da-arquitetura)
2. [Stack Tecnológica](#2-stack-tecnológica)
3. [Estrutura de Pastas](#3-estrutura-de-pastas)
4. [Schema do Banco de Dados](#4-schema-do-banco-de-dados)
5. [Autenticação e Autorização](#5-autenticação-e-autorização)
6. [Storage de Arquivos](#6-storage-de-arquivos)
7. [APIs e Endpoints](#7-apis-e-endpoints)
8. [Configurações e Variáveis de Ambiente](#8-configurações-e-variáveis-de-ambiente)
9. [Deploy e CI/CD](#9-deploy-e-cicd)

---

## 1. Visão Geral da Arquitetura

### 1.1 Diagrama de Arquitetura

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              USUÁRIOS                                        │
│                    (Browser / Desktop / Mobile)                              │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  │ HTTPS
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              VERCEL                                          │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         NEXT.JS APP                                  │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌────────────┐ │   │
│  │  │    Pages    │  │ Components  │  │    Hooks    │  │   Utils    │ │   │
│  │  │  (App Dir)  │  │    (UI)     │  │  (Logic)    │  │  (Helpers) │ │   │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └────────────┘ │   │
│  │                                                                      │   │
│  │  ┌─────────────────────────────────────────────────────────────┐   │   │
│  │  │                    API ROUTES (Server)                       │   │   │
│  │  │    /api/auth  /api/policies  /api/risks  /api/controls      │   │   │
│  │  └─────────────────────────────────────────────────────────────┘   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                      MIDDLEWARE                                      │   │
│  │          (Auth Check, Rate Limiting, Logging)                       │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────┬───────────────────────────────────────────┘
                                  │
                                  │ API Calls
                                  ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                             SUPABASE                                         │
│                                                                              │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐             │
│  │   PostgreSQL    │  │  Authentication │  │     Storage     │             │
│  │   (Database)    │  │   (Auth/JWT)    │  │    (Buckets)    │             │
│  │                 │  │                 │  │                 │             │
│  │  • Tables       │  │  • Email/Pass   │  │  • evidences    │             │
│  │  • Views        │  │  • Magic Link   │  │  • policies     │             │
│  │  • Functions    │  │  • OAuth        │  │  • attachments  │             │
│  │  • Triggers     │  │  • MFA          │  │  • exports      │             │
│  │  • RLS          │  │  • Sessions     │  │                 │             │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘             │
│                                                                              │
│  ┌─────────────────┐  ┌─────────────────┐                                   │
│  │    Realtime     │  │  Edge Functions │                                   │
│  │  (WebSockets)   │  │   (Serverless)  │                                   │
│  │                 │  │                 │                                   │
│  │  • Notificações │  │  • Cron Jobs    │                                   │
│  │  • Updates      │  │  • Webhooks     │                                   │
│  └─────────────────┘  └─────────────────┘                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Fluxo de Dados

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  Browser │────>│  Vercel  │────>│ Supabase │────>│ Response │
│          │     │ (Next.js)│     │   (DB)   │     │          │
└──────────┘     └──────────┘     └──────────┘     └──────────┘
     │                │                │                │
     │  1. Request    │                │                │
     │ ─────────────> │                │                │
     │                │  2. Auth Check │                │
     │                │ ─────────────> │                │
     │                │                │  3. Query DB   │
     │                │                │ ─────────────> │
     │                │                │  4. Data       │
     │                │  5. Process    │ <───────────── │
     │                │ <───────────── │                │
     │  6. Response   │                │                │
     │ <───────────── │                │                │
```

---

## 2. Stack Tecnológica

### 2.1 Frontend

| Tecnologia | Versão | Propósito |
|------------|--------|-----------|
| **Next.js** | 14.x | Framework React com App Router |
| **React** | 18.x | Biblioteca de UI |
| **TypeScript** | 5.x | Tipagem estática |
| **Tailwind CSS** | 3.x | Estilização utility-first |
| **shadcn/ui** | latest | Componentes de UI |
| **Lucide React** | latest | Ícones |
| **React Hook Form** | 7.x | Formulários |
| **Zod** | 3.x | Validação de schemas |
| **TanStack Query** | 5.x | Cache e estado do servidor |
| **Zustand** | 4.x | Estado global (se necessário) |

### 2.2 Backend / Database

| Tecnologia | Propósito |
|------------|-----------|
| **Supabase** | Backend-as-a-Service |
| **PostgreSQL** | Banco de dados relacional |
| **Supabase Auth** | Autenticação e autorização |
| **Supabase Storage** | Armazenamento de arquivos |
| **Supabase Realtime** | Notificações em tempo real |
| **Row Level Security** | Segurança a nível de linha |

### 2.3 Infraestrutura

| Tecnologia | Propósito |
|------------|-----------|
| **Vercel** | Hospedagem e deploy |
| **GitHub** | Controle de versão |
| **GitHub Actions** | CI/CD complementar |

### 2.4 Ferramentas de Desenvolvimento

| Ferramenta | Propósito |
|------------|-----------|
| **ESLint** | Linting de código |
| **Prettier** | Formatação de código |
| **Husky** | Git hooks |
| **Commitlint** | Padrão de commits |
| **Supabase CLI** | Desenvolvimento local |

---

## 3. Estrutura de Pastas

```
APOC/
│
├── 📁 src/
│   ├── 📁 app/                          # App Router (Next.js 14)
│   │   ├── 📁 (auth)/                   # Grupo de rotas públicas
│   │   │   ├── 📁 login/
│   │   │   │   └── page.tsx
│   │   │   ├── 📁 register/
│   │   │   │   └── page.tsx
│   │   │   ├── 📁 forgot-password/
│   │   │   │   └── page.tsx
│   │   │   └── layout.tsx
│   │   │
│   │   ├── 📁 (dashboard)/              # Grupo de rotas protegidas
│   │   │   ├── 📁 dashboard/
│   │   │   │   └── page.tsx
│   │   │   ├── 📁 policies/
│   │   │   │   ├── page.tsx             # Lista de políticas
│   │   │   │   ├── 📁 [id]/
│   │   │   │   │   └── page.tsx         # Detalhe da política
│   │   │   │   └── 📁 new/
│   │   │   │       └── page.tsx         # Nova política
│   │   │   ├── 📁 risks/
│   │   │   │   ├── page.tsx
│   │   │   │   ├── 📁 [id]/
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── 📁 new/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── 📁 matrix/
│   │   │   │       └── page.tsx         # Matriz de riscos
│   │   │   ├── 📁 controls/
│   │   │   │   ├── page.tsx
│   │   │   │   ├── 📁 [id]/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── 📁 frameworks/
│   │   │   │       └── 📁 [framework]/
│   │   │   │           └── page.tsx
│   │   │   ├── 📁 evidences/
│   │   │   │   ├── page.tsx
│   │   │   │   ├── 📁 [id]/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── 📁 upload/
│   │   │   │       └── page.tsx
│   │   │   ├── 📁 tasks/
│   │   │   │   ├── page.tsx
│   │   │   │   └── 📁 [id]/
│   │   │   │       └── page.tsx
│   │   │   ├── 📁 reports/
│   │   │   │   ├── page.tsx
│   │   │   │   └── 📁 [type]/
│   │   │   │       └── page.tsx
│   │   │   ├── 📁 settings/
│   │   │   │   ├── page.tsx
│   │   │   │   ├── 📁 organization/
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── 📁 users/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── 📁 frameworks/
│   │   │   │       └── page.tsx
│   │   │   └── layout.tsx               # Layout com sidebar
│   │   │
│   │   ├── 📁 api/                      # API Routes
│   │   │   ├── 📁 auth/
│   │   │   │   └── 📁 [...supabase]/
│   │   │   │       └── route.ts
│   │   │   ├── 📁 policies/
│   │   │   │   ├── route.ts             # GET, POST
│   │   │   │   └── 📁 [id]/
│   │   │   │       └── route.ts         # GET, PUT, DELETE
│   │   │   ├── 📁 risks/
│   │   │   │   ├── route.ts
│   │   │   │   └── 📁 [id]/
│   │   │   │       └── route.ts
│   │   │   ├── 📁 controls/
│   │   │   │   ├── route.ts
│   │   │   │   └── 📁 [id]/
│   │   │   │       └── route.ts
│   │   │   ├── 📁 evidences/
│   │   │   │   ├── route.ts
│   │   │   │   └── 📁 [id]/
│   │   │   │       └── route.ts
│   │   │   ├── 📁 tasks/
│   │   │   │   ├── route.ts
│   │   │   │   └── 📁 [id]/
│   │   │   │       └── route.ts
│   │   │   ├── 📁 reports/
│   │   │   │   └── route.ts
│   │   │   └── 📁 webhooks/
│   │   │       └── route.ts
│   │   │
│   │   ├── layout.tsx                   # Root layout
│   │   ├── page.tsx                     # Landing page
│   │   ├── loading.tsx                  # Global loading
│   │   ├── error.tsx                    # Global error
│   │   ├── not-found.tsx                # 404 page
│   │   └── globals.css                  # Estilos globais
│   │
│   ├── 📁 components/
│   │   ├── 📁 ui/                       # Componentes shadcn/ui
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   ├── dialog.tsx
│   │   │   ├── dropdown-menu.tsx
│   │   │   ├── form.tsx
│   │   │   ├── input.tsx
│   │   │   ├── select.tsx
│   │   │   ├── table.tsx
│   │   │   ├── tabs.tsx
│   │   │   ├── toast.tsx
│   │   │   └── ...
│   │   │
│   │   ├── 📁 layout/                   # Componentes de layout
│   │   │   ├── sidebar.tsx
│   │   │   ├── header.tsx
│   │   │   ├── footer.tsx
│   │   │   ├── breadcrumb.tsx
│   │   │   └── mobile-nav.tsx
│   │   │
│   │   ├── 📁 dashboard/                # Componentes do dashboard
│   │   │   ├── compliance-score.tsx
│   │   │   ├── risk-summary.tsx
│   │   │   ├── tasks-widget.tsx
│   │   │   ├── alerts-widget.tsx
│   │   │   ├── framework-status.tsx
│   │   │   └── activity-feed.tsx
│   │   │
│   │   ├── 📁 policies/                 # Componentes de políticas
│   │   │   ├── policy-card.tsx
│   │   │   ├── policy-form.tsx
│   │   │   ├── policy-viewer.tsx
│   │   │   ├── policy-history.tsx
│   │   │   └── policy-acceptance.tsx
│   │   │
│   │   ├── 📁 risks/                    # Componentes de riscos
│   │   │   ├── risk-card.tsx
│   │   │   ├── risk-form.tsx
│   │   │   ├── risk-matrix.tsx
│   │   │   ├── risk-heatmap.tsx
│   │   │   └── treatment-plan.tsx
│   │   │
│   │   ├── 📁 controls/                 # Componentes de controles
│   │   │   ├── control-card.tsx
│   │   │   ├── control-form.tsx
│   │   │   ├── control-status.tsx
│   │   │   ├── framework-selector.tsx
│   │   │   └── gap-analysis.tsx
│   │   │
│   │   ├── 📁 evidences/                # Componentes de evidências
│   │   │   ├── evidence-card.tsx
│   │   │   ├── evidence-uploader.tsx
│   │   │   ├── evidence-viewer.tsx
│   │   │   └── expiration-badge.tsx
│   │   │
│   │   ├── 📁 tasks/                    # Componentes de tarefas
│   │   │   ├── task-card.tsx
│   │   │   ├── task-form.tsx
│   │   │   ├── task-list.tsx
│   │   │   └── task-kanban.tsx
│   │   │
│   │   ├── 📁 reports/                  # Componentes de relatórios
│   │   │   ├── report-builder.tsx
│   │   │   ├── chart-compliance.tsx
│   │   │   ├── chart-risks.tsx
│   │   │   └── export-button.tsx
│   │   │
│   │   └── 📁 shared/                   # Componentes compartilhados
│   │       ├── data-table.tsx
│   │       ├── search-input.tsx
│   │       ├── status-badge.tsx
│   │       ├── date-picker.tsx
│   │       ├── file-upload.tsx
│   │       ├── confirm-dialog.tsx
│   │       ├── empty-state.tsx
│   │       └── loading-skeleton.tsx
│   │
│   ├── 📁 lib/
│   │   ├── 📁 supabase/
│   │   │   ├── client.ts                # Cliente browser
│   │   │   ├── server.ts                # Cliente server
│   │   │   ├── middleware.ts            # Helper para middleware
│   │   │   └── admin.ts                 # Cliente admin (service role)
│   │   │
│   │   ├── 📁 validations/              # Schemas Zod
│   │   │   ├── policy.ts
│   │   │   ├── risk.ts
│   │   │   ├── control.ts
│   │   │   ├── evidence.ts
│   │   │   ├── task.ts
│   │   │   └── user.ts
│   │   │
│   │   ├── utils.ts                     # Funções utilitárias
│   │   ├── constants.ts                 # Constantes da aplicação
│   │   └── permissions.ts               # Helpers de permissão
│   │
│   ├── 📁 hooks/                        # Custom hooks
│   │   ├── use-auth.ts
│   │   ├── use-policies.ts
│   │   ├── use-risks.ts
│   │   ├── use-controls.ts
│   │   ├── use-evidences.ts
│   │   ├── use-tasks.ts
│   │   ├── use-toast.ts
│   │   └── use-debounce.ts
│   │
│   ├── 📁 services/                     # Camada de serviços
│   │   ├── policies.service.ts
│   │   ├── risks.service.ts
│   │   ├── controls.service.ts
│   │   ├── evidences.service.ts
│   │   ├── tasks.service.ts
│   │   ├── reports.service.ts
│   │   └── notifications.service.ts
│   │
│   ├── 📁 types/                        # Tipos TypeScript
│   │   ├── database.types.ts            # Gerado pelo Supabase
│   │   ├── policy.types.ts
│   │   ├── risk.types.ts
│   │   ├── control.types.ts
│   │   ├── evidence.types.ts
│   │   ├── task.types.ts
│   │   └── index.ts
│   │
│   └── 📁 config/                       # Configurações
│       ├── site.ts                      # Configurações do site
│       ├── navigation.ts                # Itens de navegação
│       └── frameworks.ts                # Configurações de frameworks
│
├── 📁 supabase/
│   ├── 📁 migrations/                   # Migrations SQL
│   │   ├── 00001_initial_schema.sql
│   │   ├── 00002_policies_table.sql
│   │   ├── 00003_risks_table.sql
│   │   └── ...
│   │
│   ├── 📁 seed/                         # Dados iniciais
│   │   ├── frameworks.sql
│   │   ├── requirements.sql
│   │   └── sample_data.sql
│   │
│   └── config.toml                      # Configuração Supabase
│
├── 📁 public/
│   ├── 📁 images/
│   │   ├── logo.svg
│   │   └── favicon.ico
│   └── 📁 icons/
│
├── 📄 .env.local                        # Variáveis de ambiente (local)
├── 📄 .env.example                      # Exemplo de variáveis
├── 📄 .gitignore
├── 📄 .eslintrc.json
├── 📄 .prettierrc
├── 📄 tailwind.config.ts
├── 📄 tsconfig.json
├── 📄 next.config.js
├── 📄 package.json
├── 📄 README.md
├── 📄 PLANO_MVP_COMPLIANCE.md
└── 📄 ARQUITETURA_TECNICA.md
```

---

## 4. Schema do Banco de Dados

### 4.1 Diagrama Entidade-Relacionamento

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        SCHEMA DO BANCO DE DADOS                              │
└─────────────────────────────────────────────────────────────────────────────┘

                              ┌─────────────────┐
                              │  organizations  │
                              ├─────────────────┤
                              │ id (PK)         │
                              │ name            │
                              │ cnpj            │
                              │ industry        │
                              │ created_at      │
                              └────────┬────────┘
                                       │
           ┌───────────────────────────┼───────────────────────────┐
           │                           │                           │
           ▼                           ▼                           ▼
┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐
│     users       │         │   departments   │         │   frameworks    │
├─────────────────┤         ├─────────────────┤         ├─────────────────┤
│ id (PK)         │         │ id (PK)         │         │ id (PK)         │
│ org_id (FK)     │         │ org_id (FK)     │         │ name            │
│ email           │         │ name            │         │ version         │
│ full_name       │         │ description     │         │ description     │
│ role            │         │ head_user_id    │         │ is_active       │
│ department_id   │         └────────┬────────┘         └────────┬────────┘
│ is_active       │                  │                           │
└────────┬────────┘                  │                           │
         │                           │                           │
         │         ┌─────────────────┘                           │
         │         │                                             │
         ▼         ▼                                             ▼
┌─────────────────────────┐                           ┌─────────────────┐
│       policies          │                           │  requirements   │
├─────────────────────────┤                           ├─────────────────┤
│ id (PK)                 │                           │ id (PK)         │
│ org_id (FK)             │                           │ framework_id(FK)│
│ title                   │                           │ code            │
│ content                 │                           │ title           │
│ version                 │                           │ description     │
│ status                  │                           │ category        │
│ category                │                           └────────┬────────┘
│ owner_id (FK)           │                                    │
│ approved_by (FK)        │                                    │
│ approved_at             │                                    │
│ effective_date          │                                    │
│ review_date             │                                    │
│ created_at              │                                    │
│ updated_at              │                                    │
└────────┬────────────────┘                                    │
         │                                                     │
         │                                                     │
         ▼                                                     ▼
┌─────────────────────────┐                           ┌─────────────────┐
│   policy_acceptances    │                           │    controls     │
├─────────────────────────┤                           ├─────────────────┤
│ id (PK)                 │                           │ id (PK)         │
│ policy_id (FK)          │                           │ org_id (FK)     │
│ user_id (FK)            │                           │ requirement_id  │
│ accepted_at             │                           │ name            │
│ ip_address              │                           │ description     │
└─────────────────────────┘                           │ status          │
                                                      │ owner_id (FK)   │
                                                      │ implementation  │
                                                      │ test_frequency  │
                                                      │ last_tested     │
                                                      │ created_at      │
                                                      │ updated_at      │
                                                      └────────┬────────┘
                                                               │
         ┌─────────────────────────────────────────────────────┤
         │                                                     │
         ▼                                                     ▼
┌─────────────────────────┐                           ┌─────────────────┐
│      evidences          │                           │      risks      │
├─────────────────────────┤                           ├─────────────────┤
│ id (PK)                 │                           │ id (PK)         │
│ org_id (FK)             │                           │ org_id (FK)     │
│ control_id (FK)         │                           │ control_id (FK) │
│ title                   │                           │ title           │
│ description             │                           │ description     │
│ file_path               │                           │ category        │
│ file_type               │                           │ probability     │
│ file_size               │                           │ impact          │
│ valid_from              │                           │ inherent_risk   │
│ valid_until             │                           │ residual_risk   │
│ uploaded_by (FK)        │                           │ status          │
│ created_at              │                           │ owner_id (FK)   │
└─────────────────────────┘                           │ treatment_plan  │
                                                      │ due_date        │
                                                      │ created_at      │
                                                      │ updated_at      │
                                                      └────────┬────────┘
                                                               │
                                                               ▼
                                                      ┌─────────────────┐
                                                      │ risk_treatments │
                                                      ├─────────────────┤
                                                      │ id (PK)         │
                                                      │ risk_id (FK)    │
                                                      │ action          │
                                                      │ responsible (FK)│
                                                      │ due_date        │
                                                      │ status          │
                                                      │ completed_at    │
                                                      └─────────────────┘

┌─────────────────────────┐         ┌─────────────────────────┐
│        tasks            │         │     notifications       │
├─────────────────────────┤         ├─────────────────────────┤
│ id (PK)                 │         │ id (PK)                 │
│ org_id (FK)             │         │ user_id (FK)            │
│ title                   │         │ title                   │
│ description             │         │ message                 │
│ type                    │         │ type                    │
│ priority                │         │ related_type            │
│ status                  │         │ related_id              │
│ assignee_id (FK)        │         │ is_read                 │
│ created_by (FK)         │         │ created_at              │
│ due_date                │         └─────────────────────────┘
│ related_type            │
│ related_id              │         ┌─────────────────────────┐
│ completed_at            │         │     audit_logs          │
│ created_at              │         ├─────────────────────────┤
│ updated_at              │         │ id (PK)                 │
└─────────────────────────┘         │ org_id (FK)             │
                                    │ user_id (FK)            │
                                    │ action                  │
                                    │ entity_type             │
                                    │ entity_id               │
                                    │ old_values              │
                                    │ new_values              │
                                    │ ip_address              │
                                    │ created_at              │
                                    └─────────────────────────┘
```

### 4.2 Descrição das Tabelas

#### 4.2.1 Tabela: `organizations`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | Identificador único (PK) |
| name | VARCHAR(255) | Nome da organização |
| cnpj | VARCHAR(18) | CNPJ formatado |
| industry | VARCHAR(100) | Setor de atuação |
| logo_url | TEXT | URL do logo |
| settings | JSONB | Configurações personalizadas |
| created_at | TIMESTAMP | Data de criação |
| updated_at | TIMESTAMP | Data de atualização |

#### 4.2.2 Tabela: `users`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | Referência auth.users (PK) |
| org_id | UUID | Organização (FK) |
| email | VARCHAR(255) | Email único |
| full_name | VARCHAR(255) | Nome completo |
| role | ENUM | admin, compliance_officer, manager, user |
| department_id | UUID | Departamento (FK) |
| avatar_url | TEXT | URL do avatar |
| is_active | BOOLEAN | Usuário ativo |
| last_login | TIMESTAMP | Último acesso |
| created_at | TIMESTAMP | Data de criação |

#### 4.2.3 Tabela: `frameworks`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | Identificador único (PK) |
| name | VARCHAR(100) | Nome (LGPD, ISO 27001, etc.) |
| version | VARCHAR(50) | Versão do framework |
| description | TEXT | Descrição |
| icon | VARCHAR(50) | Ícone identificador |
| total_requirements | INTEGER | Total de requisitos |
| is_active | BOOLEAN | Framework ativo |
| created_at | TIMESTAMP | Data de criação |

#### 4.2.4 Tabela: `requirements`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | Identificador único (PK) |
| framework_id | UUID | Framework (FK) |
| code | VARCHAR(50) | Código do requisito |
| title | VARCHAR(255) | Título |
| description | TEXT | Descrição completa |
| category | VARCHAR(100) | Categoria/Seção |
| guidance | TEXT | Orientação de implementação |
| order_index | INTEGER | Ordem de exibição |

#### 4.2.5 Tabela: `controls`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | Identificador único (PK) |
| org_id | UUID | Organização (FK) |
| requirement_id | UUID | Requisito (FK) |
| name | VARCHAR(255) | Nome do controle |
| description | TEXT | Descrição |
| status | ENUM | not_started, in_progress, implemented, not_applicable |
| owner_id | UUID | Responsável (FK) |
| implementation_details | TEXT | Detalhes da implementação |
| test_frequency | ENUM | monthly, quarterly, annually, as_needed |
| last_tested_at | TIMESTAMP | Última verificação |
| next_test_date | DATE | Próxima verificação |
| created_at | TIMESTAMP | Data de criação |
| updated_at | TIMESTAMP | Data de atualização |

#### 4.2.6 Tabela: `policies`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | Identificador único (PK) |
| org_id | UUID | Organização (FK) |
| title | VARCHAR(255) | Título da política |
| content | TEXT | Conteúdo (Markdown/HTML) |
| version | VARCHAR(20) | Versão (1.0, 1.1, etc.) |
| status | ENUM | draft, pending_approval, approved, archived |
| category | VARCHAR(100) | Categoria |
| owner_id | UUID | Responsável (FK) |
| approved_by | UUID | Aprovador (FK) |
| approved_at | TIMESTAMP | Data de aprovação |
| effective_date | DATE | Data de vigência |
| review_date | DATE | Data de revisão |
| requires_acceptance | BOOLEAN | Requer aceite |
| file_path | TEXT | Arquivo anexo |
| created_at | TIMESTAMP | Data de criação |
| updated_at | TIMESTAMP | Data de atualização |

#### 4.2.7 Tabela: `policy_versions`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | Identificador único (PK) |
| policy_id | UUID | Política (FK) |
| version | VARCHAR(20) | Número da versão |
| content | TEXT | Conteúdo desta versão |
| changed_by | UUID | Alterado por (FK) |
| change_summary | TEXT | Resumo das mudanças |
| created_at | TIMESTAMP | Data de criação |

#### 4.2.8 Tabela: `policy_acceptances`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | Identificador único (PK) |
| policy_id | UUID | Política (FK) |
| user_id | UUID | Usuário (FK) |
| policy_version | VARCHAR(20) | Versão aceita |
| accepted_at | TIMESTAMP | Data do aceite |
| ip_address | VARCHAR(45) | IP do aceite |

#### 4.2.9 Tabela: `risks`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | Identificador único (PK) |
| org_id | UUID | Organização (FK) |
| control_id | UUID | Controle relacionado (FK, opcional) |
| title | VARCHAR(255) | Título do risco |
| description | TEXT | Descrição |
| category | VARCHAR(100) | Categoria |
| probability | INTEGER | Probabilidade (1-5) |
| impact | INTEGER | Impacto (1-5) |
| inherent_risk | INTEGER | Risco inerente (calculado) |
| residual_risk | INTEGER | Risco residual |
| status | ENUM | identified, analyzing, treating, accepted, closed |
| owner_id | UUID | Responsável (FK) |
| treatment_plan | TEXT | Plano de tratamento |
| due_date | DATE | Data limite |
| created_at | TIMESTAMP | Data de criação |
| updated_at | TIMESTAMP | Data de atualização |

#### 4.2.10 Tabela: `evidences`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | Identificador único (PK) |
| org_id | UUID | Organização (FK) |
| control_id | UUID | Controle (FK) |
| title | VARCHAR(255) | Título |
| description | TEXT | Descrição |
| file_path | TEXT | Caminho no Storage |
| file_name | VARCHAR(255) | Nome original do arquivo |
| file_type | VARCHAR(100) | MIME type |
| file_size | INTEGER | Tamanho em bytes |
| valid_from | DATE | Válido a partir de |
| valid_until | DATE | Válido até |
| uploaded_by | UUID | Enviado por (FK) |
| tags | TEXT[] | Tags para busca |
| created_at | TIMESTAMP | Data de criação |

#### 4.2.11 Tabela: `tasks`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | Identificador único (PK) |
| org_id | UUID | Organização (FK) |
| title | VARCHAR(255) | Título |
| description | TEXT | Descrição |
| type | ENUM | control_implementation, evidence_collection, policy_review, risk_treatment, general |
| priority | ENUM | low, medium, high, critical |
| status | ENUM | pending, in_progress, completed, cancelled |
| assignee_id | UUID | Responsável (FK) |
| created_by | UUID | Criado por (FK) |
| due_date | DATE | Data limite |
| related_type | VARCHAR(50) | Tipo relacionado (policy, risk, control, etc.) |
| related_id | UUID | ID do item relacionado |
| completed_at | TIMESTAMP | Data de conclusão |
| created_at | TIMESTAMP | Data de criação |
| updated_at | TIMESTAMP | Data de atualização |

#### 4.2.12 Tabela: `audit_logs`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | Identificador único (PK) |
| org_id | UUID | Organização (FK) |
| user_id | UUID | Usuário (FK) |
| action | VARCHAR(50) | Ação (create, update, delete, view) |
| entity_type | VARCHAR(50) | Tipo de entidade |
| entity_id | UUID | ID da entidade |
| old_values | JSONB | Valores anteriores |
| new_values | JSONB | Novos valores |
| ip_address | VARCHAR(45) | Endereço IP |
| user_agent | TEXT | User agent |
| created_at | TIMESTAMP | Data da ação |

#### 4.2.13 Tabela: `notifications`
| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | UUID | Identificador único (PK) |
| user_id | UUID | Usuário (FK) |
| title | VARCHAR(255) | Título |
| message | TEXT | Mensagem |
| type | ENUM | info, warning, alert, success |
| related_type | VARCHAR(50) | Tipo relacionado |
| related_id | UUID | ID relacionado |
| is_read | BOOLEAN | Lida |
| read_at | TIMESTAMP | Data de leitura |
| created_at | TIMESTAMP | Data de criação |

### 4.3 Índices Recomendados

```sql
-- Índices de busca frequente
CREATE INDEX idx_controls_org_status ON controls(org_id, status);
CREATE INDEX idx_controls_owner ON controls(owner_id);
CREATE INDEX idx_policies_org_status ON policies(org_id, status);
CREATE INDEX idx_risks_org_status ON risks(org_id, status);
CREATE INDEX idx_tasks_assignee_status ON tasks(assignee_id, status);
CREATE INDEX idx_tasks_due_date ON tasks(due_date) WHERE status != 'completed';
CREATE INDEX idx_evidences_control ON evidences(control_id);
CREATE INDEX idx_evidences_expiring ON evidences(valid_until) WHERE valid_until IS NOT NULL;
CREATE INDEX idx_notifications_user_unread ON notifications(user_id, is_read) WHERE is_read = false;
CREATE INDEX idx_audit_logs_entity ON audit_logs(entity_type, entity_id);
CREATE INDEX idx_audit_logs_user ON audit_logs(user_id, created_at);
```

### 4.4 Views Úteis

```sql
-- View: Resumo de compliance por framework
CREATE VIEW vw_compliance_summary AS
SELECT 
    o.id as org_id,
    f.id as framework_id,
    f.name as framework_name,
    COUNT(c.id) as total_controls,
    COUNT(c.id) FILTER (WHERE c.status = 'implemented') as implemented,
    COUNT(c.id) FILTER (WHERE c.status = 'in_progress') as in_progress,
    COUNT(c.id) FILTER (WHERE c.status = 'not_started') as not_started,
    ROUND(
        COUNT(c.id) FILTER (WHERE c.status = 'implemented')::numeric / 
        NULLIF(COUNT(c.id) FILTER (WHERE c.status != 'not_applicable'), 0) * 100, 2
    ) as compliance_percentage
FROM organizations o
CROSS JOIN frameworks f
LEFT JOIN requirements r ON r.framework_id = f.id
LEFT JOIN controls c ON c.requirement_id = r.id AND c.org_id = o.id
GROUP BY o.id, f.id, f.name;

-- View: Riscos por nível
CREATE VIEW vw_risk_matrix AS
SELECT 
    org_id,
    COUNT(id) FILTER (WHERE inherent_risk >= 20) as critical_risks,
    COUNT(id) FILTER (WHERE inherent_risk >= 12 AND inherent_risk < 20) as high_risks,
    COUNT(id) FILTER (WHERE inherent_risk >= 6 AND inherent_risk < 12) as medium_risks,
    COUNT(id) FILTER (WHERE inherent_risk < 6) as low_risks,
    COUNT(id) as total_risks
FROM risks
WHERE status NOT IN ('closed', 'accepted')
GROUP BY org_id;

-- View: Tarefas pendentes com detalhes
CREATE VIEW vw_pending_tasks AS
SELECT 
    t.*,
    u.full_name as assignee_name,
    CASE 
        WHEN t.due_date < CURRENT_DATE THEN 'overdue'
        WHEN t.due_date = CURRENT_DATE THEN 'due_today'
        WHEN t.due_date <= CURRENT_DATE + INTERVAL '7 days' THEN 'due_soon'
        ELSE 'on_track'
    END as urgency
FROM tasks t
LEFT JOIN users u ON u.id = t.assignee_id
WHERE t.status IN ('pending', 'in_progress');
```

---

## 5. Autenticação e Autorização

### 5.1 Fluxo de Autenticação

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         FLUXO DE AUTENTICAÇÃO                                │
└─────────────────────────────────────────────────────────────────────────────┘

┌──────────┐         ┌──────────┐         ┌──────────┐         ┌──────────┐
│  Login   │         │  Vercel  │         │ Supabase │         │   App    │
│   Page   │         │  Server  │         │   Auth   │         │  (Home)  │
└────┬─────┘         └────┬─────┘         └────┬─────┘         └────┬─────┘
     │                    │                    │                    │
     │  1. Email/Senha    │                    │                    │
     │ ─────────────────> │                    │                    │
     │                    │  2. signIn()       │                    │
     │                    │ ─────────────────> │                    │
     │                    │                    │                    │
     │                    │  3. JWT + Session  │                    │
     │                    │ <───────────────── │                    │
     │                    │                    │                    │
     │  4. Set Cookies    │                    │                    │
     │ <───────────────── │                    │                    │
     │                    │                    │                    │
     │  5. Redirect       │                    │                    │
     │ ──────────────────────────────────────────────────────────> │
     │                    │                    │                    │
```

### 5.2 Níveis de Acesso (RBAC)

| Role | Descrição | Permissões |
|------|-----------|------------|
| **admin** | Administrador | Acesso total, gestão de usuários |
| **compliance_officer** | Oficial de Compliance | CRUD em tudo exceto usuários/config |
| **manager** | Gestor de Área | CRUD na sua área, visualiza tudo |
| **user** | Colaborador | Visualização, aceite de políticas, tarefas próprias |

### 5.3 Matriz de Permissões

| Recurso | Admin | Compliance Officer | Manager | User |
|---------|-------|-------------------|---------|------|
| **Dashboard** | ✅ Total | ✅ Total | ✅ Área | ✅ Básico |
| **Políticas** - Ver | ✅ | ✅ | ✅ | ✅ |
| **Políticas** - Criar | ✅ | ✅ | ❌ | ❌ |
| **Políticas** - Editar | ✅ | ✅ | ❌ | ❌ |
| **Políticas** - Aprovar | ✅ | ✅ | ❌ | ❌ |
| **Riscos** - Ver | ✅ | ✅ | ✅ Área | ❌ |
| **Riscos** - CRUD | ✅ | ✅ | ✅ Área | ❌ |
| **Controles** - Ver | ✅ | ✅ | ✅ Área | ❌ |
| **Controles** - CRUD | ✅ | ✅ | ✅ Área | ❌ |
| **Evidências** - Ver | ✅ | ✅ | ✅ Área | ❌ |
| **Evidências** - Upload | ✅ | ✅ | ✅ Área | ❌ |
| **Tarefas** - Ver próprias | ✅ | ✅ | ✅ | ✅ |
| **Tarefas** - Criar | ✅ | ✅ | ✅ | ❌ |
| **Relatórios** | ✅ | ✅ | ✅ Área | ❌ |
| **Configurações** | ✅ | ❌ | ❌ | ❌ |
| **Usuários** | ✅ | ❌ | ❌ | ❌ |

### 5.4 Row Level Security (RLS)

```sql
-- Exemplo de política RLS para controles
CREATE POLICY "Users can view controls of their organization"
ON controls FOR SELECT
USING (
    org_id = (SELECT org_id FROM users WHERE id = auth.uid())
);

CREATE POLICY "Managers can edit controls of their department"
ON controls FOR UPDATE
USING (
    org_id = (SELECT org_id FROM users WHERE id = auth.uid())
    AND (
        (SELECT role FROM users WHERE id = auth.uid()) IN ('admin', 'compliance_officer')
        OR owner_id = auth.uid()
    )
);
```

---

## 6. Storage de Arquivos

### 6.1 Estrutura de Buckets

```
Supabase Storage
│
├── 📁 evidences/                    # Evidências de compliance
│   └── {org_id}/
│       └── {control_id}/
│           └── {timestamp}_{filename}
│
├── 📁 policies/                     # Documentos de políticas
│   └── {org_id}/
│       └── {policy_id}/
│           └── v{version}_{filename}
│
├── 📁 attachments/                  # Anexos gerais
│   └── {org_id}/
│       └── {entity_type}/
│           └── {entity_id}/
│               └── {filename}
│
├── 📁 exports/                      # Relatórios exportados
│   └── {org_id}/
│       └── {year}/{month}/
│           └── {report_type}_{timestamp}.pdf
│
└── 📁 avatars/                      # Fotos de perfil
    └── {user_id}.{ext}
```

### 6.2 Políticas de Storage

| Bucket | Acesso Público | Tamanho Máximo | Tipos Permitidos |
|--------|----------------|----------------|------------------|
| evidences | ❌ Privado | 50MB | pdf, doc, docx, xls, xlsx, png, jpg |
| policies | ❌ Privado | 20MB | pdf, doc, docx |
| attachments | ❌ Privado | 10MB | pdf, doc, docx, png, jpg, txt |
| exports | ❌ Privado | 100MB | pdf, xlsx, csv |
| avatars | ✅ Público | 2MB | png, jpg, webp |

---

## 7. APIs e Endpoints

### 7.1 Estrutura de Endpoints

```
API Routes (Next.js)
│
├── /api/auth/
│   ├── POST   /callback          # Callback do Supabase Auth
│   └── POST   /signout           # Logout
│
├── /api/policies/
│   ├── GET    /                  # Listar políticas
│   ├── POST   /                  # Criar política
│   ├── GET    /:id               # Obter política
│   ├── PUT    /:id               # Atualizar política
│   ├── DELETE /:id               # Deletar política
│   ├── POST   /:id/approve       # Aprovar política
│   ├── GET    /:id/versions      # Histórico de versões
│   └── POST   /:id/accept        # Registrar aceite
│
├── /api/risks/
│   ├── GET    /                  # Listar riscos
│   ├── POST   /                  # Criar risco
│   ├── GET    /:id               # Obter risco
│   ├── PUT    /:id               # Atualizar risco
│   ├── DELETE /:id               # Deletar risco
│   ├── GET    /matrix            # Matriz de riscos
│   └── POST   /:id/treatment     # Adicionar tratamento
│
├── /api/controls/
│   ├── GET    /                  # Listar controles
│   ├── POST   /                  # Criar controle
│   ├── GET    /:id               # Obter controle
│   ├── PUT    /:id               # Atualizar controle
│   ├── DELETE /:id               # Deletar controle
│   ├── POST   /:id/test          # Registrar teste
│   └── GET    /by-framework/:id  # Controles por framework
│
├── /api/evidences/
│   ├── GET    /                  # Listar evidências
│   ├── POST   /                  # Upload de evidência
│   ├── GET    /:id               # Obter evidência
│   ├── PUT    /:id               # Atualizar evidência
│   ├── DELETE /:id               # Deletar evidência
│   ├── GET    /:id/download      # Download do arquivo
│   └── GET    /expiring          # Evidências vencendo
│
├── /api/tasks/
│   ├── GET    /                  # Listar tarefas
│   ├── POST   /                  # Criar tarefa
│   ├── GET    /:id               # Obter tarefa
│   ├── PUT    /:id               # Atualizar tarefa
│   ├── DELETE /:id               # Deletar tarefa
│   └── PUT    /:id/status        # Atualizar status
│
├── /api/reports/
│   ├── GET    /compliance        # Relatório de compliance
│   ├── GET    /risks             # Relatório de riscos
│   ├── GET    /controls          # Relatório de controles
│   ├── POST   /export            # Exportar relatório
│   └── GET    /dashboard         # Dados do dashboard
│
├── /api/users/
│   ├── GET    /                  # Listar usuários
│   ├── POST   /                  # Criar usuário
│   ├── GET    /:id               # Obter usuário
│   ├── PUT    /:id               # Atualizar usuário
│   ├── DELETE /:id               # Desativar usuário
│   └── GET    /me                # Usuário atual
│
├── /api/frameworks/
│   ├── GET    /                  # Listar frameworks
│   └── GET    /:id/requirements  # Requisitos do framework
│
└── /api/notifications/
    ├── GET    /                  # Listar notificações
    ├── PUT    /:id/read          # Marcar como lida
    └── PUT    /read-all          # Marcar todas como lidas
```

### 7.2 Padrão de Resposta

```typescript
// Resposta de sucesso
{
  "success": true,
  "data": { ... },
  "meta": {
    "page": 1,
    "per_page": 20,
    "total": 100,
    "total_pages": 5
  }
}

// Resposta de erro
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Dados inválidos",
    "details": [
      { "field": "title", "message": "Campo obrigatório" }
    ]
  }
}
```

---

## 8. Configurações e Variáveis de Ambiente

### 8.1 Variáveis Necessárias

```env
# .env.local

# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_APP_NAME=APOC - Plataforma de Compliance

# Email (opcional - para notificações)
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=user@example.com
SMTP_PASSWORD=password
SMTP_FROM=noreply@example.com

# Outros
NODE_ENV=development
```

### 8.2 Variáveis no Vercel

| Variável | Ambiente | Descrição |
|----------|----------|-----------|
| NEXT_PUBLIC_SUPABASE_URL | Production, Preview | URL do projeto Supabase |
| NEXT_PUBLIC_SUPABASE_ANON_KEY | Production, Preview | Chave pública |
| SUPABASE_SERVICE_ROLE_KEY | Production, Preview | Chave de serviço (secret) |
| NEXT_PUBLIC_APP_URL | Production | URL de produção |

---

## 9. Deploy e CI/CD

### 9.1 Fluxo de Deploy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           FLUXO DE CI/CD                                     │
└─────────────────────────────────────────────────────────────────────────────┘

┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  Local   │     │  GitHub  │     │  Vercel  │     │Production│
│   Dev    │     │  (main)  │     │  Build   │     │   Live   │
└────┬─────┘     └────┬─────┘     └────┬─────┘     └────┬─────┘
     │                │                │                │
     │  1. git push   │                │                │
     │ ─────────────> │                │                │
     │                │  2. Webhook    │                │
     │                │ ─────────────> │                │
     │                │                │                │
     │                │                │  3. Build      │
     │                │                │  Next.js       │
     │                │                │                │
     │                │                │  4. Deploy     │
     │                │                │ ─────────────> │
     │                │                │                │
     │                │  5. Status     │                │
     │                │ <───────────── │                │
     │                │                │                │
```

### 9.2 Branches e Ambientes

| Branch | Ambiente | URL |
|--------|----------|-----|
| main | Production | apoc.vercel.app |
| develop | Preview | apoc-develop.vercel.app |
| feature/* | Preview | apoc-feature-*.vercel.app |

### 9.3 Checklist de Deploy

```
□ Variáveis de ambiente configuradas no Vercel
□ Supabase configurado (tabelas, RLS, storage)
□ Domínio personalizado (opcional)
□ SSL automático (Vercel)
□ Migrations executadas
□ Seeds iniciais aplicados
□ Testes passando
□ Build sem erros
```

---

## 10. Próximos Passos

### 10.1 Ordem de Implementação

```
1. Setup do Projeto
   □ Criar projeto Next.js
   □ Configurar Supabase
   □ Setup de autenticação
   □ Estrutura de pastas

2. Infraestrutura
   □ Schema do banco de dados
   □ Row Level Security
   □ Storage buckets
   □ Seed de frameworks (LGPD, ISO 27001)

3. Core Features (MVP)
   □ Autenticação e perfis
   □ Dashboard básico
   □ CRUD de Políticas
   □ CRUD de Riscos
   □ CRUD de Controles
   □ Upload de Evidências
   □ Sistema de Tarefas

4. Polish
   □ Relatórios básicos
   □ Notificações
   □ Responsividade
   □ Testes

5. Deploy
   □ Deploy na Vercel
   □ Configurar domínio
   □ Monitoramento
```

---

**Documento criado em**: Janeiro/2026  
**Versão**: 1.0  
**Stack**: Next.js 14 + Supabase + Vercel

---

> 💡 Este documento serve como guia técnico para implementação. Revise e ajuste conforme necessidades específicas do projeto.

