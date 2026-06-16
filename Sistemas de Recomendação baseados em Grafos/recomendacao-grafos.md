# Sistemas de Recomendação baseados em Grafos

**Aluno:** Angel Rafael Souza Da Silva
**Instituição:** UNEMAT — Universidade do Estado de Mato Grosso
**Curso:** Sistemas de Informação — 4º Período
**Disciplina:** Estruturas de Dados — Atividade Avaliativa N3
**Professor:** Me. Cides S. Bezerra
**Data:** 09 de junho de 2026

---

## Resumo

Este trabalho apresenta uma pesquisa teórica sobre **Sistemas de Recomendação baseados em Grafos**,
tema estudado na disciplina de Estruturas de Dados. Discute-se o que são, como funcionam, os
principais frameworks e bibliotecas open-source disponíveis e aplicações do mundo real, com figuras
ilustrativas e referências bibliográficas.

---

## Sumário

- [I.1 — O que são](#i1--o-que-são-os-sistemas-de-recomendação-baseados-em-grafos)
- [I.2 — Como funcionam](#i2--como-funcionam)
- [I.3 — Frameworks e bibliotecas open-source](#i3--frameworks-e-bibliotecas-open-source)
- [I.4 — Aplicações do mundo real](#i4--aplicações-do-mundo-real)
- [Referências bibliográficas](#referências-bibliográficas)

---

# I.1 — O que são os Sistemas de Recomendação baseados em Grafos

## I.1.1 O problema da recomendação

Um **sistema de recomendação** (*recommender system*) é um software que sugere itens relevantes a um
usuário — produtos numa loja, filmes num streaming, músicas, pessoas para seguir, vagas de emprego.
Ele é peça central de plataformas como Amazon, Netflix, YouTube, Spotify e Instagram, e responde por
uma fatia enorme do engajamento e da receita dessas empresas.

As abordagens clássicas são duas:

1. **Filtragem baseada em conteúdo (*content-based*).** Recomenda itens parecidos com os que o usuário
   já gostou, comparando atributos (gênero do filme, categoria do produto). Limitação: tende a sugerir
   "mais do mesmo" e ignora o comportamento de outros usuários.
2. **Filtragem colaborativa (*collaborative filtering*).** Recomenda com base no comportamento de
   usuários **semelhantes** ("quem comprou isto também comprou aquilo"). É a base do famoso método de
   **fatoração de matrizes** (*matrix factorization*). Limitação: a matriz usuário×item é
   **gigantesca e esparsa** (quase tudo é vazio) e captura mal relações indiretas.

## I.1.2 A virada para grafos

Os **Sistemas de Recomendação baseados em Grafos** reformulam o problema: em vez de uma tabela/matriz,
os dados são modelados como um **grafo**, onde **usuários e itens são nós** e as **interações**
(comprou, assistiu, curtiu, avaliou) são **arestas**. Esse é um **grafo bipartido** usuário–item.

> **Definição.** *Um sistema de recomendação baseado em grafos modela usuários, itens (e
> opcionalmente atributos e contexto) como nós de um grafo, e as interações entre eles como arestas;
> recomendar passa a ser um problema de **encontrar nós relevantes percorrendo o grafo** — por
> caminhos, passeios aleatórios ou aprendizado de representações (embeddings) sobre a estrutura.*

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%20R1.png" alt="Grafo bipartido usuário–item" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura R1 — Grafo bipartido usuário–item.</strong></figcaption>
</figure>

A **Figura R1** mostra a estrutura fundamental. À esquerda, os usuários; à direita, os itens; cada
aresta é uma interação. Recomendar um item ao usuário **U** equivale a prever quais arestas **ainda
não existem** mas **deveriam existir** — um problema clássico de **predição de links**
(*link prediction*) em grafos.

## I.1.3 Por que grafos são vantajosos para recomendação

- **Relações de ordem superior (*high-order connectivity*).** Num grafo, "usuário → item → outro
  usuário → outro item" é um caminho natural de 3 saltos. Isso captura padrões do tipo "pessoas como
  você gostaram disto" sem cálculos custosos sobre uma matriz esparsa.
- **Combate à esparsidade e ao *cold start*.** Mesmo um usuário novo, com poucas interações, está
  conectado à estrutura global do grafo e pode receber recomendações por caminhos indiretos.
- **Integração de conhecimento heterogêneo.** Atributos, categorias, marcas, contexto social
  (amizades) podem virar nós/arestas, formando um **grafo de conhecimento** que enriquece a sugestão
  e ainda a torna **explicável** ("recomendado porque seu amigo curtiu").

Em resumo, representar recomendação como grafo transforma "preencher uma matriz" em "navegar e
aprender sobre uma rede de relações" — o que é mais expressivo e escalável.

---

# I.2 — Como funcionam

As técnicas de recomendação baseada em grafos podem ser agrupadas em **três famílias** principais, da
mais simples à mais moderna.

## I.2.1 Passeios aleatórios (*random walks*)

A ideia: a partir do nó do usuário, um "caminhante" anda aleatoriamente pelas arestas do grafo;
itens visitados com **maior frequência/probabilidade** são os mais recomendáveis. As técnicas mais
conhecidas são o **PageRank Personalizado** (*Personalized PageRank*) e o **Random Walk with
Restart** (a cada passo há uma chance de "voltar" ao usuário de origem, mantendo as recomendações
próximas a ele).

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%20R2.png" alt="Passeio aleatório num grafo de recomendação" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura R2 — Passeio aleatório (random walk) num grafo de recomendação.</strong></figcaption>
</figure>

A **Figura R2** ilustra o passeio aleatório com reinício. O caminhante explora a vizinhança do
usuário; quanto mais "passa" por um item, maior a chance de ele ser recomendado. O "reinício" evita
que o passeio se perca em regiões distantes e irrelevantes do grafo.

## I.2.2 Embeddings de grafo (*graph embeddings*)

Aqui, cada nó (usuário/item) é convertido em um **vetor** (embedding) que captura sua posição na
estrutura do grafo. Nós próximos no grafo ficam próximos no espaço vetorial. Algoritmos clássicos:
**DeepWalk** e **node2vec**, que geram sequências por passeios aleatórios e aprendem os vetores de
forma parecida com o *word2vec* do processamento de linguagem. Com os embeddings prontos, recomendar
vira **buscar os itens cujo vetor é mais próximo** do vetor do usuário.

## I.2.3 Redes Neurais de Grafos (*Graph Neural Networks*, GNNs)

É o estado da arte. Uma **GNN** aprende a representação de cada nó **agregando informação dos
vizinhos**, camada após camada (*message passing*): cada usuário "absorve" características dos itens
com que interagiu, e cada item "absorve" características dos usuários que o consumiram, repetidamente.
Isso captura relações de **alta ordem** automaticamente. Modelos famosos:

- **GraphSAGE** — aprende a agregar vizinhanças e generaliza para nós novos (indutivo), bom para
  *cold start*.
- **PinSage** — GNN do Pinterest para grafos web-scale (bilhões de nós).
- **NGCF** e **LightGCN** — GNNs desenhadas especificamente para filtragem colaborativa; o LightGCN
  simplifica a arquitetura e virou *baseline* de referência.
- **KGAT** (*Knowledge Graph Attention Network*) — combina grafo de conhecimento + atenção para
  recomendações explicáveis.

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%20R3.png" alt="Message passing em uma Graph Neural Network" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura R3 — Message passing em uma Graph Neural Network.</strong></figcaption>
</figure>

A **Figura R3** explica o mecanismo central das GNNs. Em cada camada, um nó atualiza seu vetor
combinando os vetores dos vizinhos. Após **k** camadas, o vetor do nó resume informação de vizinhos a
até **k saltos** de distância — é assim que a GNN "enxerga" relações indiretas que a filtragem
colaborativa tradicional não captura.

## I.2.4 O fluxo completo

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%20R4.png" alt="Pipeline de um recomendador baseado em grafo" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura R4 — Pipeline de um recomendador baseado em grafo.</strong></figcaption>
</figure>

A **Figura R4** resume o funcionamento de ponta a ponta. As interações viram o grafo; o modelo
aprende representações ou probabilidades; cada item candidato recebe uma **pontuação**; e os de maior
pontuação formam a lista **Top-N** mostrada ao usuário. O ciclo se retroalimenta: novas interações
realimentam o grafo, melhorando as próximas recomendações.

---

# I.3 — Frameworks e bibliotecas open-source

| Ferramenta | Linguagem/Base | Licença | Destaque |
|---|---|---|---|
| **PyTorch Geometric (PyG)** | Python / PyTorch | MIT | Biblioteca líder de GNNs; implementa GraphSAGE, LightGCN, etc. |
| **Deep Graph Library (DGL)** | Python / PyTorch, TF | Apache 2.0 | GNNs em escala, multi-backend; muito usada em pesquisa e indústria |
| **RecBole** | Python | MIT | Framework dedicado a recomendação; +100 modelos, vários baseados em grafo |
| **Microsoft Recommenders** | Python | MIT | Coletânea de algoritmos e *best practices* de recomendação |
| **Neo4j Graph Data Science (GDS)** | Java/Cypher | open core | Algoritmos de grafo (PageRank, node2vec, link prediction) sobre Neo4j |
| **NetworkX** | Python | BSD | Manipulação de grafos e algoritmos clássicos (ótimo para prototipar) |
| **Spotlight** | Python / PyTorch | MIT | Modelos de recomendação (incl. sequenciais) |
| **Cornac** | Python | Apache 2.0 | Comparação de modelos de recomendação multimodais |
| **OpenNE / GraphVite** | Python | MIT/Apache | Embeddings de grafo (DeepWalk, node2vec, LINE) em escala |

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%20R5.png" alt="Principais bibliotecas de GNN e recomendação" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura R5 — Logos/stack das principais bibliotecas de GNN e recomendação.</strong></figcaption>
</figure>

A **Figura R5** organiza o ecossistema. Para **pesquisa e GNNs**, PyG e DGL dominam; para **montar e
avaliar recomendadores rapidamente**, RecBole e Microsoft Recommenders; e para quem já tem os dados em
um **banco de grafos**, o Neo4j GDS oferece algoritmos prontos (PageRank, node2vec, predição de links)
sem sair do banco.

## Pilha típica
Um projeto real costuma combinar: um **banco de grafos** (Neo4j) ou armazenamento de logs → uma
biblioteca de **GNN** (PyG/DGL) ou framework de recomendação (RecBole) → e um serviço que entrega o
**Top-N** em tempo real. Para grande escala, embeddings são pré-calculados e a busca dos vizinhos mais
próximos é feita com bancos vetoriais (FAISS, Milvus).

---

# I.4 — Aplicações do mundo real

- **Pinterest — PinSage.** Caso público mais famoso de GNN em recomendação: um grafo com **bilhões**
  de pins e boards alimenta as sugestões visuais da plataforma. É a referência de "GNN em web-scale".

- **Alibaba / Taobao.** Usa GNNs (Euler, AliGraph) sobre o gigantesco grafo de usuários×produtos para
  recomendação e busca no e-commerce.

- **Amazon.** O célebre "quem comprou X também comprou Y" é, na essência, navegação em um grafo de
  coocorrência de produtos; abordagens modernas usam GNNs sobre o grafo de compras.

- **Netflix / YouTube / Spotify.** Recomendação de filmes, vídeos e músicas a partir de grafos de
  interação usuário–conteúdo (e, no Spotify, também grafos de playlists e artistas relacionados).

- **Uber Eats.** Publicou o uso de GraphSAGE para recomendar pratos e restaurantes, modelando
  usuários, restaurantes e itens de menu como grafo.

- **LinkedIn / Twitter(X).** Recomendação de **pessoas para seguir/conhecer** ("People You May Know")
  é um problema de **predição de links** no grafo social — talvez o exemplo mais direto de grafo
  aplicado a recomendação.

- **E-commerce em geral / recomendação explicável.** Grafos de conhecimento (produto–marca–categoria)
  permitem justificar a sugestão ("recomendado porque combina com sua compra anterior"), aumentando a
  confiança do usuário.

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%20R6.png" alt="Recomendação em rede social (People You May Know)" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura R6 — Recomendação em rede social (People You May Know).</strong></figcaption>
</figure>

A **Figura R6** conecta a teoria à prática: sugerir uma nova amizade é prever uma aresta que ainda não
existe entre dois nós que compartilham muitos vizinhos. É o mesmo princípio de predição de links usado
para recomendar produtos, vídeos e músicas.

---


---

# Referências bibliográficas

1. PEROZZI, B.; AL-RFOU, R.; SKIENA, S. *DeepWalk: Online Learning of Social Representations.* KDD,
   2014.
2. GROVER, A.; LESKOVEC, J. *node2vec: Scalable Feature Learning for Networks.* KDD, 2016.
3. HAMILTON, W.; YING, R.; LESKOVEC, J. *Inductive Representation Learning on Large Graphs (GraphSAGE).*
   NeurIPS, 2017.
4. YING, R. et al. *Graph Convolutional Neural Networks for Web-Scale Recommender Systems (PinSage).*
   KDD, 2018.
5. WANG, X. et al. *Neural Graph Collaborative Filtering (NGCF).* SIGIR, 2019.
6. HE, X. et al. *LightGCN: Simplifying and Powering Graph Convolution Network for Recommendation.*
   SIGIR, 2020.
7. WANG, X. et al. *KGAT: Knowledge Graph Attention Network for Recommendation.* KDD, 2019.
8. WU, S. et al. *A Survey on Graph Neural Networks for Recommender Systems.* ACM Computing Surveys,
   2022.
9. FEY, M.; LENSSEN, J. E. *Fast Graph Representation Learning with PyTorch Geometric.* ICLR Workshop,
   2019. Disponível em: <https://pytorch-geometric.readthedocs.io/>.
10. DEEP GRAPH LIBRARY (DGL). *Documentação.* Disponível em: <https://www.dgl.ai/>.
11. RECBOLE. *A Unified, Comprehensive and Efficient Recommendation Library.* Disponível em:
    <https://recbole.io/>.
12. NEO4J. *Graph Data Science Library — Documentação.* Disponível em:
    <https://neo4j.com/docs/graph-data-science/>.

