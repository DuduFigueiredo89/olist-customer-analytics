# Olist Customer & Payment Analytics

## 📌 Visão Geral

Este projeto realiza uma análise de comportamento de clientes e meios de pagamento utilizando o **Brazilian E-Commerce Public Dataset by Olist**.

O objetivo é transformar dados transacionais de e-commerce em informações relevantes para tomada de decisão, com foco em:

- comportamento de pagamento;
- valor e perfil dos clientes;
- segmentação RFM;
- recompra;
- retenção por coortes;
- previsão de recompra com Machine Learning;
- recomendações estratégicas para aumento da recorrência e do valor do cliente.

A análise foi desenvolvida com uma perspectiva executiva, buscando conectar métricas analíticas a decisões relacionadas a **crescimento, retenção e geração de valor**.

---

## 🎯 Objetivo do Projeto

O projeto busca responder principalmente à seguinte questão:

> **Como o comportamento de compra e pagamento dos clientes pode ajudar a identificar oportunidades de aumento de recorrência e geração de valor no e-commerce?**

A partir dessa questão, foram explorados temas como:

- Quais são os principais meios de pagamento utilizados?
- Como o parcelamento se relaciona com o ticket das compras?
- Quais clientes concentram maior valor financeiro?
- Qual é a taxa de recompra observada?
- Quanto tempo um cliente leva para realizar uma segunda compra?
- Quais categorias apresentam maior recorrência?
- Como a retenção evolui ao longo das coortes?
- É possível identificar clientes com maior propensão à recompra?

---

## 📊 Dataset

O projeto utiliza o **Brazilian E-Commerce Public Dataset by Olist**, composto por aproximadamente 100 mil pedidos realizados entre **2016 e 2018**.

As principais bases utilizadas são:

| Dataset | Descrição |
|---|---|
| `olist_orders_dataset` | Informações dos pedidos e datas da jornada |
| `olist_order_items_dataset` | Produtos, vendedores, preços e frete |
| `olist_order_payments_dataset` | Forma de pagamento, parcelas e valores |
| `olist_customers_dataset` | Identificação dos clientes |
| `olist_order_reviews_dataset` | Avaliações dos pedidos |
| `olist_products_dataset` | Informações dos produtos |
| `olist_sellers_dataset` | Informações dos vendedores |
| `olist_geolocation_dataset` | Informações geográficas |
| `product_category_name_translation` | Tradução das categorias |

Para análises de comportamento, recompra e retenção foi utilizado o campo **`customer_unique_id`**, que permite identificar o mesmo cliente ao longo de diferentes pedidos.

---

## 🗂️ Estrutura do Projeto

```text
olist-customer-analytics/
│
├── README.md
├── requirements.txt
├── .gitignore
│
├── data/
│   ├── raw/
│   └── processed/
│
├── notebooks/
│   ├── 01_data_quality.ipynb
│   ├── 02_payment_analysis.ipynb
│   ├── 03_customer_rfm.ipynb
│   ├── 04_repurchase_analysis.ipynb
│   ├── 05_cohort_retention_analysis.ipynb
│   └── 06_repurchase_prediction.ipynb
│
├── models/
│   └── repurchase_90d_pipeline.joblib
│
├── reports/
│   └── relatorio_executivo_olist_investidores.pdf
│
└── presentation/
    └── apresentacao_executiva_olist_investidores.pptx
```

---

# 🔎 Etapas da Análise

## 01 — Data Quality & Data Preparation

O primeiro notebook realiza a exploração e validação das nove bases do projeto.

Principais etapas:

- análise de dimensões;
- identificação de valores nulos;
- análise de duplicidades;
- integridade entre tabelas;
- validação da granularidade;
- análise dos status dos pedidos;
- tratamento de datas;
- criação de uma tabela consolidada no nível do pedido.

Um ponto importante dessa etapa foi evitar **duplicação de receita** durante os joins.

Como as tabelas de pagamentos e itens possuem granularidades diferentes, ambas foram agregadas para o nível de `order_id` antes da construção da base analítica.

---

## 02 — Payment Analysis

Análise do comportamento financeiro dos pedidos.

Foram avaliados:

- meios de pagamento;
- participação financeira;
- ticket médio;
- parcelamento;
- número de pagamentos por pedido;
- relação entre parcelamento e ticket.

Os principais meios de pagamento encontrados foram:

- cartão de crédito;
- boleto;
- voucher;
- cartão de débito.

A análise também mostrou que o parcelamento apresenta relação com o valor da compra, embora não possa ser interpretado isoladamente como causa de maior ticket.

---

## 03 — Customer RFM

Foi aplicada a metodologia **RFM — Recency, Frequency e Monetary** para segmentar os clientes.

