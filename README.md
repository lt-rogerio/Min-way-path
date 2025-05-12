# Relatório de atividades - CC0328

## 1. Leitura do Arquivo de Grafo

Um dos principais desafios que precederam a implementação dos algoritmos foi a leitura do arquivo de grafo. Não havia garantia de que as arestas estivessem organizadas em uma ordem específica, nem era possível prever quantas conexões cada vértice teria. Diante dessa imprevisibilidade, busquei uma estrutura de dados que:

* **Economizasse memória**, já que grafos de ruas costumam ser esparsos (a maioria dos nós tem poucas arestas).
* **Agrupasse rapidamente** todas as conexões de um nó, mesmo quando as arestas chegassem completamente embaralhadas.

## 2. Estrutura de Dados Escolhida

**Compressed Row Storage (CRS)**

Optei pela CRS porque, ao contrário de uma matriz de adjacência tradicional, ela armazena apenas os valores válidos:

* **Acesso eficiente por linha**
  No Dijkstra (e em outros algoritmos de caminhos mínimos), é fundamental listar rápido todos os vizinhos de um vértice `u`. A CRS mantém as conexões de cada nó armazenadas de forma contígua, permitindo varrer apenas o trecho relevante do vetor.

* **Independência da ordem de leitura**
  As arestas podem vir em qualquer sequência no arquivo de entrada, mas a CRS agrupa todas as conexões de um mesmo vértice em um bloco contínuo na memória. Assim, não é preciso pré-ordenar nada — basta ler e inserir.

### Representação Interna

```csharp
out int[] rowsptr;   // Tamanho V+1: marca o início/fim de cada linha em indices[]/weights[]
out int[] indices;  // Tamanho E: armazena, para cada aresta, o índice do vértice destino
out int[] weights;  // Tamanho E: armazena, para cada aresta, o peso A[i][j]
```

## 3. Complexidade e Comportamento na Prática

* **Tempo de Execução**
  Cada aresta é processada exatamente uma vez durante o "relaxamento" em Dijkstra, o que representa `O(E)`. Além disso, cada operação de inserção ou remoção na fila de prioridade custa `O(log V)`. No pior caso, a complexidade total é `O((E + V) log V)` — eficiente para a maioria dos grafos esparsos.

* **Uso de Memória**
  Graças à CRS, a memória total consumida é `O(V + E)`, visto que armazenamos apenas as arestas existentes e um vetor de ponteiros de início/fim para cada vértice.

* **Cenários Degenerados**
  Se o grafo for denso (E ≈ V²), a complexidade tende a `O(V² log V)`. Nesses casos, outras técnicas, como o algorítmo de Floyd–Warshall (`O(V³)`), podem se tornar competitivas para valores menores de V.

---

## 4. Exemplo visual

Para garantir que tudo está funcionando conforme o esperado, utilizei um grafo de exemplo com cinco nós:

```plaintext
     (1)
  0 ———→ 1 ———→ 4
  |      ↘
  |       (2)
  ↓          2
  2 ———→ 3 ———→ 4
     (3)     (1)
```

Em CRS, podemos detalhar as arestas, pesos e marcadores de ponteiro em formato de tabela:

| Ponteiro | Aresta | Peso |
| :------: | :----: | :--: |
|     0    |  0 → 1 |   1  |
|          |  0 → 2 |   4  |
|     2    |  1 → 2 |   2  |
|          |  1 → 4 |   7  |
|     4    |  2 → 3 |   3  |
|     5    |  3 → 4 |   1  |

* **rowsptr** = { 0, 2, 4, 5, 6, 6 } marca onde cada bloco de arestas começa em `indices` e `weights`.
* **indices**  = { 1, 2, 2, 4, 3, 4 } define o destino de cada aresta.
* **weights**  = { 1, 4, 2, 7, 3, 1 } define o peso correspondente.
