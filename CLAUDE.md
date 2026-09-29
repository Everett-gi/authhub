# AuthHub — Contexto do Projeto

> Handoff para o Claude Code. Este arquivo é **autossuficiente**: traz o blueprint do projeto e,
> no fim, a base de segurança e as convenções do portfólio. A central do portfólio é
> [Everett-gi/Projetos-e-ideias](https://github.com/Everett-gi/Projetos-e-ideias). Referências de qualidade:
> [DocSage](https://github.com/Everett-gi/docsage) e [PwnCheck](https://github.com/Everett-gi/pwncheck)
> (estrutura, testes, CI e as lições do modo tutorial).
> **Status:** 📋 blueprint pronto — a construção começa pela fase 1.

## Modo tutorial

O autor programa bem em **C e C++** (lógica, ponteiros, memória, compilação) e
está aprendendo Python; **Java é novo para ele** — este projeto também é o curso de Java. Se outro projeto Java do portfólio já tiver ensinado o básico, não repita: aponte para a lição de lá.

- Explique cada passo e o **porquê**, com analogias a C/C++ quando ajudarem (ex.: JVM × compilação nativa, referências × ponteiros, garbage collector × RAII e `try-with-resources`, interfaces × classes abstratas puras, generics × templates, Maven × CMake + gerenciador de pacotes, anotações do Spring × atributos/macros).
  Não explique lógica de programação básica; foque no que é novo.
- Cada fase concluída ganha uma lição em `docs/tutorial/fase-N-<tema>.md`, com exercícios e
  respostas no fim — o modelo são as lições do PwnCheck.
- Confirme rodando código qualquer afirmação técnica antes de escrevê-la numa lição.
- Sessões na nuvem rodam em Linux (bash). Ao indicar comandos para o autor rodar na máquina
  dele, use PowerShell (Windows 11, VS Code, repositórios clonados em `C:\dev`).
- Commits no padrão Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `ci:`, `chore:`),
  um assunto por commit. Rode lint e testes sobre o estado final, logo antes de cada commit.

## Prioridades

**1) Segurança · 2) Clareza do código · 3) Funcionalidade.** Nessa ordem.

## O que é

Servidor central de **autenticação e autorização**: login, registro, OAuth2/OIDC, MFA.
É reutilizável como *single sign-on* pelos outros projetos Java — construa este primeiro.

**Valor de portfólio:** auth é a base de qualquer sistema; um servidor próprio demonstra
domínio de segurança de verdade.

## Stack

Java 21 · Spring Boot 3.3 · Spring Authorization Server · Spring Security · Spring Data JPA ·
Flyway · PostgreSQL 16 · Maven · JUnit 5 + Testcontainers.

> **Versões:** o blueprint cita Spring Boot 3.3, cuja linha já saiu do suporte gratuito.
> No scaffold, use as versões estáveis atuais do Spring Boot e do JDK LTS (confira no
> start.spring.io) e registre a escolha na lição da fase 1. Segurança vem antes de fidelidade ao blueprint.

## Modelo de dados

- **users** (id, email único, password_hash, enabled, created_at)
- **roles** (id, name) + **user_roles** (user_id, role_id)
- **refresh_tokens** (id, user_id, token_hash, expires_at, revoked)
- **password_reset_tokens** (id, user_id, token_hash, expires_at, used)
- **totp_secrets** (user_id, secret_encrypted, confirmed)
- **oauth_clients** (client_id, client_secret_hash, redirect_uris, scopes)

## Endpoints principais

- `POST /auth/register` · `POST /auth/login` · `POST /auth/token/refresh` · `POST /auth/logout`
- `POST /auth/password/forgot` · `POST /auth/password/reset`
- `POST /auth/mfa/setup` · `POST /auth/mfa/verify`
- OIDC: `/.well-known/openid-configuration` · `/oauth2/authorize` · `/oauth2/token` · `/userinfo`
- Admin (ROLE_ADMIN): gestão de papéis e clientes

