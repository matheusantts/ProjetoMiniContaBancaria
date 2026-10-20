# Aula 06 — Frontend com React

**Objetivo:** construir a interface que consome a API: login, contas, depósito, saque e extrato.

## 1. Conceitos de React
- **Componente:** função que recebe `props` e devolve JSX (interface).
- **JSX:** sintaxe parecida com HTML dentro do JavaScript.
- **Estado (`useState`):** dados que, ao mudar, fazem o componente **re-renderizar**.
- **Efeitos (`useEffect`):** executar algo fora da renderização (ex.: buscar dados). Veremos que o TanStack Query reduz o uso manual disso.
- **Componente controlado:** o valor do `<input>` vive no estado (`value` + `onChange`).
- **Props × estado:** props vêm de fora (somente leitura); estado é interno.

```tsx
function FormDeposito({ contaId }: { contaId: number }) {
  const [valor, setValor] = useState('');
  return (
    <form onSubmit={...}>
      <input value={valor} onChange={e => setValor(e.target.value)} />
      <button>Depositar</button>
    </form>
  );
}
```

## 2. TypeScript
Tipos espelham os DTOs da API:
```ts
export interface Conta { id: number; numero: string; titular: string; saldo: number }
```
O compilador avisa se você usar um campo que não existe. (Para exibição, formate com `Intl.NumberFormat('pt-BR', {style:'currency', currency:'BRL'})`. O cálculo e a regra ficam no backend.)

## 3. Rotas (React Router)
SPA (*single page application*): uma página HTML só; o Router troca componentes conforme a URL.
```
/login   /registro   /contas   /contas/:id   (rotas protegidas exigem token)
```
**Rota protegida:** componente que redireciona para `/login` se não houver token.

## 4. Camada de API (`src/api/`)
Centralize o `fetch` num cliente que: define a URL base (`VITE_API_URL`), injeta `Authorization: Bearer <token>`, converte erros HTTP em exceções com a `mensagem` do backend e, em `401`, faz logout.
Variáveis do Vite só são expostas se começarem com `VITE_`.

## 5. TanStack Query (estado de servidor)
Separa **estado de servidor** (vem da API) de **estado de UI**.
- `useQuery(['contas'], listarContas)`: busca + cache + estados `isLoading`/`error`.
- `useMutation(depositar, { onSuccess: () => queryClient.invalidateQueries(['conta', id]) })`: após depositar, os dados são rebuscados e a tela atualiza.

## 6. Telas
1. **Login/Registro:** formulário, mostra erro da API.
2. **Contas:** lista + botão "Nova conta" (informa titular).
3. **Detalhe:** titular, número, saldo, forms de depósito/saque (erro 422 → "Saldo insuficiente"), extrato abaixo.

UX mínima: estado de carregando, mensagens de erro claras, desabilitar botão durante envio, validar valor > 0 no cliente (a validação **real** continua no servidor).

## 7. Testes (Vitest + Testing Library)
Testam o comportamento como o usuário vê: renderizar, digitar, clicar, esperar texto.
```tsx
render(<FormDeposito ... />);
await user.type(screen.getByLabelText('Valor'), '100');
await user.click(screen.getByRole('button', { name: /depositar/i }));
expect(await screen.findByText(/sucesso/i)).toBeInTheDocument();
```
A API é simulada (mock) nos testes.

## Tarefas
- **6.1** Router + layout + rota protegida. **6.2** Cliente HTTP. **6.3** Login/Registro. **6.4** Lista/criação de contas. **6.5** Detalhe com depósito/saque. **6.6** Extrato. **6.7** Erros/loading. **6.8** Testes.
- Comece pelo fluxo mais curto (login → listar contas) e só então avance.
