# Dashboard-de-Desempenho-e-Viabilidade-Operacional-EV-vs.-ICE
Este projeto consiste em um dashboard interativo desenvolvido no **Power BI** para comparar o desempenho operacional, a viabilidade financeira e o impacto ambiental entre veículos elétricos (**EV**) e veículos movidos a combustão interna (**ICE**).

A solução processa dados operacionais de frotas e viagens (coletados via dados estruturados em planilhas Excel) e aplica modelagem dimensional e inteligência de negócios para extrair insights estratégicos para gestão de mobilidade corporativa.

---

##  Visão Geral do Dashboard

O painel permite analisar em tempo real as diferenças térmicas, energéticas e financeiras entre as tecnologias de propulsão:

- **Indicadores Principais (KPIs):**
  - **Consumo Médio Dinâmico:** Exibição adaptativa de consumo por distância em `kWh/100 km` para EVs e `L/100 km` para veículos a combustão.
  - **Economia por Frenagem Regenerativa:** Valor financeiro recuperado em R$ através do reaproveitamento da energia cinética dos veículos elétricos.
  - **Autonomia Média (km):** Capacidade de alcance médio da frota cadastrada.
  - **CO₂ Evitado (kg):** Volume de emissões de poluentes evitados com o uso da frota elétrica em substituição aos motores diesel/gasolina.

- **Visualizações de Negócio:**
  - **Gráfico Comparativo de Custo Operacional:** Análise lado a lado do custo de abastecimento/carregamento por tipo de veículo (calculado com base em tarifas de energia R$/kWh e combustível R$/L).
  - **Consumo Bruto vs. Energia Regenerada por Viagem:** Gráfico de colunas empilhadas discriminando o volume de consumo e o retorno por frenagem regenerativa em cada rota realizada.

---

##  Arquitetura e Modelagem de Dados

O projeto foi estruturado seguindo o padrão de modelagem **Star Schema** para otimização de performance e clareza no relacionamento dos dados:

1. **`Dimensão de Veículos` (Tabela 2):** Contém os metadados dos veículos, como ID, Modelo, Tipo de Propulsão (`EV` vs `ICE`) e Autonomia.
2. **`Dados operacionais diários` (Tabela 1):** Contém os registros das rotas realizadas, incluindo Distância Percorrida, Consumo Bruto, Energia Regenerada e Emissões de CO₂.

**Relacionamento:** `Tabela2[ID_Veiculo]`  → `Tabela1[ID_Veiculo]` .

---

## Regras de Negócio e Fórmulas DAX

As métricas do projeto utilizam regras matemáticas avançadas em DAX com suporte a contextos de filtro dinâmicos:

### 1. Custo Operacional Total de Abastecimento (R$)
$$\text{Custo EV} = (\text{Consumo Bruto} - \text{Energia Regenerada}) \times \text{Tarifa (R\$/kWh)}$$$$\text{Custo ICE} = \text{Consumo Bruto} \times \text{Preço Combustível (R\$/L)}$$
```dax
Custo_Total_R$ = 
SUMX(
    Tabela2,
    VAR Tipo = Tabela2[Tipo]
    VAR Consumo = CALCULATE(SUM(Tabela1[Consumo Bruto_L_ou_(kWh)]))
    VAR Regeneracao = CALCULATE(SUM(Tabela1[Energia_Regenerada_(kWh)]))
    VAR ConsumoLiquido = Consumo - Regeneracao
    RETURN
    IF(
        Tipo = "EV",
        ConsumoLiquido * 0.85,    -- Tarifa média de energia (R$/kWh)
        Consumo * 5.90             -- Preço médio do combustível (R$/L)
    )
)
``` 
## 2.Custo Comparativo Dinâmico
Permite visualizar as duas barras (EV vs ICE) simultaneamente na visão geral e filtrar individualmente ao selecionar o tipo de veículo no painel:
```dax
Custo_Total_Comparativo_R$ = 
IF(
    ISFILTERED(Tabela2[Tipo]),
    [Custo_Total_R$],
    CALCULATE([Custo_Total_R$], ALL(Tabela2[Tipo]))
)
```
## 3. Redução de Pegada de Carbono (CO₂ Evitado)
Calcula a diferença entre as emissões teóricas do motor a combustão e as emissões reais nulas da frota elétrica:
```dax
CO2_Evitado_kg = 
VAR CO2_ICE = CALCULATE(SUM(Tabela1[Emissao CO2 (kg)]), Tabela2[Tipo] = "ICE")
VAR CO2_EV = CALCULATE(SUM(Tabela1[Emissao CO2 (kg)]), Tabela2[Tipo] = "EV")
VAR TotalEvitado = CO2_ICE - CO2_EV
RETURN
IF(
    HASONEVALUE(Tabela2[Tipo]),
    IF(SELECTEDVALUE(Tabela2[Tipo]) = "EV", TotalEvitado, 0),
    TotalEvitado
)
```
<img width="546" height="444" alt="image" src="https://github.com/user-attachments/assets/ceeca15a-8507-4db5-9821-a768c93c5492" />


# Principais Insights do Projeto
- **Eficiência de Custos:** Embora os veículos elétricos possuam um custo de aquisição mais elevado, a operação demonstra uma redução de ~68% nos custos de rodagem por viagem em comparação com os modelos a combustão.

- **Reaproveitamento Energético:** A regeneração em rotas urbanas e de desaceleração reduz significativamente o consumo líquido dos EVs, gerando economia financeira direta.

- **Sustentabilidade:** A substituição das rotas por veículos elétricos eliminou 100% das emissões diretas de CO₂ das viagens analisadas.

 ## Tecnologias Utilizadas
- **Power BI Desktop:** Modelagem de dados, DAX dinâmico e desenvolvimento visual.

- **Microsoft Excel:** Estruturação e tratamento inicial dos dados operacionais da frota.

- **DAX (Data Analysis Expressions):** Criação de inteligência de tempo, agregação condicional e context switches.
