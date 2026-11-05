# Documento de Requisitos de Software (DRS) – Motor Fiscal

**Versão:** 0.3 (rascunho)\
**Autor:** Eduardo Henrique dos Santos Emilio\
**Data:** 05/11/2026

> Esta versão redefine o escopo: o Motor Fiscal é um simulador de valores de venda, sem leitura de XML. Pontos ainda abertos estão na seção 12. A versão 0.3 mantém o framework do front-end em aberto, entre React e Angular (seções 8 e 12).

## 1. Introdução e escopo

O Motor Fiscal é uma calculadora web que simula o valor de uma venda (preço, tributos e margem) a partir do tipo de operação, dos custos e dos parâmetros fiscais informados.

**Usuários.** Analista de custos e preços, vendedor e contador, que precisam testar cenários antes de fechar uma venda.

**Dentro do escopo (v1.0)**

- Simular uma venda em 8 tipos de operação (seção 2).
- Calcular 13 tributos e contribuições, inclusive IBS, CBS e IS (seção 3).
- Compor o preço a partir de custos, markup, comissão e descontos (seção 4).
- Exibir a memória de cálculo, item a item, e comparar cenários.

**Fora do escopo (v1.0)**

- Leitura de XML, emissão ou transmissão de NF-e.
- Apuração mensal de impostos, escrituração (SPED) e geração de guias.
- Cálculo de serviços (ISS).

O resultado é uma estimativa para apoio à decisão e não substitui a apuração feita pelo contador.

## 2. Tipos de venda

O tipo de venda define quais tributos incidem, quais créditos entram no custo e como o custo é formado; o motor trata o tipo como parâmetro de entrada.

| Tipo | Quando ocorre | O que muda no cálculo |
| --- | --- | --- |
| Venda simples (produto produzido) | O fabricante vende o que industrializou. | Custo = material + mão de obra + maquinário + embalagem; incidem ICMS, PIS, COFINS e, se o NCM for tributado, IPI; créditos sobre insumos. |
| Revenda (produto comprado) | Compra e vende sem transformar. | Custo de aquisição líquido dos créditos (ICMS, PIS, COFINS, conforme o regime); sem IPI na saída; ST ou monofasia podem alterar a incidência. |
| Venda para consumo final | O destinatário é o consumidor final. | DIFAL e FCP em operação interestadual; não há repasse de crédito ao cliente. |
| Venda de Kits | Produtos vendidos em conjunto (kit ou combo). | Preço e desconto rateados entre os itens, cada um com sua própria tributação. Definição a confirmar (seção 12). |
| Remessa para industrialização | Envio de material a terceiro para transformação. | Em regra não é venda: ICMS e IPI podem ter suspensão, diferimento ou não incidência conforme UF e produto; regra parametrizável. |
| Venda para exterior | Exportação. | Em regra ICMS, IPI, PIS e COFINS não incidem e os créditos de entrada são mantidos; sem DIFAL; câmbio e custos de exportação informados pelo usuário. |
| Revenda de produto importado | Compra importado e revende no mercado interno. | Custo de entrada inclui II, IPI, PIS/COFINS-Importação e ICMS-Importação, com os créditos aplicáveis; em regra, ICMS interestadual de 4% na saída. |
| Venda de produto com percentual de importação (%FCI) | Produto fabricado com parcela importada. | O Conteúdo de Importação da FCI define o ICMS interestadual: acima de 40%, 4%; até 40%, 7% ou 12% conforme a região de destino. |

## 3. Tributos e contribuições

O motor calcula 13 itens entre tributos e regimes especiais; cada um depende de parâmetros por UF, NCM, regime e data, que precisam vir de uma base atualizável (seção 7).

