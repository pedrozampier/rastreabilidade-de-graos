# 📄 Product Requirements Document (PRD)

**Projeto:** Rastreabilidade de Grãos
**Versão:** 1.0.0
**Status:** 🟡 Em Definição (MVP)

---

## 🎯 1. Visão Geral e Objetivo

O agronegócio brasileiro movimenta bilhões de toneladas de grãos por ano, porém a rastreabilidade das informações entre produção, armazenagem, transporte e comercialização ainda é feita de forma manual e fragmentada — gerando perdas, fraudes e falta de transparência.

O **Rastreabilidade de Grãos** é uma plataforma web que permite que produtores rurais e operadores cadastrem lotes de produção e registrem cada movimentação ao longo da cadeia: entrada em silo, saída para transporte, venda, controle de qualidade e destino final.

**Objetivo principal:** garantir rastreabilidade completa e auditável de lotes de grãos, do campo ao destino, com acesso seguro via autenticação JWT.

---

## 📖 2. Glossário Ubíquo

| Termo | Definição |
|:------|:----------|
| **Lote** | Unidade de produção identificada por cultura, safra, peso e origem (propriedade rural). |
| **Movimentação** | Evento que altera o estado ou localização de um lote (entrada em armazém, saída, venda, transporte, análise). |
| **Produtor** | Usuário administrador dono dos lotes. Responsável pelo cadastro e gestão. |
| **Operador** | Usuário que registra movimentações, sem permissão para editar lotes. |
| **Cultura** | Tipo de grão: soja, milho, trigo, sorgo, etc. |
| **Safra** | Identificador do ciclo agrícola (ex: 2024/2025). |
| **Destino** | Local de chegada de uma movimentação (silo, transportadora, compradora). |

---

## 👤 3. Atores e Permissões

| Ator | Descrição | Permissões |
|:-----|:----------|:-----------|
| **Produtor (Admin)** | Proprietário dos lotes | CRUD completo de lotes; visualizar todas as movimentações; gerenciar operadores |
| **Operador** | Funcionário de campo/armazém | Registrar e listar movimentações; visualizar lotes (somente leitura) |
| **Visitante** | Usuário não autenticado | Acesso apenas às rotas públicas (login, cadastro) |

---

## 📝 4. Escopo Funcional (User Stories)

### Autenticação (cobre ID8)
- Como **visitante**, eu quero me cadastrar com nome, e-mail, senha e papel (produtor/operador) para que eu possa acessar a plataforma.
- Como **visitante**, eu quero fazer login e receber um token JWT para que eu possa acessar rotas protegidas.
- Como **usuário autenticado**, eu quero fazer logout para que minha sessão seja encerrada com segurança.

### Gestão de Lotes (cobre ID5, ID6, ID7)
- Como **produtor**, eu quero cadastrar um novo lote informando cultura, safra, peso (kg) e propriedade de origem para que eu possa iniciar o rastreio.
- Como **produtor**, eu quero listar todos os meus lotes com filtro por cultura e safra para que eu possa ter uma visão geral da produção.
- Como **produtor**, eu quero visualizar o detalhe de um lote específico incluindo todo o histórico de movimentações para que eu possa auditar a rastreabilidade completa.
- Como **produtor**, eu quero editar as informações de um lote (antes da primeira movimentação) para que eu possa corrigir dados de cadastro.
- Como **produtor**, eu quero encerrar/arquivar um lote para que ele saia do fluxo ativo sem ser deletado.

### Movimentações (cobre ID7, relação 1:N)
- Como **operador**, eu quero registrar uma movimentação em um lote informando tipo (entrada, saída, transporte, venda, análise), destino, data e observações para que o histórico fique completo.
- Como **operador**, eu quero listar todas as movimentações de um lote ordenadas por data para que eu possa acompanhar o fluxo cronológico.
- Como **produtor**, eu quero visualizar um relatório de todas as movimentações com filtro por tipo e período para que eu possa gerar relatórios de auditoria.

### Qualidade e Validação (cobre ID6)
- Como **operador**, eu quero registrar uma movimentação do tipo "análise" com campos de umidade (%) e impureza (%) para que a qualidade do grão seja documentada.

### Segurança e Infraestrutura (cobre ID8, ID9, ID15, ID16, ID17)
- Como **produtor**, eu quero que somente eu possa criar e editar meus lotes para que dados de terceiros não sejam alterados.
- Como **operador**, eu quero que minha tentativa de editar um lote retorne erro 403 para que o sistema respeite as permissões de papel.
- Como **desenvolvedor**, eu quero que credenciais sensíveis (DATABASE_URL, JWT_SECRET) sejam lidas via variáveis de ambiente para que não fiquem expostas no repositório.
- Como **desenvolvedor**, eu quero que o pipeline de CI execute os testes automaticamente a cada Pull Request para que código quebrado não chegue à develop.

### Documentação e Frontend (cobre ID12, ID13, ID14)
- Como **desenvolvedor**, eu quero que a API exponha documentação Swagger interativa para que o frontend possa consumir os endpoints com contrato definido.
- Como **usuário**, eu quero uma interface web Angular com telas de login, dashboard de lotes e registro de movimentações para que eu possa usar o sistema sem conhecer a API diretamente.

---

## 🛡️ 5. Regras de Negócio

1. Um lote só pode ser editado **antes** de ter qualquer movimentação registrada.
2. O peso de um lote deve ser **maior que zero**.
3. Movimentações são **imutáveis** após registro — não podem ser editadas ou deletadas (apenas visualizadas).
4. Somente o **produtor dono do lote** pode registrar movimentações nele; operadores de outro produtor são bloqueados.
5. O tipo de movimentação `analise` exige os campos `umidade` e `impureza`; os demais tipos tornam esses campos opcionais.
6. Um lote **arquivado** não aceita novas movimentações.
7. O token JWT expira em **24 horas**; o cliente deve renovar via re-login.
8. Senhas armazenadas com **bcrypt** (min. 10 rounds).

---

## 🚫 6. Fora de Escopo (Non-goals)

- Integração com ERPs ou sistemas de bolsa de grãos.
- Geolocalização em tempo real dos caminhões.
- App mobile nativo.
- Notificações por SMS ou e-mail.
- Relatórios em PDF/Excel nesta versão.

---

## ⚙️ 7. Requisitos Não Funcionais

| Requisito | Detalhe |
|:----------|:--------|
| **Segurança** | JWT obrigatório em todas as rotas privadas; RBAC por papel |
| **Validação** | DTOs com class-validator + ValidationPipe (whitelist: true) |
| **Performance** | Listagens paginadas (page/limit) |
| **Observabilidade** | Logs de erro estruturados via Exception Filters globais |
| **Responsividade** | Frontend mobile-first (Angular + Tailwind) |
| **Auditabilidade** | Movimentações imutáveis garantem trilha de auditoria |

---

## 🛠️ 8. Tech Stack

| Camada | Tecnologia |
|:-------|:-----------|
| Backend | NestJS (módulos, controllers, services, guards, interceptors) |
| ORM | Prisma + PostgreSQL |
| Auth | JWT + bcrypt + Passport |
| Testes | Jest + Supertest (TDD) |
| Frontend | Angular 17+ Standalone |
| CI/CD | GitHub Actions |
| Infra | Render + Neon.tech + Vercel |
