# Aula 05 — Segurança: Spring Security + JWT

**Objetivo:** só usuários autenticados acessam a API, e cada um só vê suas contas.

## 1. Autenticação × Autorização
- **Autenticação:** *quem é você?* (login e senha) → falha = `401`.
- **Autorização:** *o que você pode fazer?* (só suas contas) → falha = `403`/`404`.

## 2. Senhas: nunca em texto puro
Guardamos um **hash**: função de mão única. Para validar, geramos o hash da senha digitada e comparamos. Usamos **BCrypt**: inclui *salt* aleatório (hashes diferentes para a mesma senha) e é propositalmente lento (dificulta força bruta).
```java
@Bean PasswordEncoder passwordEncoder() { return new BCryptPasswordEncoder(); }
```
O antigo campo `senhaConta` do código legado vira `senha_hash` no `usuario`.

## 3. Sessão × token (stateless)
Em vez de o servidor "lembrar" quem está logado, ele entrega um **token** assinado e o cliente o reenvia a cada requisição. Isso é **stateless**: o servidor não guarda sessão.

## 4. JWT (JSON Web Token)
Três partes em Base64: `header.payload.assinatura`.
- **payload** (claims): `sub` (id do usuário), `exp` (expiração).
- **assinatura**: HMAC com uma chave secreta só do servidor; prova que ninguém alterou o token.
- ⚠️ O payload é **legível** (só não é alterável). **Nunca coloque senha ou dados sensíveis nele.**
- Token expira rápido (ex.: 1 h). A chave vem de variável de ambiente (`JWT_SECRET`, ≥ 256 bits).

Fluxo:
```
POST /api/auth/login {email, senha} ──► valida ──► { "token": "eyJ..." }
GET /api/contas   Authorization: Bearer eyJ...  ──► filtro valida ──► controller
```

## 5. Peças do Spring Security
- **`SecurityFilterChain`**: define regras: `/api/auth/**` e Swagger públicos; o resto exige autenticação; sessão `STATELESS`; CSRF desligado (não usamos cookies de sessão).
- **Filtro JWT** (`OncePerRequestFilter`): lê o header `Authorization`, valida o token e coloca o usuário no `SecurityContext`.
- **`UserDetailsService`**: carrega usuário pelo e-mail.
- No controller, o id do usuário vem do contexto autenticado (`@AuthenticationPrincipal`) — **nunca** de um parâmetro enviado pelo cliente, senão qualquer um se passaria por outro (falha IDOR).

## 6. CORS
O frontend (`localhost:5173`) e a API (`localhost:8080`) são **origens diferentes**; o navegador bloqueia por padrão. Configure CORS permitindo apenas a origem do frontend (lida de config), os métodos e o header `Authorization`.

## 7. Boas práticas
- Mensagem de login genérica ("credenciais inválidas"): não revele se o e-mail existe.
- Valide força mínima da senha e e-mail no registro.
- Nunca logue senha/token.
- Armazenamento do token no frontend: `localStorage` é simples (adequado ao projeto de estudo), mas vulnerável a XSS; em produção avalie cookies `HttpOnly`.

## Tarefas
- **5.1** `AuthController` + `AuthService`: registro (e-mail único, BCrypt).
- **5.2** Login → JWT (biblioteca `jjwt` ou o suporte OAuth2 Resource Server do Spring).
- **5.3** Filtro + `SecurityFilterChain`.
- **5.4** Trocar o usuário fixo temporário pelo usuário autenticado; teste: usuário B não acessa conta de A (404).
- **5.5** CORS.
- **5.6** Testes: sem token → 401; token inválido/expirado → 401; senha errada → 401; registro com e-mail duplicado → 409.