| Item | Esfera | Como entra no cálculo | Parâmetros necessários |
| --- | --- | --- | --- |
| ICMS | Estadual | Calculado "por dentro" sobre o valor da operação, com alíquota interna ou interestadual (7%, 12%, 4%), redução de base e crédito de entradas. | UF origem e destino, NCM, CST, alíquota, redução de base |
| IPI | Federal | Alíquota da TIPI por NCM sobre a saída do industrial; não cumulativo. A partir de 2027 é zerado para a maioria dos produtos, exceto os da Zona Franca de Manaus. | NCM, alíquota, crédito de insumos |
| PIS | Federal | Regime cumulativo (0,65%) ou não cumulativo (1,65%) sobre a receita, com crédito no não cumulativo. | Regime, alíquota, crédito |
| COFINS | Federal | Regime cumulativo (3%) ou não cumulativo (7,6%) sobre a receita, com crédito no não cumulativo. | Regime, alíquota, crédito |
| Substituição Tributária (ST) | Estadual | ICMS da cadeia seguinte: base = valor + MVA (ou pauta); ICMS-ST = base × alíquota interna do destino − ICMS próprio. | NCM/CEST, MVA, protocolo ou convênio, UF |
| Monofasia | Federal | PIS e COFINS concentrados no fabricante ou importador; o revendedor sai com alíquota zero. | NCM, indicador de monofásico, alíquotas |
| DIFAL | Estadual | Diferença entre a alíquota interna do destino e a interestadual, em venda interestadual a consumidor final. | Alíquota interna do destino, método de base conforme a UF |
| FCI | Informação | Ficha de Conteúdo de Importação; define o percentual de conteúdo importado e, com ele, a alíquota interestadual de ICMS. | Valor da parcela importada, valor da operação |
| FCP | Estadual | Adicional de ICMS para o Fundo de Combate à Pobreza; soma-se ao ICMS, à ST e ao DIFAL. | Alíquota por UF e produto |
| IBS | Estadual e municipal | Substitui ICMS e ISS de forma gradual; não cumulativo pleno. | Ano de referência, alíquota por ano e ente |
| CBS | Federal | Substitui PIS e COFINS a partir de 2027; não cumulativa plena. | Ano de referência, alíquota |
| IS (Imposto Seletivo) | Federal | Incide sobre bens e serviços prejudiciais à saúde ou ao meio ambiente, a partir de 2027. | Produto, alíquota |

**Transição da reforma.** Em 2026 vale a fase de teste (CBS 0,9% e IBS 0,1%); em 2027 a CBS substitui PIS e COFINS, entra o IS e o IPI é zerado para a maioria dos produtos (mantido para a Zona Franca de Manaus); de 2029 a 2032 o IBS sobe 10, 20, 30 e 40% enquanto ICMS e ISS caem; em 2033 a transição termina. Por isso a data de referência da simulação é um parâmetro obrigatório, já que os tributos antigos e novos coexistem.

## 4. Fatores de custo e formação de preço

O preço de venda é o custo direto dividido pela parcela do preço que sobra depois de tributos, comissão e lucro; os fatores abaixo alimentam essa conta.

| Fator | Natureza | Como entra no cálculo |
| --- | --- | --- |
| Material | Custo direto (R$/un) | Custo de compra líquido de créditos; na venda simples, soma dos insumos. |
| Mão de obra | Custo direto (R$/un) | Horas por unidade × custo-hora com encargos. |
| Maquinário | Custo direto (R$/un) | Tempo de máquina × custo-hora (depreciação, energia, manutenção). |
| Embalagem | Custo direto (R$/un) | Custo da embalagem por unidade vendida. |
| Comissão | % sobre o preço | Entra no divisor do preço. |
| Carga tributária | % sobre o preço | Resultado do motor fiscal (seção 3); entra no divisor, exceto IPI e ICMS-ST, somados por fora. |
| Desconto incondicional | % sobre a lista | Destacado na nota; reduz o valor da operação e a base dos tributos. |
| Desconto condicional | % sobre a lista | Concedido sob condição posterior (pagamento antecipado, por exemplo); não reduz a base dos tributos. |
| Lucro | % sobre o preço ou R$/un | Meta definida pelo usuário; entra no divisor. |
| Markup | Multiplicador sobre o custo | Resultado: preço ÷ custo direto; também aceito como entrada. |
| Rentabilidade | % sobre a receita líquida | Resultado: lucro ÷ receita líquida, depois de tributos, comissão e descontos. |

