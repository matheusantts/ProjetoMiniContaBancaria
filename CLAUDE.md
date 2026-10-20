# CLAUDE.md

Guia para o Claude Code neste repositório. Leia antes de qualquer tarefa.

## O projeto

**Mini Sistema Bancário Fullstack**: evolução de um app de terminal em Java para uma aplicação web.
O projeto também é **didático**: o autor (Matheus) está aprendendo fullstack. Cada fase tem uma aula em `docs/aulas/`.

- Plano e checklist de tarefas: `PLANO.md`
- Aulas (conceitos + passo a passo): `docs/aulas/`
- Código legado (console, referência): `MiniSistemaBancario/` (mover para `legado-console/` na Fase 1)

## Stack (decisões já tomadas, não rediscutir)

| Camada | Tecnologia |
|---|---|
| Backend | Java 21, Spring Boot 3.x, **Maven** |
| Persistência | Spring Data JPA, PostgreSQL, Flyway (H2 ou Testcontainers nos testes) |
| Segurança | Spring Security + JWT (autenticação com login) |
| Docs da API | springdoc-openapi (Swagger UI) |
| Frontend | React + Vite + TypeScript, React Router, TanStack Query |
| Testes | JUnit 5, MockMvc, Mockito; Vitest + Testing Library |
| Infra | Docker Compose (api + web + postgres). **Só roda local, sem deploy** |

Monorepo: `/backend`, `/frontend`, `docker-compose.yml` na raiz.

## Estrutura alvo

```
backend/src/main/java/com/matheusantts/minibanco/
  config/       segurança, CORS, OpenAPI
  controller/   endpoints REST (finos, sem regra de negócio)
  service/      regras de negócio (@Transactional)
  repository/   interfaces Spring Data
  domain/       entidades JPA e enums
  dto/          records de request/response
  exception/    exceções de negócio + @RestControllerAdvice
backend/src/main/resources/db/migration/   migrações Flyway (V1__, V2__...)
frontend/src/{pages,components,api,hooks}/
```

## Regras de negócio (invioláveis)

- Dinheiro é sempre `BigDecimal` (backend) e `NUMERIC(15,2)` (banco). **Nunca `double`/`float`.**
- Depósito e saque exigem valor **> 0** (senão HTTP 400).
- Saque com saldo insuficiente → HTTP 422, saldo inalterado.
- Número de conta é **único e gerado pelo servidor**, nunca enviado pelo cliente.
- Todo depósito/saque gera uma linha em `transacao` (extrato) na mesma transação do banco.
- Um usuário só acessa **as próprias contas** (checar dono em toda operação).
- Senhas: só hash (BCrypt). Nunca logar senha, token ou hash.

## Convenções de código

- Código, nomes de classes/métodos/colunas e mensagens de commit: **português sem acento no identificador** quando for domínio (`ContaService`, `depositar`, `saldo`); termos técnicos em inglês são ok (`Controller`, `Repository`).
- Comentários só quando explicam o "porquê". Seguir a densidade do código ao redor.
- Controller → Service → Repository; controller nunca acessa repository direto.
- Entidades JPA **não** saem pela API: sempre DTO (`record`).
- Validação de entrada com Bean Validation (`@NotNull`, `@Positive`...) nos DTOs.
- Erros padronizados via `@RestControllerAdvice` (corpo JSON com `status`, `mensagem`, `timestamp`).
- Schema só muda via migração Flyway nova; **nunca editar migração já aplicada** e `ddl-auto` fica em `validate`.
- Frontend: componentes funcionais + hooks, chamadas HTTP só em `src/api/`, estado de servidor com TanStack Query.

## Comandos

```bash
# Subir tudo (api + web + banco)
docker compose up --build

# Backend
cd backend && ./mvnw test              # testes
cd backend && ./mvnw spring-boot:run   # roda local (precisa do postgres)

# Frontend
cd frontend && npm install && npm run dev
cd frontend && npm test && npm run lint
```

Portas: web `5173` (dev) / `3000` (docker), api `8080`, postgres `5432`, Swagger `http://localhost:8080/swagger-ui.html`.

## Como trabalhar neste repo

1. Siga `PLANO.md` na ordem das fases; marque os itens `[x]` ao concluir.
2. Antes de implementar uma fase, leia a aula correspondente em `docs/aulas/`. Se o código divergir da aula, atualize a aula.
3. Ao concluir uma tarefa: rode testes/lint da camada alterada antes de commitar.
4. Commits pequenos, um por tarefa, mensagem em português no imperativo (`Adiciona ContaService com depósito e saque`).
5. Não criar PR sem o usuário pedir. Não commitar `out/`, `target/`, `node_modules/`, `.env`, `.idea/`.
6. Explique conceitos novos de forma didática ao usuário: ele quer aprender, não só receber código pronto.
7. Segredos (senha do banco, chave JWT) vão em variáveis de ambiente / `.env` (com `.env.example` commitado), nunca no código.
