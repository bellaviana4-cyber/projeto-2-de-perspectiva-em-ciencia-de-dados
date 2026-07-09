# Cobertura de imprensa e a transição elétrica: uma análise do Motor1 UOL entre 2016 e 2026

Projeto acadêmico desenvolvido em dupla para a disciplina **Perspectiva em Ciência de Dados**, com foco na análise de notícias automotivas publicadas pelo site **Motor1 UOL** entre 2016 e 2026.

O objetivo central do trabalho é investigar como a cobertura jornalística sobre o setor automotivo brasileiro acompanhou a transição para veículos eletrificados, especialmente a presença de temas como carros elétricos, híbridos, SUVs, direção autônoma, diesel e marcas automotivas ao longo do tempo.

## Pergunta de pesquisa

**A cobertura de imprensa do Motor1 antecipou ou apenas acompanhou a transição elétrica no mercado automotivo brasileiro?**

A partir dessa pergunta, o projeto foi estruturado em etapas de coleta, tratamento, análise exploratória, mineração de texto, modelagem supervisionada, explicabilidade e validação externa com dados de mercado.

## Fonte dos dados

Os dados foram coletados por meio de **web scraping** no site Motor1 UOL, considerando páginas de notícias automotivas publicadas entre 2016 e 2026.

O dataset principal reúne aproximadamente **26,6 mil notícias** e contém informações como:

- título da notícia;
- subtítulo;
- texto completo;
- categoria;
- autor;
- data de publicação;
- ano e mês da publicação;
- termos de interesse identificados;
- marcas automotivas mencionadas;
- variáveis textuais criadas nas etapas de processamento.

O arquivo principal utilizado no projeto está localizado em:

```text
projeto/dados/noticias_motor1_2016_2026.csv
```

Além da base principal, o projeto também gera arquivos intermediários usados nas análises, como bases tratadas, matrizes Bag-of-Words, matrizes TF-IDF e bases específicas para modelagem.

Arquivos intermediários grandes, como matrizes completas de Bag-of-Words e TF-IDF, podem não estar versionados por tamanho. Nesses casos, eles devem ser recriados localmente a partir dos notebooks de pré-processamento.

## Dashboard HTML

O projeto inclui um **dashboard em HTML** construído com o dataset das notícias extraídas via web scraping.

O dashboard foi desenvolvido para apresentar os principais resultados de forma visual e interativa. Ele reúne gráficos e tabelas relacionados à evolução temporal das notícias, presença de temas associados à eletrificação, participação das marcas, distribuição dos assuntos e outros recortes relevantes da cobertura jornalística.

Essa etapa complementa os notebooks analíticos, facilitando a interpretação dos padrões encontrados ao longo do projeto e servindo como material de apoio para apresentação dos resultados.

## Estrutura do repositório

```text
projeto-2-de-perspectiva-em-ciencia-de-dados/
|
|-- README.md
|-- pyproject.toml
|-- uv.lock
|
`-- projeto/
    |-- dados/
    |   |-- noticias_motor1_2016_2026.csv
    |   `-- demais arquivos gerados durante o processamento e a modelagem
    |
    `-- notebooks/
        |-- webscrapping/
        |   `-- coleta das notícias por web scraping
        |
        |-- pln/
        |   `-- limpeza, normalização textual, Bag-of-Words e TF-IDF
        |
        |-- analise_temporal/
        |   `-- evolução dos temas, termos e marcas ao longo do tempo
        |
        |-- clusterizacao/
        |   `-- agrupamento não supervisionado de notícias
        |
        |-- modelagem/
        |   `-- modelos supervisionados, SHAP e previsão de volume de notícias
        |
        |-- rede_neural_shap/
        |   `-- classificação com rede neural e interpretação com SHAP
        |
        `-- validacao_externa/
            `-- comparação entre cobertura jornalística e dados reais de mercado
