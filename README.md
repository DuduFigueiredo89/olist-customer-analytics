# Olist Customer & Payment Analytics

## 📌 Visão Geral

Este projeto realiza uma análise de **Customer Analytics, comportamento de pagamentos, retenção e propensão à recompra** utilizando o **Brazilian E-Commerce Public Dataset by Olist**.

O objetivo é transformar dados transacionais de e-commerce em informações relevantes para tomada de decisão, com foco em:

- comportamento de pagamento;
- valor e perfil dos clientes;
- segmentação RFM;
- recompra;
- retenção por coortes;
- previsão de recompra com Machine Learning;
- priorização de clientes para estratégias de CRM;
- recomendações para aumento de recorrência e geração de valor.

A análise foi desenvolvida com uma perspectiva executiva, conectando métricas analíticas a decisões relacionadas a **aquisição, retenção, recorrência e valor do cliente**.

---

## 🎯 Objetivo do Projeto

O projeto busca responder principalmente à seguinte questão:

> **Como o comportamento de compra e pagamento dos clientes pode ajudar a identificar oportunidades de aumento de recorrência e geração de valor no e-commerce?**

A partir dessa questão, foram explorados temas como:

- Quais são os principais meios de pagamento utilizados?
- Qual é a diferença entre um meio aparecer em um pedido e ser o meio principal da compra?
- Como o parcelamento em cartão de crédito se relaciona com o ticket?
- Quais clientes concentram maior valor financeiro?
- Qual é a taxa observada de clientes recorrentes?
- Quanto tempo um cliente leva para realizar uma segunda compra?
- Qual é a recompra em janelas comparáveis de 30, 60, 90 e 180 dias?
- Quais categorias apresentam maior recorrência?
- Como as coortes de aquisição evoluem ao longo do tempo?
- Qual valor econômico é acumulado pelas diferentes safras de clientes?
- É possível identificar clientes com maior propensão à recompra em até 90 dias?

---

## 📊 Dataset

O projeto utiliza o **Brazilian E-Commerce Public Dataset by Olist**, composto por aproximadamente 100 mil pedidos realizados entre **2016 e 2018**.

As principais bases utilizadas são:

| Dataset | Descrição |
|---|---|
| `olist_orders_dataset` | Informações dos pedidos e datas da jornada |
| `olist_order_items_dataset` | Produtos, vendedores, preços e frete |
| `olist_order_payments_dataset` | Meios de pagamento, parcelas e valores |
| `olist_customers_dataset` | Identificação dos clientes |
| `olist_order_reviews_dataset` | Avaliações dos pedidos |
| `olist_products_dataset` | Informações dos produtos |
| `olist_sellers_dataset` | Informações dos vendedores |
| `olist_geolocation_dataset` | Informações geográficas |
| `product_category_name_translation` | Tradução das categorias |

Para análises de comportamento, recompra e retenção é utilizado o campo **`customer_unique_id`**, que permite identificar o mesmo consumidor ao longo de diferentes pedidos.

O campo `customer_id` identifica o registro associado a cada pedido e, portanto, não é utilizado como identificador principal para medir recorrência.

Os arquivos originais do dataset não são versionados neste repositório. Para reproduzir as análises, eles devem ser adicionados localmente ao diretório:

```text
data/raw/
```

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
│   │   └── .gitkeep
│   └── processed/
│       └── .gitkeep
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
│   └── .gitkeep
│
├── reports/
│   └── relatorio_executivo_olist_investidores.pdf
│
└── presentation/
    └── apresentacao_executiva_olist_investidores.pptx
