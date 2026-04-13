# 📐 Software Design Document (SDD)

**Projeto:** Rastreabilidade de Grãos
**Versão:** 1.0.0
**Status:** 🟡 Em Definição (MVP)

---

## 🏗️ 1. Arquitetura Geral

```
rastreabilidade-de-graos/   ← Monorepo
├── backend/                ← NestJS API (porta 3000)
│   ├── src/
│   │   ├── auth/           ← Módulo JWT (login, register, guards)
│   │   ├── users/          ← Módulo de usuários
│   │   ├── lotes/          ← Módulo de lotes (entidade principal)
│   │   ├── movimentacoes/  ← Módulo de movimentações (1:N com lotes)
│   │   ├── prisma/         ← PrismaService singleton
│   │   └── main.ts         ← Bootstrap + Swagger + Pipes globais
│   ├── prisma/
│   │   └── schema.prisma
│   └── test/               ← Testes e2e
└── frontend/               ← Angular App (porta 4200)
    └── src/app/
        ├── core/           ← AuthService, interceptors, guards
        ├── features/       ← Pages (login, dashboard, lotes, movimentacoes)
        └── shared/         ← Componentes reutilizáveis
```

---

## 🗄️ 2. Modelo de Dados

### 2.1. Dicionário de Entidades

| Entidade | Tabela | Descrição |
|:---------|:-------|:----------|
| Usuário | `users` | Produtores e operadores autenticados |
| Lote | `lotes` | Unidade de produção rastreável |
| Movimentação | `movimentacoes` | Eventos do lote (1:N com lotes) |

#### Tabela `users`
| Campo | Tipo | Restrições |
|:------|:-----|:-----------|
| `id` | UUID | PK, auto |
| `name` | String | NOT NULL |
| `email` | String | UNIQUE, NOT NULL |
| `password` | String | bcrypt hash |
| `role` | Enum | `PRODUTOR` \| `OPERADOR` |
| `createdAt` | DateTime | DEFAULT now() |

#### Tabela `lotes`
| Campo | Tipo | Restrições |
|:------|:-----|:-----------|
| `id` | UUID | PK, auto |
| `cultura` | String | NOT NULL (soja, milho, trigo...) |
| `safra` | String | NOT NULL (ex: 2024/2025) |
| `pesoKg` | Decimal | NOT NULL, > 0 |
| `propriedade` | String | NOT NULL |
| `status` | Enum | `ATIVO` \| `ARQUIVADO` |
| `produtorId` | UUID | FK → users.id |
| `createdAt` | DateTime | DEFAULT now() |

#### Tabela `movimentacoes`
| Campo | Tipo | Restrições |
|:------|:-----|:-----------|
| `id` | UUID | PK, auto |
| `tipo` | Enum | `ENTRADA` \| `SAIDA` \| `TRANSPORTE` \| `VENDA` \| `ANALISE` |
| `destino` | String | NOT NULL |
| `data` | DateTime | NOT NULL |
| `observacoes` | String | nullable |
| `umidade` | Decimal | nullable (obrigatório se tipo=ANALISE) |
| `impureza` | Decimal | nullable (obrigatório se tipo=ANALISE) |
| `loteId` | UUID | FK → lotes.id |
| `operadorId` | UUID | FK → users.id |
| `createdAt` | DateTime | DEFAULT now() |

### 2.2. Diagrama ER (Mermaid)

```mermaid
erDiagram
    USERS {
        uuid id PK
        string name
        string email
        string password
        enum role
        datetime createdAt
    }

    LOTES {
        uuid id PK
        string cultura
        string safra
        decimal pesoKg
        string propriedade
        enum status
        uuid produtorId FK
        datetime createdAt
    }

    MOVIMENTACOES {
        uuid id PK
        enum tipo
        string destino
        datetime data
        string observacoes
        decimal umidade
        decimal impureza
        uuid loteId FK
        uuid operadorId FK
        datetime createdAt
    }

    USERS ||--o{ LOTES : "produtor cadastra"
    USERS ||--o{ MOVIMENTACOES : "operador registra"
    LOTES ||--o{ MOVIMENTACOES : "lote possui (1:N)"
```

---

## 📡 3. Contratos da API (Endpoints)

**Base URL:** `https://api.rastreabilidade-graos.com/api/v1`
**Auth:** Bearer Token (JWT) no header `Authorization`

### 3.1. Auth

| Método | Rota | Auth | Descrição |
|:-------|:-----|:-----|:----------|
| POST | `/auth/register` | Público | Cadastrar novo usuário |
| POST | `/auth/login` | Público | Login → retorna JWT |
| GET | `/auth/me` | JWT | Perfil do usuário logado |

**POST /auth/register — Body:**
```json
{
  "name": "Pedro Zampier",
  "email": "pedro@email.com",
  "password": "Senha@123",
  "role": "PRODUTOR"
}
```