```latex
P = \frac{C_d}{1 - (T + K + L)} \qquad \text{Markup} = \frac{P}{C_d}
```

Onde P é o preço líquido, C\_d o custo direto (material + mão de obra + maquinário + embalagem), T a carga tributária, K a comissão e L o lucro, todos em percentual do preço. O preço de lista é P ÷ (1 − desconto incondicional).

## 5. Requisitos funcionais

O simulador recebe o tipo de venda, os custos e o contexto fiscal, e devolve preço, tributos e margem com a memória de cálculo.

**Entrada**

- **RF01** – Selecionar o tipo de venda entre os 8 da seção 2.
- **RF02** – Informar UF de origem e destino, NCM, regime tributário do vendedor (Simples, Presumido ou Real), perfil do destinatário (contribuinte ou consumidor final) e data de referência.
- **RF03** – Informar os custos por unidade (material, mão de obra, maquinário, embalagem), a comissão, o lucro ou markup desejado e os descontos.
- **RF04** – Informar o valor da parcela importada ou o %FCI quando o tipo de venda exigir.
- **RF05** – Simular vários itens na mesma venda, com rateio de preço e desconto (venda casada).

**Cálculo**

- **RF06** – Calcular os tributos aplicáveis ao tipo de venda, à UF, ao NCM e à data de referência.
- **RF07** – Calcular o preço de venda a partir dos custos e da meta de lucro (modo direto).
- **RF08** – Calcular lucro, markup e rentabilidade a partir de um preço informado (modo reverso).
- **RF09** – Aplicar créditos de entrada de ICMS, IPI, PIS e COFINS quando o regime permitir.
- **RF10** – Calcular IBS, CBS e IS conforme o ano de referência da transição.

**Saída**

- **RF11** – Exibir a memória de cálculo tributo a tributo, com base, alíquota e valor.
- **RF12** – Exibir a composição do preço: custos, tributos, comissão, descontos e lucro.
- **RF13** – Comparar cenários lado a lado (por exemplo, outra UF, outro tipo de venda ou outro ano).
- **RF14** – Exportar o resultado em CSV e JSON.

**Parâmetros fiscais**

- **RF15** – Buscar alíquotas e regras por UF, NCM, regime e data (seção 7).
- **RF16** – Permitir que o usuário substitua uma alíquota manualmente, com aviso visível no resultado.
- **RF17** – Sinalizar quando um parâmetro estiver ausente ou fora da vigência.

## 6. Regras de negócio e ordem de cálculo

O motor calcula em sete passos fixos, porque IPI, ST e DIFAL dependem do preço já formado.

1. Resolver o contexto (tipo, UF, NCM, regime, data) e buscar os parâmetros fiscais.
2. Apurar o custo direto e abater os créditos de entrada.
3. Montar o divisor do preço: ICMS (com FCP), PIS e COFINS (ou CBS e IBS, conforme a data), IS, comissão e lucro.
4. Calcular o preço líquido P.
5. Calcular por fora os tributos que não entram no divisor: IPI sobre P; depois ICMS-ST e DIFAL.
6. Aplicar os descontos para chegar ao preço de lista e ao valor final.
7. Derivar markup e rentabilidade e registrar a memória de cálculo.

**Regras**

