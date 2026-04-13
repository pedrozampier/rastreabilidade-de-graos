# 🌾 Rastreabilidade de Grãos

> Plataforma web para rastrear lotes de produção agrícola e suas movimentações ao longo da cadeia produtiva — do campo ao destino final.

## 👤 Autor

- Pedro Zampier ([@pedrozampier](https://github.com/pedrozampier))

## 🚀 Stack Tecnológica

| Camada | Tecnologia |
|:-------|:-----------|
| **Backend** | NestJS + Prisma ORM + PostgreSQL |
| **Frontend** | Angular 17+ (Standalone + Signals) |
| **Auth** | JWT (JSON Web Tokens) + Guards |
| **Testes** | Jest (TDD) |
| **CI/CD** | GitHub Actions |
| **Deploy** | Render (API) + Neon.tech (DB) + Vercel (Frontend) |

## 🔗 Documentação

- 📄 [PRD — Product Requirements Document](./docs/prd.md)
- 📐 [SDD — Software Design Document](./docs/sdd.md)
- ✅ [Checklist de Indicadores de Desempenho](./docs/checklist.md)

## 🌐 Links de Produção

- **API:** _Em breve_
- **Frontend:** _Em breve_
- **Swagger:** _Em breve_

---

## ⚡ Quick Start

### Pré-requisitos
- Node.js 20+
- PostgreSQL (local ou Neon.tech)

### Backend

```bash
cd backend
npm install
cp .env.example .env
npx prisma migrate dev
npm run start:dev
```

### Frontend

```bash
cd frontend
npm install
ng serve
```

### Testes

```bash
cd backend
npm run test
npm run test:e2e
```

## 🌿 GitFlow

```
main       ← produção (PR obrigatório, Ruleset ativo)
develop    ← integração (PR obrigatório, Ruleset ativo)
feature/*  ← novas funcionalidades
fix/*      ← correções
```