```

As pastas `data/raw/`, `data/processed/` e `models/` são mantidas no repositório por meio de arquivos `.gitkeep`.

Os datasets, arquivos processados e artefatos serializados de Machine Learning, como `.joblib` e `.pkl`, não são versionados. Eles são gerados localmente durante a execução do projeto.

---

# 🔎 Etapas da Análise

## 01 — Data Quality & Data Preparation

O primeiro notebook realiza a exploração, validação e preparação das nove bases do projeto.

Principais etapas:

- análise de dimensões;
- identificação de valores nulos;
- análise de duplicidades;
- validação de chaves;
- integridade referencial;
- análise dos status dos pedidos;
- conversão e validação de datas;
- análise da cardinalidade de `customer_unique_id`;
- validação das diferentes granularidades;
- criação das camadas analíticas.

Um ponto central dessa etapa é evitar **duplicação de receita** durante os joins.

Como as tabelas de pagamentos e itens possuem granularidades diferentes, ambas são agregadas previamente ao nível de `order_id`.

O notebook gera três camadas principais:

```text
orders_base
    ↓
customer_orders
    ↓
customer_summary_base
```

Onde:

- `orders_base`: uma linha por pedido;
- `customer_orders`: pedidos entregues válidos, ainda no nível do pedido;
- `customer_summary_base`: uma linha por `customer_unique_id`.

Também são executados **sanity checks** para garantir a unicidade e a consistência das bases produzidas.

---

## 02 — Payment Analysis

O segundo notebook analisa o comportamento financeiro dos pedidos.

São avaliados:

- meios de pagamento;
- participação financeira;
- associação entre pedido e meio de pagamento;
- meio principal de cada pedido;
- ticket médio;
- distribuição de parcelas;
- parcelamento em cartão de crédito;
- relação entre parcelamento e ticket;
- composição de meios de pagamento por faixa de ticket.

### Associação x meio principal

Um mesmo pedido pode utilizar mais de um meio de pagamento.

Por isso, o projeto diferencia:

**`order_association_share`**

Percentual de pedidos em que determinado meio aparece.

Como um pedido pode usar mais de um meio, essas participações podem somar mais de 100%.

**`primary_order_share`**

Percentual de pedidos em que determinado meio é o principal, definido como aquele responsável pelo maior valor financeiro dentro do pedido.

Nesse caso, cada pedido possui apenas um meio principal e as participações somam 100%.

### Parcelamento

Registros com:

```text
payment_installments = 0
```

são preservados na base original, mas excluídos das análises específicas de parcelamento.

A interpretação econômica do número de parcelas é concentrada principalmente nos pedidos cujo meio principal é **cartão de crédito**.

Essa distinção evita atribuir o mesmo significado de parcelamento a meios de pagamento diferentes.

---

## 03 — Customer RFM

O terceiro notebook aplica a metodologia **RFM — Recency, Frequency e Monetary** para transformar o histórico transacional em uma visão de cliente.

### Recency

Número de dias desde a última compra entregue até uma data fixa de referência.

### Frequency

Quantidade de pedidos entregues realizados pelo consumidor.

Como grande parte da base possui apenas uma compra, o score de Frequency é baseado em faixas comportamentais:

```text
1 pedido  → F1
2 pedidos → F2
3 pedidos → F3
4 pedidos → F4
5+        → F5
```

Isso evita forçar quintis artificiais sobre clientes com a mesma frequência.

### Monetary

Valor total pago pelo consumidor.

Valores ausentes de `payment_value` **não são convertidos automaticamente em zero**. Pedidos sem valor financeiro conhecido são identificados antes da construção da métrica Monetary.

### Segmentos

Os clientes são classificados em grupos estratégicos como:

- Champions;
- Loyal Customers;
- Potential Loyalists;
- New High Value;
- New Customers;
- Promising;
- Need Attention;
- Can't Lose Them;
- At Risk;
- Hibernating.

Também são analisados:

- concentração de receita;
- curva de Pareto;
- matriz Recency × Monetary;
- matriz Frequency × Monetary;
- clientes de maior valor;
- clientes de alto valor em risco;
- meio principal de pagamento por segmento.

---

## 04 — Repurchase Analysis

O quarto notebook analisa a conversão da **primeira para a segunda compra**.

Neste projeto, primeira compra significa:

> **primeiro pedido entregue observado do consumidor.**

São analisadas janelas de:

- 30 dias;
- 60 dias;
- 90 dias;
- 180 dias.

### Censura à direita

Um cuidado metodológico importante é considerar somente clientes que tiveram tempo suficiente para completar cada janela de análise.

Por exemplo, para medir recompra em 90 dias, entram no denominador apenas consumidores cuja primeira compra ocorreu pelo menos 90 dias antes do fim do período observado.

Isso evita comparar consumidores antigos com clientes adquiridos perto do final do dataset.

### Target principal

A janela de **90 dias** é utilizada como referência principal para modelagem por oferecer um equilíbrio entre:

- tempo suficiente para observar recompra;
- tamanho da população elegível;
- utilidade operacional para CRM.

A janela é uma **decisão analítica e operacional**, e não uma propriedade universal do negócio.

### Variáveis analisadas

A recompra é estudada em relação a:

- meio principal de pagamento;
- parcelamento no cartão de crédito;
- ticket da primeira compra;
- categoria dominante da primeira compra;
- categoria da segunda compra;
- coorte de aquisição.

O notebook também produz a base:

```text
repurchase_90d_model_base.csv
```

utilizada posteriormente pelo modelo de Machine Learning.

---

## 05 — Cohort & Retention Analysis

O quinto notebook organiza os clientes de acordo com o mês da primeira compra entregue.

A análise acompanha as coortes em:

```text
M0 → M1 → M2 → M3 → ... → M12
```

São analisadas três dimensões diferentes.

### 1. Retenção mensal exata

Responde:

> **Qual percentual da coorte realizou uma compra exatamente em Mk?**

Um cliente pode comprar em M1, ficar inativo em M2 e voltar em M3.

### 2. Recompra acumulada

Responde:

> **Qual percentual da população já realizou uma segunda compra até Mk?**

Para essa análise, é utilizada uma população madura com o mesmo horizonte completo de observação, mantendo o denominador comparável ao longo da curva.

### 3. Valor econômico

São utilizadas duas métricas principais:

#### Revenue Index

\[
\text{Revenue Index}_{M_k}
=
\frac{\text{Receita da coorte em } M_k}
{\text{Receita da coorte em } M_0}
\]

O Revenue Index:

- não representa percentual de clientes retidos;
- não é uma métrica de SaaS NRR;
- pode superar 100%;
- compara a receita mensal posterior com a receita gerada no mês de aquisição.

#### Receita acumulada por cliente adquirido

Funciona como uma **proxy descritiva de LTV observado**.

Não representa CLV predito, pois o dataset não possui informações como:

- CAC;
- margem;
- custo promocional;
- taxa de desconto;
- valor futuro esperado.

### Último mês completo

Para evitar distorções causadas por um mês final parcialmente observado, a maturidade das coortes é calculada usando o **último mês calendário completo disponível**.

Células que ainda não poderiam ser observadas aparecem como `NaN`, e não como zero.

O notebook também cria benchmarks para comparar cada coorte com a retenção média de clientes elegíveis em M1, M3 e M6.

---

## 06 — Repurchase Prediction

O sexto notebook constrói um modelo de Machine Learning para estimar a propensão de um cliente realizar uma segunda compra em até **90 dias**.

### Target

```text
repurchase_90d
```

Onde:

```text
1 = segunda compra entregue em até 90 dias
0 = não realizou segunda compra entregue nesse período
```

### Momento operacional do score

O score é calculado:

> **após a conclusão da primeira compra entregue.**

São utilizadas apenas características originadas nesse primeiro pedido.

Nenhuma informação sobre compras posteriores é utilizada como feature.

### Split temporal

Para respeitar a natureza temporal do problema, o dataset é dividido em:

```text
Treino
   ↓
