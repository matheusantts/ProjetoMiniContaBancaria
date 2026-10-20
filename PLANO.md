# Plano do Projeto: Mini Sistema Bancário Fullstack

Evolução do app de terminal (Java puro) para aplicação web completa.
**Stack:** React + Vite + TS · Java 21 + Spring Boot 3 (Maven) · PostgreSQL · JWT · Docker Compose (somente local).

## Visão da arquitetura

```
 Navegador ──► React (Vite/nginx) ──HTTP/JSON──► Spring Boot API ──JPA──► PostgreSQL
                  :3000                              :8080                   :5432
```

## Requisitos funcionais (do README original + evolução)

- [ ] RF1 Criar conta (titular + número único gerado)
- [ ] RF2 Depositar
- [ ] RF3 Sacar (validando saldo)
- [ ] RF4 Consultar saldo + dados do titular
- [ ] RF5 Interface interativa (substitui o menu em `while`)
- [ ] RF6 Cadastro e login de usuários (JWT); cada usuário vê só suas contas
- [ ] RF7 Extrato de transações

## Fases e tarefas

Cada fase tem uma aula em `docs/aulas/`. Marque `[x]` conforme concluir.

### Fase 0 — Conceitos e arquitetura → [aula 00](docs/aulas/00-arquitetura-e-conceitos.md)
- [ ] 0.1 Ler a aula: cliente/servidor, REST, camadas, JSON
- [ ] 0.2 Desenhar (no papel ou drawio) o fluxo "depositar" de ponta a ponta

### Fase 1 — Fundação do repositório e ambiente → [aula 01](docs/aulas/01-fundacao-ambiente.md)
- [ ] 1.1 Instalar/conferir JDK 21, Maven, Node 20+, Docker, Git
- [ ] 1.2 Criar `.gitignore` na raiz; remover `out/` do Git
- [ ] 1.3 Mover `MiniSistemaBancario/` para `legado-console/`
- [ ] 1.4 Gerar esqueleto Spring Boot em `/backend` (Spring Initializr)
- [ ] 1.5 Gerar esqueleto React+Vite+TS em `/frontend`
- [ ] 1.6 `docker-compose.yml` com PostgreSQL + `.env.example`
- [ ] 1.7 Atualizar README (como rodar)

### Fase 2 — Banco de dados e migrações → [aula 02](docs/aulas/02-banco-de-dados-flyway.md)
- [ ] 2.1 Modelar tabelas `usuario`, `conta`, `transacao` (diagrama ER)
- [ ] 2.2 Configurar datasource e Flyway
- [ ] 2.3 Escrever `V1__criar_tabelas.sql`
- [ ] 2.4 Subir app e confirmar migração aplicada

### Fase 3 — Domínio e persistência (JPA) → [aula 03](docs/aulas/03-dominio-jpa.md)
- [ ] 3.1 Entidades `Usuario`, `Conta`, `Transacao` + enum `TipoTransacao`
- [ ] 3.2 Repositories (`ContaRepository`, etc.)
- [ ] 3.3 Gerador de número de conta único
- [ ] 3.4 Teste de repository (`@DataJpaTest`)

### Fase 4 — Regras de negócio e API REST → [aula 04](docs/aulas/04-service-controller-rest.md)
- [ ] 4.1 DTOs (records) + Bean Validation
- [ ] 4.2 `ContaService`: criar, consultar, depositar, sacar, extrato
- [ ] 4.3 Exceções de negócio + `@RestControllerAdvice`
- [ ] 4.4 `ContaController` com os endpoints
- [ ] 4.5 Testes unitários do service (Mockito) e de controller (MockMvc)
- [ ] 4.6 Swagger/OpenAPI

### Fase 5 — Segurança (Spring Security + JWT) → [aula 05](docs/aulas/05-seguranca-jwt.md)
- [ ] 5.1 Registro (`/api/auth/registro`) com BCrypt
- [ ] 5.2 Login que devolve JWT
- [ ] 5.3 Filtro JWT + `SecurityFilterChain` (stateless)
- [ ] 5.4 Garantir que cada usuário acessa só suas contas
- [ ] 5.5 CORS para o frontend
- [ ] 5.6 Testes de segurança (401/403)

### Fase 6 — Frontend React → [aula 06](docs/aulas/06-frontend-react.md)
- [ ] 6.1 Rotas (React Router) e layout
- [ ] 6.2 Cliente HTTP em `src/api/` com token no header
- [ ] 6.3 Telas de login e registro
- [ ] 6.4 Lista de contas + criar conta
- [ ] 6.5 Detalhe da conta: saldo, titular, depósito, saque
- [ ] 6.6 Extrato
- [ ] 6.7 Tratamento de erros e estados de loading
- [ ] 6.8 Testes com Vitest + Testing Library

### Fase 7 — Docker e fechamento → [aula 07](docs/aulas/07-docker-e-fechamento.md)
- [ ] 7.1 Dockerfile do backend (multi-stage)
- [ ] 7.2 Dockerfile do frontend (build + nginx)
- [ ] 7.3 Compose completo: `docker compose up --build` sobe tudo
- [ ] 7.4 Teste manual ponta a ponta
- [ ] 7.5 CI no GitHub Actions (build + testes) — opcional
- [ ] 7.6 README final com prints/instruções

### Evoluções futuras (fora do escopo inicial)
Transferência entre contas, tipos de conta, paginação do extrato, dashboard com gráficos.

## Definição de pronto (por tarefa)

Código compila, testes da camada passam, regras do `CLAUDE.md` respeitadas, checklist marcado e commit feito.