**POST /auth/login — Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "user": { "id": "uuid", "name": "Pedro", "role": "PRODUTOR" }
}
```

### 3.2. Lotes

| Método | Rota | Auth | Role | Descrição |
|:-------|:-----|:-----|:-----|:----------|
| POST | `/lotes` | JWT | PRODUTOR | Criar lote |
| GET | `/lotes` | JWT | Todos | Listar lotes (paginado) |
| GET | `/lotes/:id` | JWT | Todos | Detalhe do lote + movimentações |
| PATCH | `/lotes/:id` | JWT | PRODUTOR (dono) | Editar lote (sem movimentações) |
| PATCH | `/lotes/:id/arquivar` | JWT | PRODUTOR (dono) | Arquivar lote |

**POST /lotes — Body:**
```json
{
  "cultura": "soja",
  "safra": "2024/2025",
  "pesoKg": 25000.50,
  "propriedade": "Fazenda São João"
}
```

### 3.3. Movimentações

| Método | Rota | Auth | Role | Descrição |
|:-------|:-----|:-----|:-----|:----------|
| POST | `/lotes/:loteId/movimentacoes` | JWT | OPERADOR ou PRODUTOR | Registrar movimentação |
| GET | `/lotes/:loteId/movimentacoes` | JWT | Todos | Listar movimentações do lote |
| GET | `/movimentacoes` | JWT | PRODUTOR | Todas as movimentações (com filtros) |

**POST /lotes/:loteId/movimentacoes — Body:**
```json
{
  "tipo": "ANALISE",
  "destino": "Laboratório Agro SP",
  "data": "2025-03-15T10:00:00Z",
  "observacoes": "Análise pré-colheita",
  "umidade": 13.5,
  "impureza": 0.8
}
```

---

## 🔐 4. Segurança e Autenticação

### JWT Strategy
- Algoritmo: `HS256`
- Expiração: `24h`
- Payload: `{ sub: userId, email, role }`

### Guards NestJS
| Guard | Proteção |
|:------|:---------|
| `JwtAuthGuard` | Valida token em todas as rotas privadas |
| `RolesGuard` | Verifica papel (`@Roles(Role.PRODUTOR)`) |
| `LoteOwnerGuard` | Verifica se o produtor é dono do lote |

---

## 🧩 5. DTOs Principais

```typescript
// CreateLoteDto
export class CreateLoteDto {
  @IsString() @IsNotEmpty() cultura: string;
  @IsString() @IsNotEmpty() safra: string;
  @IsNumber() @Min(0.01) pesoKg: number;
  @IsString() @IsNotEmpty() propriedade: string;
}

// CreateMovimentacaoDto
export class CreateMovimentacaoDto {
  @IsEnum(TipoMovimentacao) tipo: TipoMovimentacao;
  @IsString() @IsNotEmpty() destino: string;
  @IsDateString() data: string;
  @IsString() @IsOptional() observacoes?: string;
  @IsNumber() @IsOptional() umidade?: number;
  @IsNumber() @IsOptional() impureza?: number;
}

// RegisterDto
export class RegisterDto {
  @IsString() @IsNotEmpty() name: string;
  @IsEmail() email: string;
  @IsString() @MinLength(8) password: string;
  @IsEnum(Role) role: Role;
}
```

---

## ⚙️ 6. Variáveis de Ambiente

```env
# backend/.env (nunca comitar — listado no .gitignore)
DATABASE_URL="postgresql://user:pass@host/db?schema=public"
JWT_SECRET="seu-segredo-super-seguro"
JWT_EXPIRES_IN="24h"
PORT=3000
```

---

## 🎨 7. Design Tokens (Tailwind — Frontend)

```javascript
// tailwind.config.js
theme: {
  extend: {
    colors: {
      primary:    "#2D6A4F",  // verde campo
      accent:     "#F4A261",  // laranja colheita
      background: "#F9F7F0",  // bege claro
      surface:    "#FFFFFF",
      text:       "#1B1B1B",
      success:    "#52B788",
      danger:     "#E63946",
      muted:      "#9CA3AF",
    },
    fontFamily: {
      sans: ["Inter", "sans-serif"],
    },
    borderRadius: {
      DEFAULT: "0.75rem",
    }
  }
}
```

---

## 🚦 8. Mapa de Rotas Frontend (Angular)

| Rota | Componente | Guard | Descrição |
|:-----|:-----------|:------|:----------|
| `/login` | `LoginPage` | Público | Formulário de login |
| `/register` | `RegisterPage` | Público | Cadastro de usuário |
| `/dashboard` | `DashboardPage` | `AuthGuard` | Painel com resumo dos lotes |
| `/lotes` | `LotesListPage` | `AuthGuard` | Listagem paginada dos lotes |
| `/lotes/new` | `LoteFormPage` | `AuthGuard` + `RoleGuard(PRODUTOR)` | Cadastrar novo lote |
| `/lotes/:id` | `LoteDetailPage` | `AuthGuard` | Detalhe + histórico de movimentações |
| `/lotes/:id/movimentacao` | `MovimentacaoFormPage` | `AuthGuard` | Registrar nova movimentação |