## Foco de segurança

Hash **Argon2**; JWT access curto (15 min) + refresh com **rotação**; **RBAC**;
**rate limit + lockout** no login; **TOTP** para MFA; **PKCE** no fluxo OAuth2;
proteção contra *email enumeration* (resposta uniforme); segredos TOTP criptografados.

## Plano de build

1. Cadastro + hash Argon2 + schema Flyway
2. Login + emissão de JWT (access + refresh)
3. Rotação e revogação de refresh token
4. Reset de senha por token (e-mail)
5. MFA/TOTP (setup + verificação)
6. OAuth2/OIDC (authorization code + PKCE, userinfo)
7. RBAC + endpoints de admin
8. Deploy (ver `docs/DEPLOY.md`)

## Como começar

1. Gere o esqueleto no [Spring Initializr](https://start.spring.io): projeto **Maven**, Java,
   dependências Spring Web, Spring Security, OAuth2 Authorization Server, Spring Data JPA,
   Flyway, PostgreSQL Driver, Validation e Testcontainers. Extraia na **raiz deste repositório**, preservando `README.md`,
   `CLAUDE.md`, `.gitignore` e `.gitattributes` daqui (junte ao nosso `.gitignore` as regras
   do gerado).
2. Crie já na fase 1 o CI em `.github/workflows/ci.yml` (build e testes com `./mvnw verify`
   e o scan OWASP Dependency-Check), um `docker-compose.yml` com PostgreSQL 16 para
   desenvolvimento e um `.env.example` com valores fictícios.
3. Mantenha a lógica de domínio testável sem banco (testes unitários rápidos) e use
   Testcontainers nos testes de integração. Se o ambiente não tiver Docker, os testes
   unitários precisam continuar rodando.
4. Siga o plano de build, uma fase por vez, com testes a cada etapa. Este é o primeiro
   projeto Java do portfólio: ele serve de *single sign-on* para os outros.

## Base de segurança do portfólio (Definition of Done)

Vale para todos os projetos do portfólio, este incluído:

- **Segredos** só em variáveis de ambiente (`.env` fora do Git e da imagem). Nunca hardcoded.
- **Senhas** com hash Argon2/BCrypt. **Nunca** texto puro ou hash fraco.
- **Autenticação** por JWT (access + refresh com rotação) onde houver login.
- **Autorização** com verificação de posse por recurso (anti-IDOR) e RBAC quando aplicável.
- **SQL Injection**: acesso ao banco só via ORM / parâmetros ligados. Nunca concatenar SQL.
- **Validação de entrada** com schema/DTO e limite de tamanho de payload.
- **Rate limiting** em login e endpoints sensíveis.
- **Cabeçalhos de segurança** (HSTS, CSP, X-Content-Type-Options) e **HTTPS** em produção.
- **Dependências** escaneadas no CI + Dependabot.
- **Logs** sem segredos nem dados pessoais.
- **LGPD**: coleta mínima, aceite de termos e exclusão de conta quando houver dados de usuário.
- Um **`SECURITY.md`** descrevendo essas medidas aplicadas a este repositório (crie-o na
  primeira fase com código e mantenha-o atualizado).

## Convenções de código

- Comentários e documentação do código em **português**; nomes de código em **inglês**.
- Tipos explícitos sempre (em Python, *type hints*).
- Toda função pura nova vem acompanhada de teste.
- Mensagens de erro da API/interface em português (é o usuário que lê).
- Ao criar uma variável de ambiente nova, atualize o `.env.example` (valores fictícios).

## Ferramentas — Java (Spring Boot)

| | |
|---|---|
| Migrations | Flyway |
| Auth | Spring Security |
| Testes | JUnit 5 + Testcontainers |
| Lint | Checkstyle / SpotBugs |
| Scan de segurança | OWASP Dependency-Check |

## Deploy

Guia em `docs/DEPLOY.md` (porta interna 8080).
