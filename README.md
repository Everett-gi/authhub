# AuthHub

Servidor central de autenticação e autorização — login, registro, OAuth2/OIDC e MFA — construído com foco em segurança.

> 📋 **Em construção.** O blueprint está pronto e a implementação começa pela fase 1 do
> roadmap abaixo. Parte do portfólio
> [Projetos-e-ideias](https://github.com/Everett-gi/Projetos-e-ideias).

## O que é

Servidor central de **autenticação e autorização**: login, registro, OAuth2/OIDC, MFA.
É reutilizável como *single sign-on* pelos outros projetos Java.

## Stack (planejada)

Java 21 · Spring Boot 3.3 · Spring Authorization Server · Spring Security · Spring Data JPA ·
Flyway · PostgreSQL 16 · Maven · JUnit 5 + Testcontainers.

## Modelo de dados (planejado)

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

## Segurança

Hash **Argon2**; JWT access curto (15 min) + refresh com **rotação**; **RBAC**;
**rate limit + lockout** no login; **TOTP** para MFA; **PKCE** no fluxo OAuth2;
proteção contra *email enumeration* (resposta uniforme); segredos TOTP criptografados.

## Roadmap

1. Cadastro + hash Argon2 + schema Flyway
2. Login + emissão de JWT (access + refresh)
3. Rotação e revogação de refresh token
4. Reset de senha por token (e-mail)
5. MFA/TOTP (setup + verificação)
6. OAuth2/OIDC (authorization code + PKCE, userinfo)
7. RBAC + endpoints de admin
8. Deploy

---

Desenvolvido em modo tutorial: cada fase concluída ganha uma lição em `docs/tutorial/`.
