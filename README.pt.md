#  Warehouse KPI Dashboard

Este projeto apresenta um dashboard interativo desenvolvido no Power BI para monitoramento de indicadores logísticos em operações de armazéns. A solução foi construída com base em dados simulados e tem como objetivo demonstrar a capacidade de consolidar, modelar e visualizar informações críticas para a tomada de decisão operacional.

---

##  Objetivo

Fornecer uma visão clara e confiável do desempenho dos armazéns, com foco em:

- Produtividade por operação e turno
- Acuracidade de inventário por SKU e unidade
- Custos operacionais por armazém e período
- Nível de serviço e cumprimento de SLA (OTIF)

---

##  Tecnologias utilizadas

- **Power BI**: Modelagem de dados, DAX, Power Query e visualizações interativas
- **Excel / CSV**: Simulação e estruturação dos dados operacionais
- **SQL (conceitual)**: Base para integração e extração de dados
- **Python (conceitual)**: Aplicações futuras para automação e predição

---

##  Estrutura dos dados

Os dados estão organizados em quatro arquivos principais:

| Arquivo | Descrição |
|--------|-----------|
| `warehouse_operations.csv` | Volume de itens processados, tempo de operação e erros por turno |
| `inventory_accuracy.csv` | Comparação entre estoque físico e sistema, com cálculo de acuracidade |
| `operational_costs.csv` | Custos fixos e variáveis por armazém e mês |
| `service_level.csv` | Tempo real vs. SLA por pedido, com indicador OTIF |

FactServiceLevel — Grain: uma linha por Pedido por Data (colunas chave: Pedido, Data), contendo SLA (min), Tempo Real (min) e OTIF.

FactOperations (ou FactExceptions) — Grain: uma linha por Operação por Turno por Data (colunas chave: Operação, Turno, Data), contendo Tempo Processado (min) e Erros.

TableExpectedCost (FactOperational_Costs) — Grain: uma linha por Armazém por Mês/Data (colunas chave: Armazém, Month ou Data agregada por mês), contendo Custo Valor €, Custo Total € e Itens Processados.

FactInventory_Accuracy — Grain: uma linha por Armazém por SKU por Data (colunas chave: Armazem, SKU, Data), contendo Quantidade Sistema, Quantidade Física e Acuracidade (%).


##  Indicadores calculados

- **Produtividade Média** = Itens Processados ÷ Tempo Total  
- **Taxa de erro (%)** = (Erros ÷ Itens Processados) × 100
- **Volume Total Processado** = Soma do Total de Itens Processados
- **Acuracidade de inventário (%)** = (Qtd. Física ÷ Qtd. Sistema) × 100  
- **Custo por item** = Custo Total ÷ Itens Processados  
- **OTIF (%)** = % de pedidos entregues dentro do SLA


##  Fórmulas DAX utilizadas
As medidas abaixo foram desenvolvidas para calcular indicadores logísticos com precisão e flexibilidade. Elas demonstram domínio de funções como CALCULATE, DIVIDE, AVERAGEX e uso de contexto com ALLEXCEPT.