```

## Metodologia e principais achados por etapa

### Etapa 0 - Coleta dos dados

`projeto/notebooks/webscrapping/webscraping.ipynb`

A primeira etapa consistiu na coleta das notícias do Motor1 UOL por meio de web scraping.

Foram acessadas páginas de arquivo mensal do site, com extração de links, títulos, datas, autores, categorias, subtítulos e textos completos das notícias. A coleta foi estruturada com checkpoints para permitir a retomada do processo em caso de interrupção.

Depois da extração, os dados foram consolidados em um dataset único e avaliados quanto à duplicidade, consistência das datas, preenchimento dos principais campos e cobertura temporal.

**Resultado:** foi gerada uma base final com aproximadamente 26,6 mil notícias, pronta para as etapas de análise textual e modelagem.

### Pré-processamento de texto - PLN

`projeto/notebooks/pln/vetorizacao-dados.ipynb`

Após a coleta, os textos foram preparados para as etapas de análise em linguagem natural.

As principais operações realizadas foram:

- padronização dos textos;
- conversão para letras minúsculas;
- remoção de acentos;
- remoção de pontuações;
- remoção de stopwords em português;
- remoção de trechos recorrentes e ruídos do site;
- criação de colunas auxiliares de data, tema, marca e texto tratado.

Também foram construídas representações vetoriais dos textos por meio de:

- **Bag-of-Words**;
- **TF-IDF**.

Essas matrizes foram reaproveitadas nas etapas de clusterização, classificação supervisionada e rede neural.

### Etapa 1 - Tendência temporal de termos

`projeto/notebooks/analise_temporal/tendencia_termos.ipynb`

Nesta etapa, foi analisada a evolução dos principais termos associados ao setor automotivo ao longo dos anos.

Os temas observados incluíram:

- elétrico;
- híbrido;
- SUV;
- diesel;
- direção autônoma.

**Principais interpretações:**

- **Elétrico** e **híbrido** cresceram de forma forte e estatisticamente significativa, com teste de Mann-Kendall indicando p < 0,01.
- A presença de notícias com termos ligados a elétrico e híbrido passou de cerca de 3% em 2016 para mais de 40% em 2025.
- **SUV** também apresentou crescimento estatisticamente significativo, porém de forma mais moderada.
- **Direção autônoma** e **diesel** não apresentaram tendência estatisticamente significativa no período analisado.

A análise temporal mostra que a eletrificação se tornou progressivamente mais presente na pauta jornalística automotiva, principalmente nos anos mais recentes.

### Etapa 2 - Clustering e tópicos com t-SNE

`projeto/notebooks/clusterizacao/clustering_topicos.ipynb`

A etapa de clusterização teve como objetivo identificar agrupamentos temáticos nas notícias sem informar previamente ao algoritmo quais eram os termos de interesse da análise temporal.

Foram utilizadas técnicas de representação textual, redução de dimensionalidade e agrupamento, incluindo:

- TF-IDF;
- SVD;
- t-SNE;
- K-Means.

**Principais interpretações:**

- O agrupamento não supervisionado encontrou clusters que coincidem parcialmente com os termos de elétrico e híbrido.
- Esse resultado reforça que a lista de termos usada na análise temporal captura um agrupamento temático real dentro do conjunto de notícias.
- A análise também revelou um tópico fora da lista original: a cobertura de **Fórmula 1**. Isso reforça a validade do método, pois o algoritmo foi capaz de identificar um tema relevante sem receber essa informação previamente.
- O coeficiente de silhueta ficou baixo, em torno de 0,06 a 0,08. Essa limitação é esperada para textos jornalísticos de um mesmo nicho, pois as notícias compartilham vocabulário semelhante e os grupos não são completamente separados.

Os clusters não devem ser interpretados como categorias rígidas, mas como evidências de padrões textuais presentes na base.

### Etapa 3 - Cruzamento entre termo, marca e sentimento

`projeto/notebooks/analise_temporal/cruzamento_marca.ipynb`

Nesta etapa, o projeto analisou a relação entre marcas automotivas e temas de interesse, especialmente a presença do tema elétrico na cobertura de cada marca.

Foram avaliados aspectos como:

- volume de notícias por marca;
- proporção de notícias sobre elétricos dentro da cobertura de cada marca;
- evolução inicial e final da presença dos temas;
- diferenças entre marcas tradicionais e marcas mais associadas à eletrificação;
- sentimento textual com base em abordagem léxica simples.

**Principais interpretações:**

- Em volume bruto, o tema elétrico aparece mais nas marcas que a imprensa mais cobre em geral, como **Volkswagen**, **Toyota**, **BMW** e **Ford**.
- Em proporção dentro da cobertura de cada marca nos anos iniciais, especialmente entre 2016 e 2019, **Audi** e **Nissan** aparecem à frente.
- Esse resultado sugere um possível pioneirismo editorial pontual em torno dessas marcas, já que elas aparecem proporcionalmente mais associadas à eletrificação antes de o tema se tornar dominante.
- A análise de sentimento, baseada em léxico simples, indica que a maioria das marcas tende a ser retratada de forma mais positiva em notícias sobre elétricos.
- A **Tesla** aparece como exceção, com sentimento médio mais baixo mesmo dentro da cobertura de elétricos, possivelmente por estar associada a notícias sobre problemas, controvérsias, atrasos ou decisões de mercado.

Essa etapa mostrou que a análise por marca acrescenta nuance ao resultado geral. A cobertura como um todo acompanha a transição elétrica, mas algumas marcas aparecem associadas ao tema em momentos anteriores.

### Etapa 4 - Classificação supervisionada

`projeto/notebooks/modelagem/classificacao_supervisionada.ipynb`

A classificação supervisionada foi utilizada para avaliar se os padrões textuais identificados anteriormente poderiam ser reconhecidos por modelos preditivos.

Foram construídos modelos com Regressão Logística sobre TF-IDF para prever, a partir do conteúdo das notícias:

- o cluster associado à notícia;
- a marca relacionada ao texto;
- a presença de temas ligados à eletrificação.

**Principais interpretações:**

- O modelo conseguiu prever o cluster da etapa de agrupamento com cerca de **97% de acurácia**, contra um baseline de aproximadamente 35%.
- Esse resultado indica que os clusters formam regiões textuais coesas e bem separáveis no espaço de vocabulário.
- Na previsão de marca, o modelo alcançou cerca de **90% de acurácia**, mesmo com remoção do nome da marca do texto, contra um baseline de aproximadamente 15%.
- Isso sugere que cada marca possui um vocabulário próprio, associado a nomes de modelos, termos técnicos, posicionamento de mercado e temas recorrentes.
- O desempenho é menor em marcas de nicho, como BYD, cujo vocabulário se sobrepõe ao de outras marcas do segmento elétrico.

Essa etapa funcionou como uma checagem de consistência: se os rótulos e agrupamentos não fossem informativos, os modelos supervisionados tenderiam a apresentar desempenho próximo aos baselines.

### Classificação complementar - Rede neural e SHAP

`projeto/notebooks/rede_neural_shap/rede_neural_shap_bow.ipynb`

O projeto também inclui uma etapa de modelagem com rede neural para classificar notícias relacionadas a carros elétricos.

Foi utilizado um modelo do tipo **MLPClassifier** com representações textuais em Bag-of-Words. O desempenho foi avaliado por métricas de classificação, incluindo:

- acurácia;
- precisão;
- valor preditivo positivo;
- valor preditivo negativo;
- recall;
- F1-score;
- curva ROC;
- AUC.

**Principais interpretações:**

- A rede neural obteve **84,3% de acurácia** na classificação de notícias sobre carros elétricos.
- A curva ROC apresentou **AUC de 91,96%**, indicando boa capacidade de separação entre notícias relacionadas e não relacionadas ao tema.
- A explicabilidade via SHAP, usando KernelExplainer, mostrou que os termos mais importantes para a classificação foram semanticamente coerentes com o problema.
- Entre os termos mais relevantes aparecem palavras como `bateria`, `kwh`, `hibrido`, `ev` e `autonomia`.

Esse resultado indica que o modelo não se baseou apenas em padrões aleatórios do texto, mas em palavras diretamente relacionadas ao tema de eletrificação.

### Etapa 5 - Modelagem numérica com SHAP e Conformal Prediction

`projeto/notebooks/modelagem/modelagem_volume.ipynb`

Outra etapa do projeto consistiu na modelagem do volume mensal de notícias relacionadas a veículos elétricos.

Foi construído um modelo de regressão com **Random Forest** para estimar a quantidade de publicações mensais sobre o tema. A interpretação do modelo foi feita com **SHAP**, permitindo identificar quais variáveis mais contribuíram para as previsões.

Também foi utilizada uma abordagem de **Conformal Prediction** para representar a incerteza das estimativas. Essa técnica cria intervalos preditivos ao redor das previsões sem depender da suposição de normalidade dos erros.

**Principais interpretações:**

- O modelo de Random Forest apresentou desempenho superior ao baseline baseado na média do treino.
- O R² do baseline ficou negativo, o que indica que uma previsão constante pela média histórica não foi capaz de acompanhar a evolução da série temporal.
- O modelo principal apresentou R² em torno de **0,75**, indicando que conseguiu capturar parte relevante da variação mensal no volume de notícias sobre elétricos.
- A análise com SHAP apontou **ano** e **volume total de notícias do mês** como fatores importantes para a previsão.
- Isso sugere que o crescimento da cobertura está ligado à passagem do tempo e ao volume geral de publicações do site, e não apenas a ruídos isolados da base.
- Os intervalos de Conformal Prediction foram usados para mostrar que a previsão de volume deve ser interpretada com incerteza, especialmente em meses com comportamento fora do padrão.

Essa etapa não buscou apenas prever a quantidade de notícias, mas também entender quais fatores ajudam a explicar o aumento ou a redução da cobertura sobre eletrificação.

**Decisão de escopo:** o componente de difusão ou VAE previsto no planejamento inicial não foi implementado. Como a série mensal possui cerca de 120 observações, um modelo generativo profundo não teria dados suficientes para treinamento confiável. Por isso, a modelagem foi concentrada em Random Forest, SHAP e Conformal Prediction.

### Etapa 6 - Validação externa com dados de mercado

`projeto/notebooks/validacao_externa/validacao_externa.ipynb`

A etapa final comparou a cobertura jornalística sobre veículos eletrificados com dados externos de vendas do mercado automotivo brasileiro, utilizando informações de fontes como ABVE e Anfavea.

O objetivo foi avaliar se a imprensa teria antecipado o crescimento do mercado de eletrificados ou se apenas acompanhou um movimento já observado nas vendas reais.

**Principais interpretações:**

- A proporção de cobertura sobre elétricos e híbridos cresce junto com os dados de vendas de veículos eletrificados no Brasil entre 2016 e 2025.
- Não foi observado um descolamento claro entre imprensa e mercado que permitisse afirmar uma antecipação sistemática da cobertura.
- A correlação de Pearson ficou forte, em torno de **0,87**, em diferentes defasagens testadas.
- As diferenças entre os cenários de cobertura antecipando vendas, cobertura contemporânea e cobertura atrasada foram pequenas demais para afirmar uma direção com confiança.
- A principal limitação dessa etapa é o número reduzido de pontos anuais, cerca de 10 anos, o que limita o poder estatístico dos testes de defasagem.

A validação externa reforça a conclusão de que a cobertura jornalística acompanhou de perto a transição elétrica, mas não permite afirmar antecipação sistemática do mercado.

## Síntese dos principais resultados

Os principais resultados encontrados foram:

- a presença de termos relacionados a veículos elétricos e híbridos cresceu de forma forte ao longo do período analisado;
- a cobertura sobre eletrificação tornou-se mais frequente principalmente nos anos mais recentes;
- o crescimento dos termos elétrico e híbrido foi estatisticamente significativo pelo teste de Mann-Kendall;
- SUV também apresentou crescimento, porém mais moderado;
- direção autônoma e diesel não apresentaram tendência estatisticamente significativa;
- algumas marcas aparecem mais associadas ao tema elétrico pelo volume total de notícias;
- outras marcas, como Audi e Nissan, se destacam proporcionalmente nos anos iniciais;
- os clusters textuais identificam agrupamentos coerentes com temas reais da cobertura, ainda que com separação limitada;
- os modelos supervisionados conseguem reconhecer padrões associados a temas, marcas e clusters;
- a rede neural apresentou desempenho consistente na identificação de notícias sobre carros elétricos;
- a análise com SHAP indicou que os modelos utilizaram termos semanticamente compatíveis com o problema estudado;
- a modelagem de volume indicou que ano e volume total de notícias são variáveis importantes para explicar o crescimento da cobertura;
- a validação externa mostrou forte associação entre cobertura jornalística e crescimento do mercado de eletrificados, mas sem evidência suficiente de antecipação sistemática.

## Tecnologias utilizadas

O projeto foi desenvolvido em Python, com uso de bibliotecas voltadas para coleta, tratamento, análise textual, visualização, modelagem e explicabilidade.

Principais tecnologias e bibliotecas utilizadas:

- Python;
- Jupyter Notebook;
- pandas;
- numpy;
- requests;
- BeautifulSoup;
- scikit-learn;
- matplotlib;
- plotly;
- SHAP;
- scipy;
- pymannkendall;
- MAPIE;
- uv.

## Como executar o projeto

Este projeto utiliza `uv` para gerenciamento de ambiente e dependências.

Para instalar as dependências:

```bash
uv sync
```

Para abrir os notebooks:

```bash
uv run jupyter lab
```

Ordem recomendada de execução:

```text
1. projeto/notebooks/webscrapping/webscraping.ipynb
   Gera noticias_motor1_2016_2026.csv.