### Variáveis

**Recency**

Número de dias desde a última compra.

**Frequency**

Quantidade de pedidos realizados pelo cliente.

**Monetary**

Valor total movimentado pelo cliente.

Os clientes foram classificados em grupos estratégicos como:

- Champions;
- Loyal Customers;
- Potential Loyalists;
- New High Value;
- At Risk;
- Can't Lose Them;
- Hibernating.

---

## 04 — Repurchase Analysis

A análise de recompra teve como objetivo identificar clientes que realizaram uma segunda compra.

Foram analisadas janelas de:

- 30 dias;
- 60 dias;
- 90 dias;
- 180 dias.

Um cuidado metodológico importante foi considerar apenas clientes com **tempo suficiente de observação** para cada janela.

### Principais resultados

- aproximadamente **3,0% dos clientes** realizaram mais de uma compra;
- a recompra em até **90 dias foi de aproximadamente 2,28%**;
- entre os clientes que recompram, a mediana até a segunda compra ficou próxima de **29 dias**;
- aproximadamente **68% das segundas compras acontecem em até 90 dias**.

Esses resultados indicam que os primeiros meses após a aquisição representam uma janela importante para estratégias de retenção.

---

## 05 — Cohort & Retention Analysis

Os clientes foram organizados de acordo com o mês da primeira compra.

A análise acompanha o comportamento das coortes em:

```text
M0 → M1 → M2 → M3 → ... → M12
```

Foram construídas análises de:

- retenção mensal;
- recompra acumulada;
- retenção de receita;
- receita acumulada por cliente;
- comportamento por meio de pagamento.

A análise de coortes complementa a taxa simples de recompra, permitindo observar como diferentes safras de clientes evoluem ao longo do tempo.

---

## 06 — Repurchase Prediction

Foi construído um modelo de Machine Learning para estimar a probabilidade de um cliente realizar uma segunda compra em até **90 dias**.

### Target

```text
repurchase_90d
```

Onde:

```text
1 = cliente realizou nova compra em até 90 dias
0 = cliente não realizou nova compra nesse período
```

Foram utilizados apenas dados conhecidos no momento da primeira compra, evitando **data leakage**.

### Modelos avaliados

- Dummy Classifier;
- Logistic Regression;
- Random Forest.

Para respeitar a natureza temporal do problema, foi utilizado **split temporal**, em vez de divisão aleatória dos dados.

---

## 🤖 Resultado do Modelo

O **Random Forest** apresentou o melhor desempenho na etapa de validação.

No conjunto de teste temporal, o modelo apresentou aproximadamente:

| Métrica | Resultado |
|---|---:|
| ROC-AUC | 0,59 |
| PR-AUC | 0,029 |
| Lift @ Top 10% | **1,85x** |
| Recall @ Top 10% | 18,5% |
| Precision @ Top 10% | 3,19% |

Apesar do desempenho absoluto moderado, o modelo apresentou capacidade de **ranking de clientes**.

A taxa natural de recompra no conjunto de teste ficou próxima de **1,73%**.

Entre os 10% de clientes com maior score de propensão, essa taxa subiu para aproximadamente:

**3,19%**

representando um lift de aproximadamente:

**1,85x**

O modelo pode, portanto, ser utilizado como mecanismo de **priorização de clientes para campanhas**, e não como uma previsão determinística de recompra.

---

# 📈 Principais Insights

## 1. Recorrência é o principal desafio

Apesar do volume significativo de pedidos, apenas cerca de:

**3% dos clientes realizaram mais de uma compra.**

Isso indica forte dependência da aquisição de novos consumidores.

---

## 2. Existe uma janela crítica de 30–90 dias

Entre os clientes que realizam uma segunda compra:

- aproximadamente metade retorna em até 30 dias;
- cerca de 68% retorna em até 90 dias.

Esse período representa uma janela relevante para estratégias de CRM e retenção.

---

## 3. O valor dos clientes é concentrado

A análise RFM mostrou concentração significativa da receita.

Aproximadamente:

**20% dos clientes concentram mais de 50% da receita observada.**

Isso reforça a importância de diferenciar estratégias de relacionamento de acordo com o valor do cliente.

---

## 4. A segunda compra não precisa ocorrer na mesma categoria

Uma parcela significativa dos clientes realiza a segunda compra em uma categoria diferente da primeira.

Esse comportamento sugere oportunidades de:

- cross-sell;
- recomendação de produtos;
- campanhas personalizadas;
- exploração de categorias complementares.

---

## 5. Pagamento funciona melhor como variável complementar

Os meios de pagamento apresentaram diferenças relativamente pequenas na taxa de recompra.

Isso sugere que payment type isoladamente possui poder limitado para explicar a recorrência, mas pode contribuir quando combinado com:

