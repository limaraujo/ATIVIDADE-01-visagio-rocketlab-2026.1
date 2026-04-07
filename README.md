# ATIVIDADE-01-visagio-rocketlab-2026.1

## 📋 Visão Geral

Projeto de Engenharia de Dados implementado na plataforma **Databricks**, utilizando arquitetura **Medalhão** (Bronze → Silver → Gold) com o dataset **Olist E-Commerce**.

O projeto realiza:
- **Ingestão** de dados brutos em CSV e API (cotação do dólar) na camada Bronze
- **Transformação e curadoria** de dados na camada Silver
- **Criação de Data Marts analíticos** na camada Gold com KPIs de receita, volume e satisfação

---

## 🏗️ Arquitetura Medalhão

### 📦 Camada Bronze
Armazena dados brutos com rastreabilidade de ingestão. Cada tabela inclui metadados `timestamp_ingestion`.

**Tabelas:**
- `tb_customers` - Dados brutos de clientes
- `tb_orders` - Pedidos brutos
- `tb_order_items` - Itens dos pedidos
- `tb_order_payments` - Pagamentos dos pedidos
- `tb_order_reviews` - Avaliações de pedidos
- `tb_products` - Catálogo de produtos
- `tb_sellers` - Dados de vendedores
- `tb_geolocalizacao` - Geolocalização
- `tb_product_category_name_translation` - Tradução de categorias
- `tb_cotacao_dolar` - Cotação diária (API Banco Central)

### 🔄 Camada Silver
Dados transformados, padronizados e enriquecidos. Pronta para análises.

**Dimensões:**
- `dim_consumidores` - Clientes únicos dedplicados
- `dim_produtos` - Catálogo de produtos enriquecido
- `dim_vendedores` - Vendedores dedplicados
- `dim_categoria_produtos_traducao` - Mapping de categorias
- `dim_cotacao_dolar` - Série histórica com preenchimento de feriados

**Fatos:**
- `fat_pedidos` - Pedidos com status traduzido e métricas de entrega
- `fat_itens_pedidos` - Itens dos pedidos enriquecidos
- `fat_pagamentos_pedidos` - Pagamentos com tipo traduzido
- `fat_avaliacoes_pedidos` - Avaliações com validação de integridade
- `fat_pedido_total` - Agregação de pedidos + pagamentos + cotação

### 📊 Camada Gold
Data Marts otimizados para consultas analíticas.

**Tabelas:**
- `fat_vendas_comercial` - Agregações de receita e volume por período/categoria
- `dim_satisfacao_cliente` - Métricas de satisfação por cliente
- `fat_vendas_por_produto` - Performance de produtos
- `fat_vendas_por_vendedor` - Análises de vendedores

---

## 📁 Estrutura do Repositório

```
.
├── notebooks/                          # Especificação dos jobs Databricks
│   ├── Atividade_land_to_bronze.ipynb         # Ingestão dos dados (Bronze)
│   ├── Atividade_bronze_to_silver.ipynb       # Transformação e curadoria (Silver)
│   └── Atividade_silver_to_gold.ipynb         # Analytics e KPIs (Gold)
├── config/
│   └── arquivo.yaml                   # Configuração dos jobs (Databricks Workflows)
├── utils/                             # Funções utilitárias Python (reservado)
├── Imagens/                           # Diagramas e documentação visual
├── requirements.txt                   # Dependências Python
└── README.md                          # Este arquivo
```

---

## 🚀 Como Executar

### Pré-requisitos
- Acesso ao **Databricks Workspace**
- Dataset **Olist E-Commerce** carregado em volume ou camada Landing
- Python 3.9+ (para desenvolvimento local)

### No Databricks

1. **Fazer upload dos notebooks**
   ```
   Workspace → Usuário → Atividade 01 - RockLab
   ```
   Upload dos três notebooks (`Atividade_*.ipynb`)