Validação
   ↓
Teste out-of-time
```

O split é realizado por **coortes mensais de aquisição**, garantindo que uma mesma coorte não seja dividida entre conjuntos diferentes.

Isso aproxima a avaliação do cenário real:

> treinar com clientes adquiridos no passado e prever clientes adquiridos posteriormente.

### Modelos avaliados

- Dummy Classifier;
- Logistic Regression;
- Random Forest.

### Métricas

Como a recompra em 90 dias é um evento raro, a avaliação prioriza:

- PR-AUC;
- ROC-AUC;
- Brier Score;
- Precision@5/10/20%;
- Recall@5/10/20%;
- Lift@5/10/20%;
- desempenho por decil;
- curva de captura.

A acurácia não é utilizada como métrica principal.

### Interpretação

O modelo deve ser entendido como ferramenta de:

> **ranking e priorização de clientes**

e não como uma previsão determinística de comportamento individual.

Também é calculada **Permutation Importance** no conjunto de teste para avaliar quais features mais contribuem para o poder de ordenação do modelo.

Importância preditiva não implica causalidade.

---

# 🤖 Resultado do Modelo

Na execução de referência do projeto, o **Random Forest** apresentou o melhor desempenho na validação.

No conjunto de teste temporal, foram observados aproximadamente:

| Métrica | Resultado |
|---|---:|
| ROC-AUC | 0,59 |
| PR-AUC | 0,029 |
| Lift @ Top 10% | **1,85x** |
| Recall @ Top 10% | 18,5% |
| Precision @ Top 10% | 3,19% |

A taxa natural de recompra no conjunto de teste ficou próxima de:

**1,73%**

Entre os 10% de clientes com maior score de propensão, a taxa observada de recompra ficou próxima de:

**3,19%**

representando aproximadamente:

**1,85x de lift**

sobre a média do teste.

Esses resultados indicam que o modelo possui capacidade de **concentrar uma proporção maior de clientes que recompram no topo do ranking**, embora seu poder preditivo absoluto seja moderado.

> Os valores acima devem ser validados novamente após a execução completa dos notebooks revisados, pois alterações metodológicas podem produzir pequenas diferenças nos resultados finais.

Os artefatos serializados gerados durante o treinamento são armazenados localmente em:

```text
models/
```

e não são versionados no repositório.

---

# 📈 Principais Insights

## 1. Recorrência é um dos principais desafios

Na execução de referência, aproximadamente:

**3% dos clientes realizaram mais de uma compra entregue no período observado.**

Isso indica forte dependência de aquisição contínua de novos consumidores.

É importante distinguir essa taxa observada das métricas ajustadas por janela temporal.

---

## 2. Existe uma janela relevante após a primeira compra

Na análise de referência:

- a mediana do tempo até a segunda compra ficou próxima de **29 dias**;
- aproximadamente **68% das segundas compras observadas ocorreram em até 90 dias**.

Esse comportamento torna a janela pós-primeira compra especialmente relevante para estratégias de CRM.

---

## 3. O valor dos clientes é concentrado

A análise RFM indicou concentração significativa da receita.

Na execução de referência:

**aproximadamente 20% dos clientes concentraram mais de 50% da receita observada.**

Isso reforça a importância de diferenciar estratégias de relacionamento de acordo com:

- valor;
- recência;
- frequência;
- risco de perda.

---

## 4. A segunda compra não precisa ocorrer na mesma categoria

Parte relevante dos clientes realiza a segunda compra em uma categoria diferente da primeira.

Esse comportamento sugere oportunidades de:

- cross-sell;
- recomendação personalizada;
- exploração de categorias complementares;
- ampliação de share of wallet.

---

## 5. Pagamento funciona melhor como sinal complementar

O meio de pagamento apresenta diferenças de comportamento, mas não deve ser interpretado isoladamente como causa de recompra.

Sua utilidade é maior quando combinado com variáveis como:

- ticket;
- parcelamento;
- categoria;
- valor do frete;
- quantidade de itens;
- complexidade do pedido;
- momento da aquisição.

---

## 6. Retenção mensal e recompra acumulada contam histórias diferentes

Em um marketplace transacional, o cliente não precisa comprar todos os meses para continuar tendo valor.

Por isso:

```text
Retenção mensal
```

mede atividade em um mês específico, enquanto:

```text
Recompra acumulada
```

mede a conversão progressiva da primeira para a segunda compra.

As duas métricas devem ser analisadas separadamente.

---

# 💡 Recomendações Estratégicas

## 1. Programa de Segunda Compra

Criar ações específicas durante os primeiros **30–90 dias** após a primeira compra entregue.

Objetivo:

> aumentar a conversão de compradores de primeira viagem em clientes recorrentes.

Possíveis iniciativas:

- comunicação pós-compra;
- incentivo moderado à segunda compra;
- recomendação de categorias complementares;
- personalização por ticket e categoria inicial.

---

## 2. Estratégias de Cross-Sell

Utilizar o comportamento entre primeira e segunda compra para identificar categorias complementares.

O objetivo não deve ser apenas repetir a compra anterior, mas ampliar o relacionamento do cliente com o marketplace.

---

## 3. CRM Baseado em Valor

Utilizar a segmentação RFM para diferenciar estratégias.

### Champions

- benefícios exclusivos;
- programas de fidelidade;
- early access;
- referrals.

### Loyal Customers

- bundles;
- recomendação personalizada;
- expansão de share of wallet.

### New High Value

- onboarding diferenciado;
- incentivo à segunda compra;
- cross-sell.

### At Risk / Can't Lose Them

- ações seletivas de win-back;
- comunicação contextual;
- priorização conforme valor histórico.

### Hibernating

- comunicação de baixo custo;
- evitar subsídios indiscriminados.

---

## 4. Campanhas Orientadas por Propensão

Utilizar o modelo de recompra para ordenar clientes por prioridade.

Em vez de impactar indiscriminadamente toda a base:

```text
Base total
    ↓
