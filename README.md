# Cobertura de imprensa e a transição elétrica: uma análise do Motor1 (2016–2026)

Projeto 2 da disciplina **Perspectiva em Ciência de Dados**, feito em dupla, a partir de notícias automotivas coletadas do site [motor1.uol.com.br](https://motor1.uol.com.br).

## Pergunta de pesquisa

> **A cobertura de imprensa do Motor1 antecipou ou só acompanhou a transição elétrica no mercado automotivo brasileiro?**

Todas as etapas do projeto foram desenhadas como evidências para responder essa pergunta central, em vez de serem análises soltas e desconectadas entre si.

## Resposta curta (a versão longa está na seção de achados)

A cobertura do Motor1 **acompanhou de perto** a transição elétrica — com crescimento forte e estatisticamente significativo (Mann-Kendall, p < 0,01) — e, para marcas específicas como Nissan e Audi, chegou a **antecipar** a pauta antes de ela virar tendência geral de mercado. Mas, olhando o site como um todo e comparando com dados reais de venda (ABVE/Anfavea), **não há evidência estatística suficiente de que o Motor1 tenha antecipado a transição de forma sistemática** — a correlação entre cobertura e vendas é forte (~0,87) em qualquer defasagem testada, sem uma direção clara. A conclusão mais honesta é: acompanhamento próximo e contemporâneo, com pioneirismo pontual em algumas marcas, não uma antecipação generalizada.

## Sobre os dados

- **Fonte:** notícias de carro do [motor1.uol.com.br](https://motor1.uol.com.br), coletadas via web scraping (`requests` + `BeautifulSoup`).
- **Período:** 2016 a 2026 (~26,6 mil notícias).
- **Campos coletados:** título, subtítulo, texto completo, categoria, autor e data de publicação.
- O dataset bruto está em `projeto/dados/noticias_motor1_2016_2026.csv`. Arquivos intermediários grandes (matrizes Bag-of-Words e TF-IDF completas) não são versionados — veja [Como rodar](#como-rodar) para gerá-los localmente.

## Estrutura do repositório

```
projeto/
├── dados/                       # CSVs de entrada e de saída de cada notebook
└── notebooks/
    ├── webscrapping/             # Coleta dos dados (Etapa 0)
    ├── pln/                      # Pré-processamento de texto, Bag-of-Words e TF-IDF
    ├── analise_temporal/         # Etapas 1 e 3
    ├── clusterizacao/            # Etapa 2
    ├── modelagem/                # Etapas 4 e 5
    ├── rede_neural_shap/         # Classificação complementar (elétrico x não elétrico)
    └── validacao_externa/        # Etapa 6
```

## Metodologia e principais achados, etapa por etapa

### Etapa 0 — Coleta dos dados
`notebooks/webscrapping/webscraping.ipynb`
Scraping estático via páginas de arquivo mensal do site, com checkpoint incremental para retomar a coleta caso seja interrompida. Resultado: dataset final verificado quanto a duplicatas, tipos de dado e cobertura temporal.

### Pré-processamento de texto (PLN)
`notebooks/pln/vetorizacao-dados.ipynb`
Limpeza de texto (remoção de boilerplate do site, acentos, pontuação, stopwords em português) e construção das representações **Bag-of-Words** e **TF-IDF**, reaproveitadas nas etapas seguintes.

### Etapa 1 — Tendência temporal de termos
`notebooks/analise_temporal/tendencia_termos.ipynb`
- **Elétrico** e **híbrido** crescem de forma forte e estatisticamente significativa (Mann-Kendall, p < 0,01): de ~3% das notícias em 2016 para mais de 40% em 2025.
- **SUV** também cresce de forma significativa, porém mais moderada.
- **Direção autônoma** e **diesel** não apresentam tendência estatisticamente significativa no período.

### Etapa 2 — Clustering / tópicos com t-SNE
`notebooks/clusterizacao/clustering_topicos.ipynb`
Agrupamento não supervisionado (TF-IDF + SVD + t-SNE + K-Means), feito **sem informar ao algoritmo** os termos de interesse da Etapa 1.
- Encontrou clusters que coincidem parcialmente com os termos de elétrico/híbrido, confirmando que a lista de termos da Etapa 1 captura um agrupamento temático real.
- Revelou um tópico fora da lista original (cobertura de Fórmula 1), o que reforça a validade do recorte da análise.
- Limitação assumida: coeficiente de silhueta baixo (0,06–0,08), esperado para texto jornalístico de um nicho temático único.

### Etapa 3 — Cruzamento termo × marca + sentimento
`notebooks/analise_temporal/cruzamento_marca.ipynb`
- Em volume bruto, elétrico é mais coberto nas marcas que a imprensa mais cobre em geral (Volkswagen, Toyota, BMW, Ford).
- Em **proporção** dentro da cobertura de cada marca nos anos iniciais (2016–2019), **Audi** e **Nissan** aparecem à frente — indício de pioneirismo editorial em torno dessas marcas.
- Análise de sentimento (léxico simples): a maioria das marcas é retratada de forma mais positiva em notícias sobre elétrico; a Tesla é exceção, com sentimento médio mais baixo mesmo em cobertura de elétrico.

### Etapa 4 — Classificação supervisionada (checagem de consistência)
`notebooks/modelagem/classificacao_supervisionada.ipynb`
Usa Regressão Logística sobre TF-IDF para checar, de forma supervisionada, se os agrupamentos das etapas anteriores são reais:
- **Cluster:** ~97% de acurácia ao prever o cluster da Etapa 2 a partir do texto (baseline: 35%) — os clusters formam regiões coesas e bem separáveis no espaço de vocabulário.
- **Marca:** ~90% de acurácia ao prever a marca mesmo removendo o nome da marca do texto (baseline: ~15%) — cada marca tem um vocabulário próprio (nomes de modelo, jargão), com pior desempenho em marcas de nicho como BYD, cujo vocabulário se sobrepõe ao de outras marcas do segmento elétrico.

### Classificação complementar — Rede neural + SHAP
`notebooks/rede_neural_shap/rede_neural_shap_bow.ipynb`
Um MLPClassifier sobre Bag-of-Words prevê se uma notícia é sobre carro elétrico com **acurácia de 84,3%** e **ROC AUC de 91,96%**. A explicabilidade via SHAP (KernelExplainer) confirma aderência semântica: os termos mais importantes são `bateria`, `kwh`, `hibrido`, `ev` e `autonomia`.

### Etapa 5 — Modelagem numérica (SHAP + Conformal Prediction)
`notebooks/modelagem/modelagem_volume.ipynb`
Um Random Forest prevê o volume mensal de notícias sobre elétrico (R² = 0,75 contra baseline negativo).
- SHAP aponta **ano** e **volume total de notícias do mês** como fatores mais importantes — o crescimento está ligado à passagem do tempo e ao volume geral do site, não a fatores artificiais.
- Conformal Prediction (via `mapie`) fornece intervalos de confiança de 90% sem assumir normalidade dos erros.
- **Decisão de escopo:** o componente de difusão/VAE previsto no planejamento original não foi implementado — com apenas 120 observações mensais, um modelo generativo profundo não teria dados suficientes para treinar de forma confiável, então o tempo foi investido em aprofundar SHAP e conformal prediction.

### Etapa 6 — Validação externa (cobertura × vendas reais)
`notebooks/validacao_externa/validacao_externa.ipynb`
Cruza a proporção de cobertura sobre elétrico/híbrido com dados reais de venda de veículos eletrificados no Brasil (ABVE/Anfavea).
- As duas curvas crescem **juntas** ao longo de 2016–2025, sem descolamento brusco.
- Correlação de Pearson forte (~0,87) em qualquer defasagem testada (cobertura antecipando vendas, contemporânea, ou atrasada) — as diferenças entre os cenários são pequenas demais para afirmar direção com confiança.
- **Limitação assumida:** com apenas 10 pontos anuais, o teste de defasagem tem baixo poder estatístico. Uma extensão natural seria repetir a análise com dados mensais de vendas.

## Limitações gerais do projeto

- A extração de **marca** é feita por regex sobre o título; notícias sem marca reconhecida (~23%) ficam fora das análises que dependem dela.
- O rótulo "elétrico" (usado em várias etapas) é heurístico, construído por regras textuais, não por anotação manual — reproduz uma definição operacional, não uma classificação editorial validada por humanos.
- Séries anuais (Etapa 1 e Etapa 6) têm poucos pontos (10 anos), o que limita o poder estatístico de testes de tendência e defasagem.
- Os coeficientes de silhueta baixos na Etapa 2 indicam que os clusters de texto jornalístico sobre carros não são fortemente separados — a leitura qualitativa dos termos por cluster complementa (e é mais informativa que) a métrica quantitativa.

## Como rodar

Este projeto usa [uv](https://docs.astral.sh/uv/) para gerenciar dependências (`pyproject.toml` / `uv.lock`).

```bash
uv sync
uv run jupyter lab
```

**Ordem recomendada de execução** (alguns notebooks dependem de arquivos gerados pelos anteriores):

1. `webscrapping/webscraping.ipynb` — gera `noticias_motor1_2016_2026.csv`
2. `pln/vetorizacao-dados.ipynb` — gera `bow_motor1.csv`, `tfidf_motor1.csv` e `noticias_motor1_processado.csv` (não versionados por tamanho; precisam ser gerados localmente antes do notebook de rede neural)
3. `analise_temporal/tendencia_termos.ipynb` (Etapa 1)
4. `clusterizacao/clustering_topicos.ipynb` (Etapa 2)
5. `analise_temporal/cruzamento_marca.ipynb` (Etapa 3)
6. `modelagem/classificacao_supervisionada.ipynb` (Etapa 4)
7. `rede_neural_shap/rede_neural_shap_bow.ipynb` (depende do passo 2)
8. `modelagem/modelagem_volume.ipynb` (Etapa 5)
9. `validacao_externa/validacao_externa.ipynb` (Etapa 6)

Cada notebook é independente na leitura dos dados de entrada (recalcula a limpeza de texto quando necessário), então também é possível rodar cada um isoladamente, desde que `noticias_motor1_2016_2026.csv` já exista.
