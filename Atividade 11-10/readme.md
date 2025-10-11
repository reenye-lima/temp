# Funções DAX (com exemplos)

> Funções vistas na aula do dia 11/10/2025

## Premissas do modelo utilizado
- Tabelas no exemplo:
  - **Chamados**: `[IDChamado]`, `[DataAbertura]`, `[DataFechamento]`, `[IDCliente]`, `[IDTécnico]`, `[TipoProblema]`, `[Status]`
  - **Clientes**: `[IDCliente]`, `[Cliente]`, `[Setor]`
  - **Técnicos**: `[IDTécnico]`, `[Nome]`, `[Especialidade]`

- Relacionamentos:

    |Tabela      |Cardinalidade|Tabela       |Campo Chave|
    |------------|-------------|-------------|-----------|
    |**Clientes**| 1:N         | **Chamados**| IDCliente |
    |**Técnicos**| 1:N         | **Chamados**| IDTécnico |

---

## COUNT
**O que faz:** Conta células **não em branco** de uma coluna (numérica ou texto).

```DAX
Total de Chamados = COUNT(Chamados[IDChamado])
```

> Dica: Para contar linhas independentemente de nulos por coluna, prefira `COUNTROWS` com uma tabela filtrada.

---

## COUNTROWS
**O que faz:** Conta linhas de uma **tabela**.

```DAX
Total de Chamados = COUNTROWS(Chamados)
```
---

## DISTINCTCOUNT
**O que faz:** Conta **valores distintos** em uma coluna.

```DAX
Clientes Únicos = DISTINCTCOUNT(Chamados[IDCliente])
```
---

## CALCULATE
**O que faz:** **Recalcula** uma expressão **modificando o contexto de filtro**.

```DAX
Chamados Fechados = 
CALCULATE(
    COUNT(Chamados[IDChamado]),
    Chamados[Status] = "fechado"
)
```
> Dica: `CALCULATE` aceita filtros diretos (coluna = valor), expressões booleanas e funções de filtro (`FILTER`, `ALL`, `REMOVEFILTERS`, etc.).  
> **Regra de ouro:** primeiro argumento é a **expressão** (geralmente uma medida), depois vem os **filtros**.

---

## DAY / MONTH / YEAR
**O que fazem:** Extraem dia, mês e ano de uma data (retornam inteiros).

```DAX
Dia da Venda = DAY(Chamados[DataAbertura])        -- coluna calculada
Mes da Venda = MONTH(Chamados[DataAbertura])      -- coluna calculada
Ano da Venda = YEAR(Chamados[DataAbertura])       -- coluna calculada
```

> Recomenda-se manter uma tabela Calendário completa (com colunas de dia, mês, ano, nome do mês, etc.) para facilitar filtros e hierarquias.

---

## DATEDIFF
**O que faz:** Calcula diferença entre duas datas em unidades escolhidas.

Sintaxe: `DATEDIFF(DataInicial, DataFinal, Unidade)`

Unidades comuns: `DAY`, `MONTH`, `YEAR`, `HOUR`, `MINUTE`, `SECOND`.

```DAX
Duração do Chamado = 
DATEDIFF(
    Chamados[DataAbertura],
    Chamados[DataFechamento],
    DAY
)
```
---

## DATESINPERIOD
**O que faz:** Retorna um conjunto de **datas contíguas** relativo a uma **data final** no contexto.

Sintaxe:  
`DATESINPERIOD(<ColunaData>, <DataFinal>, <NúmeroDeIntervalos>, <Unidade>)`

Ex.: últimos 30 dias desde último chamado aberto:

```DAX
DATESINPERIOD(
    Chamados[DataAbertura],
    MAX(Chamados[DataAbertura]),
    -30,
    DAY
)

```
---

## IF
**O que faz:** Retorna resultados condicionais.

Sintaxe básica: `IF( Condição, ResultadoSeVerdadeiro, ResultadoSeFalso )`

```DAX
Avaliação por Tempo = 
IF(
    Chamados[Duração do Chamado] > 5,
    "Ruim",
    "Bom"
)
```

```DAX
Avaliação por Tempo = 
IF(
    Chamado[Duração do Chamado] > 5,
    "Ruim",
    IF(
        Chamado[Duração do Chamado] > 3,
        "Médio",
        "Bom"
    )
)
```
---

## Combinações úteis (mini-receitas)

**1) Contagem de chamados no mês atual**
```DAX
Chamados Mes Atual = 
CALCULATE(
    COUNTROWS(Chamado),
    MONTH(Chamado[DataAbertura]) = MONTH(TODAY()),
    YEAR(Chamado[DataAbertura]) = YEAR(TODAY())
)
```

**2) Contagem de chamados "Em Andamento" no mês atual**
```DAX
Chamados Mes Atual = 
CALCULATE(
    COUNTROWS(Chamado),
    MONTH(Chamado[DataAbertura]) = MONTH(TODAY()),
    YEAR(Chamado[DataAbertura]) = YEAR(TODAY()),
    Chamados[Status] = "Em Andamento"
)
```

**3) Quantidade de chamados abertos nos últimos 30 dias desde o último chamado aberto**
```DAX
Chamados 30 dias = 
CALCULATE(
    COUNTROWS(Chamado),
    DATESINPERIOD(
        Chamado[DataAbertura],
        MAX(Chamado[DataAbertura]),
        -30,
        DAY
    )
)
```