Score de propensão
    ↓
Ranking
    ↓
Top 5% / 10% / 20%
    ↓
Campanha priorizada
```

O objetivo é concentrar recursos nos grupos em que há maior densidade observada de clientes que posteriormente recompram.

---

## 5. Acompanhamento por Coorte

Acompanhar cada nova safra utilizando um scorecard com:

- clientes adquiridos;
- retenção M1;
- retenção M3;
- retenção M6;
- delta versus benchmark;
- recompra acumulada;
- Revenue Index;
- receita acumulada por cliente.

Isso permite separar:

> crescimento por aquisição

de:

> melhora real na qualidade do relacionamento.

---

# 📊 Storytelling Executivo

A jornada analítica do projeto pode ser resumida como:

```text
Aquisição de clientes
        ↓
Primeira compra entregue
        ↓
Comportamento de pagamento
        ↓
Valor e segmentação
        ↓
Segunda compra
        ↓
Retenção por coortes
        ↓
Propensão à recompra
        ↓
CRM orientado por dados
```

A principal oportunidade identificada está na transição:

```text
1ª COMPRA → 2ª COMPRA
```

O projeto sugere que aumentar essa conversão pode:

- reduzir a dependência de aquisição contínua;
- aumentar o valor gerado pela base existente;
- melhorar a eficiência de ações de CRM;
- criar relacionamentos mais duradouros com clientes de maior potencial.

---

# 🛠️ Tecnologias Utilizadas

- Python
- Pandas
- NumPy
- Matplotlib
- Seaborn
- Scikit-learn
- Joblib
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

Adicione os arquivos originais do **Brazilian E-Commerce Public Dataset by Olist** em:

```text
data/raw/
```

Os arquivos de dados não estão incluídos no repositório.

---

## Ordem de execução

Os notebooks devem ser executados na seguinte ordem:

```text
01_data_quality.ipynb
        ↓
