# 🚍 RideSmart - Otimização de Rotas Urbanas com Grafos e Metaheurísticas

## 📌 Visão Geral

O **RideSmart** é um projeto desenvolvido para a disciplina de **Estrutura de Dados II (AED II)** com o objetivo de modelar redes urbanas reais utilizando grafos extraídos do OpenStreetMap e aplicar algoritmos de otimização para auxiliar na escolha de rotas e pontos de embarque no transporte público.

O sistema utiliza dados geográficos reais da cidade de Natal/RN para representar a malha viária como um grafo ponderado, permitindo a análise de caminhos mínimos entre diferentes pontos da cidade.

---

## 🎯 Objetivos

* Modelar uma rede urbana como um grafo ponderado.
* Extrair dados geográficos reais utilizando OSMnx.
* Identificar pontos de embarque do transporte público.
* Implementar algoritmos clássicos de caminhos mínimos.
* Comparar desempenho e qualidade das soluções encontradas.
* Investigar o uso de metaheurísticas para apoio à tomada de decisão.

---

## 🗺️ Modelagem do Problema

A cidade é representada por um grafo:

* **Vértices (Nós):** cruzamentos, interseções e pontos da malha viária.
* **Arestas:** ruas conectando os vértices.
* **Pesos:** distância física das vias em metros.

O problema consiste em encontrar a melhor rota considerando:

* Origem (A)
* Destino (B)
* Conjunto de pontos de embarque (P)

Buscando minimizar:

Custo Total = Caminhada(A → P) + Viagem(P → B)

---

## ⚙️ Tecnologias Utilizadas

### Linguagem

* Python 3.x

### Bibliotecas

* OSMnx
* NetworkX
* NumPy
* Pandas
* Matplotlib
* GeoPandas
* Heapq

---

## 🧠 Algoritmos Implementados

### 1. Dijkstra Tradicional

Implementação clássica utilizando busca linear para seleção do próximo vértice.

**Complexidade:**

O(V²)

---

### 2. Dijkstra com Heap

Versão otimizada utilizando fila de prioridade (`heapq`).

**Complexidade:**

O((V + E) log V)

---

### 3. A*

Algoritmo heurístico para busca de caminhos mínimos.

Utiliza distância euclidiana como heurística.

**Vantagem:**

* Explora menos vértices.
* Melhor desempenho em redes urbanas grandes.

---

### 4. Particle Swarm Optimization (PSO)

Metaheurística inspirada no comportamento coletivo de enxames.

Utilizada para seleção de pontos de embarque candidatos e otimização da função de custo global.

---

## 🚌 Extração dos Pontos de Embarque

Os pontos de interesse são obtidos diretamente do OpenStreetMap utilizando:

```python
tags = {"highway": "bus_stop"}
```

Os pontos encontrados são convertidos para os vértices mais próximos da rede viária, permitindo sua utilização pelos algoritmos de caminhos mínimos.

---

## 📊 Avaliação Experimental

Os algoritmos são comparados considerando:

* Tempo de execução
* Distância total percorrida
* Número de nós visitados
* Qualidade da solução
* Impacto de congestionamentos simulados

---

## 📂 Estrutura do Projeto

```text
RideSmart/
│
├── RideSmart.ipynb
├── README.md
├── data/
│
├── algorithms/
│   ├── dijkstra.py
│   ├── dijkstra_heap.py
│   ├── astar.py
│   └── pso.py
│
├── images/
│
└── results/
```

---

## 🚀 Como Executar

### Clonar o repositório

```bash
git clone <url-do-repositorio>
```

### Entrar na pasta

```bash
cd RideSmart
```

### Instalar dependências

```bash
pip install osmnx networkx geopandas matplotlib pandas numpy
```

### Executar notebook

```bash
jupyter notebook RideSmart.ipynb
```

---

## 📈 Resultados Esperados

* Construção automática da rede urbana.
* Identificação de pontos de ônibus.
* Cálculo de rotas ótimas.
* Comparação entre algoritmos clássicos e metaheurísticas.
* Visualização gráfica das rotas geradas.

---

## 🎓 Contexto Acadêmico

Projeto desenvolvido para a disciplina de **Estrutura de Dados II**, explorando conceitos de:

* Grafos
* Caminhos mínimos
* Estruturas de dados avançadas
* Análise de complexidade
* Otimização
* Metaheurísticas

---

## 👨‍💻 Autor

Felix Estudos

Curso de Engenharia da Computação

Universidade Federal do Rio Grande do Norte (UFRN)

---

## 📄 Licença

Este projeto possui finalidade acadêmica e educacional.
Todos os dados geográficos utilizados são provenientes do OpenStreetMap.

