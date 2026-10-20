# Aula 02 — Banco de dados e migrações (Flyway)

**Objetivo:** modelar os dados e criar o schema de forma versionada.

## 1. Banco relacional
Dados ficam em **tabelas** (linhas = registros, colunas = campos). Relacionamentos são feitos por **chaves**:
- **PK (primary key):** identifica a linha de forma única (`id`).
- **FK (foreign key):** referencia a PK de outra tabela (`conta.usuario_id → usuario.id`).
- **UNIQUE:** impede duplicatas (e-mail, número da conta).
- **NOT NULL / CHECK:** regras garantidas pelo próprio banco.

## 2. Modelo do projeto

```
usuario (1) ──< conta (1) ──< transacao
```
Um usuário tem várias contas; uma conta tem várias transações.

```sql
usuario:   id BIGSERIAL PK, nome, email UNIQUE, senha_hash, criado_em
conta:     id BIGSERIAL PK, numero UNIQUE, titular, saldo NUMERIC(15,2) >= 0,
           usuario_id FK, versao BIGINT, criada_em
transacao: id BIGSERIAL PK, conta_id FK, tipo, valor NUMERIC(15,2) > 0,
           saldo_apos NUMERIC(15,2), criada_em
```

### Por que `NUMERIC(15,2)` e não `double`?
`double` usa ponto flutuante binário: `0.1 + 0.2 = 0.30000000000000004`. Em dinheiro isso é inaceitável. `NUMERIC(15,2)` é decimal exato: até 15 dígitos, 2 casas.

### Por que uma tabela `transacao`?
O saldo é um número que muda; o **extrato** é o histórico que explica o saldo. Guardamos `saldo_apos` para auditar. Em bancos reais, o histórico é a fonte da verdade.

### Por que a coluna `versao`?
Para **lock otimista** (aula 04): evita que dois saques simultâneos leiam o mesmo saldo e gerem saldo errado.

## 3. Migrações com Flyway
Alterar o banco "no braço" não é rastreável. **Flyway** aplica scripts SQL versionados, em ordem, e registra o que já rodou na tabela `flyway_schema_history`.

- Arquivos em `src/main/resources/db/migration/`
- Nome: `V1__criar_tabelas.sql` (V + versão + `__` + descrição)
- **Regra de ouro:** nunca edite uma migração já aplicada (o Flyway detecta o checksum e falha). Para mudar, crie `V2__...`.

Configuração (`application.yml`):
```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/minibanco
    username: ${DB_USER}
    password: ${DB_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: validate   # Hibernate só confere; quem cria o schema é o Flyway
    open-in-view: false
  flyway:
    enabled: true
```
`${DB_USER}` lê variável de ambiente — por isso o segredo não fica no código.

## Tarefas

- **2.1** Desenhe o diagrama ER (pode ser no drawio) com as 3 tabelas e as relações.
- **2.2** Configure datasource/JPA/Flyway no `application.yml`.
- **2.3** Escreva `V1__criar_tabelas.sql` com PKs, FKs, UNIQUE, `CHECK (saldo >= 0)`, `CHECK (valor > 0)` e índices em FKs.
- **2.4** `docker compose up -d db`, rode a API e confira:
  ```bash
  docker compose exec db psql -U $DB_USER minibanco -c "\dt"
  ```
  Devem aparecer as tabelas + `flyway_schema_history`.

## Desafio
Tente inserir uma conta com saldo negativo direto no `psql` e veja o `CHECK` barrar. As regras críticas devem ser garantidas **também no banco**, não só no código.
