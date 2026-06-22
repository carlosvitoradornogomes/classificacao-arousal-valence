
 CLASSIFICADOR AROUSAL-VALENCE: Q1-Q4 (MODELO CIRCUMPLEXO DE RUSSELL)


Dissertação de Mestrado:
  "Caracterizacao espectral de emocoes musicais no espaco arousal-valence:
   uma analise quadrante por quadrante"

Autor:
  Eng. Carlos Vitor Adorno Gomes -- PPGEEC/UFBA
  
Orientador:
  Dr. Eduardo Furtado de Simas Filho -- PPGEEC/UFBA

Dataset utilizado:
  DEAM (Database for Emotion Analysis using Music), 1802 faixas musicais
  com anotacoes dinamicas de arousal e valence.


--------------------------------------------------------------------------------
 1. VISAO GERAL
--------------------------------------------------------------------------------

Este repositorio contem um notebook Jupyter (Classificador_Arousal_Valence.ipynb)
que reproduz a metodologia do artigo acima para classificar faixas musicais em
quatro quadrantes emocionais do modelo circumplexo de Russell:

  Q1 - Alto Arousal  / Alta  Valence  -> Animado / Euforico
  Q2 - Baixo Arousal / Alta  Valence  -> Calmo   / Sereno
  Q3 - Baixo Arousal / Baixa Valence  -> Triste  / Melancolico
  Q4 - Alto Arousal  / Baixa Valence  -> Tenso   / Raivoso

O particionamento em quadrantes e feito pelas medianas globais de arousal e
valence do dataset (garantindo aproximadamente 450 amostras por classe).

O pipeline extrai 5 descritores espectrais/energeticos de cada faixa, aplica
testes estatisticos nao-parametricos (Kruskal-Wallis + post-hoc de Dunn) e
treina um classificador Random Forest com validacao cruzada estratificada.


--------------------------------------------------------------------------------
 2. ESTRUTURA DO PIPELINE (SECOES DO NOTEBOOK)
--------------------------------------------------------------------------------

  1. Importacoes de bibliotecas
  2. Configuracao dos caminhos de arquivos (CSVs e pasta de audio)
  3. Carregamento das anotacoes DEAM e particionamento em Q1-Q4
  4. Pre-processamento e definicao das funcoes de extracao de features
     espectrais (janela de Hann, PSD de Welch, centroide, largura de banda,
     roll-off, ZCR, energia RMS)
  5. Extracao de features dos arquivos .mp3 do DEAM (ou carregamento de um
     CSV de features ja extraidas)
  6. Construcao do dataset final (merge features + anotacoes + quadrante)
  7. Analise estatistica: teste de Kruskal-Wallis e tamanho de efeito (eta2)
  8. Histogramas e boxplots das features por quadrante
  8.5. Visualizacao do espaco de caracteristicas via PCA (2 componentes)
  9. Densidade espectral de potencia (PSD) media por quadrante (metodo de
     Welch)
  10. Classificador Random Forest com validacao cruzada estratificada k=5
  11. Resumo final dos resultados

--------------------------------------------------------------------------------
 3. DESCRITORES (FEATURES) EXTRAIDOS
--------------------------------------------------------------------------------

  - spectral_centroid   : centroide espectral (Hz)
  - spectral_bandwidth  : largura de banda espectral (Hz)
  - spectral_rolloff    : frequencia de roll-off a 85% da energia (Hz)
  - zcr                 : taxa de cruzamentos por zero
  - rms_energy          : energia RMS do sinal normalizado

