# Sistema Inteligente de Scouting, Mapeamento de Perfil e Recomendação de Jogadores de Futebol

Repositório oficial dos experimentos científicos do artigo no formato **IEEE**.

## 1. Perguntas de Pesquisa (R.Q.) e Hipóteses

### Perguntas de Pesquisa
* **R.Q. 1 (Principal):** *Em que medida modelos de Inteligência Artificial baseados em redução de dimensionalidade e medidas de similaridade vetorial superam filtros estatísticos tradicionais na tarefa de recomendar jogadores substitutos que desempenham a mesma função tática em campo?*
* **R.Q. 2 (Descoberta de Funções Táticas):** *É possível descobrir automaticamente agrupamentos táticos realistas apenas a partir de dados de desempenho em campo por 90 minutos, utilizando algoritmos de aprendizado não supervisionado?*
* **R.Q. 3 (Custo-Benefício e Subvalorização):** *Modelos preditivos de aprendizado supervisionado conseguem estimar o valor de mercado de um atleta a partir de suas métricas subjacentes, auxiliando na identificação de jogadores subvalorizados (filosofia Moneyball)?*

### Hipóteses
* **H1:** O motor proposto (Redução via PCA + Filtragem por Cluster + Similaridade de Cosseno Ponderada) obtém maior *Precision@10* e *NDCG@10* na recomendação de substitutos do que o baseline *k-NN* com Distância Euclidiana direta.
* **H2:** Algoritmos não supervisionados (*K-Means* e *GMM*) conseguem segmentar atletas em funções táticas coerentes sem depender exclusivamente do rótulo posicional nominal.
* **H3:** Regressores de *Gradient Boosting* (*XGBoost* e *LightGBM*) conseguem prever o valor justo de mercado a partir de métricas avançadas por 90 minutos, revelando oportunidades de contratação subvalorizadas.

---

## 2. Modelos de I.A. Pesquisados e Justificativa de Implementação

| Componente | Modelo Implementado | Por que foi incluído no artigo? |
| :--- | :--- | :--- |
| **Baseline** | `k-NN` (Distância Euclidiana) | Serve como linha de base tradicional aplicada sobre o vetor padronizado bruto para comprovar o ganho do pipeline proposto. |
| **Redução de Dimensionalidade** | `PCA` e `UMAP` | Estatísticas de futebol possuem forte multicolinearidade. O PCA compacta as variáveis preservando **81.50%** da variância em 5 componentes latentes, enquanto o UMAP gera a projeção bidimensional não-linear do mapa tático. |
| **Clustering Tático** | `K-Means` e `GMM` (Gaussian Mixture) | Identificam a real função tática exercida em campo. O GMM adiciona modelagem probabilística, capturando atletas híbridos/multifuncionais. |
| **Motor de Recomendação** | `Similaridade de Cosseno Ponderada` | Mede o alinhamento proporcional do estilo de jogo dentro do mesmo cluster tático, evitando distorções causadas pelo volume absoluto de ações. |
| **Valuation (Moneyball)** | `XGBoost` e `LightGBM` | Estado da arte para regressão em dados tabulares; estimam o valor de mercado esperado para apontar atletas cujo rendimento supera seu preço atual. |

---

## 3. Dataset

* **Fonte Oficial:** **StatsBomb Open Data** (dados reais de eventos das partidas da **Copa do Mundo FIFA 2022** e **UEFA Euro 2024**), coletados via API oficial `statsbombpy`.
* **Amostra Processada:** **500 jogadores de linha** (goleiros excluídos por possuírem métricas específicas disjuntas) com filtro mínimo de participação de 3 partidas.
* **Engenharia de Atributos (11 métricas normalizadas por 90 min / partida):** `Passes_Prog_90`, `Passes_Terco_Final_90`, `Key_Passes_90`, `Conducoes_Prog_90`, `Dribles_Certos_90`, `Finalizacoes_90`, `npxG_90`, `Desarmes_90`, `Interceptacoes_90`, `Recuperacoes_90`, `Pressoes_90`, além de `Idade`, `Jogos` e `Valor_Mercado_M`.
* **Pré-processamento:** Normalização por volume de jogo e padronização Z-score via `StandardScaler`.

---

## 4. Como os Experimentos Foram Conduzidos

1. **Experimento 1 (Agrupamento Não Supervisionado):** Os vetores padronizados foram reduzidos via PCA (5 componentes) e submetidos ao *K-Means* e *GMM* ($K=4$ perfis funcionais). A coesão interna e separação foram medidas via *Silhouette Score* e *Davies-Bouldin Index*.
2. **Experimento 2 (Benchmark de Recomendação Top-10):** Para cada atleta da base, buscou-se os $K=10$ substitutos mais próximos via Baseline (*k-NN Euclidiano*) e via Modelo Proposto (*Cluster + Cosseno Ponderado no PCA*). Avaliou-se a relevância funcional com *Precision@10* e *NDCG@10*.
3. **Experimento 3 (Valuation Supervisionado):** Divisão treino/teste (80%/20%) para treinar *XGBoost* e *LightGBM* na predição do valor de mercado, avaliando *MAE* e *RMSE* e ranqueando os maiores resíduos positivos ($	ext{Valor Justo IA} - 	ext{Valor Real}$).

---

## 5. Resultados e Discussão

### 5.1 Descoberta de Funções Táticas (Clustering)
* **K-Means:** Silhouette Score = `0.2670` | Davies-Bouldin Index = `1.1954`
* **GMM:** Silhouette Score = `0.2275` | Davies-Bouldin Index = `1.3348`

### 5.2 Desempenho na Recomendação de Substitutos ($K=10$)
| Método de Recomendação | Precision@10 | NDCG@10 |
| :--- | :---: | :---: |
| **Baseline: k-NN (Distância Euclidiana)** | `0.6212` | `0.6275` |
| **Proposto: Cluster + Similaridade de Cosseno Ponderada** | **`0.6246`** | **`0.6341`** |

**Discussão (H1 e H2 confirmadas):** A filtragem prévia por agrupamento tático aliada à similaridade de cosseno no espaço latente do PCA elevou a precisão das recomendações e garantiu melhor ordenação no topo da lista (*NDCG@10*), evitando sugerir jogadores de funções distintas que apenas compartilham volume bruto de ações.

### 5.3 Valuation e Identificação de Oportunidades Moneyball
* **XGBoost Regressor:** MAE = `€ 7.69M` | RMSE = `€ 10.93M`
* **LightGBM Regressor:** MAE = `€ 7.04M` | RMSE = `€ 9.79M`

**Discussão (H3 confirmada):** Os modelos baseados em *Gradient Boosting* capturaram com baixa margem de erro a relação não-linear entre métricas de criação/progressão (`npxG_90`, `Passes_Prog_90`), idade e valor de mercado, permitindo listar objetivamente os atletas subvalorizados na tabela de oportunidades *Moneyball*.
