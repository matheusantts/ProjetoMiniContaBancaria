# Aula 03 — Domínio e persistência (JPA)

**Objetivo:** representar as tabelas como classes Java e acessá-las sem escrever SQL manual.

## 1. ORM, JPA, Hibernate, Spring Data
- **ORM** (Object-Relational Mapping): mapeia objetos ↔ linhas de tabelas.
- **JPA**: a especificação (anotações como `@Entity`).
- **Hibernate**: a implementação que o Spring usa por baixo.
- **Spring Data JPA**: gera automaticamente a implementação de repositórios a partir de interfaces.

## 2. Entidade
```java
@Entity
@Table(name = "conta")
public class Conta {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true, length = 12)
    private String numero;

    @Column(nullable = false)
    private String titular;

    @Column(nullable = false, precision = 15, scale = 2)
    private BigDecimal saldo = BigDecimal.ZERO;

    @ManyToOne(fetch = FetchType.LAZY, optional = false)
    @JoinColumn(name = "usuario_id")
    private Usuario usuario;

    @Version
    private Long versao;
    // construtor protegido para o JPA + getters
}
```
Conceitos:
- `@Id` + `IDENTITY`: o banco gera o id (BIGSERIAL).
- `@ManyToOne` / `@OneToMany`: relacionamentos. `LAZY` = só carrega do banco quando acessar (evita consultas desnecessárias).
- `@Version`: ativa o **lock otimista** — o Hibernate inclui a versão no `UPDATE ... WHERE versao = ?`; se outra transação alterou antes, lança `OptimisticLockingFailureException`.
- `@Enumerated(EnumType.STRING)` para o enum `TipoTransacao`. **Sempre STRING**: gravar a posição (ORDINAL) quebra se alguém reordenar o enum.

### Encapsulamento (lembra do README original?)
Os atributos continuam `private`. Mas em vez de `setSaldo`, o domínio expõe **comportamentos**:
```java
public void depositar(BigDecimal valor) { saldo = saldo.add(valor); }
public void sacar(BigDecimal valor) {
    if (saldo.compareTo(valor) < 0) throw new SaldoInsuficienteException();
    saldo = saldo.subtract(valor);
}
```
Isso é encapsulamento de verdade: ninguém "seta" saldo arbitrário.

### `BigDecimal`
- Nunca compare com `==`/`equals` (2.0 ≠ 2.00); use `compareTo`.
- Imutável: `saldo.add(x)` **devolve** novo objeto.

## 3. Repositórios
```java
public interface ContaRepository extends JpaRepository<Conta, Long> {
    Optional<Conta> findByIdAndUsuarioId(Long id, Long usuarioId);
    List<Conta> findByUsuarioId(Long usuarioId);
    boolean existsByNumero(String numero);
}
```
Basta a interface: o Spring Data cria a implementação lendo o **nome do método** (*derived queries*). `JpaRepository` já traz `save`, `findById`, `findAll`, `delete`...

## 4. Número de conta único
O requisito fala em "número de conta único gerado". Estratégia simples: gerar número aleatório de 8 dígitos + dígito verificador e conferir com `existsByNumero`, repetindo se colidir. O `UNIQUE` do banco é a rede de segurança final.

## 5. Teste de repository
`@DataJpaTest` sobe só a camada JPA com um banco em memória (H2) ou Testcontainers:
```java
@DataJpaTest
class ContaRepositoryTest {
    @Autowired ContaRepository repo;
    @Test void salvaEBuscaPorNumero() { ... }
}
```
Padrão **AAA**: *Arrange* (monta), *Act* (executa), *Assert* (verifica).

## Tarefas
- **3.1** Criar `Usuario`, `Conta`, `Transacao`, `TipoTransacao` em `domain/`, com `depositar`/`sacar` em `Conta`.
- **3.2** Criar os repositories.
- **3.3** Criar `GeradorNumeroConta` (component) com teste.
- **3.4** Escrever `@DataJpaTest`: salvar conta, buscar por usuário, violar unicidade do número.
- Teste unitário puro de `Conta.sacar` (saldo suficiente, insuficiente, exatamente igual ao saldo).