- **RN01** – ICMS interestadual: 7% ou 12% conforme as regiões de origem e destino; 4% para importados e para produtos com conteúdo de importação acima de 40%.
- **RN02** – Desconto incondicional reduz a base dos tributos; desconto condicional não reduz.
- **RN03** – Se a soma dos percentuais do divisor chegar a 100% ou mais, a simulação é rejeitada com mensagem explicativa.
- **RN04** – Produto monofásico: PIS e COFINS com alíquota zero na revenda.
- **RN05** – ICMS-ST = base ST × alíquota interna do destino − ICMS próprio, com base ST = (valor + IPI + outras despesas) × (1 + MVA).
- **RN06** – DIFAL só se aplica a venda interestadual a consumidor final.
- **RN07** – Exportação: sem ICMS, IPI, PIS, COFINS e DIFAL, mantidos os créditos de entrada.
- **RN08** – A data de referência decide quais tributos incidem, conforme a transição de 2026 a 2033.
- **RN09** – Valores monetários em precisão decimal, com uma única regra de arredondamento, testada.
- **RN10** – Nenhuma alíquota fica fixa no código: todas vêm da base de parâmetros ou do usuário.

## 7. Origem dos dados fiscais

Alíquotas e códigos de tributação ficam em um banco PostgreSQL com data de vigência; o usuário só substitui valores manualmente e a API externa fica como evolução. O banco guarda apenas parâmetros fiscais, nunca simulações.

| Opção | Vantagens | Limitações | Papel recomendado |
| --- | --- | --- | --- |
| Banco de dados | Atualiza sem novo deploy; guarda vigência (importante na transição 2026–2033); consulta por UF, NCM e regime. | Alguém precisa manter e validar a carga inicial. | **Fonte principal** |
| DLL ou biblioteca embutida | Rápida e sem infraestrutura. | Cada mudança de alíquota exige recompilar e publicar; amarra a stack (uma DLL .NET não roda em back-end TS). | Só para o **código de cálculo**, nunca para os dados |
| API externa | Dados atualizados por terceiros. | Dependência e custo de terceiros; risco de indisponibilidade; fica difícil reproduzir uma simulação passada. | **Evolução** (v2): sincronizar o banco |
| Usuário informa | Zero manutenção; cobre casos especiais (regime especial, benefício fiscal). | Erro de digitação; resultado sem rastreabilidade. | **Substituição manual** sinalizada no resultado (RF16) |

**Modelo mínimo de parâmetros.** Uma linha por regra, com: tributo, UF de origem, UF de destino, NCM, regime, tipo de operação, alíquota ou MVA, início e fim da vigência e fonte legal. Assim o motor escolhe a linha vigente na data de referência e a memória de cálculo mostra qual regra foi usada.

**Carga inicial (v1).** Arquivos de seed versionados no repositório (CSV ou JSON) carregados no banco, com as fontes oficiais citadas em cada linha. A sincronização automática fica para a v2.

## 8. Arquitetura e tecnologias

A stack está definida, exceto o framework do front-end, ainda em avaliação entre React e Angular: PostgreSQL para os parâmetros fiscais, C# com ASP.NET Core para o motor de cálculo e a API, Tailwind CSS no estilo e xUnit nos testes.

| Camada | Tecnologia | Por que |
| --- | --- | --- |
| Banco de dados | PostgreSQL (driver Npgsql, com EF Core ou Dapper) | Guarda alíquotas e códigos de tributação com vigência; o tipo `daterange` impede vigências sobrepostas. |
| Back-end | C# com ASP.NET Core | Tipo `decimal` nativo para dinheiro; motor de cálculo isolado em biblioteca; API REST fina. |
| Front-end | **Opção A:** React com TypeScript e Vite. **Opção B:** Angular com TypeScript e Angular CLI. Nas duas, Tailwind CSS. | Formulários que mudam por tipo de venda e comparação de cenários; stack pedida em vagas; Tailwind acelera o estilo. Escolha em aberto (seção 12). |
| Testes | xUnit | Casos por tipo de venda e por tributo com `[Theory]`, concentrados no motor de cálculo. |

**Camadas**

