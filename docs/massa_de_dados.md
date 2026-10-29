# Massa de dados

Dados fictícios criados para o case: nomes, CPFs (só respeitam o dígito verificador), contas, cartões e estabelecimentos são inventados.
Simulam 6 cargas, cada uma numa pasta de `data/landing/` com o nome da data de referência.

| Pasta | O que chega |
|---|---|
| `20261001` | Carga inicial: cadastro completo e transações de janeiro a setembro/2026 |
| `20261002` a `20261006` | Cargas diárias: transações do dia anterior (01 a 05/10), atrasadas, reenvios, estornos e alterações de cadastro |

Os arquivos de cadastro (clientes, contas e cartões) das cargas diárias trazem só o que mudou no dia.

## Quantidade de linhas

| Arquivo | 20261001 | 20261002 | 20261003 | 20261004 | 20261005 | 20261006 | Total |
|---|---:|---:|---:|---:|---:|---:|---:|
| clientes | 60 | 3 | 2 | 1 | 1 | 1 | 68 |
| contas | 72 | 1 | 1 | 2 | 1 | 2 | 79 |
| cartoes | 82 | 0 | 2 | 1 | 1 | 1 | 87 |
| transacoes_cartao | 14.451 | 83 | 75 | 102 | 83 | 82 | 14.876 |
| estornos | 145 | 4 | 4 | 3 | 4 | 3 | 163 |

## Problemas colocados de propósito

As transações com problema de qualidade têm ID começando com `TRX9`, pra facilitar a conferência na quarentena.

**Clientes**
- CPF inválido: CLI0017 (10 dígitos) e CLI0042 (com letras). Contas, cartões e transações deles caem em cascata.
- CLI0020 chega duas vezes no mesmo arquivo (20261002) com cidades diferentes. Vale a das 16:40 (Canoas).
- CLI0033 chega em 20261003 com CPF inválido: vai pra quarentena e a silver mantém a versão anterior.
- Mudanças de cidade: CLI0005 (20261002), CLI0008 (20261003), CLI0047 (20261005).
- Clientes novos com conta e cartão: CLI0061 (20261004) e CLI0062 (20261006).

**Contas**
- Sem cliente: CTA0071 e CTA0072 (clientes CLI0999 e CLI0998 não existem).
- Status escrito de jeitos diferentes: CTA0003, CTA0011, CTA0024, CTA0038 (`ativa`, ` Ativa `...).
- Status inválido: CTA0015 com `XPTO` (20261002) e CTA0056 vazio (20261005).
- Mudanças de status: CTA0027 bloqueada (20261003) e reativada (20261006); CTA0040 encerrada (20261004).

**Cartões**
- Cancelados no meio do período: CRT0007 (maio), CRT0029 (julho), CRT0046 (agosto). Param de movimentar depois disso.
- CRT0013 cancelado em 20261003 e CRT0058 bloqueado em 20261005.
- Sem conta válida: os cartões das contas CTA0071/CTA0072 e os cartões CRT0081/CRT0082 (contas que não existem).

**Transações**
- 40 linhas duplicadas na carga inicial e 2 por carga diária.
- Reenvio: 3 transações de cada carga chegam de novo na carga seguinte.
- Atrasadas: de 8 a 15 por carga diária, com data de dias até meses antes (a mais antiga é de março).
- Valor negativo, valor não numérico (`abc`, vazio, vírgula decimal), data inválida (30/02, 31/09, mês 13, vazia, formato dd/mm/aaaa) e cartão inexistente (CRT9999, CRT9998, CRT8888): TRX900001 a TRX900081.

**Estornos**
- 160 estornos de transações existentes, a maioria dias depois da compra.
- Estornos tardios: 3 por carga diária, estornando compras de junho a setembro.
- Transação inexistente: EST0161 (TRX999999), EST0162 (TRX999998) e EST0163 (TRX999997).

## Cenários de negócio

- **Mês zerado:** CLI0010 não movimenta em junho e volta em julho.
- **Cartão cancelado para de movimentar:** clientes dos cartões cancelados aparecem com meses zerados depois do cancelamento.
- **Contas abertas durante o ano:** CLI0051 (março) a CLI0055 (julho), só aparecem no resumo a partir da abertura.
- **Picos de gasto:** CLI0004, CLI0012, CLI0023, CLI0031 e CLI0045 têm um mês com gasto bem acima do normal.
- **Estabelecimento que dispara:** o Pet Shop Amigo Fiel lidera o ranking de agosto.
