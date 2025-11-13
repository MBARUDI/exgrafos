# exercício 1 grafos
Modelar um serviço de streaming (como Netflix, Spotify, Amazon Prime Video, etc.) usando grafos é uma excelente maneira de representar as relações complexas entre usuários, conteúdo e metadados.
Com prazer\! Aqui está um resumo completo da nossa conversa sobre como construir um algoritmo de recomendação de músicas baseado em grafos, formatado em Markdown.

# exercício 2 grafos

## 📝 Algoritmo de Recomendação de Músicas em Grafos

A conversa detalhou a criação de um sistema de recomendação de músicas utilizando a tecnologia de **Grafos (Neo4j)** e a linguagem de consulta **Cypher**.

-----

### 1\. Modelagem do Grafo (Nós e Arestas)

Definimos as entidades e as interações que compõem o sistema:

#### **Nós (Entidades)**

  * **(:User)**: Usuário (`userId`, `nome`)
  * **(:Song)**: Música (`songId`, `titulo`, `lancamento`)
  * **(:Artist)**: Artista (`artistId`, `nome`)
  * **(:Genre)**: Gênero (`nome`)

#### **Arestas (Relacionamentos) e Propriedades**

As arestas capturam interações e associações, sendo crucial a propriedade `count` para ponderação:

| Tipo de Aresta | Conexão | Propriedades Chave |
| :--- | :--- | :--- |
| **[:LISTENED]** | `(:User) $\to$ (:Song)` | `count` (Número de escutas), `last_listened` (Recência) |
| **[:LIKES]** | `(:User) $\to$ (:Song)` | `timestamp` |
| **[:FOLLOWS]** | `(:User) $\to$ (:Artist)` | `timestamp` |
| **[:PERFORMED\_BY]** | `(:Song) $\to$ (:Artist)` | Nenhuma |
| **[:HAS\_GENRE]** | `(:Song) $\to$ (:Genre)` | Nenhuma |

-----

### 2\. Implementação da Lógica de Recomendação (Cypher)

Focamos em uma consulta de **Filtragem Colaborativa Ponderada** que usa as propriedades das arestas:

  * **Objetivo:** Encontrar usuários semelhantes (`u2`) a um usuário alvo (`u1`) e recomendar músicas que `u2` escutou, mas `u1` não.
  * **Métrica de Similaridade:** A consulta calcula a similaridade com base no produto das contagens de escuta em comum (`l1.count * l2.count`) para dar maior peso às interações intensas.

**Lógica da Consulta:**

```cypher
MATCH (u1:User {...})-[l1:LISTENED]->(s:Song)<-[l2:LISTENED]-(u2:User)
// Cálculo da similaridade ponderada
WITH u1, u2, sum(l1.count * l2.count) AS weighted_similarity
// Encontra as músicas recomendadas e calcula o score final
MATCH (u2)-[rec_listen:LISTENED]->(recommended_song:Song)
WHERE NOT (u1)-[:LISTENED]->(recommended_song)
// Final_score = (weighted_similarity do vizinho) * (rec_listen.count do vizinho)
RETURN recommended_song.titulo, sum(...) AS final_recommendation_score
ORDER BY final_recommendation_score DESC
```

-----

### 3\. Inserção de Dados (População do Grafo)

Discutimos a forma mais eficiente de popular o grafo, garantindo a integridade dos dados:

  * **Criação de Nós e Arestas:** Uso do comando **`MERGE`** para criar entidades e relacionamentos, evitando duplicatas e definindo as propriedades (`SET l.count = 15`).
  * **Importação Massiva de Dados:** O comando **`LOAD CSV WITH HEADERS`** foi sugerido como a ferramenta essencial para importar grandes volumes de interações de um arquivo CSV, convertendo *strings* em números (`toInteger()`) e datas (`date()`).
  * **Otimização:** Foi enfatizada a necessidade de criar **Índices/Restrições de Unicidade** (`CREATE CONSTRAINT ... IS UNIQUE`) nos campos de ID (`userId`, `songId`) para garantir alta performance nas operações de `MATCH` e `MERGE`.

-----
