# AuthHub — Contexto do Projeto

> Handoff para o Claude Code. Convenções compartilhadas em `../../README.md`.
> **Status:** blueprint (a construir). Referência de qualidade: `../../python/docsage`.

## O que é
Servidor central de **autenticação e autorização**: login, registro, OAuth2/OIDC, MFA.
É reutilizável como *single sign-on* pelos outros projetos Java — construa este primeiro.

**Valor de portfólio:** auth é a base de qualquer sistema; um servidor próprio demonstra
domínio de segurança de verdade.

## Stack
Java 21 · Spring Boot 3.3 · Spring Authorization Server · Spring Security · Spring Data JPA ·
Flyway · PostgreSQL 16 · Maven · JUnit 5 + Testcontainers.

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
8. Deploy (ver `../../DEPLOY-GERAL.md`)

## Como começar
Scaffold a partir de `../../../projeto-template` (use `Dockerfile.java`). Abra o Claude Code
nesta pasta e siga as fases acima, uma por vez, com testes a cada etapa.
