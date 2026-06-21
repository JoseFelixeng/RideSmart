# 🚗 RideSmart — Modelagem e Análise de Rotas Urbanas com Grafos

Projeto final desenvolvido para a disciplina de **Algoritmos e Estrutura de Dados II**, com o objetivo de simular a lógica de escolha de rota de um aplicativo de mobilidade urbana sobre a malha viária real de **Natal/RN**, usando grafos, algoritmos de caminho mínimo e um modelo de trânsito sintético.

---

## 👨‍💻 Integrantes

* Felipe Gabriel Bezerra Da Silva
* José Felix Rodrigues Anselmo
* Laíze Suélia da Silva Oliveira
* Lucas Henrique Alvez De Queiroz

---

## 🎯 Objetivo

Dado um ponto de origem `A`, um destino `B` e uma distância máxima de caminhada `X`, o sistema escolhe o melhor ponto de embarque `P` (acessível a pé a partir de `A`, dentro do limite `X`) tal que a rota completa **A → P (caminhada) → P → B (carro)** seja a mais vantajosa, seguindo diferentes critérios de custo.

O projeto cobre:

* modelagem da rede viária de Natal como grafo (OSMnx + NetworkX);
* implementação de 4 algoritmos de caminho mínimo, do zero;
* um modelo de trânsito sintético calibrado por tipo de via (BPR + capacidade HCM + centralidade + hotspots geográficos);
* comparação de 5 cenários de roteamento exigidos pelo enunciado;
* análise crítica dos resultados obtidos.

---

## 📂 Estrutura do Projeto

```
RideSmart/
│
├── RideSmart.ipynb              # Notebook principal, executável de ponta a ponta
├── README.md
│
└── saidas/                      # Gerado automaticamente na execução
    ├── mapa_transito_sintetico.png   # Mapa de Natal colorido por nível de congestionamento
    ├── distribuicoes_trafego.png     # Histogramas: CF, fluxo OD, centralidade vs hotspot
    ├── rotas_ridesmart.png           # Painel: Dijkstra Heap vs A* | Direto vs Com Embarque
    ├── caminho_direto_AB.png         # Rota direta A→B isolada (baseline sem caminhada)
    └── rotas_zoom_4algoritmos.png    # Zoom em A→P→B, um painel por algoritmo (2×2)
```

---

## ⚙️ Instalação

O notebook foi desenvolvido para rodar em **Google Colab** ou no **Jupyter Notebook** (Python 3.11).

```bash
pip install osmnx networkx matplotlib numpy
```

Todas as demais dependências (`heapq`, `math`, `random`, `time`) fazem parte da biblioteca padrão do Python.

---

## ▶️ Execução

Abra `RideSmart.ipynb` e execute todas as células **em ordem** (`Ambiente de execução → Executar tudo`, no Colab, ou `Kernel → Restart & Run All`, no Jupyter). O notebook baixa o grafo viário de Natal/RN via OSMnx na primeira execução, esse passo pode levar alguns minutos dependendo da conexão.

> ⚠️ Uma observação importante **A célula** do modelo de trânsito sintético (`apply_synthetic_traffic`) precisa rodar **depois** de qualquer célula que recrie o grafo `G` do zero, e **apenas uma vez** por execução — chamá-la novamente em uma célula de visualização recalcula a centralidade do zero (custosa) e sobrescreve o período de tráfego já definido. Caso a célula seja pulada ou o grafo recriado sem reaplicá-la, o atributo `traffic_time` fica ausente nas arestas e o Cenário 3 falha silenciosamente. Rodar tudo em ordem, de cima para baixo, evita esse problema.

---

## 🔄 Metodologia

1. **Aquisição do grafo** — download da malha viária de Natal/RN via `ox.graph_from_place`, nos modos `drive` (carro) e `walk` (pedestre).
2. **Coleta de pontos de embarque** — POIs de táxi e estacionamento via `ox.features.features_from_place`, mapeados para os nós mais próximos do grafo de carro.
3. **Modelo de trânsito sintético** — classificação das vias (incluindo identificação de BRs federais), atribuição de capacidade (HCM), centralidade de intermediação calculada por amostragem (`k=300`, método de Brandes) para viabilizar o tempo de execução em um grafo urbano de larga escala, hotspots geográficos com decaimento gaussiano calibrado em escala real (`σ ≈ 1,3 km`) e ancorados nos principais corredores da cidade, fluxo OD sintético e função BPR calibrada por categoria de via.
4. **Implementação dos algoritmos** — Dijkstra simples (O(V²)), Dijkstra com heap (O((V+E) log V)), A* com heurística Haversine, e Bellman-Ford como algoritmo adicional da literatura.
5. **Seleção do ponto de embarque** — busca do `P` ótimo entre os candidatos alcançáveis a pé, minimizando o custo total da rota composta.
6. **Comparação dos 5 cenários** — menor distância, mais rápido sem trânsito, mais rápido com trânsito sintético, rota direta sem caminhada, e ganho obtido ao caminhar.
7. **Visualização** — mapa de congestionamento da cidade, histogramas de distribuição do modelo de tráfego, um plot dedicado do caminho direto A→B (baseline sem caminhada), e um painel individual por algoritmo mostrando a rota A→P→B sobre o grafo.

