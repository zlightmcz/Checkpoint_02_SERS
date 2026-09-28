# APIs de Energia Renovável e Aprendizado de Máquina

## 👥 Integrantes
| Gustavo Guedes Pereira — RM 569779
| Lucas Angelo — RM 569530
| Gustavo de Souza — RM 570746
| Murillo Boyadjian | 570774
| Arthur Tae — RM 570647
| Gabriel Rodrigues — RM 569322

**Disciplina / Turma:** 1CCPJ
**Professor(a):** Andre Tritiack

---

## 🎯 Objetivo

Consultar duas APIs públicas, organizar os dados em arquivos CSV e resolver **duas tarefas de aprendizado de máquina**, comparando **três algoritmos** em cada uma:

| Tarefa | Tipo | Pergunta |
|--------|------|----------|
| **1 — ANEEL** | Classificação | A partir da potência e da localização de um empreendimento, é possível classificar sua fonte como **Solar, Eólica ou Hidráulica**? |
| **2 — Open-Meteo** | Regressão | Dadas as condições meteorológicas e a hora local em Petrolina (PE), qual é a **radiação solar horizontal** (W/m²) naquela hora? |

---

## 🗂️ Fontes e período dos dados

Ambas as consultas são públicas e **não exigem login, token ou chave de API**.

### Tarefa 1 — SIGA / ANEEL
- **Fonte:** [SIGA – Sistema de Informações de Geração da ANEEL](https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel), via API CKAN/DataStore (`datastore_search`).
- **Período:** cadastro atual de empreendimentos (retrato do momento da consulta, sem recorte temporal; empreendimentos em várias fases).
- **Tipos consultados (`SigTipoGeracao`):** `UFV` (solar), `EOL` (eólica) e `UHE`, `PCH`, `CGH` (agrupadas como *Hidráulica*).
- **Limite por consulta:** 1200 registros por tipo.
- **Atributos de entrada (X):** `potencia_kw`, `latitude`, `longitude`. **Alvo (y):** `fonte`.
- **Não são usados como entrada:** `SigTipoGeracao`, `CodCEG`, nome e descrições do empreendimento (revelariam a classe).
- **Arquivo gerado:** `aneel_classificacao_orange.csv` (3.876 linhas).

| Classe | Exemplos |
|--------|---------:|
| Hidráulica (UHE 221 + PCH 537 + CGH 718) | 1.476 |
| Solar (UFV) | 1.200 |
| Eólica (EOL) | 1.200 |

> As quantidades são limitadas por tipo e **não representam** a participação de cada fonte na matriz elétrica brasileira.

