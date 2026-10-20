# Aula 04 — Regras de negócio e API REST

**Objetivo:** expor as operações bancárias como endpoints seguros de erro e bem testados.

## 1. Injeção de dependência (DI)
Em vez de `new ContaRepository()`, você declara o que precisa no construtor e o Spring entrega (container IoC):
```java
@Service
public class ContaService {
    private final ContaRepository contas;
    public ContaService(ContaRepository contas) { this.contas = contas; }
}
```
Estereótipos: `@Component` (genérico), `@Service`, `@Repository`, `@RestController`. Facilita testes: no teste você passa um *mock*.

## 2. DTOs com `record` e validação
```java
public record OperacaoRequest(@NotNull @Positive @Digits(integer = 13, fraction = 2) BigDecimal valor) {}
public record ContaResponse(Long id, String numero, String titular, BigDecimal saldo) {}
```
- `record` = classe imutável concisa (Java 16+).
- Anotações do **Bean Validation** barram entrada inválida antes do service. No controller: `@Valid @RequestBody`.
- Resposta **nunca** devolve a entidade: evita vazar campos e loops de serialização.

## 3. Service e transação
```java
@Transactional
public TransacaoResponse sacar(Long contaId, Long usuarioId, BigDecimal valor) {
    Conta conta = contas.findByIdAndUsuarioId(contaId, usuarioId)
        .orElseThrow(ContaNaoEncontradaException::new);
    conta.sacar(valor);                       // regra no domínio
    Transacao t = transacoes.save(new Transacao(conta, SAQUE, valor, conta.getSaldo()));
    return TransacaoResponse.de(t);
}
```
**`@Transactional`**: tudo ou nada. Se a gravação do extrato falhar, o saldo também volta atrás (**atomicidade** — o "A" de ACID).

**Concorrência:** dois saques simultâneos de R$ 80 em saldo de R$ 100. Sem proteção, ambos leem 100 e o saldo fica 20 (errado). Com `@Version`, o segundo `UPDATE` falha → capturamos e devolvemos `409 Conflict` (ou tentamos de novo).

## 4. Controller
```java
@RestController
@RequestMapping("/api/contas")
public class ContaController {
    @PostMapping("/{id}/saques")
    public TransacaoResponse sacar(@PathVariable Long id,
                                   @Valid @RequestBody OperacaoRequest req,
                                   @AuthenticatedPrincipal ...) { ... }
}
```
Controller fino: converte HTTP ↔ chamada de service. Anotações: `@GetMapping`, `@PostMapping`, `@PathVariable` (da URL), `@RequestBody` (JSON), `@ResponseStatus`.

### Endpoints
| Método | Rota | Resultado |
|---|---|---|
| POST | `/api/contas` | 201 + conta (número gerado) |
| GET | `/api/contas` | contas do usuário |
| GET | `/api/contas/{id}` | titular + saldo |
| POST | `/api/contas/{id}/depositos` | 201 + transação |
| POST | `/api/contas/{id}/saques` | 201, ou 422 se saldo insuficiente |
| GET | `/api/contas/{id}/extrato` | lista de transações |

## 5. Tratamento global de erros
Exceções de negócio (`SaldoInsuficienteException`, `ContaNaoEncontradaException`) viram respostas HTTP num único lugar:
```java
@RestControllerAdvice
class TratadorDeErros {
    @ExceptionHandler(SaldoInsuficienteException.class)
    ResponseEntity<ErroResponse> saldo(SaldoInsuficienteException e) { ... 422 ... }
    @ExceptionHandler(MethodArgumentNotValidException.class) // 400
    @ExceptionHandler(ObjectOptimisticLockingFailureException.class) // 409
}
```
Corpo padrão: `{ "status": 422, "mensagem": "Saldo insuficiente", "timestamp": "..." }`.
Conta de outro usuário → responda **404** (não 403) para não revelar que ela existe.

## 6. Testes
- **Unitário do service (Mockito):** mocka os repositories; testa regras sem banco.
- **Web (`@WebMvcTest` + MockMvc):** testa rotas, validação e códigos HTTP sem subir tudo.
- Casos obrigatórios: depósito ok, valor 0/negativo (400), saque ok, saque com saldo insuficiente (422), saque do saldo exato (ok, fica 0), conta inexistente (404).

## 7. Swagger/OpenAPI
`springdoc-openapi-starter-webmvc-ui` gera documentação interativa em `/swagger-ui.html`: dá para testar a API sem frontend. Excelente para estudar.

## Tarefas
- **4.1** DTOs + validação. **4.2** `ContaService`. **4.3** exceções + advice. **4.4** `ContaController`. **4.5** testes. **4.6** Swagger.
- Teste manual com curl:
```bash
curl -X POST localhost:8080/api/contas/1/depositos \
  -H 'Content-Type: application/json' -d '{"valor": 100.00}'
```
(Antes da Fase 5, use um usuário fixo temporário; remova ao ligar a segurança.)
