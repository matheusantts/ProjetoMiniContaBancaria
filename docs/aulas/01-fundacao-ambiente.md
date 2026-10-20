# Aula 01 — Fundação do repositório e ambiente

**Objetivo:** organizar o repositório, instalar as ferramentas e gerar os esqueletos de backend, frontend e banco.

## 1. Ferramentas e para que servem

| Ferramenta | Função |
|---|---|
| **JDK 21** | Compila e executa Java. Versão LTS (suporte longo). |
| **Maven** | Gerenciador de build e dependências. O `pom.xml` lista as bibliotecas; o Maven as baixa e empacota o projeto. O `./mvnw` (wrapper) baixa a versão certa sozinho. |
| **Node.js 20+ / npm** | Executa JavaScript fora do navegador; npm instala pacotes do frontend. |
| **Vite** | Ferramenta de desenvolvimento do frontend: servidor local com recarga instantânea e build otimizado. |
| **Docker** | Empacota programas em *containers* (ambientes isolados e reproduzíveis). |
| **Docker Compose** | Sobe vários containers juntos a partir de um arquivo `docker-compose.yml`. |
| **Git** | Versionamento. |

Conferir instalação:
```bash
java -version      # 21+
./mvnw -v          # (dentro de /backend, depois de gerado)
node -v && npm -v
docker --version && docker compose version
```

## 2. Monorepo
Backend e frontend no **mesmo repositório**, em pastas separadas. Vantagem: uma única fonte de verdade, commits que alteram API e tela juntos, um só `docker compose`.

## 3. `.gitignore`
Lista o que **não** vai para o Git: arquivos gerados (`target/`, `node_modules/`, `out/`), configuração de IDE (`.idea/`) e segredos (`.env`). Arquivos `.class` commitados (como `out/`) poluem o repositório e geram conflitos.

```gitignore
out/
target/
node_modules/
dist/
.idea/
*.iml
.env
```
Para remover do Git o que já foi commitado: `git rm -r --cached out/`.

## 4. Spring Boot e o Spring Initializr
**Spring Framework** é o ecossistema Java mais usado em backend. **Spring Boot** elimina a configuração manual: traz servidor web embutido (Tomcat) e configuração automática por convenção. O **Spring Initializr** (start.spring.io) gera o projeto com as dependências escolhidas.

Escolhas: Maven · Java 21 · Spring Boot 3.x · pacote `com.matheusantts.minibanco` · dependências:

- **Spring Web** — endpoints REST
- **Spring Data JPA** — acesso ao banco
- **PostgreSQL Driver** — conexão
- **Flyway Migration** (+ `flyway-database-postgresql`) — versionar o schema
- **Validation** — Bean Validation
- **Spring Security** — autenticação
- **Lombok** (opcional) — reduz boilerplate; **vamos preferir `record`s e escrever getters à mão onde necessário** para você ver o que acontece
- **Spring Boot DevTools** — recarga em desenvolvimento

## 5. React + Vite + TypeScript
**React** constrói interfaces com *componentes* (funções que devolvem HTML-like, o JSX). **TypeScript** adiciona tipos ao JavaScript e pega erros antes de rodar.
```bash
npm create vite@latest frontend -- --template react-ts
cd frontend && npm install
npm install react-router-dom @tanstack/react-query
```

## 6. Docker Compose com PostgreSQL
Em vez de instalar o Postgres no seu computador, rodamos ele num container:

```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_DB: minibanco
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    ports: ["5432:5432"]
    volumes: [pgdata:/var/lib/postgresql/data]
volumes:
  pgdata:
```
- **volume**: guarda os dados fora do container para não perdê-los ao recriá-lo.
- **`.env`**: guarda segredos fora do código. Commit apenas o `.env.example` (com valores fictícios).

## Tarefas

- **1.1** Conferir as versões instaladas (comandos acima).
- **1.2** Criar `.gitignore` e `git rm -r --cached out/`.
- **1.3** `git mv MiniSistemaBancario legado-console` (o histórico é preservado).
- **1.4** Gerar o backend no Initializr e extrair em `/backend`; rodar `./mvnw test` (deve passar).
- **1.5** Criar o frontend com Vite em `/frontend`; rodar `npm run dev` e abrir `localhost:5173`.
- **1.6** Criar `docker-compose.yml` + `.env.example`; `docker compose up -d db` e conectar com `docker compose exec db psql -U $DB_USER minibanco`.
- **1.7** Reescrever o README: o que é, stack, como rodar.

## Checklist de "feito"
Backend sobe sem erro (pode falhar sem datasource até a Fase 2 — ok), frontend abre, Postgres responde.
