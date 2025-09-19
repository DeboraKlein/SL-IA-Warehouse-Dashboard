# 📦 Warehouse KPI Dashboard

Este projeto apresenta um dashboard interativo desenvolvido no Power BI para monitoramento de indicadores logísticos em operações de armazéns. A solução foi construída com base em dados simulados e tem como objetivo demonstrar a capacidade de consolidar, modelar e visualizar informações críticas para a tomada de decisão operacional.

---

## 🎯 Objetivo

Fornecer uma visão clara e confiável do desempenho dos armazéns, com foco em:

- Produtividade por operação e turno
- Acuracidade de inventário por SKU e unidade
- Custos operacionais por armazém e período
- Nível de serviço e cumprimento de SLA (OTIF)

---

## 🧠 Tecnologias utilizadas

- **Power BI**: Modelagem de dados, DAX, Power Query e visualizações interativas
- **Excel / CSV**: Simulação e estruturação dos dados operacionais
- **SQL (conceitual)**: Base para integração e extração de dados
- **Python (conceitual)**: Aplicações futuras para automação e predição

---

## 📁 Estrutura dos dados

Os dados estão organizados em quatro arquivos principais:

| Arquivo | Descrição |
|--------|-----------|
| `warehouse_operations.csv` | Volume de itens processados, tempo de operação e erros por turno |
| `inventory_accuracy.csv` | Comparação entre estoque físico e sistema, com cálculo de acuracidade |
| `operational_costs.csv` | Custos fixos e variáveis por armazém e mês |
| `service_level.csv` | Tempo real vs. SLA por pedido, com indicador OTIF |

---

## 📊 Indicadores calculados

- **Produtividade** = Itens Processados ÷ Tempo Total  
- **Taxa de erro (%)** = (Erros ÷ Itens Processados) × 100  
- **Acuracidade de inventário (%)** = (Qtd. Física ÷ Qtd. Sistema) × 100  
- **Custo por item** = Custo Total ÷ Itens Processados  
- **OTIF (%)** = % de pedidos entregues dentro do SLA

---

## 📈 Visualizações

O dashboard inclui:

- Gráficos de linha e barras por operação e armazém  
- Cartões de KPIs com metas e variações  
- Filtros dinâmicos por data, unidade, operação e SKU  
- Tabelas detalhadas com drill-through por pedido e SKU

---

## 🧩 Próximos passos

- Integração com dados reais via SQL Server ou API  
- Automação de atualizações com Power Automate  
- Aplicação de modelos preditivos com Python (forecast de demanda, risco de ruptura)  
- Expansão para indicadores de transporte e armazenagem avançada

---
## 🚀 Como usar

### 1. Clone este repositório:
   ```bash
   git clone https://github.com/DeboraKlein/Warehouse-KPI-Dashboard.git
````
### 2. Abra o arquivo .pbix no Power BI Desktop

### 3. Navegue pelas páginas do dashboard para explorar os KPIs

Os dados utilizados são simulados e estão disponíveis na pasta /data


---


## 📷 Exemplos de visualizações

![Dashboard - Visão Geral](./images/dashboard-visao-geral.png)
![Dashboard - Inventário e SLA](./images/dashboard-inventario-sla.png)

## 🔗 Projetos relacionados

- [Logistics Dashboard](https://github.com/DeboraKlein/Logistics-Dashboard): Análise de desempenho de frota e entregas



## 👩‍💻 Sobre mim

Este projeto foi desenvolvido por **Debora Klein**, Analista de Dados com experiência em BI, planejamento financeiro e inteligência operacional.  
🔗 [linkedin.com/in/débora-klein](https://linkedin.com/in/débora-klein)  
💻 [github.com/DeboraKlein](https://github.com/DeboraKlein)

---