###  Acuracidade Média por Armazém
````
Acuracidade Média por Armazém = 
VAR TotalSistema = CALCULATE(
    SUM(FactInventory_Accuracy[Quantidade Sistema]), 
    ALLEXCEPT(FactInventory_Accuracy, FactInventory_Accuracy[Armazem])
)
VAR TotalFisica = CALCULATE(
    SUM(FactInventory_Accuracy[Quantidade Física]), 
    ALLEXCEPT(FactInventory_Accuracy, FactInventory_Accuracy[Armazem])
)
RETURN DIVIDE(TotalFisica, TotalSistema, 0)
Calcula a acuracidade por armazém, preservando o contexto de agrupamento.
````
###  Custo por Item
````
Custo por Item = 
DIVIDE(
    SUM(FactOperational_Costs[Custo Total]), 
    SUM(FactOperational_Costs[Itens Processados])
)
Mede o custo unitário por item processado.
````
###  Volume Total Processado
````
Volume Total Processado = 
SUM('FactOperations'[Itens Processados])
Soma total dos itens movimentados.
````
###  Produtividade Média
```
Produtividade Média = 
AVERAGEX(
    'FactOperations', 
    'FactOperations'[Itens Processados] / FactOperations[Tempo Total (min)]
)
Avalia a produtividade média por operação com base no tempo.
````
###  Taxa de Erro (%)
````
Taxa de Erro (%) = 
DIVIDE(
    SUM('FactOperations'[Erros]), 
    SUM(FactOperations[Itens Processados])
)
Calcula a proporção de erros em relação ao volume processado.
````
###  OTIF (%) por Mês
````
OTIF (%) por Mês = 
VAR TotalPedidos = COUNTROWS(FactService_Level)
VAR EntregasNoPrazo = CALCULATE(
    COUNTROWS(FactService_Level),
    FactService_Level[OTIF] = "Sim"
)
RETURN DIVIDE(EntregasNoPrazo, TotalPedidos)
Mede o percentual de pedidos entregues dentro do prazo (On Time In Full).
````
###  Soma Total Sistema
````
Soma total sistema = 
SUM(FactInventory_Accuracy[Quantidade Sistema])
````
## Documentação de Duplicidades — FactOperations
###  Objetivo
Identificar e tratar duplicidades na tabela fato FactOperations, garantindo granularidade correta por operação, turno, data e armazém.

###  Chaves duplicadas encontradas
As seguintes chaves compostas apresentaram duplicidade (2 registros cada):

Picking|Manhã|2025-06-09

Picking|Noite|2025-07-11

Recebimento|Tarde|2025-02-12

Packing|Manhã|2025-04-29

Picking|Noite|2025-01-03

###  Investigação realizada
Criação da coluna FactOperationsKey Composta por Operação|Turno|Data, posteriormente expandida para incluir Armazem.

Verificação de duplicidade via DAX Medida criada para listar chaves com mais de uma ocorrência:

````
DuplicateKeysList = 
VAR DuplicateKeys =
    FILTER(
        GROUPBY(
            FactOperations,
            FactOperations[FactOperationsKey],
            "Cnt", COUNTX(CURRENTGROUP(), 1)
        ),
        [Cnt] > 1
    )
RETURN
    CONCATENATEX(DuplicateKeys, FactOperations[FactOperationsKey] & " (" & [Cnt] & " vezes)", ", ")
````
Inspeção das linhas duplicadas Verificou-se que os registros representavam o mesmo evento, com valores numéricos que podiam ser somados.

###  Correção aplicada
Agregação por FactOperationsKey no Power Query:

Somados: Itens Processados, Tempo Total (min), Erros

Preservados: Operação, Turno, Data, Armazem (primeira ocorrência não nula)

Coluna auxiliar OriginalCount criada para auditoria

Remoção de colunas temporárias e tabelas de teste Após validação, elementos usados apenas para diagnóstico foram excluídos do modelo.

###  Validação pós-correção
Medidas comparativas:

Rows Count = COUNTROWS(FactOperations)

DistinctKey Count = DISTINCTCOUNT(FactOperations[FactOperationsKey])

Acuracidade de Inventário ajustada de 96,85% → 96,80%

Cartão múltiplo com todas as medidas foi usado para comparar valores antes e depois da correção.

###  Observações finais
A diferença de 0,05 p.p. na acuracidade foi considerada aceitável.

A modelagem agora garante 1 linha por evento operacional, respeitando o grain definido.

A lógica de agregação foi documentada e validada com medidas e visuais.

##  Visualizações

O dashboard inclui:

- Gráficos de linha e barras por operação e armazém  
- Cartões de KPIs com indicadores  
- Filtros dinâmicos por data, unidade, operação e SKU  
  
---

##  Próximos passos

- Integração com dados reais via SQL Server ou API  
- Automação de atualizações com Power Automate  
- Aplicação de modelos preditivos com Python (forecast de demanda, risco de ruptura)  
- Expansão para indicadores de transporte e armazenagem avançada

---
##  Como usar

### 1. Clone este repositório:
   ```bash
   git clone https://github.com/DeboraKlein/Warehouse-KPI-Dashboard.git
````
### 2. Abra o arquivo .pbix no Power BI Desktop

### 3. Navegue pelas páginas do dashboard para explorar os KPIs

Os dados utilizados são simulados e estão disponíveis na pasta /data


---

##  Link do Dashboard  
###  [Acesse o dashboard publicado aqui](https://app.powerbi.com/view?r=eyJrIjoiNjA5NDRhYmMtMmZmNC00MmI5LTk1MGYtMWNiZWNlMTQ5NjZjIiwidCI6IjY1OWNlMmI4LTA3MTQtNDE5OC04YzM4LWRjOWI2MGFhYmI1NyJ9)





##  Exemplos de visualizações

![Dashboard - Visão Geral](https://github.com/user-attachments/assets/029dcdf7-fb70-4744-a408-f5692a86ccf5)
![Dashboard - Inventário e SLA](https://github.com/user-attachments/assets/679a8d4f-8a06-425f-aafe-75c9e989f19c)

##  Projetos relacionados

- [Logistics Dashboard](https://github.com/DeboraKlein/Logistics-Dashboard): Análise de desempenho de frota e entregas



##  Sobre mim

Este projeto foi desenvolvido por **Debora Klein**, Analista de Dados com experiência em BI, planejamento financeiro e inteligência operacional.  
 [linkedin.com/in/débora-klein](https://linkedin.com/in/débora-klein)  
 [github.com/DeboraKlein](https://github.com/DeboraKlein)

---