2. projeto/notebooks/pln/vetorizacao-dados.ipynb
   Realiza a limpeza textual e gera bow_motor1.csv,
   tfidf_motor1.csv e noticias_motor1_processado.csv.

3. projeto/notebooks/analise_temporal/tendencia_termos.ipynb
   Analisa a evolução dos termos ao longo do tempo.

4. projeto/notebooks/clusterizacao/clustering_topicos.ipynb
   Realiza a clusterização e a análise de tópicos.

5. projeto/notebooks/analise_temporal/cruzamento_marca.ipynb
   Analisa a relação entre termos, marcas e sentimento.

6. projeto/notebooks/modelagem/classificacao_supervisionada.ipynb
   Avalia a consistência dos agrupamentos e rótulos por modelos supervisionados.

7. projeto/notebooks/rede_neural_shap/rede_neural_shap_bow.ipynb
   Classifica notícias sobre carros elétricos com rede neural e interpreta os resultados com SHAP.

8. projeto/notebooks/modelagem/modelagem_volume.ipynb
   Modela o volume mensal de notícias sobre elétricos com SHAP e Conformal Prediction.

9. projeto/notebooks/validacao_externa/validacao_externa.ipynb
   Compara a cobertura jornalística com dados externos de mercado.
```

Cada notebook é independente na leitura dos dados de entrada principais, mas alguns arquivos intermediários precisam ser gerados antes de etapas específicas. Por exemplo, o notebook de rede neural depende das matrizes textuais criadas na etapa de PLN.

## Limitações

O projeto possui algumas limitações metodológicas:

- a identificação de marcas foi baseada em regras textuais e expressões regulares aplicadas principalmente ao título;
- notícias sem marca reconhecida ficam fora das análises que dependem dessa variável;
- os rótulos de temas, como “elétrico” e “híbrido”, foram definidos por heurísticas de palavras-chave, e não por anotação manual;
- a análise de sentimento utilizou uma abordagem léxica simples, sem validação humana dos sentimentos atribuídos;
- séries anuais possuem poucos pontos, o que limita o poder estatístico de testes de tendência e defasagem;
- os coeficientes de silhueta baixos na clusterização indicam que os grupos textuais não são fortemente separados;
- os resultados representam a cobertura de um veículo jornalístico específico e não necessariamente toda a imprensa automotiva brasileira;
- os dados coletados dependem da estrutura disponível no site no momento da extração.

## Conclusão

A análise indica que a cobertura do Motor1 UOL acompanhou de perto a transição elétrica no mercado automotivo brasileiro entre 2016 e 2026.

Embora existam sinais pontuais de antecipação editorial em algumas marcas e temas, os resultados não sustentam a conclusão de que a cobertura jornalística tenha antecipado de forma sistemática o avanço dos veículos eletrificados no mercado.

O principal achado é que a imprensa analisada reagiu de maneira próxima e progressiva às transformações do setor, refletindo o crescimento da eletrificação como tema cada vez mais importante na indústria automotiva.