Parametros de processamento:
  - Taxa de amostragem (SR)     : 22050 Hz
  - Tamanho da janela (L)       : 2048 amostras (~46,4 ms a 44100 Hz)
  - Sobreposicao (hop)          : 50% (L // 2)
  - Limiar de roll-off          : 85% da energia espectral
  - Janela utilizada            : Hann


--------------------------------------------------------------------------------
 4. ESTRUTURA DE PASTAS ESPERADA
--------------------------------------------------------------------------------

  projeto/
  |-- Classificador_Arousal_Valence.ipynb
  |-- README.txt
  |-- Images/                          (criada automaticamente pelo notebook)
  |   |-- circumplex_scatter.png
  |   |-- histogramas.png
  |   |-- boxplots.png
  |   |-- pca_espaco_caracteristicas.png
  |   |-- resultados_espaco_caracteristicas.txt
  |   |-- PSD.png
  |   |-- dunn_posthoc.png
  |   |-- matriz_confusao.png
  |   |-- importancia_features.png
  |   |-- tabela_describe_features.png
  |   |-- tabela_kruskal_wallis.png
  |   |-- tabela_dunn_posthoc.png
  |   |-- tabela_metricas_por_quadrante.png
  |   `-- tabela_resumo_resultados.png
  `-- features_extraidas.csv           (gerado pela Secao 5, se aplicavel)

Os dados do DEAM (audio e anotacoes) NAO estao incluidos neste repositorio
devido ao tamanho e a licenca do dataset. Veja a Secao 6 para obtencao.


--------------------------------------------------------------------------------
 5. DEPENDENCIAS
--------------------------------------------------------------------------------

Requer Python 3.9+ e as seguintes bibliotecas:

  numpy
  pandas
  matplotlib
  scipy
  scikit-posthocs
  seaborn
  librosa
  tqdm
  scikit-learn
  jupyter (ou jupyterlab)

Instalacao via pip:

  pip install numpy pandas matplotlib scipy scikit-posthocs seaborn \
              librosa tqdm scikit-learn jupyter

Observacao: o librosa depende do ffmpeg/libsndfile para leitura de arquivos
.mp3. Em sistemas Linux/Debian/Ubuntu, instale com:

  sudo apt-get install ffmpeg libsndfile1


--------------------------------------------------------------------------------
 6. OBTENCAO DO DATASET DEAM
--------------------------------------------------------------------------------

O dataset DEAM (Database for Emotion Analysis using Music) esta disponivel
publicamente para fins academicos em:

  https://cvml.unige.ch/databases/DEAM/

Sao necessarios:
  - Os arquivos de audio (.mp3)
  - As anotacoes dinamicas (per-second) de arousal e valence:
      annotations/annotations averaged per song/dynamic (per second
      annotations)/arousal.csv
      annotations/annotations averaged per song/dynamic (per second
      annotations)/valence.csv

Apos o download, ajuste os caminhos na Secao 2 do notebook (ver Secao 7
abaixo).


--------------------------------------------------------------------------------
 7. CONFIGURACAO ANTES DE EXECUTAR
--------------------------------------------------------------------------------

Na Secao 2 do notebook ("Caminhos dos Arquivos"), ajuste as 4 variaveis
abaixo para o seu ambiente. Os valores padrao do notebook sao apenas
EXEMPLOS da maquina original do autor e NAO existirao no seu computador:

  PATH_AROUSAL  -> CSV de anotacoes de arousal do DEAM
  PATH_VALENCE  -> CSV de anotacoes de valence do DEAM
  PATH_AUDIO    -> pasta contendo os arquivos .mp3 do DEAM
  PATH_FEATURES -> CSV de features ja extraidas (opcional, ver Secao 8)

Se os caminhos nao forem corrigidos, o notebook falhara com
FileNotFoundError ao tentar carregar os arquivos.


--------------------------------------------------------------------------------
 8. COMO EXECUTAR
--------------------------------------------------------------------------------

1. Clone o repositorio e instale as dependencias (Secao 5).
2. Baixe o dataset DEAM e ajuste os caminhos (Secoes 6 e 7).
3. Abra o notebook:

     jupyter notebook Classificador_Arousal_Valence.ipynb

4. Execute as celulas em ordem, de cima para baixo.

   - Se for a PRIMEIRA execucao (sem CSV de features pre-extraidas):
     execute a celula da Secao 5 ("Extracao de Features das Musicas do
     DEAM"). Ela percorre todos os arquivos .mp3, extrai os 5 descritores
     e salva o resultado em "features_extraidas.csv".

   - Se voce JA POSSUI um CSV de features extraidas previamente:
     pule a celula de extracao e execute a celula alternativa da Secao
     "Alternativa: carregar CSV de features pre-extraidas", que apenas le
     o CSV indicado em PATH_FEATURES.

5. As demais secoes (estatistica, visualizacoes, PCA, PSD, Random Forest e
   resumo final) podem ser executadas em sequencia sem intervencao manual.


--------------------------------------------------------------------------------
 9. SAIDAS GERADAS
--------------------------------------------------------------------------------

Com SAVE_FIGURES = True (padrao), todas as figuras e tabelas de resultado
sao salvas automaticamente na pasta "Images/":

  Graficos:
    - circumplex_scatter.png        : dispersao no espaco arousal-valence
    - histogramas.png               : histogramas das features por quadrante
    - boxplots.png                  : boxplots das features por quadrante
    - pca_espaco_caracteristicas.png: projecao PCA 2D das features
    - PSD.png                       : PSD media por quadrante (Welch)
    - dunn_posthoc.png              : heatmap dos p-valores do post-hoc de Dunn
    - matriz_confusao.png           : matriz de confusao 4x4 (Random Forest)
    - importancia_features.png      : importancia das features (Gini)

  Tabelas (imagens .png geradas a partir dos resultados numericos):
    - tabela_describe_features.png        : estatisticas descritivas
    - tabela_kruskal_wallis.png           : resultados do teste de Kruskal-Wallis
    - tabela_dunn_posthoc.png             : p-valores do post-hoc de Dunn
    - tabela_metricas_por_quadrante.png   : precisao/recall/F1 por quadrante
    - tabela_resumo_resultados.png        : resumo final (Tabela 2 do artigo)

  Outros arquivos:
    - resultados_espaco_caracteristicas.txt : trecho LaTeX com a secao do PCA,
                                               pronto para inclusao no artigo
    - features_extraidas.csv                : features extraidas (Secao 5)


--------------------------------------------------------------------------------
 10. METODOLOGIA ESTATISTICA
--------------------------------------------------------------------------------

  - Teste de Kruskal-Wallis: teste nao-parametrico para comparar as 5
    features entre os 4 quadrantes (extensao do teste de Mann-Whitney para
    multiplos grupos).

  - Tamanho de efeito (eta2): eta2 = (H - (k-1)) / (n-k), com k=4 grupos.

  - Post-hoc de Dunn com correcao de Bonferroni: identifica quais pares de
    quadrantes diferem significativamente entre si (aplicado ao centroide
    espectral, feature de maior eta2).


--------------------------------------------------------------------------------
 11. CLASSIFICADOR
--------------------------------------------------------------------------------

  - Algoritmo       : Random Forest (scikit-learn RandomForestClassifier)
  - Hiperparametros  : n_estimators=100, random_state=42
  - Validacao        : K-Fold estratificado, k=5 (shuffle=True, random_state=42)
  - Metricas         : acuracia, F1-macro, precisao-macro, recall-macro
                       (media +- desvio-padrao entre os folds)
  - Baseline aleatorio de referencia (4 classes balanceadas): 25,0%


--------------------------------------------------------------------------------
 12. REPRODUTIBILIDADE
--------------------------------------------------------------------------------

Todas as etapas com componente aleatorio utilizam random_state/seed fixos
(np.random.seed(42), random_state=42), garantindo resultados reprodutiveis
entre execucoes, dado o mesmo dataset de entrada.


--------------------------------------------------------------------------------
 13. LICENCA E CITACAO
--------------------------------------------------------------------------------

Este codigo e fornecido para fins academicos e de pesquisa. Ao utiliza-lo,
cite o artigo original:

GOMES, C. V. A. Caracterização espectral de emoções musicais no espaço arousal-valence: uma análise quadrante por quadrante. Dissertação (Mestrado) — Universidade Federal da Bahia, Salvador, 2026.


O dataset DEAM possui sua propria licenca de uso academico; consulte os
termos em https://cvml.unige.ch/databases/DEAM/ antes de redistribuir
qualquer dado derivado do audio original.


--------------------------------------------------------------------------------
 14. CONTATO / MANUTENCAO
--------------------------------------------------------------------------------

Para reportar problemas ou sugerir melhorias neste notebook, abra uma
"Issue" no repositorio do projeto no GitHub.

================================================================================
