<div align="center">

# 🏋️ Gym Pass API

**API RESTful para gestão de academias, check-ins e usuários focada em SOLID e Clean Architecture.**

[![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Node.js](https://img.shields.io/badge/Node.js-339933?style=flat-square&logo=nodedotjs&logoColor=white)](https://nodejs.org/)
[![Fastify](https://img.shields.io/badge/Fastify-000000?style=flat-square&logo=fastify&logoColor=white)](https://fastify.dev/)
[![Prisma](https://img.shields.io/badge/Prisma-2D3748?style=flat-square&logo=prisma&logoColor=white)](https://www.prisma.io/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Vitest](https://img.shields.io/badge/Vitest-6E9F18?style=flat-square&logo=vitest&logoColor=white)](https://vitest.dev/)
[![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white)](https://github.com/features/actions)

</div>

---

## Sobre o Projeto

O **Gym Pass App** é uma API backend desenvolvida para o gerenciamento de academias, permitindo o cadastro de usuários, busca de academias próximas (geolocalização) e realização de check-ins diários.

O grande diferencial deste projeto é a sua fundação arquitetural. Ele foi construído aplicando rigorosamente os princípios **SOLID**, **Clean Architecture** (separação em *Controllers*, *Services* e *Repositories*) e **Design Patterns** (como *Factory* e *Repository*). Além disso, a aplicação possui uma esteira completa de testes automatizados (Unitários e End-to-End) integrados a um pipeline de CI/CD.

---

## Índice

- [Especificações e Regras](#especificações-e-regras)
- [Arquitetura e Padrões](#arquitetura-e-padrões)
- [Tecnologias](#tecnologias)
- [Endpoints da API](#endpoints-da-api)
- [Como Executar Localmente](#como-executar-localmente)
- [Testes e Ambientes Isolados](#testes-e-ambientes-isolados)
- [Estrutura de Pastas](#estrutura-de-pastas)

---

## Especificações e Regras

A aplicação foi guiada por requisitos rigorosos de negócio e sistema:

### ✅ Requisitos Funcionais (RFs)
- [x] O usuário deve poder se cadastrar e se autenticar.
- [x] O usuário deve poder visualizar seu perfil (quando logado).
- [x] O usuário deve poder obter o número total de check-ins realizados.
- [x] O usuário deve poder visualizar seu histórico completo de check-ins.
- [x] O usuário deve poder buscar academias pelo nome ou proximidade (até 10km).
- [x] O usuário deve poder realizar check-in em uma academia.
- [x] O administrador deve poder validar o check-in de um usuário.
- [x] O administrador deve poder cadastrar novas academias.

### 🔒 Regras de Negócio (RNs)
- [x] O usuário não pode se cadastrar com um e-mail duplicado.
- [x] O usuário não pode fazer mais de um check-in no mesmo dia.
- [x] O usuário só pode fazer check-in se estiver a menos de 100 metros da academia.
- [x] O check-in só pode ser validado até 20 minutos após sua criação.
- [x] O check-in só pode ser validado por administradores.
- [x] Apenas administradores podem cadastrar academias.

### ⚙️ Requisitos Não-Funcionais (RNFs)
- [x] A senha do usuário deve estar criptografada (`bcryptjs`).
- [x] Os dados da aplicação devem estar persistidos em um banco PostgreSQL.
- [x] Todas as listas de dados devem ser paginadas, retornando 20 itens por página.
- [x] O usuário deve ser identificado por um JWT (JSON Web Token) e Refresh Token.

---

## Arquitetura e Padrões

O projeto foi construído utilizando conceitos avançados de engenharia de software e recursos modernos do Node.js, separando o "mundo externo" (HTTP, Banco de Dados) do "coração" da aplicação (Regras de Negócio).

~~~text
┌──────────────────────────────────────────────────────────┐
│                      Infrastructure                      │
│       Fastify Routes · Prisma ORM · Zod Validation       │
│                                                          │
│   ┌──────────────────────────────────────────────────┐   │
│   │                    HTTP Layer                    │   │
│   │      Controllers · Middlewares (JWT/Roles)       │   │
│   │                                                  │   │
│   │   ┌──────────────────────────────────────────┐   │   │
│   │   │               Application                │   │   │
│   │   │     Services (Regras de Negócio Puras)   │   │   │
│   │   │                                          │   │   │
│   │   │   ┌──────────────────────────────────┐   │   │   │
│   │   │   │              Domain              │   │   │   │
│   │   │   │      Repository Interfaces       │   │   │   │
│   │   │   └──────────────────────────────────┘   │   │   │
│   │   └──────────────────────────────────────────┘   │   │
│   └──────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
~~~

- **Repository Pattern:** O acesso aos dados é abstraído via interfaces (`src/repositories`). Isso permitiu implementar repositórios em memória (`in-memory`) para executar a suíte de testes unitários sem tocar no banco de dados.
- **Dependency Injection & Factories:** Os *Services* (casos de uso) não instanciam suas próprias dependências. Em vez disso, funções *Factory* são responsáveis por injetar os repositórios corretos, mantendo o código limpo.
- **RBAC (Role-Based Access Control):** Middlewares personalizados para verificar a *Role* do usuário (`verify-user-role.ts`), garantindo que apenas `ADMINS` acessem rotas sensíveis.

---

## Tecnologias

| Categoria       | Tecnologia                 |
| --------------- | -------------------------- |
| Linguagem       | TypeScript                 |
| Framework Web   | Fastify                    |
| Banco de Dados  | PostgreSQL                 |
| ORM             | Prisma                     |
| Validação       | Zod                        |
| Autenticação    | JWT (`@fastify/jwt`)       |
| Encriptação     | bcryptjs                   |
| Datas           | Day.js                     |
| Testes          | Vitest / Supertest         |

---

## Endpoints da API

A API é protegida por **Autenticação JWT** e controle de acesso baseado em cargos (**RBAC**).

### Usuários e Autenticação
| Método | Rota             | Descrição                                         | Autenticação / Cargo |
| ------ | ---------------- | ------------------------------------------------- | -------------------- |
| POST   | `/users`         | Criação de uma nova conta de usuário              | ❌ Público           |
| POST   | `/sessions`      | Autenticação (Login) e geração de JWT             | ❌ Público           |
| PATCH  | `/token/refresh` | Renovação silenciosa do token de acesso           | ❌ Público           |
| GET    | `/me`            | Busca o perfil do usuário logado                  | ✅ Autenticado       |

### Academias (Gyms)
| Método | Rota             | Descrição                                         | Autenticação / Cargo |
| ------ | ---------------- | ------------------------------------------------- | -------------------- |
| GET    | `/gyms/search`   | Busca academias pelo nome (Paginado)              | ✅ Autenticado       |
| GET    | `/gyms/nearby`   | Busca academias num raio de 10km (Geolocalização) | ✅ Autenticado       |
| POST   | `/gyms`          | Cadastra uma nova academia no sistema             | 🛡️ **ADMIN** |

### Check-ins
| Método | Rota                           | Descrição                                     | Autenticação / Cargo |
| ------ | ------------------------------ | --------------------------------------------- | -------------------- |
| GET    | `/check-ins/history`           | Histórico de check-ins do usuário             | ✅ Autenticado       |
| GET    | `/check-ins/metrics`           | Contagem total de check-ins do usuário        | ✅ Autenticado       |
| POST   | `/gyms/:gymId/check-ins`       | Realiza um check-in em uma academia específica| ✅ Autenticado       |
| PATCH  | `/check-ins/:checkInId/validate`| Valida e aprova um check-in pendente         | 🛡️ **ADMIN** |

---

## Como Executar Localmente

### Pré-requisitos
- Node.js (v18+)
- Docker e Docker Compose

### Passo a passo

**1. Clone o repositório:**
~~~bash
git clone https://github.com/gabriel-smartins/03-api-node-solid.git
cd 03-api-node-solid
~~~

**2. Instale as dependências:**
~~~bash
npm install
~~~

**3. Configure o Banco de Dados:**
Renomeie o arquivo `.env.example` para `.env`. Em seguida, suba o container do PostgreSQL:
~~~bash
docker-compose up -d
~~~

**4. Execute as Migrations do Prisma:**
~~~bash
npx prisma migrate dev
~~~

**5. Inicie o servidor:**
~~~bash
npm run start:dev
~~~

---

## Testes e Ambientes Isolados

Este projeto possui uma arquitetura de testes robusta suportada pelo **Vitest**.

### Testes Unitários
Testam os *Services* isoladamente utilizando Repositórios em Memória. São extremamente rápidos e validam as regras de negócio.
~~~bash
npm run test           # Roda todos os testes unitários
npm run test:watch     # Modo de observação
~~~

### Testes End-to-End (E2E)
Testam a aplicação do começo ao fim (Rotas -> Controllers -> Banco de Dados) simulando requisições HTTP via `Supertest`.

🌟 **Diferencial:** O projeto utiliza um pacote customizado (`vitest-environment-prisma`) configurado nos scripts do `package.json`. Ele garante que **cada arquivo de teste E2E suba um schema de banco de dados totalmente isolado** e o destrua ao final, evitando conflitos de dados entre os testes rodando em paralelo.

~~~bash
npm run test:e2e       # Roda a suíte de integração
~~~

### Cobertura de Código
~~~bash
npm run test:coverage  # Gera o relatório de cobertura de testes (C8)
~~~

---

## Estrutura de Pastas

~~~text
03-api-node-solid/
│
├── .github/workflows/         # Pipeline de CI/CD (GitHub Actions)
├── prisma/                    # Schemas, migrations e ambiente Vitest customizado
│
└── src/
    ├── @types/                # Definições globais de tipagem do Fastify JWT
    ├── env/                   # Validação estrita de variáveis de ambiente
    ├── http/                  # Camada de Apresentação (Rotas HTTP)
    │   ├── controllers/       # Controladores de domínio (check-ins, gyms, users)
    │   └── middlewares/       # Interceptadores (verify-jwt, verify-user-role)
    │
    ├── lib/                   # Configuração de libs de terceiros (Prisma Client)
    │
    ├── repositories/          # Padrão Repository
    │   ├── in-memory/         # Implementação para testes unitários
    │   └── prisma/            # Implementação real conectada ao PostgreSQL
    │
    ├── services/              # Regras de Negócio e Casos de Uso (Core da Aplicação)
    ├── utils/                 # Funções auxiliares genéricas
    │
    ├── app.ts                 # Configuração do Fastify e rotas globais
    └── server.ts              # Entry-point da aplicação HTTP
~~~

---

<div align="center">

Desenvolvido por Gabriel. Focado em qualidade de código, Clean Architecture e Testes Automatizados.

</div>
