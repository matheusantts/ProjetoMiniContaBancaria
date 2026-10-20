# Aula 00 — Arquitetura e conceitos

**Objetivo:** entender o "mapa" do que vamos construir antes de escrever código.

## 1. O que é "fullstack"?

Uma aplicação web tem duas metades:

- **Frontend** (o que roda no navegador): telas, botões, formulários. Aqui: **React**.
- **Backend** (o que roda no servidor): regras de negócio, segurança, acesso ao banco. Aqui: **Java + Spring Boot**.
- **Banco de dados**: onde os dados ficam guardados de forma permanente. Aqui: **PostgreSQL**.

No seu projeto de terminal, tudo estava em um único programa e os dados morriam ao fechar. Agora cada responsabilidade vai para um lugar.

## 2. Cliente/servidor e HTTP

O navegador (cliente) faz **requisições HTTP**; o servidor devolve **respostas**.
Uma requisição tem: **método** (verbo), **URL**, **cabeçalhos** (headers) e, às vezes, **corpo** (body).

| Método | Significado | Exemplo no projeto |
|---|---|---|
| GET | ler | `GET /api/contas/1` |
| POST | criar / executar ação | `POST /api/contas/1/depositos` |
| PUT/PATCH | alterar | (não usaremos agora) |
| DELETE | remover | (não usaremos agora) |

A resposta traz um **status code**: `200 OK`, `201 Created`, `400` (pedido inválido), `401` (não autenticado), `403` (sem permissão), `404` (não existe), `422` (regra de negócio violada, como saldo insuficiente), `500` (erro no servidor).

## 3. REST e JSON

**REST** é um estilo de desenhar APIs: URLs representam *recursos* (substantivos: `contas`, `extrato`) e o método HTTP diz a ação. **JSON** é o formato de texto que usamos para trocar dados:

```json
POST /api/contas/1/depositos
{ "valor": 150.00 }

→ 201 Created
{ "contaId": 1, "tipo": "DEPOSITO", "valor": 150.00, "saldoApos": 350.00 }
```

## 4. Arquitetura em camadas (backend)

```
Controller  →  Service  →  Repository  →  Banco
 (HTTP)        (regras)     (consultas)
```

- **Controller:** recebe a requisição, valida o formato e chama o service. Não tem regra de negócio.
- **Service:** onde mora a lógica ("saque só se houver saldo"). É aqui que o antigo `ContaBancaria.saque()` vai viver.
- **Repository:** conversa com o banco.
- **DTO** (Data Transfer Object): objeto que define o que entra e sai pela API, para não expor entidades internas.

Por que separar? Facilita testar, trocar partes e entender o código. Cada classe tem **uma responsabilidade**.

## 5. Mapeando o código antigo para o novo

| Antes (terminal) | Depois |
|---|---|
| `Scanner` lendo `nextDouble()` | formulário React → JSON no body |
| `System.out.println` | resposta JSON + tela React |
| `ContaBancaria` | entidade `Conta` + `ContaService` |
| `switch` do menu | rotas/botões no React + endpoints REST |
| variável `saldo` em memória | coluna `saldo` no PostgreSQL |
| `double` | `BigDecimal` / `NUMERIC(15,2)` |

## 6. Fluxo completo: "depositar R$ 100"

1. Usuário digita 100 no formulário React e clica em **Depositar**.
2. React envia `POST /api/contas/1/depositos` com `{ "valor": 100 }` e o token JWT no header.
3. O filtro de segurança valida o token e identifica o usuário.
4. `ContaController` valida o JSON (`valor > 0`) e chama `ContaService.depositar`.
5. O service busca a conta, confere se é do usuário, soma o saldo, grava uma `Transacao` — tudo numa **transação de banco**.
6. A resposta volta como JSON; o React atualiza o saldo na tela.

## Tarefas

- **0.1** Releia as seções acima e explique com suas palavras a diferença entre Controller e Service.
- **0.2** Desenhe (papel ou `Arquivos do projeto/*.drawio`) o fluxo do item 6 com setas entre navegador, API e banco.

## Para pesquisar depois
Status codes HTTP (MDN), "REST API design", "separation of concerns".