02_payment_analysis.ipynb
        ↓
03_customer_rfm.ipynb
        ↓
04_repurchase_analysis.ipynb
        ↓
05_cohort_retention_analysis.ipynb
        ↓
06_repurchase_prediction.ipynb
```

A ordem é importante porque existem dependências entre os outputs processados.

### Notebook 01

Gera, entre outros:

```text
orders_base.csv
customer_orders.csv
customer_summary_base.csv
```

### Notebook 02

Gera, entre outros:

```text
payment_behavior_order_level.csv
payment_type_summary.csv
primary_payment_summary.csv
payment_executive_kpis.csv
```

### Notebook 03

Gera:

```text
customer_rfm.csv
rfm_segment_summary.csv
rfm_executive_kpis.csv
```

### Notebook 04

Gera:

```text
repurchase_customer_level.csv
repurchase_window_summary.csv
repurchase_speed_summary.csv
repurchase_90d_model_base.csv
repurchase_executive_kpis.csv
```

### Notebook 05

Gera:

```text
cohort_customer_retention_matrix.csv
cohort_weighted_retention_curve.csv
cohort_cumulative_repeat_curve.csv
cohort_revenue_index_matrix.csv
cohort_cumulative_revenue_per_customer.csv
cohort_summary.csv
cohort_executive_kpis.csv
```

### Notebook 06

Gera:

```text
repurchase_model_metrics.csv
repurchase_test_scored_customers.csv
repurchase_score_deciles.csv
repurchase_campaign_simulation.csv
repurchase_capture_curve.csv
repurchase_permutation_importance.csv
repurchase_model_executive_kpis.csv
```

Além de artefatos locais em:

```text
models/
```

como:

```text
repurchase_90d_pipeline.joblib
repurchase_90d_metadata.json
```

Esses arquivos não são versionados no GitHub.

---

# 📁 Relatório e Apresentação Executiva

O projeto também possui materiais executivos consolidados:

- [Relatório executivo](reports/relatorio_executivo_olist_investidores.pdf)
- [Apresentação executiva](presentation/apresentacao_executiva_olist_investidores.pptx)

O relatório consolida:

- principais resultados;
- implicações para o negócio;
- limitações;
- recomendações estratégicas.

A apresentação executiva sintetiza a narrativa analítica para uma audiência orientada à tomada de decisão.

---

# 🔗 Repositório

GitHub:

https://github.com/DuduFigueiredo89/olist-customer-analytics

---

# ⚠️ Limitações

O dataset representa transações históricas ocorridas entre **2016 e 2018**.

Portanto, os resultados devem ser interpretados como um estudo analítico sobre a base pública disponibilizada e **não representam necessariamente o desempenho atual da Olist**.

Além disso:

- associação estatística não implica causalidade;
- Monetary representa valor pago, não margem;
- o dataset não contém CAC ou custo promocional;
- clientes adquiridos próximos ao final da base possuem menor tempo de acompanhamento;
- métricas de recompra precisam respeitar o tempo disponível de observação;
- coortes recentes não podem ser comparadas em horizontes ainda não observáveis;
- o mês final parcialmente observado não é tratado como mês completo nas métricas de maturidade;
- Revenue Index não representa retenção de clientes nem SaaS NRR;
- receita acumulada por cliente é uma proxy de LTV observado, não CLV predito;
- o modelo de Machine Learning utiliza apenas informações disponíveis no dataset público;
- a baixa taxa de recompra torna o problema fortemente desbalanceado;
- desempenho preditivo pode variar ao longo do tempo;
- importância de feature não representa causalidade;
- o modelo de propensão deve ser interpretado como ferramenta de priorização e ranking;
- medir o efeito real de campanhas exigiria experimentação, como testes A/B ou uplift modeling.

---

# 👤 Autor

**Eduardo Figueiredo**

GitHub: [DuduFigueiredo89](https://github.com/DuduFigueiredo89)

Projeto desenvolvido como parte de estudos e portfólio em **Data Analytics e Machine Learning**, com foco em comportamento de clientes, retenção, pagamentos e modelagem preditiva.

---

## 📌 Conclusão

O projeto demonstra como dados transacionais podem ser utilizados para evoluir de uma análise descritiva de vendas para uma visão mais ampla de **Customer Analytics**.

A combinação de:

```text
Pagamentos
    +
RFM
    +
Recompra
    +
Coortes
    +
Machine Learning
```

permite responder não apenas quanto os clientes compraram, mas também:

- quem gera mais valor;
- quem voltou a comprar;
- quando a recompra acontece;
- como diferentes safras evoluem;
- quais comportamentos estão associados à recorrência;
- quais clientes podem ser priorizados em ações futuras de retenção.

A principal oportunidade encontrada está na conversão entre a **primeira e a segunda compra**, conectando análise de comportamento, segmentação, retenção longitudinal e modelos preditivos a possíveis estratégias de CRM e crescimento orientado por dados.