1. **Front-end (React ou Angular, com TypeScript e Tailwind CSS):** formulários, comparação de cenários e exibição da memória de cálculo.
2. **API REST:** valida a entrada e expõe os endpoints de simulação e de consulta de parâmetros.
3. **Motor de cálculo:** biblioteca separada, sem acesso a banco nem a rede, com funções puras e testáveis.
4. **Repositório de parâmetros fiscais:** PostgreSQL com alíquotas e códigos de tributação, consultado por UF, NCM, regime e data (via Npgsql, com EF Core ou Dapper); não guarda simulações.

Manter o motor isolado das camadas 2 e 4 é a decisão que mais pesa: ele concentra as regras fiscais, é o que mais muda com a legislação e pode ser testado sem subir API nem banco.

**Front-end: React ou Angular (em avaliação).** A escolha afeta só a camada 1: o motor de cálculo, a API REST, o PostgreSQL e os testes em xUnit são iguais nas duas opções.

| Critério | Opção A: React (Vite) | Opção B: Angular (CLI) |
| --- | --- | --- |
| Natureza | Biblioteca de interface; roteamento, formulários e estado vêm de bibliotecas escolhidas por nós. | Framework completo, com roteamento, formulários reativos, cliente HTTP e injeção de dependência incluídos. |
| Formulários que mudam por tipo de venda (RNF06) | React Hook Form com Zod, por exemplo. | Reactive Forms nativos. |
| Afinidade com C# | Menor: estilo funcional, com mais liberdade de organização. | Maior: classes, serviços injetados e tipagem forte. |
| Mercado de vagas | Maior volume geral. | Forte em empresas grandes e sistemas corporativos. |
| Curva de aprendizado | Menor no início, com mais decisões de arquitetura a tomar. | Maior no início, com mais convenções prontas. |
| Build e execução | Vite (`npm run dev`). | Angular CLI (`ng serve`). |
| Tailwind CSS | Suportado. | Suportado. |

A decisão deve sair antes do marco 1 (seção 11), que entrega a tela mínima. Depois dela, esta seção e a seção 12 passam a citar só o framework escolhido.

## 9. Requisitos não funcionais

A prioridade é que o cálculo seja correto, rastreável e fácil de atualizar quando a legislação mudar.

- **RNF01 – Precisão:** todo valor monetário e toda alíquota em tipo decimal, nunca ponto flutuante binário.
- **RNF02 – Rastreabilidade:** cada resultado guarda a regra, a vigência e a fonte usadas, para reproduzir a simulação depois.
- **RNF03 – Testabilidade:** cobertura mínima de 90% no motor de cálculo, em xUnit, com um caso de teste por tipo de venda e por tributo.
- **RNF04 – Desempenho:** uma simulação com até 50 itens responde em menos de 1 segundo.
- **RNF05 – Manutenibilidade:** regras fiscais isoladas do restante do código, sem alíquota fixa; adicionar um tributo ou um ano da transição não exige reescrever o motor.
- **RNF06 – Usabilidade:** formulário que só mostra os campos exigidos pelo tipo de venda escolhido; mensagens de erro em português.
- **RNF07 – Segurança:** validação de toda entrada na API; nenhum dado de cliente é necessário nem armazenado na v1.
- **RNF08 – Transparência:** o resultado informa que é uma estimativa e destaca qualquer alíquota informada manualmente.

## 10. Critérios de aceitação

A v1.0 é aceita quando os cenários abaixo produzem o resultado esperado, conferido contra uma planilha de referência.