2. **Configurar volumes**
   - Criar catalog: `ecommerce`
   - Criar schema: `bronze`, `silver`, `gold`
   - Criar volume: `/Volumes/ecommerce/bronze/raw_data`
   - Fazer upload dos CSVs do Olist

3. **Executar pipeline**
   - Executar `Atividade_land_to_bronze.ipynb` (~ 5 min)
   - Executar `Atividade_bronze_to_silver.ipynb` (~ 10 min)
   - Executar `Atividade_silver_to_gold.ipynb` (~ 5 min)

4. **Scheduler automático**
   Usar arquivo `config/arquivo.yaml` para configurar job recorrente:
   ```
   Schedule: 24h UTC (13:00 horário de São Paulo)
   Ordem: Bronze → Silver → Gold
   ```

### Localmente (desenvolvimento)

```bash
# Criar ambiente virtual
python -m venv venv
source venv/bin/activate  # Linux/Mac
# ou
venv\Scripts\activate  # Windows

# Instalar dependências
pip install -r requirements.txt

# Executar testes
jupyter lab notebooks/Atividade_land_to_bronze.ipynb
```

---

## 🔑 Principais Regras de Negócio

### Bronze
- Sem transformações de negócio
- Rastreabilidade via `timestamp_ingestion`
- Validação mínima (arquivo não vazio)

### Silver
- **Deduplicação Sênior**: Mantém apenas o registro mais recente por PK
- **Tradução de status**: Campos em português
- **Validação referencial**: Consistency checks entre tabelas
- **Preenchimento de datas**: Cotação do dólar propagada para feriados
- **Padronização**: nomes em português, maiúsculas/minúsculas consistentes

### Gold
- **Agregação por período e categoria**: Vendas comerciais
- **Cálculo de ticket médio**: em BRL e USD
- **ZORDER clustering**: Otimização física por chaves primárias
- **Arredondamento financeiro**: 2 casas decimais

---

## 📊 Dependências

| Pacote | Versão | Uso |
|--------|--------|-----|
| **pyspark** | ≥3.5.0 | Databricks Runtime (incluído) |
| **requests** | ≥2.31.0 | Consumo da API do Banco Central |
| **python-dateutil** | ≥2.8.2 | Manipulação de datas |
| **pandas** | ≥2.0.3 | Análise (opcional em desenvolvimento) |
| **jupyter** | ≥1.0.0 | IDE para testes locais |

---

## 📈 Métricas Geradas

### Receita
- `receita_total_brl`: Valor em reais
- `receita_total_usd`: Valor em dólares (com cotação diária)

### Volume
- `total_pedidos`: Contagem de pedidos únicos
- `qtd_itens_vendidos`: Contagem de itens

### Satisfação
- `nota_media_avaliacao`: Score médio 1-5
- `percentual_atraso_entrega`: % pedidos entregues após prazo

### Time-to-Market
- `tempo_entrega_dias`: Dias entre pedido e entrega
- `diferenca_entrega_dias`: Atraso em relação ao estimado

---

## 🔧 Troubleshooting

| Erro | Solução |
|------|---------|
| `FileNotFoundError: arquivo.csv` | Verificar volume `/Volumes/ecommerce/bronze/raw_data` |
| `NullPointerException cotacao_dolar` | Executar Atividade_land_to_bronze primeiro |
| `DeltaAnalysisException` | Usar `overwriteSchema=true` no write |
| `API Banco Central timeout` | Aumentar timeout de requisição em `land_to_bronze` |

---

## 📝 Autor

Desenvolvido como atividade de **Engenharia de Dados** - RockLab 2026.1

---

## 📚 Referências

- [Databricks Delta Lake](https://docs.databricks.com/delta/)
- [Medallion Architecture](https://www.databricks.com/en/blog/2022/06/24/five-step-guide-medallion-lakehouse-architecture.html)
- [Olist E-Commerce Dataset](https://www.kaggle.com/olistbr/brazilian-ecommerce)
- [API Banco Central do Brasil](https://www3.bcb.gov.br/api/)