### Tarefa 2 — Open-Meteo (histórico)
- **Fonte:** [Open-Meteo Historical Weather API](https://open-meteo.com/en/docs/historical-weather-api) (`archive-api.open-meteo.com/v1/archive`). Dados estimados por modelos/reanálise, não medições de um sensor específico.
- **Local:** Petrolina (PE) — latitude `-9.39`, longitude `-40.50`.
- **Período:** **01/04/2025 a 30/06/2025**, resolução horária, fuso `America/Recife` (hora local).
- **Recorte:** apenas horas de **7h a 17h** (2.184 horários recebidos → **1.001 linhas válidas**).
- **Atributos de entrada (X):** `temperatura_c`, `umidade_pct`, `nuvens_pct`, `vento_kmh`, `hora`. **Alvo (y):** `radiacao_w_m2` (radiação de onda curta, W/m²).
- **Arquivo gerado:** `meteo_regressao_orange.csv`.

---

## ▶️ Como executar o notebook

**Notebook:** `Aula_APIs_Energia_Renovavel_ML__3_.ipynb`

### Opção A — Google Colab (mais simples)
1. Abra o notebook no [Google Colab](https://colab.research.google.com/) (*Arquivo → Abrir notebook → GitHub* ou faça upload do `.ipynb`).
2. Execute **Ambiente de execução → Executar tudo**. As bibliotecas já vêm instaladas.

### Opção B — Localmente
Requisitos: **Python 3.9+** e acesso à internet (para consultar as APIs).

```bash
# 1. Clone o repositório
git clone <URL_DO_REPOSITORIO>
cd <NOME_DO_REPOSITORIO>

# 2. (Opcional) Crie um ambiente virtual
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

# 3. Instale as dependências
pip install pandas numpy matplotlib scikit-learn jupyter

# 4. Abra o notebook
jupyter notebook Aula_APIs_Energia_Renovavel_ML__3_.ipynb
```

Execute as células **em ordem**. O notebook:
1. Consulta a API da ANEEL e gera `aneel_classificacao_orange.csv`;
2. Consulta a API do Open-Meteo e gera `meteo_regressao_orange.csv`;
3. Treina e avalia os modelos das duas tarefas, exibindo tabelas e gráficos.

> **Observações:** a coleta usa apenas `json`, `urllib` e `csv` (bibliotecas padrão). Como o SIGA é atualizado continuamente, uma nova consulta pode gerar contagens e métricas ligeiramente diferentes das reportadas abaixo. Os CSVs já incluídos no repositório permitem reproduzir a análise sem refazer as consultas.

---

## 🧪 Metodologia

| Item | Tarefa 1 (classificação) | Tarefa 2 (regressão) |
|------|--------------------------|----------------------|
| Divisão | Estratificada 80/20, `random_state=42` | Temporal: primeiras 80% das horas (800) para treino, últimas 20% (201) para teste, sem embaralhar |
| Escala | `StandardScaler` dentro de `Pipeline` (ajustado só no treino) para Regressão Logística e k-NN | `StandardScaler` no `Pipeline` da Regressão Linear |
| Modelos | Regressão Logística, Random Forest, k-NN (k=5) | Regressão Linear, Random Forest, Gradient Boosting (100 árvores) |
| Métricas | Accuracy, Precision, Recall e F1 (média **macro**), matriz de confusão | MAE (W/m²), MSE ((W/m²)²), R², gráfico real × previsto |

---

## 📊 Resumo dos resultados

### Tarefa 1 — Classificação da fonte (776 exemplos de teste)

| Modelo | Accuracy | Precision (macro) | Recall (macro) | F1 (macro) |
|--------|---------:|------------------:|---------------:|-----------:|
| Regressão Logística | 0,825 | 0,828 | 0,821 | 0,820 |
| **Random Forest** | **0,976** | **0,977** | **0,974** | **0,975** |
| k-NN | 0,965 | 0,966 | 0,964 | 0,965 |

- **Melhor modelo: Random Forest**, seguido de perto pelo k-NN. A Regressão Logística fica bem atrás, pois um limite linear não separa bem as classes.
- **Principais confusões:** na Regressão Logística, a classe **Solar** foi a mais errada (51 de 240 previstas como Eólica e 22 como Hidráulica). No Random Forest, os erros restantes se concentram em Solar → Hidráulica (7) e Solar → Eólica (5).
- **Por que pode não bastar em uso real:** a alta acurácia vem em grande parte da **localização** (usinas hidráulicas seguem rios; eólicas se concentram em regiões de vento, como o Nordeste e o Sul) e da faixa de **potência** típica de cada fonte. Além disso, o conjunto é uma amostra limitada e levemente desbalanceada, e a coordenada é aproximada. Não há garantia de generalização para novos empreendimentos ou outras regiões.

### Tarefa 2 — Radiação solar em Petrolina (201 horas de teste)

| Algoritmo | MAE (W/m²) | MSE ((W/m²)²) | R² |
|-----------|-----------:|--------------:|---:|
| Regressão Linear | 145,20 | 30.034,20 | 0,360 |
| **Random Forest** | **66,80** | **7.307,42** | **0,844** |
| Gradient Boosting | 67,08 | 7.444,70 | 0,841 |

- **Melhor modelo: Random Forest**, praticamente empatado com o Gradient Boosting; ambos reduzem o erro absoluto a menos da metade do da Regressão Linear.
- **Interpretação:** a radiação tem um formato de "sino" ao longo do dia (baixa às 7h, pico próximo ao meio-dia), relação **não linear** com a hora, que a Regressão Linear não captura. Os modelos de árvores modelam bem essa curva e a interação entre hora e nebulosidade.
- **Limitação:** o teste cobre as últimas ~18 dias do período (fim de junho), então a avaliação reflete uma janela curta e sazonal. Além disso, radiação em W/m² **não equivale** à energia em kWh nem à geração de um sistema fotovoltaico, que depende também de área, eficiência, inclinação, temperatura do módulo, sombreamento e perdas.

---

## 📁 Estrutura do repositório

```
.
├── Aula_APIs_Energia_Renovavel_ML__3_.ipynb   # notebook executável
├── aneel_classificacao_orange.csv             # dados da Tarefa 1
├── meteo_regressao_orange.csv                 # dados da Tarefa 2
└── README.md
```

## ⚠️ Limitações gerais

- Os dados da ANEEL refletem o cadastro no momento da consulta e podem mudar.
- O Open-Meteo fornece dados de reanálise/modelo, não medições locais.
- Não foi feito ajuste de hiperparâmetros nem validação cruzada; os resultados usam configurações padrão dos algoritmos e uma única divisão treino/teste.

## 📚 Referências

- ANEEL — [Dados Abertos / SIGA](https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel)
- Open-Meteo — [Historical Weather API](https://open-meteo.com/en/docs/historical-weather-api)
- scikit-learn — <https://scikit-learn.org/>
