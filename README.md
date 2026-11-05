# Simulador de Venda Fiscal

Calculadora web que simula o valor de uma venda no Brasil: preço de venda, tributos e margem, a partir do tipo de operação, dos custos e dos parâmetros fiscais informados.

> **Status:** em planejamento. O escopo está especificado no DRS (Documento de Requisitos de Software) e o desenvolvimento ainda não começou.

## O que o projeto faz

- Simula uma venda em 8 tipos de operação.
- Calcula tributos e contribuições com base em UF, NCM, regime tributário e data de referência.
- Forma o preço a partir de custos, comissão, lucro e descontos, nos dois sentidos: da meta ao preço e do preço à rentabilidade.
- Mostra a memória de cálculo tributo a tributo e compara cenários lado a lado.
- Não lê XML nem emite nota fiscal, e não grava simulações.

> O resultado é uma estimativa para apoio à decisão e não substitui a apuração feita por um contador.

### Tipos de venda

| Tipo | Descrição |
| --- | --- |
| Venda simples | Produto produzido pelo próprio vendedor |
| Revenda | Produto comprado para revender |
| Venda para consumo final | Destinatário é o consumidor final |
| Venda de Kits | Produtos vendidos em conjunto (kit ou combo) |
| Remessa para industrialização | Envio de material a terceiro para transformação |
| Venda para exterior | Exportação |
| Revenda de produto importado | Compra importado e revende no mercado interno |
| Venda com percentual de importação (%FCI) | Produto com parcela importada |

### Tributos e fatores de custo

- **Tributos:** ICMS, IPI, PIS, COFINS, Substituição Tributária, Monofasia, DIFAL, FCI, FCP, InovarAuto, IBS, CBS e IS.
- **Custos e preço:** material, mão de obra, maquinário, embalagem, comissão, carga tributária, lucro, markup, rentabilidade, descontos condicional e incondicional, frete, seguroo e outras despesas.

A transição da Reforma Tributária (2026 a 2033) é tratada pela **data de referência** da simulação, já que tributos antigos e novos coexistem nesse período.

## Stack

| Camada | Tecnologia |
| --- | --- |
| Banco de dados | PostgreSQL |
| Back-end | C# com ASP.NET Core |
| Front-end | React, Vite e Tailwind CSS |
| Testes | xUnit |

## Arquitetura

1. **Front-end (React):** formulários que mudam conforme o tipo de venda, comparação de cenários e memória de cálculo.
2. **API REST (ASP.NET Core):** valida a entrada e expõe os endpoints de simulação e de consulta de parâmetros.
3. **Motor de cálculo (biblioteca C#):** regras fiscais em funções puras, sem acesso a banco nem a rede, o que permite testá-las isoladamente.
4. **PostgreSQL:** guarda apenas alíquotas e códigos de tributação, com data de vigência. Nenhuma simulação é gravada.

Decisões que valem destacar:

- Valores monetários e alíquotas usam `decimal`, nunca ponto flutuante binário.
- Nenhuma alíquota fica fixa no código: vem do banco ou é informada pelo usuário, com aviso visível no resultado.
- O banco é carregado inicialmente por arquivos de seed versionados, com a fonte legal de cada regra.

## Estrutura prevista

```text
simulador-venda-fiscal/
├── docs/                          # DRS e decisões de projeto
├── src/
│   ├── SimuladorFiscal.Core/      # motor de cálculo (sem I/O)
│   ├── SimuladorFiscal.Data/      # acesso ao PostgreSQL
│   └── SimuladorFiscal.Api/       # ASP.NET Core
├── tests/
│   └── SimuladorFiscal.Core.Tests/  # xUnit
└── web/                           # React + TypeScript + Vite + Tailwind
```

## Como rodar (previsto)

Os comandos abaixo valem quando o projeto for criado; ajuste os nomes conforme a estrutura final.

**Pré-requisitos:** .NET SDK, Node.js e PostgreSQL (local ou em Docker).

```bash
# 1. Banco de dados e carga inicial dos parâmetros fiscais
#    (configure a connection string em src/SimuladorFiscal.Api)

# 2. API
dotnet run --project src/SimuladorFiscal.Api

# 3. Front-end
cd web
npm install
npm run dev
```

## Testes

```bash
dotnet test
```

Cada tipo de venda e cada tributo tem casos de teste próprios, e a meta é cobertura mínima de 90% no motor de cálculo. Um exemplo de referência: com custo direto de R$ 100, ICMS de 18%, PIS de 1,65%, COFINS de 7,60%, comissão de 5,00% e lucro de 10,00%, o preço líquido é de aproximadamente R$ 173,16.

## Roadmap

1. Núcleo de preço e tela mínima.
2. Tributos de base: ICMS, PIS, COFINS e IPI, com créditos.
3. Base de parâmetros no PostgreSQL, com vigência e substituição manual.
4. Operações especiais: ST, monofasia, DIFAL, FCP, consumo final e exportação.
5. Importação e remessas: FCI, %FCI, revenda de importado, industrialização e venda casada.
6. Reforma Tributária: IBS, CBS e IS com transição por data.
7. Comparação de cenários e exportação em CSV e JSON.

## Autor

Eduardo Henrique dos Santos Emilio

## Licença

AGPL-3.0.