---

## 🧩 Modelagem do Grafo

| Elemento | Representação |
|---|---|
| Nós | Interseções e cruzamentos de ruas de Natal/RN |
| Arestas | Segmentos de via entre interseções |
| Peso `length` | Distância em metros |
| Peso `travel_time` | Tempo de percurso sem trânsito (segundos) |
| Peso `traffic_time` | Tempo de percurso com trânsito sintético (segundos), via função BPR |

O grafo de carro (`G`) e o grafo pedestre (`G_walk`) são tratados separadamente; a rota final combina um trecho em cada um deles.

---

## 🚦 Algoritmos Implementados

| Algoritmo | Complexidade | Papel no projeto |
|---|---|---|
| Dijkstra Simples | O(V²) | Linha de base, sem estrutura de dados auxiliar |
| Dijkstra com Heap | O((V+E) log V) | Versão otimizada com fila de prioridade |
| A* (heurística Haversine) | O((V+E) log V) | Busca direcionada ao destino |
| Bellman-Ford | O(V·E) | Algoritmo adicional; robusto a pesos negativos, usado como referência de corretude |

---

## 📊 Resultados

**Grafo de Natal/RN:** 18.581 nós, 48.193 arestas. 691 POIs de embarque coletados (19 táxis + 672 estacionamentos), mapeados para 570 nós únicos.

**Origem (A):** Praia Shopping — **Destino (B):** CMEI Professora Francisca Célia Martins de Souza (zona norte), ≈ 19,2 km em linha de carro.

### Comparação dos algoritmos (rota direta A→B, peso `length`)

| Algoritmo | Distância | Tempo de execução | Nós expandidos |
|---|---:|---:|---:|
| Dijkstra Simples | 19.155,8 m | 41,951 s | — |
| Dijkstra Heap | 19.155,8 m | 0,216 s | — |
| A* | 19.155,8 m | 0,107 s | 7.311 |
| Bellman-Ford* | 19.155,8 m | 11,425 s | — |

*Bellman-Ford executado em subgrafo local (raio ≈ 23 km) por questão de desempenho.*

Os quatro algoritmos convergiram para exatamente a mesma distância, validando a corretude das implementações. O Dijkstra com Heap foi **~194× mais rápido** que o Dijkstra Simples e **~53× mais rápido** que o Bellman-Ford. O A* expandiu apenas 7.311 dos 18.581 nós do grafo — **60,7% a menos** — graças à heurística geográfica admissível.

### Comparação dos 5 cenários

| Cenário | P escolhido | Caminhada | Carro | Total |
|---|---|---:|---:|---:|
| 1 — Menor distância | 503307536 | 219,0 m | 18.825,5 m | 19.044,4 m |
| 2 — Mais rápido sem trânsito | 502216497 | 211,5 s | 1.005,3 s | 1.216,8 s |
| 3 — Mais rápido com trânsito sintético | 3801130604 | 496,2 s | 1.086,9 s | 1.583,1 s |
| 4 — Direto, sem caminhada | — | — | — | 19.155,8 m |
| 5 — Ganho ao caminhar | — | — | — | **+111,4 m** (positivo) |

> O Cenário 4 (rota direta A→B, sem caminhada) é visualizado isoladamente em `caminho_direto_AB.png` — um plot dedicado mostrando apenas a rota de carro entre origem e destino, usado como baseline visual para os demais cenários.

O trânsito sintético elevou o custo total da rota em **+30,1%** em relação ao cenário sem trânsito (1.583,1 s vs. 1.216,8 s), e **deslocou o ponto de embarque ótimo** de um nó para outro completamente diferente (`502216497` → `3801130604`) — evidência direta de que o modelo de tráfego influencia a decisão de roteamento, não apenas o tempo total estimado.