| Cenário | Resultado esperado |
| --- | --- |
| Custo direto R$ 100, ICMS 18%, PIS 1,65%, COFINS 7,6%, comissão 5%, lucro 10% | Divisor = 1 − 0,4225 = 0,5775; preço líquido ≈ R$ 173,16; markup ≈ 1,7316. |
| Soma dos percentuais do divisor igual ou maior que 100% | Simulação rejeitada com mensagem explicando o motivo. |
| Venda para exterior | Memória de cálculo sem ICMS, IPI, PIS, COFINS e DIFAL. |
| Venda interestadual a consumidor final | DIFAL e FCP calculados; nenhum crédito repassado ao cliente. |
| Produto com conteúdo de importação de 45% em venda interestadual | ICMS interestadual de 4%. |
| Produto com conteúdo de importação de 30% em venda interestadual | ICMS de 7% ou 12%, conforme a região de destino. |
| Mesma venda com data de referência em 2026 e em 2027 | 2026 exibe CBS 0,9% e IBS 0,1% da fase de teste; 2027 troca PIS e COFINS pela CBS. |
| Desconto incondicional de 5% | Base dos tributos reduzida em 5%. |
| Desconto condicional de 5% | Base dos tributos inalterada. |
| Alíquota substituída manualmente | Resultado destaca a substituição e a alíquota original da base. |
| Parâmetro sem vigência na data informada | Aviso de parâmetro ausente, sem resultado silencioso. |

## 11. Plano de entregas

O projeto é longo, então cada marco entrega algo utilizável e termina com um critério de saída verificável, começando pelo núcleo de preço e deixando os tributos mais complexos para depois.

1. **Núcleo de preço e tela mínima.** Custo direto, divisor, preço líquido, markup e rentabilidade, com percentuais informados à mão. *Saída:* o exemplo de R$ 173,16 passa.
2. **Tributos de base.** ICMS interno e interestadual, PIS, COFINS e IPI, com créditos, nos tipos venda simples e revenda. *Saída:* resultados conferem com a planilha de referência.
3. **Base de parâmetros.** Banco com vigência, carga inicial por seed, consulta por UF, NCM, regime e data, e substituição manual. *Saída:* nenhuma alíquota fixa no código.
4. **Operações especiais.** ST, monofasia, DIFAL, FCP, venda a consumidor final e exportação. *Saída:* cenários de aceitação desses itens passam.
5. **Importação e remessas.** FCI e %FCI, revenda de importado, remessa para industrialização e venda casada. *Saída:* cenários de 45% e 30% de conteúdo importado passam.
6. **Reforma tributária.** IBS, CBS e IS com data de referência e transição. *Saída:* a mesma venda simulada em 2026, 2027 e 2029 muda como previsto.
7. **Comparação e exportação.** Cenários lado a lado e exportação em CSV e JSON. *Saída:* comparação de dois cenários com memória de cálculo completa.


## 12. Pontos em aberto

Cinco decisões ainda mudam o escopo ou o desenho: o framework do front-end precisa de resposta antes do marco 1 e as demais, antes do marco 2.

- [ ] **Regime do vendedor:** a v1 cobre Lucro Presumido e Lucro Real, ou inclui também o Simples Nacional, que tem cálculo próprio?
- [ ] **Definições de lucro, markup e rentabilidade:** a seção 4 adota lucro como meta sobre o preço, markup como preço ÷ custo e rentabilidade sobre a receita líquida; confirmar ou ajustar.
- [x] **Persistência:** decidido: o banco guarda só alíquotas e códigos de tributação; simulações não são gravadas e a comparação de cenários vive só na sessão do navegador.
- [x] **Stack (exceto front-end):** decidido: PostgreSQL, C# com ASP.NET Core, Tailwind CSS e xUnit nos testes (seção 8).
- [ ] **Framework do front-end:** React (com Vite) ou Angular (com Angular CLI)? A seção 8 compara as duas opções; motor, API e banco não mudam com a escolha.

## 13. Fontes

O cronograma da reforma vêm da página abaixo; as demais regras (alíquotas de PIS, COFINS e ICMS, mecânica de ST e DIFAL, regras por tipo de venda) foram escritas a partir de conhecimento geral e devem ser conferidas com a legislação e com um contador antes de entrar na base de parâmetros.

- [Cronograma da Reforma Tributária, Lexos Help Center](https://intercom.help/lexos-help/pt-BR/articles/12735843-cronograma-da-reforma-tributaria): fases de 2026 a 2033, alíquotas de teste de CBS e IBS e tratamento do IPI em 2027.