- ticket;
- parcelamento;
- categoria;
- valor do frete;
- quantidade de itens;
- comportamento temporal.

---

# 💡 Recomendações Estratégicas

A partir das análises, quatro iniciativas se destacam.

### 1. Programa de Segunda Compra

Criar ações específicas durante os primeiros **30–90 dias** após a aquisição.

Objetivo:

> aumentar a conversão de compradores de primeira viagem em clientes recorrentes.

---

### 2. Estratégias de Cross-Sell

Utilizar o comportamento entre primeira e segunda compra para recomendar categorias relacionadas.

O objetivo não deve ser apenas estimular a recompra do mesmo produto, mas ampliar o relacionamento do cliente com o marketplace.

---

### 3. CRM Baseado em Valor

Utilizar a segmentação RFM para diferenciar estratégias.

**Champions**

- benefícios exclusivos;
- programas de fidelidade;
- early access.

**New High Value**

- incentivo à segunda compra;
- recomendações personalizadas.

**At Risk**

- campanhas de reativação;
- ofertas específicas.

---

### 4. Campanhas Orientadas por Propensão

Utilizar o modelo de recompra para priorizar clientes.

Em vez de impactar toda a base, campanhas podem focar grupos com maior probabilidade estimada de conversão.

---

# 📊 Storytelling Executivo

A análise sugere a seguinte jornada:

```text
Aquisição de clientes
        ↓
Primeira compra
        ↓
Comportamento de pagamento
        ↓
Valor do cliente
        ↓
Segunda compra
        ↓
Retenção
        ↓
Predição de recompra
        ↓
CRM e crescimento orientado por dados
```

A principal oportunidade identificada está na transição:

```text
1ª COMPRA → 2ª COMPRA
```

Aumentar essa conversão pode reduzir a dependência de aquisição contínua de novos clientes e aumentar o valor gerado pela base existente.

---

# 🛠️ Tecnologias Utilizadas

- Python
- Pandas
- NumPy
- Matplotlib
- Scikit-learn
- Jupyter Notebook
- Git
- GitHub
- Visual Studio Code

---

# ▶️ Como Executar o Projeto

Clone o repositório:

```bash
git clone https://github.com/DuduFigueiredo89/olist-customer-analytics.git
```

Entre na pasta:

```bash
cd olist-customer-analytics
```

Crie um ambiente virtual:

```bash
python -m venv .venv
```

No Windows:

```bash
.venv\Scripts\activate
```

Instale as dependências:

```bash
pip install -r requirements.txt
```

Adicione os arquivos originais da Olist em:

```text
data/raw/
```

Depois execute os notebooks na seguinte ordem:

```text
01_data_quality.ipynb
02_payment_analysis.ipynb
03_customer_rfm.ipynb
04_repurchase_analysis.ipynb
05_cohort_retention_analysis.ipynb
06_repurchase_prediction.ipynb
```

---

# 📁 Relatório e Apresentação Executiva

O projeto também possui materiais executivos consolidados:

- [Relatório executivo](reports/relatorio_executivo_olist_investidores.pdf)
- [Apresentação executiva](presentation/apresentacao_executiva_olist_investidores.pptx)

---

# 🔗 Repositório

GitHub:

https://github.com/DuduFigueiredo89/olist-customer-analytics

---

# ⚠️ Limitações

O dataset utilizado representa transações históricas ocorridas entre **2016 e 2018**.

Portanto, os resultados deste projeto devem ser interpretados como um estudo analítico sobre a base disponibilizada e **não representam necessariamente o desempenho atual da Olist**.

Além disso:

- associação estatística não implica causalidade;
- a janela de observação influencia métricas de recompra;
- clientes adquiridos próximos ao final do dataset possuem menor tempo de acompanhamento;
- o modelo de Machine Learning utiliza apenas informações disponíveis no dataset público.

---

# 👤 Autor

**Eduardo Figueiredo**

GitHub: [DuduFigueiredo89](https://github.com/DuduFigueiredo89)

Projeto desenvolvido como parte de estudos em **Data Analytics**, com foco em análise de dados, comportamento de clientes, retenção e Machine Learning.

---

## 📌 Conclusão

O projeto demonstra como dados transacionais podem ser utilizados para evoluir de uma análise descritiva de vendas para uma visão mais ampla de **Customer Analytics**.

A combinação de:

**Pagamentos + RFM + Recompra + Coortes + Machine Learning**

permite identificar não apenas quanto os clientes compraram, mas também:

- quem gera mais valor;
- quem voltou a comprar;
- quando a recompra acontece;
- quais comportamentos estão associados à recorrência;
- quais clientes podem ser priorizados em futuras ações de retenção.