### Modelo de trânsito sintético

CF médio: 1,017 · CF máximo: 2,286 · 87,9% das vias em condição livre (CF<1,2), 12,0% moderada, 0,1% intensa, 0% congestionada.

| Categoria de via | CF médio | CF máximo | Nº de arestas |
|---|---:|---:|---:|
| BR federal | 1,62 | 2,29 | 388 |
| Arterial principal | 1,52 | 1,78 | 46 |
| Arterial | 1,25 | 1,68 | 2.013 |
| Coletora | 1,11 | 1,50 | 9.556 |
| Local | 0,97 | 1,35 | 36.190 |

As BRs concentram o maior congestionamento sintético da cidade, como esperado dado seu papel de corredores estruturais de Natal.

---

## 🔍 Análise Crítica e Limitações

* O modelo de trânsito é sintético, calibrado por proxies (capacidade HCM, centralidade, hotspots geográficos), não por dados reais de tráfego.
* A centralidade de intermediação é calculada por amostragem (`k=300` nós-fonte) em vez do algoritmo exato — necessário para viabilizar o tempo de execução em um grafo com ~18 mil nós, mas introduz uma aproximação estatística nos valores de `betweenness` por aresta.
* O Cenário 3 depende da execução prévia, sobre o mesmo objeto `G`, da célula de trânsito sintético — se o grafo for recriado sem reaplicar o modelo, o atributo `traffic_time` fica ausente e a busca falha silenciosamente, retornando "sem candidatos" mesmo havendo pontos de embarque válidos.
* Os Cenários 2 e 3 filtram candidatos comparando `max_walk_m` (configurado em metros) contra `travel_time`, que está em **segundos** — uma mistura de unidades que torna o limite de caminhada efetivamente mais permissivo (~700 m a ~1,4 m/s) do que os 500 m nominais usados no Cenário 1.
* A busca do ponto de embarque ótimo roda um Dijkstra completo por candidato dentro de um laço; com centenas de candidatos isso já custa dezenas de segundos por cenário — uma busca única a partir do destino, no grafo invertido, reduziria esse custo para frações de segundo.
* Os pontos de embarque dependem da cobertura de POIs do OpenStreetMap, que pode estar incompleta em algumas regiões da cidade.
* O fluxo OD sintético é gerado por amostragem aleatória de pares origem-destino, sem calibração contra dados reais de demanda.
* Os grafos de carro e pedestre são tratados de forma independente, sem modelar tempos de espera, semáforos ou travessias.

---

## 🧠 Tecnologias

* Python 3.11
* OSMnx
* NetworkX
* Matplotlib
* NumPy

---

## 📚 Referências

* Boeing, G. (2017). OSMnx: New Methods for Acquiring, Constructing, Analyzing, and Visualizing Complex Street Networks. *Computers, Environment and Urban Systems*.
* Dijkstra, E. W. (1959). A Note on Two Problems in Connexion with Graphs. *Numerische Mathematik*.
* Hart, P. E., Nilsson, N. J., & Raphael, B. (1968). A Formal Basis for the Heuristic Determination of Minimum Cost Paths. *IEEE Transactions on Systems Science and Cybernetics*.
* Bellman, R. (1958). On a Routing Problem. *Quarterly of Applied Mathematics*.
* Bureau of Public Roads (1964). Traffic Assignment Manual.
* Brandes, U. (2001). A Faster Algorithm for Betweenness Centrality. *Journal of Mathematical Sociology*.
* Transportation Research Board (2016). *Highway Capacity Manual*, 6th Edition.
* DNIT (2006). Manual de Estudos de Tráfego.

---

## 📌 Conclusão

O RideSmart integra **modelagem de grafos + algoritmos de caminho mínimo + simulação de tráfego** para reproduzir, em escala reduzida, a lógica de roteamento de um aplicativo de mobilidade urbana sobre a malha viária real de Natal/RN. Os quatro algoritmos implementados convergiram para os mesmos resultados, validando a corretude da implementação. O A* se destacou em eficiência de busca, e o Dijkstra com Heap na velocidade geral. O modelo de trânsito sintético, por sua vez, não apenas elevou o tempo estimado de viagem em 30%, mas efetivamente mudou qual ponto de embarque é ótimo — evidenciando que o trânsito não é só um ajuste de custo, mas um fator que reformula a decisão de roteamento, como aconteceria em um aplicativo real de mobilidade.
