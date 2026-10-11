# Tech Challenge — Fase 2 | POSTECH Data Analytics

## 1. Identificação

| Campo | Valor |
|---|---|
| Turma | POSTECH BB - Fase 2 |
| Grupo | Grupo 52 |
| Data de entrega | 10/10/2026 |

### Integrantes

| Nome completo | RM | E-mail |
|---|---|---|
| MARIANA ZARNAUSKAS AMARAL MARTINS| RM377741 | marianaz@bb.com.br |

---

## 2. Links da entrega

Estes três links são **obrigatórios** e devem ser idênticos aos do PDF de submissão.

| Item | Link |
|---|---|
| Repositório | https://github.com/marizarnauskas-star/postech-tech-challenge-fase2-template |
| Vídeo executivo (≤ 5 min) | <!-- PREENCHER: YouTube não listado / Drive com acesso liberado --> |
| Apresentação | <!-- PREENCHER: link do arquivo em `docs/` ou Drive --> |

> ⚠️ Repositório privado ou inacessível inviabiliza a avaliação da entrega.
> Confira o acesso em uma janela anônima antes de enviar.

---

## 3. O problema

No setor bancário, a concessão de crédito exige o equilíbrio entre expandir a carteira de clientes adimplentes e conter a inadimplência grave, que gera impacto direto na Provisão para Devedores Duvidosos (PDD). O objetivo deste projeto é construir um modelo preditivo de Machine Learning capaz de identificar antecipadamente solicitantes de cartão de crédito com alto risco de inadimplência, automatizando e otimizando a esteira de aprovação.

### Variável alvo

* **Definição (`TARGET`):** A variável alvo foi definida com base no histórico de pagamentos (`credit_record.csv`). Foi classificado como **Mau Pagador (`TARGET = 1`)** qualquer cliente que tenha apresentado atraso de pagamento igual ou superior a 60 dias (`STATUS` `'2'`, `'3'`, `'4'` ou `'5'`). Clientes sem atraso grave foram definidos como **Bom Pagador (`TARGET = 0`)**.
* **Justificativa de Limiar e Distribuição:** Atrasos pontuais de até 30 ou 50 dias costumam ser operacionais. A partir de 60 dias de atraso, caracteriza-se o risco real de calote na carteira. O conjunto de dados consolidado resultou em **36.457 clientes únicos**, sendo **35.841 Bons Pagadores (98,31%)** e **616 Maus Pagadores (1,69%)**, apresentando um acentuado desbalanceamento de classes inerente ao problema financeiro real.

### Dataset

| Campo | Valor |
|---|---|
| Fonte | https://www.kaggle.com/datasets/rikdifos/credit-card-approval-prediction |
| Linhas × colunas | 36.457 linhas × 57 colunas (após feature engineering e encoding) |
| Período / versão | Dados cadastrais e histórico de até 60 meses de pagamentos |
| Licença de uso | CC0: Public Domain |

Descrição das variáveis:

| Variável | Tipo | Descrição |
|---|---|---|
| `TARGET` | Int / Booleano | Variável alvo: 1 = Mau Pagador (atraso ≥ 60 dias), 0 = Bom Pagador |
| `AGE` | Float | Idade do cliente convertida em anos positivos |
| `INCOME_PER_PERSON` | Float | Renda per capita familiar (`AMT_INCOME_TOTAL / CNT_FAM_MEMBERS`) |
| `YEARS_EMPLOYED` | Float | Tempo de trabalho em anos (ajustado para aposentados) |
| `AMT_INCOME_TOTAL` | Float | Renda total anual declarada pelo cliente |
| `CNT_FAM_MEMBERS` | Float | Número total de membros da família do cliente |
| `OCCUPATION_TYPE` | Categórico | Tipo de ocupação profissional (imputado com "Desconhecido" para ausentes) |

---

## 4. Como reproduzir

```bash
git clone [https://github.com/marizarnauskas-star/postech-tech-challenge-fase2-template.git](https://github.com/marizarnauskas-star/postech-tech-challenge-fase2-template.git)
cd postech-tech-challenge-fase2-template

python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install -r requirements.txt
jupyter notebook
```

