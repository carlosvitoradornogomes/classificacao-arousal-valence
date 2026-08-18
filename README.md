# CLASSIFICADOR AROUSAL-VALENCE: Q1-Q4 (MODELO CIRCUMPLEXO DE RUSSELL)

**Dissertação de Mestrado:**  
*"Caracterização espectral de emoções musicais no espaço arousal-valence: uma análise quadrante por quadrante"*

- **Autor:** Eng. Carlos Vitor Adorno Gomes — PPGEEC/UFBA  
- **Orientador:** Prof. Dr. Eduardo Furtado Simas Filho — PPGEEC/UFBA  
- **Dataset utilizado:** DEAM (*Database for Emotion Analysis using Music*), 1.802 faixas musicais com anotações dinâmicas de arousal e valence.

---

## 1. VISÃO GERAL

Este repositório contém o notebook Jupyter [`Classificador_Arousal_Valence_DEAM.ipynb`](file:///c:/Users/vitto/Desktop/PPGEEC/Classificador_Arousal_Valence_DEAM.ipynb) (sincronizado com [`Classificador_22_07.ipynb`](file:///c:/Users/vitto/Desktop/PPGEEC/Classificador_22_07.ipynb)), que implementa um pipeline completo para a classificação de faixas musicais nos quatro quadrantes emocionais do Modelo Circumplexo de Russell:

- **Q1 (Alto Arousal / Alta Valence):** Animado / Eufórico
- **Q2 (Baixo Arousal / Alta Valence):** Calmo / Sereno
- **Q3 (Baixo Arousal / Baixa Valence):** Triste / Melancólico
- **Q4 (Alto Arousal / Baixa Valence):** Tenso / Raivoso

O particionamento em quadrantes é realizado pelas medianas globais de arousal e valence da base DEAM. O pipeline extrai descritores espectrais/energéticos, realiza validação estatística não-paramétrica, executa seleção supervisionada de atributos, treina múltiplos classificadores de Machine Learning com validação cruzada aninhada (*Nested Cross-Validation*) e compara os resultados contra a literatura científica e uma arquitetura especialista de classificadores binários paralelos.

---

## 2. ESTRUTURA DO PIPELINE (SEÇÕES DO NOTEBOOK)

1. **Importações de Bibliotecas e Configuração do Ambiente**
2. **Configuração dos Caminhos dos Arquivos** (CSVs e pasta de áudios)
3. **Carregamento das Anotações DEAM e Particionamento Q1-Q4**
4. **Pré-processamento e Extração de Features Espectrais** (janela de Hann, PSD de Welch, centroide, largura de banda, roll-off, ZCR, energia RMS)
5. **Extração de Features dos Áudios do DEAM** (com opção de carregamento direto via CSV pré-extraído `features_extraidas.csv`)
6. **Construção do Dataset Final** (merge de features + anotações + rótulos de quadrante)
7. **Análise Estatística:** Teste de Kruskal-Wallis e Tamanho de Efeito (\(\eta^2\)) + Post-hoc de Dunn com correção de Bonferroni
8. **Visualizações Estatísticas:** Histogramas e Boxplots por quadrante
8.5. **Visualização do Espaço de Características via PCA** (Projeção 2D das componentes principais)
9. **Densidade Espectral de Potência (PSD) Média por Quadrante** (Método de Welch)
10. **Análise da Relevância e Seleção de Atributos (*Feature Selection*):** Ranking explicativo e seleção supervisionada via `SelectKBest` por fold
11. **Abordagem Multiclasse Direta (4 Quadrantes Q1-Q4) e Comparação de Classificadores:** Validação Cruzada Aninhada (Nested CV 5x3 Folds) com busca de hiperparâmetros por `GridSearchCV` comparando *Random Forest*, *SVM*, *MLP (Rede Neural)*, *XGBoost / Gradient Boosting*, *LightGBM* e *DummyClassifier (Baseline Majoritário)*
12. **Abordagem de Classificadores Binários Especialistas em Paralelo (Arousal & Valence):** Dois classificadores binários independentes (`Arousal_bin` + `Valence_bin`) cujas predições são combinadas para mapeamento no espaço de 4 quadrantes
13. **Análise Comparativa Global, Benchmark da Literatura e Consolidação:** Consolidação das tabelas comparativas, gráficos de desempenho, matrizes de confusão e integração dos resultados da literatura via `literatura_deam_benchmark.csv`

---

## 3. DESCRITORES (FEATURES) EXTRAÍDOS

- `spectral_centroid`: Centroide Espectral (Hz) — indica o "brilho" do sinal sonoro.
- `spectral_bandwidth`: Largura de Banda Espectral (Hz) — mensura a dispersão de frequências.
- `spectral_rolloff`: Frequência de Roll-off a 85% da energia espectral (Hz).
- `zcr`: Taxa de Cruzamentos por Zero (*Zero Crossing Rate*).
- `rms_energy`: Energia RMS do sinal normalizado.

### Parâmetros de Processamento:
- **Taxa de amostragem ($SR$):** 22.050 Hz
- **Tamanho da janela ($L$):** 2.048 amostras ($\approx 46,4$ ms a 44,1 kHz)
- **Sobreposição (*hop*):** 50% ($L // 2$)
- **Limiar de Roll-off:** 85% da energia espectral
- **Janela de ponderação:** Hann

---

## 4. ESTRUTURA DE PASTAS ESPERADA

```text
projeto/
│-- Classificador_Arousal_Valence_DEAM.ipynb  (Notebook principal refatorado para o GitHub)
│-- Classificador_22_07.ipynb                 (Notebook de trabalho sincronizado)
│-- literatura_deam_benchmark.csv             (Tabela de benchmarks da literatura DEAM)
│-- features_extraidas.csv                    (Gerado pela Seção 5 ou fornecido previamente)
│-- README.md                                 (Documentação técnica do projeto)
│-- Images/                                   (Criada automaticamente ao salvar figuras)
│   ├── circumplex_scatter.png
│   ├── histogramas.png
│   ├── boxplots.png
│   ├── pca_espaco_caracteristicas.png
│   ├── PSD.png
│   ├── dunn_posthoc.png
│   ├── matriz_confusao.png
│   ├── importancia_features.png
│   └── tabela_resumo_resultados.png
```

*Nota: Os arquivos de áudio `.mp3` da base DEAM não são incluídos no repositório devido ao tamanho e licença de uso.*

---

## 5. DEPENDÊNCIAS

Requer Python 3.9+ e as seguintes bibliotecas:

```bash
pip install numpy pandas matplotlib scipy scikit-posthocs seaborn \
            librosa tqdm scikit-learn xgboost lightgbm jupyter
```

*Observação: O `librosa` utiliza `ffmpeg` / `libsndfile` para leitura dos arquivos `.mp3`. Em sistemas Linux/Debian/Ubuntu:*
```bash
sudo apt-get install ffmpeg libsndfile1
```

---

## 6. OBTENÇÃO DO DATASET DEAM

O dataset DEAM (*Database for Emotion Analysis using Music*) pode ser obtido para fins acadêmicos em:  
🔗 [https://cvml.unige.ch/databases/DEAM/](https://cvml.unige.ch/databases/DEAM/)

Arquivos necessários:
- Áudios em formato `.mp3`
- Anotações dinâmicas de arousal e valence:
  - `annotations/annotations averaged per song/dynamic (per second annotations)/arousal.csv`
  - `annotations/annotations averaged per song/dynamic (per second annotations)/valence.csv`

---

## 7. CONFIGURAÇÃO ANTES DE EXECUTAR

Na **Seção 2** do notebook (*Caminhos dos Arquivos*), ajuste as variáveis de ambiente:

```python
PATH_AROUSAL  = "caminho/para/arousal.csv"
PATH_VALENCE  = "caminho/para/valence.csv"
PATH_AUDIO    = "caminho/para/pasta_audios_mp3/"
PATH_FEATURES = "features_extraidas.csv"
```

---

## 8. COMO EXECUTAR

1. Clone o repositório e instale as dependências (Seção 5).
2. Configure os caminhos do dataset (Seção 7).
3. Abra o notebook no Jupyter:
   ```bash
   jupyter notebook Classificador_Arousal_Valence_DEAM.ipynb
   ```
4. Execute as células em ordem sequencial.
   - **Com CSV pré-extraído:** pule a execução da extração de áudios e utilize o bloco alternativo da Seção 5 para carregar `features_extraidas.csv`.
   - **Sem CSV pré-extraído:** execute a célula de extração da Seção 5 para gerar o arquivo `.csv` a partir das faixas `.mp3`.

---

## 9. METODOLOGIA E ESTRATÉGIAS DE MACHINE LEARNING

- **Teste de Kruskal-Wallis & Eta-quadrado ($\eta^2$):** Validação da capacidade de discriminação estatística de cada atributo entre os 4 quadrantes.
- **Seleção de Atributos (*Feature Selection*):** Avaliação de modelos utilizando o conjunto total (5 atributos) e o conjunto selecionado via `SelectKBest(f_classif)` executado estritamente dentro dos folds de treino.
- **Modelos de Aprendizado de Máquina:**
  - *Baseline Majoritário (`DummyClassifier`)*
  - *Random Forest Classifier*
  - *Support Vector Machine (SVC rbf/linear)*
  - *Multi-Layer Perceptron (MLPClassifier)*
  - *XGBoost / Gradient Boosting Classifier*
  - *LightGBM Classifier*
- **Validação Cruzada Aninhada (*Nested CV*):**
  - **Outer Loop:** 5-fold Stratified K-Fold para avaliação não viesada do desempenho fora da amostra.
  - **Inner Loop:** 3-fold Stratified K-Fold com `GridSearchCV` para otimização de hiperparâmetros em cada fold externo.
- **Abordagem Binária Especialista Paralela:** Treinamento independente de um classificador para Arousal (High vs Low) e um para Valence (High vs Low), combinando suas predições matricialmente para a atribuição final do quadrante (Q1-Q4).
- **Benchmark da Literatura:** Integração automatizada da tabela de comparação com resultados prévios na base DEAM via `literatura_deam_benchmark.csv`.

---

## 10. LICENÇA E CITAÇÃO

Este código é disponibilizado para fins acadêmicos e de pesquisa. Ao utilizá-lo, cite o trabalho:

```bibtex
@mastersthesis{gomes2026caracterizacao,
  author       = {Carlos Vitor Adorno Gomes},
  title        = {Caracterização espectral de emoções musicais no espaço arousal-valence: uma análise quadrante por quadrante},
  school       = {Universidade Federal da Bahia (UFBA)},
  year         = {2026},
  type         = {Dissertação (Mestrado em Engenharia Elétrica)},
  address      = {Salvador}
}
```
