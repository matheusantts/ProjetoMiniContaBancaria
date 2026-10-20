# Aula 07 — Docker e fechamento

**Objetivo:** subir o sistema inteiro com um comando e documentar o projeto.

## 1. Imagem × container
- **Imagem:** "receita" empacotada e imutável (sistema + app + dependências).
- **Container:** instância em execução de uma imagem.
Resolve o "na minha máquina funciona": o ambiente vai junto.

## 2. Dockerfile do backend (multi-stage)
Dois estágios: um com JDK+Maven para **compilar**, outro só com JRE para **rodar** (imagem final menor).
```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn -q dependency:go-offline
COPY src ./src
RUN mvn -q package -DskipTests

FROM eclipse-temurin:21-jre
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```
Copiar o `pom.xml` antes do `src` aproveita o **cache de camadas**: dependências só são rebaixadas se o `pom.xml` mudar.

## 3. Dockerfile do frontend
Build com Node gera arquivos estáticos; **nginx** os serve.
```dockerfile
FROM node:20 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
ARG VITE_API_URL
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf   # fallback para index.html (SPA)
```

## 4. Compose completo
Três serviços: `db`, `api` (`depends_on: db` com `healthcheck`), `web`.
- Dentro da rede do compose, serviços se acham pelo **nome** (`jdbc:postgresql://db:5432/...`, não `localhost`).
- Variáveis (`DB_USER`, `DB_PASSWORD`, `JWT_SECRET`) vêm do `.env`.
- `healthcheck` do Postgres (`pg_isready`) evita a API subir antes do banco aceitar conexões.
- Portas: `web 3000:80`, `api 8080:8080`.
Ajuste o CORS da API para aceitar `http://localhost:3000`.

Comandos úteis:
```bash
docker compose up --build        # sobe tudo
docker compose logs -f api       # acompanha logs
docker compose down              # para (dados do volume permanecem)
docker compose down -v           # para e APAGA os dados
```

## 5. Teste ponta a ponta (manual)
1. Registrar usuário → login.
2. Criar conta → depositar 200 → sacar 50 → saldo 150.
3. Tentar sacar 500 → erro de saldo insuficiente.
4. Conferir extrato (2 linhas, `saldo_apos` coerente).
5. `docker compose down` e `up` de novo → dados persistiram (volume).
6. Segundo usuário não enxerga a conta do primeiro.

## 6. CI (opcional) — GitHub Actions
Arquivo `.github/workflows/ci.yml` roda a cada push: `./mvnw test` no backend e `npm ci && npm test && npm run build` no frontend. **Integração contínua** = verificar automaticamente que nada quebrou.

## 7. README final
Deve conter: descrição, tecnologias, pré-requisitos (Docker), `cp .env.example .env`, `docker compose up --build`, URLs (app, Swagger), como rodar os testes e a estrutura de pastas.

## Tarefas
**7.1** Dockerfile back · **7.2** Dockerfile front · **7.3** Compose completo · **7.4** Teste manual · **7.5** CI · **7.6** README.

## Parabéns
Você terminou o ciclo: terminal → API REST → banco versionado → segurança → SPA → containers. Próximos passos naturais: transferências entre contas (transação envolvendo duas contas e ordem de lock), paginação do extrato e testes de integração com Testcontainers.