Baixe o dataset e coloque o arquivo bruto em `data/raw/` (os dados **não** são versionados —
veja `data/README.md`).

Depois execute os notebooks nesta ordem:

| # | Notebook | O que faz |
|---|---|---|
| 1 | `notebooks/01_eda.ipynb` | Análise exploratória |
| 2 | `notebooks/02_preprocessamento.ipynb` | Limpeza, escala e feature engineering |
| 3 | `notebooks/03_modelagem.ipynb` | Treino e comparação dos modelos |
| 4 | `notebooks/04_avaliacao.ipynb` | Métricas, importância de variáveis e conclusões |

**Semente fixa:** `RANDOM_STATE = 42`, declarada na primeira célula de cada notebook.
Rodar os notebooks na ordem acima, a partir de um ambiente limpo, deve reproduzir
exatamente os números da seção 5.

---

## 5. Resultados

| Modelo | Acurácia | Precisão | Recall | F1 | AUC-ROC |
|---|---|---|---|---|---|
| Regressão Logística (Baseline) | 0,61 | 0,03 | 0,58 | 0,05 | 0,6043 |
| Random Forest (Campeão) | 0,96 | 0,17 | 0,31 | 0,22 | **0,7476** |

**Modelo escolhido:** Random Forest — Escolhido por apresentar o maior poder discriminatório entre os clientes (ROC-AUC médio de 0,7476 vs 0,6043 do modelo linear) e por conseguir capturar relações não-lineares complexas entre as variáveis socioeconômicas sem cair em overfitting.

**Métricas priorizadas:** Priorizamos o ROC-AUC como métrica global e o Recall da classe 1 (Maus Pagadores). Em concessão de crédito desbalanceada, a acurácia isolada é enganosa (um modelo ingênuo que aprovasse todos obteria 98,3% de acurácia, mas geraria prejuízo total). Priorizar o Recall minimiza os Falsos Negativos (aprovar um mau pagador), que representam o erro de maior custo financeiro e calote direto para a instituição.

---

## 6. Principais conclusões

1. Indicadores Determinantes: As variáveis que mais influenciaram a decisão do modelo foram a Idade (AGE), a Renda Per Capita Familiar (INCOME_PER_PERSON) e a Estabilidade Profissional (YEARS_EMPLOYED), mostrando que a capacidade financeira por membro familiar é um preditor muito mais forte que a renda bruta isolada.
2. Motor de Decisão Automatizado: O modelo permite estruturar uma esteira de aprovação inteligente com três níveis: Aprovação Expressa automatizada para baixo risco, Mesa de Análise Humana focada em renda per capita para risco moderado e Negativa/Restrição de Limites preventiva para alto risco.
3. Equilíbrio Financeiro: Com Recall de 31% na esteira inicial e ROC-AUC de ~0.70 no teste retido, a instituição consegue capturar preventivamente mais de 1/3 dos inadimplentes graves logo no primeiro filtro cadastral.

### Limitações e próximos passos

1. Consulta a Dados Externos de Mercado: O modelo utiliza apenas dados cadastrais internos. Recomenda-se integrar dados do Serasa/SPC e do Open Finance para avaliar o endividamento real do solicitante em outras instituições.
2. Monitoramento de Data Drift: Alterações na economia (inflação, desemprego e juros) afetam diretamente o perfil de pagamento. É indispensável implantar métricas de monitoramento contínuo para recalibrar e re-treinar o modelo trimestralmente

---

## 7. Estrutura do repositório

```
.
├── data/          dados brutos (raw) e tratados (processed) — não versionados
├── notebooks/     análise em ordem numerada
└── docs/          apresentação executiva
```

Detalhes e convenções em [`ESTRUTURA.md`](ESTRUTURA.md).
Antes de enviar, percorra o [`CHECKLIST.md`](CHECKLIST.md).

---

## 8. Tecnologias

<!-- PREENCHER: Python 3.11, pandas, scikit-learn, ... -->
