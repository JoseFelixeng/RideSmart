# RideSmart (EasyPath) — Modelagem e Análise de Rotas Urbanas com Grafos

Projeto final da disciplina de **Teoria dos Grafos / Algoritmos em Grafos** — UFRN, 2026.

O projeto simula um problema inspirado em aplicativos de mobilidade urbana (estilo *ride-sharing*):

> Dado um ponto de origem **A**, um destino **B** e uma distância máxima que o usuário aceita caminhar, escolher o melhor ponto de embarque **P**, de forma que a rota completa **A → P (a pé) → B (carro)** seja a mais vantajosa possível.

O grafo urbano utilizado é a malha viária real de **Natal, Rio Grande do Norte, Brasil**, obtida via [OSMnx](https://osmnx.readthedocs.io/) a partir dos dados do OpenStreetMap.

---

## 👥 Integrantes

- Felipe Gabriel
- José Felix Rodrigues Anselmo
- Laize
- Lucas Henrique

---

## 📋 Sumário

- [Visão geral do problema](#-visão-geral-do-problema)
- [Modelagem do grafo](#-modelagem-do-grafo)
- [Algoritmos implementados](#-algoritmos-implementados)
- [Trânsito sintético](#-trânsito-sintético)
- [Cenários comparados](#-cenários-comparados)
- [Visualizações](#-visualizações)
- [Como executar](#-como-executar)
- [Estrutura do repositório](#-estrutura-do-repositório)
- [Referências](#-referências)

---

## 🧭 Visão geral do problema

A rota completa é composta por dois trechos:

```text
A → P   caminhada
P → B   carro
```

O usuário informa uma distância máxima de caminhada **X** (em metros). O algoritmo:

1. Filtra todos os pontos de embarque candidatos (`pickup_nodes`) que estejam a no máximo **X** metros de caminhada da origem **A**;
2. Para cada candidato **P**, calcula o custo total da rota `A → P` (a pé) + `P → B` (carro);
3. Escolhe o ponto **P** com o menor custo total.

O notebook compara cinco cenários (rota mais curta, rota mais rápida sem/com trânsito, rota sem caminhada, e o ganho obtido ao caminhar até um ponto de embarque alternativo) e quatro algoritmos clássicos de caminho mínimo.

---

## 🗺️ Modelagem do grafo

- **Nós**: interseções e cruzamentos das vias de Natal/RN.
- **Arestas**: segmentos de via (ruas, avenidas), com os seguintes pesos:
  - `length` — distância em metros;
  - `travel_time` — tempo de percurso em segundos, calculado a partir da velocidade da via;
  - `traffic_time` — tempo de percurso considerando o modelo de trânsito sintético.

São utilizados dois grafos:

- **Grafo de direção** (`network_type="drive"`) — para os trechos de carro;
- **Grafo pedestre** (`network_type="walk"`) — para os trechos de caminhada `A → P`.

Os pontos de embarque (`pickup_nodes`) são obtidos a partir de pontos de interesse reais do OpenStreetMap (pontos de táxi, estacionamentos, locadoras de veículos) e, caso insuficientes, complementados por amostragem de nós do grafo de direção.

---

## ⚙️ Algoritmos implementados

Foram implementados, **do zero**, quatro algoritmos de caminho mínimo, todos parametrizados pelo atributo de peso (`weight`), permitindo reutilizá-los com `length`, `travel_time` ou `traffic_time`:

| # | Algoritmo | Complexidade | Observações |
|---|-----------|---------------|-------------|
| 1 | **Dijkstra Simples** | O(V²) | Implementação sem fila de prioridade, usada como linha de base |
| 2 | **Dijkstra com Heap** | O((V+E) log V) | Implementação com `heapq`, usada como referência de eficiência |
| 3 | **A\*** | O((V+E) log V) (na prática, menos nós expandidos) | Heurística geográfica baseada na distância de Haversine até o destino |
| 4 | **Bellman-Ford** (algoritmo adicional) | O(V·E) | Suporta pesos negativos (não utilizados aqui); relaxa todas as arestas V−1 vezes, com detecção de ciclo negativo |

Todos os quatro algoritmos são comparados em termos de **distância encontrada**, **tempo de execução** e (quando aplicável) **número de nós expandidos**, garantindo que todos cheguem ao mesmo custo ótimo — usado como verificação de corretude das implementações.

---

## 🚦 Trânsito sintético

Para simular um congestionamento espacialmente realista e academicamente justificável, o peso `traffic_time` combina **6 componentes**:

| Componente | Referência |
|---|---|
| Capacidade por classe de via (`highway`) | HCM 6th Ed. (TRB, 2016) |
| Fator temporal por período do dia | DNIT — Manual de Estudos de Tráfego (2006) |
| Centralidade de intermediação das arestas (k-sampling) | Brandes (2001), *J. Math. Sociology* |
| Hotspots com decaimento gaussiano espacial | Sheffi (1985), *Urban Transportation Networks* |
| Fluxo OD sintético (equilíbrio de Wardrop) | Wardrop (1952), *Proc. ICE* |
| Função BPR | Bureau of Public Roads (1964) |

A função final aplicada a cada aresta é:

```
traffic_time = BPR(travel_time, fluxo_efetivo, capacidade)
fluxo_efetivo = capacidade · 0.4 · tf · (1 + 2·bet) · (1 + 2·hs) + od_contrib
```

onde `tf` é o fator temporal do período do dia, `bet` é a centralidade de intermediação normalizada e `hs` é o fator de hotspot espacial.

---

## 🧪 Cenários comparados

| # | Cenário | Peso usado |
|---|---------|-----------|
| 1 | Menor distância total | `length` |
| 2 | Rota mais rápida sem trânsito | `travel_time` |
| 3 | Rota mais rápida com trânsito sintético | `traffic_time` |
| 4 | Sem caminhada (carro direto A→B) | `length` / `travel_time` |
| 5 | Ganho obtido ao caminhar até um ponto de embarque alternativo | comparação entre P ótimo e P = origem |

---

## 📊 Visualizações

O notebook gera dois tipos de visualização sobre o mapa de Natal:

1. **Visão geral das rotas** — comparação `Dijkstra Heap vs A*` e `rota direta A→B vs rota com embarque A→P→B`, sobre o grafo completo;
2. **Comparação visual com zoom (2×2)** — um painel por algoritmo (Dijkstra Simples, Dijkstra com Heap, A* e Bellman-Ford), mostrando o trajeto completo `A → P` (caminhada) + `P → B` (carro) sobre um subgrafo recortado em torno do ponto de embarque escolhido. A figura é exportada como `rotas_zoom_4algoritmos.png`.

---

## ▶️ Como executar

O notebook foi desenvolvido e testado no **Google Colab**, mas pode ser executado em qualquer ambiente Jupyter com Python 3.

1. Abra o notebook `RideSmart.ipynb` no Google Colab (ou Jupyter local);
2. Execute as células em ordem, do início ao fim (a primeira célula instala a dependência `osmnx`);
3. Na seção **5. Definição de Origem (A) e Destino (B)**, ajuste as variáveis `ENDERECO_ORIGEM` e `ENDERECO_DESTINO` para os endereços desejados (qualquer endereço reconhecível pelo geocodificador do OpenStreetMap/Nominatim);
4. As demais células executam automaticamente: carregamento do grafo de Natal, geração do trânsito sintético, execução e comparação dos quatro algoritmos, seleção do ponto de embarque ótimo para os cinco cenários, e geração das visualizações.

> ⚠️ **Atenção**: o download do grafo de Natal e o cálculo do Dijkstra Simples (O(V²)) sobre o grafo completo podem levar alguns minutos. O cache do OSMnx (`ox.settings.use_cache = True`) é utilizado para evitar downloads repetidos.

### Dependências principais

- `osmnx`
- `networkx`
- `matplotlib`
- `numpy`

---

## 📁 Estrutura do repositório

```
.
├── RideSmart.ipynb              # Notebook principal do projeto
├── rotas_zoom_4algoritmos.png   # Figura 2x2 gerada pela Seção 13
├── relatorio/                   # Relatório curto em formato IEEE (PDF)
└── README.md                    # Este arquivo
```

---

## 📚 Referências

- Boeing, G. (2017). OSMnx: New methods for acquiring, constructing, analyzing, and visualizing complex street networks. *Computers, Environment and Urban Systems*, 65, 126-139.
- Cormen, T. H. et al. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press.
- Hart, P. E., Nilsson, N. J., & Raphael, B. (1968). A formal basis for the heuristic determination of minimum cost paths. *IEEE Transactions on Systems Science and Cybernetics*, 4(2), 100-107.
- Bellman, R. (1958). On a routing problem. *Quarterly of Applied Mathematics*, 16(1), 87-90.
- Transportation Research Board (2016). *Highway Capacity Manual*, 6th Edition.
- DNIT (2006). *Manual de Estudos de Tráfego*.
- Brandes, U. (2001). A faster algorithm for betweenness centrality. *Journal of Mathematical Sociology*, 25(2), 163-177.
- Sheffi, Y. (1985). *Urban Transportation Networks: Equilibrium Analysis with Mathematical Programming Methods*. Prentice-Hall.
- Wardrop, J. G. (1952). Some theoretical aspects of road traffic research. *Proceedings of the Institution of Civil Engineers*, 1(3), 325-362.
- Bureau of Public Roads (1964). *Traffic Assignment Manual*.
