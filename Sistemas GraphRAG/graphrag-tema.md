# Sistemas de RAG baseados em Grafos (GraphRAG)

**Aluno:** Angel Rafael Souza Da Silva
**Instituição:** UNEMAT — Universidade do Estado de Mato Grosso
**Curso:** Sistemas de Informação — 4º Período
**Disciplina:** Estruturas de Dados — Atividade Avaliativa N3
**Professor:** Me. Cides S. Bezerra
**Data:** 09 de junho de 2026

---

## Resumo

Este trabalho apresenta uma pesquisa teórica sobre **Sistemas de Geração Aumentada por Recuperação
baseados em Grafos (GraphRAG)**, tema estudado na disciplina de Estruturas de Dados. Discute-se o que
são, como funcionam, os principais frameworks e bibliotecas open-source disponíveis e aplicações do
mundo real, com figuras ilustrativas e referências bibliográficas.


---

## Sumário

- [Assunto 1 — Os Sistemas GraphRAG: o que são](#assunto-1--os-sistemas-graphrag-o-que-são)
- [Assunto 2 — Como funcionam](#assunto-2--como-funcionam)
- [Assunto 3 — Frameworks e bibliotecas open-source](#assunto-3--frameworks-e-bibliotecas-open-source)
- [Assunto 4 — Aplicações do mundo real](#assunto-4--aplicações-do-mundo-real)
- [Apêndice — RAG vetorial x GraphRAG](#apêndice--rag-vetorial-x-graphrag)
- [Referências bibliográficas](#referências-bibliográficas)

---


# Assunto 1 — Os Sistemas GraphRAG: o que são

## 1.1 Contexto: do LLM "puro" ao RAG

Os Grandes Modelos de Linguagem (*Large Language Models*, LLMs), como GPT, Gemini e Claude, são
treinados sobre enormes volumes de texto e aprendem a prever a próxima palavra de uma sequência.
Apesar de poderosos, eles têm três limitações estruturais bem conhecidas:

1. **Conhecimento congelado no tempo (*knowledge cutoff*).** O modelo só "sabe" o que existia até a
   data de corte do treinamento. Ele desconhece fatos novos e dados privados de uma empresa.
2. **Alucinações.** Quando não sabe a resposta, o modelo tende a *inventar* informações com
   aparência convincente, pois sua função é gerar texto plausível — não verificar a verdade.
3. **Falta de fonte/rastreabilidade.** O LLM puro não consegue apontar "de onde tirou" a resposta.

A técnica de **RAG (Retrieval-Augmented Generation / Geração Aumentada por Recuperação)**, proposta
por Lewis et al. (2020), resolve boa parte desses problemas. A ideia central é simples e elegante:
**antes de responder, o sistema recupera (busca) informações relevantes de uma base externa e as
fornece ao LLM como contexto.** O modelo passa a responder "com o livro aberto" em vez de "de cabeça".

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%201.png" alt="Diagrama clássico do pipeline RAG" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura 1 — Diagrama clássico do pipeline RAG.</strong></figcaption>
</figure>

A **Figura 1** ilustra o ciclo fundamental do RAG. A pergunta não vai direto ao LLM; ela primeiro aciona um **recuperador** (*retriever*) que consulta uma base de conhecimento e seleciona os trechos mais relevantes. Só então o LLM recebe a pergunta **acompanhada** desses trechos e gera a resposta. É essa etapa intermediária de recuperação que dá nome à técnica.

## 1.2 O RAG tradicional é "vetorial"

Na implementação mais comum do RAG — o **RAG vetorial** — a base de conhecimento é construída assim:

- Os documentos são quebrados em pedaços menores, chamados ***chunks*** (parágrafos ou trechos).
- Cada *chunk* é convertido em um **embedding**: um vetor numérico de centenas/milhares de dimensões
  que representa o "significado" do texto. Textos com sentido parecido ficam próximos no espaço vetorial.
- Esses vetores são guardados em um **banco de dados vetorial** (Qdrant, Weaviate, Milvus, pgvector…).

Na hora da pergunta, ela também vira um vetor, e o sistema busca os *chunks* cujos vetores são mais
**similares** (geralmente por *similaridade do cosseno*). Esses trechos são injetados no *prompt*.

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%202.png" alt="Espaço de embeddings / similaridade vetorial" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura 2 — Espaço de embeddings / similaridade vetorial.</strong></figcaption>
</figure>

A **Figura 2** mostra por que o RAG vetorial funciona — e onde ele falha. Cada ponto é um trecho de texto posicionado pelo seu significado. A busca retorna os pontos **mais próximos** da pergunta. Isso é excelente quando a resposta está concentrada em um trecho, mas repare: a proximidade é por **semelhança de texto**, não por **relação lógica entre fatos**. Dois documentos podem ser cruciais juntos e, ainda assim, estarem distantes no espaço vetorial.

## 1.3 As limitações que motivaram o GraphRAG

O RAG vetorial brilha em perguntas **locais** (a resposta cabe em um ou dois *chunks*), mas mostra
fraquezas claras:

- **Raciocínio multi-salto (*multi-hop*).** Perguntas cuja resposta exige **encadear vários fatos**
  espalhados em documentos diferentes. Ex.: "Qual o telefone do supervisor do funcionário que
  fechou o contrato nº 320?". A similaridade isolada raramente recupera toda a cadeia.
- **Perguntas globais / de síntese.** "Quais são os principais temas e riscos discutidos neste acervo
  de 1.000 relatórios?" Não existe um *chunk* único que contenha a resposta — ela exige uma visão do
  todo, e poucos trechos não cabem na janela de contexto.
- **Perda das relações.** Ao fatiar tudo em *chunks* isolados, o RAG vetorial **descarta a estrutura**
  que liga as informações (quem trabalha com quem, o que causa o quê, o que pertence a quem).

## 1.4 A proposta do GraphRAG

O **GraphRAG** (RAG baseado em Grafos) surge como evolução para atacar exatamente esses pontos. Em vez
de tratar o conhecimento como uma "pilha" de trechos soltos, ele o organiza em um
**grafo de conhecimento** (*Knowledge Graph*): uma estrutura formada por

- **Nós (vértices):** as **entidades** — pessoas, empresas, imóveis, doenças, produtos, conceitos.
- **Arestas (relações):** as **ligações** entre entidades — "é dono de", "trabalha em", "causa",
  "está localizado em", "pertence a".

> **Definição.** *GraphRAG é a técnica de RAG em que a base de recuperação é um **grafo de
> conhecimento** estruturado. Em vez de recuperar apenas trechos semelhantes, o sistema recupera
> **entidades e as relações entre elas**, possibilitando raciocínio sobre conexões e visão global.*

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%203.png" alt="Exemplo de grafo de conhecimento" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura 3 — Exemplo de grafo de conhecimento.</strong></figcaption>
</figure>

A **Figura 3** representa o coração do GraphRAG. Diferente da nuvem de pontos da Figura 2, aqui o que importa não é a proximidade, e sim as **setas** — as relações explícitas. Para responder "quais imóveis o João administra?", basta **percorrer as arestas** que saem do nó "João", sem depender de coincidência de vocabulário.

## 1.5 Origem e relevância

O termo ganhou força com a pesquisa **GraphRAG da Microsoft Research (Edge et al., 2024)**, no artigo
*"From Local to Global: A Graph RAG Approach to Query-Focused Summarization"*. Os autores mostraram
que, para perguntas **globais** sobre grandes acervos, o GraphRAG supera o RAG vetorial tanto em
**abrangência** (cobrir mais aspectos da resposta) quanto em **diversidade**, justamente porque
constrói **resumos hierárquicos** a partir da estrutura do grafo. A ideia, porém, é mais antiga: une
décadas de pesquisa em **grafos de conhecimento** (popularizados pelo *Google Knowledge Graph*, 2012)
com a onda recente de LLMs.

Em resumo, o GraphRAG **não substitui** o RAG vetorial em tudo — ele o **complementa**, adicionando
a dimensão das **relações** ao processo de recuperação. É essa dimensão que destrava raciocínio
multi-salto, síntese global e respostas explicáveis (com o "caminho" percorrido no grafo como prova).

---

# Assunto 2 — Como funcionam

O funcionamento de um sistema GraphRAG se divide em **duas grandes fases**: a **indexação**
(construção do grafo, feita uma vez, *offline*) e a **consulta** (responder perguntas, em tempo real).

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%204.png" alt="Visão geral das duas fases do GraphRAG" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura 4 — Visão geral das duas fases do GraphRAG.</strong></figcaption>
</figure>

A **Figura 4** separa o que é caro e feito uma vez (à esquerda, a indexação) do que roda a cada pergunta (à direita, a consulta). Entender essa divisão é essencial: o custo alto de usar um LLM para extrair entidades acontece **na indexação**, e não a cada pergunta do usuário.

## 2.1 Fase de Indexação (construção do grafo)

A indexação transforma textos brutos em um grafo de conhecimento consultável. As etapas típicas são:

### Etapa 1 — Chunking
Os documentos são divididos em trechos de tamanho controlado (*chunks*), com alguma sobreposição para
não cortar ideias ao meio. Cada *chunk* é a unidade que o LLM vai analisar.

### Etapa 2 — Extração de entidades e relações
Este é **o passo que define o GraphRAG**. Um LLM lê cada *chunk* e extrai, de forma estruturada:
- **Entidades** (com tipo e descrição): ex. `Pessoa: João`, `Imóvel: Apto 101`.
- **Relações** entre elas: ex. `João --(é dono de)--> Apto 101`.

A saída costuma ser uma lista de **triplas** no formato **(sujeito, predicado, objeto)** —
por exemplo, `(João, é_dono_de, Apto 101)`.

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%205.png" alt="Extração de triplas a partir de texto" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura 5 — Extração de triplas a partir de texto.</strong></figcaption>
</figure>

A **Figura 5** demonstra como uma frase comum ("João é o proprietário do apartamento 101, alugado para Maria") vira dados estruturados: `(João, é_dono_de, Apto 101)` e `(Maria, aluga, Apto 101)`. É essa conversão de **texto livre** em **fatos estruturados** que permite tratar conhecimento como grafo.

### Etapa 3 — Construção e resolução de entidades
As triplas extraídas de todos os *chunks* são unidas em um único grafo. Um passo importante é a
**resolução de entidades** (*entity resolution*): reconhecer que "João", "João Silva" e "Sr. Silva"
são a **mesma** pessoa, fundindo esses nós para evitar duplicação.

### Etapa 4 — Detecção de comunidades
Com o grafo montado, aplicam-se **algoritmos de detecção de comunidades** — o mais citado é o
**algoritmo de Leiden** (Traag et al., 2019), evolução do **Louvain**. Eles identificam **regiões
densamente conectadas** do grafo e as agrupam em **comunidades** (clusters temáticos). Isso cria uma
**hierarquia**: comunidades pequenas dentro de comunidades maiores.

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%206.png" alt="Detecção de comunidades em um grafo" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura 6 — Detecção de comunidades em um grafo.</strong></figcaption>
</figure>

Na **Figura 6**, cada cor é uma comunidade — um conjunto de entidades que se relacionam muito entre si (ex.: tudo ligado a "contratos de aluguel" de um lado, "vendas" de outro). Essas comunidades são a base para responder **perguntas globais**, pois resumem grandes regiões do conhecimento.

### Etapa 5 — Resumos hierárquicos de comunidade
Para cada comunidade, um LLM gera um **resumo em linguagem natural** ("esta comunidade trata de
contratos de aluguel vencendo no 2º semestre, envolvendo os corretores A e B..."). Esses resumos são
o segredo para responder perguntas amplas **sem reler todos os documentos**.

### Etapa 6 — Embeddings (busca semântica complementar)
Nós, relações e resumos também podem ser **vetorizados** (embeddings) e guardados em um banco
vetorial. Assim, o sistema combina **busca estrutural** (no grafo) com **busca semântica** (vetorial)
— a abordagem **híbrida**.

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%207.png" alt="Pipeline completo de indexação do GraphRAG" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura 7 — Pipeline completo de indexação do GraphRAG.</strong></figcaption>
</figure>

A **Figura 7** reúne as seis etapas anteriores em um único fluxo. Note que tudo isso roda **uma vez** (ou quando a base muda), gerando o artefato final: o grafo enriquecido com comunidades, resumos e embeddings, pronto para ser consultado.

## 2.2 Fase de Consulta (responder a pergunta)

Na consulta, o GraphRAG tipicamente oferece **dois modos de busca**, escolhidos conforme o tipo de
pergunta:

### Busca Local (*Local Search*)
Para perguntas **específicas sobre entidades**. O sistema:
1. Identifica as entidades citadas na pergunta.
2. Localiza os nós correspondentes no grafo.
3. **Navega pela vizinhança** — relações diretas e *multi-hop* (vários saltos).
4. Reúne entidades, relações e *chunks* associados como contexto para o LLM.

### Busca Global (*Global Search*)
Para perguntas **amplas / de síntese**. O sistema usa os **resumos de comunidade** num padrão
**map-reduce**:
- **Map:** cada resumo de comunidade gera uma **resposta parcial** à pergunta.
- **Reduce:** as respostas parciais são **combinadas** em uma resposta final coerente.

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%208.png" alt="Busca Local x Busca Global" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura 8 — Busca Local x Busca Global.</strong></figcaption>
</figure>

A **Figura 8** contrasta os dois modos. A busca **local** é como caminhar pelas ruas vizinhas de um endereço (a entidade); a busca **global** é como olhar o mapa inteiro da cidade resumido por bairros (as comunidades). Perguntas factuais usam a local; perguntas de "qual o panorama geral?" usam a global.

### O diferencial: raciocínio *multi-hop*
A maior vantagem prática aparece em perguntas que exigem **vários saltos**:

> *"Quais clientes do corretor que vendeu o Imóvel X também têm contratos vencendo este mês?"*

No grafo, basta percorrer as arestas:
`Imóvel X → (vendido por) → Corretor → (atende) → Clientes → (possuem) → Contratos → (vencimento) → mês`.

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%209.png" alt="Caminho de raciocínio multi-hop no grafo" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura 9 — Caminho de raciocínio multi-hop no grafo.</strong></figcaption>
</figure>

A **Figura 9** mostra, em destaque, o "caminho" percorrido para responder à pergunta acima. Esse caminho é também a **explicação** da resposta — uma vantagem de transparência que o RAG vetorial não oferece, já que ele entrega apenas trechos soltos.

---

# Assunto 3 — Frameworks e bibliotecas open-source

Existe hoje um ecossistema maduro de ferramentas open-source para construir sistemas GraphRAG. Elas se
dividem, grosso modo, em: **orquestradores/frameworks** (constroem e consultam o grafo), **bancos de
grafos** (armazenam) e **bancos vetoriais** (busca semântica complementar).

## 3.1 Orquestradores e frameworks de GraphRAG

### Microsoft GraphRAG
A **implementação de referência**, publicada junto com o artigo da Microsoft Research. Licença **MIT**.
Implementa todo o pipeline descrito no Assunto 2: extração de entidades/relações via LLM, construção
do grafo, detecção de comunidades (Leiden), resumos hierárquicos e buscas local/global. É escrita em
Python e voltada principalmente a perguntas globais sobre acervos privados.
Repositório: `github.com/microsoft/graphrag`.

### LangChain / LangGraph
Framework popular para aplicações com LLMs (licença MIT). Oferece o **`LLMGraphTransformer`**, que
converte texto em grafo automaticamente, e integrações nativas com bancos de grafos como o **Neo4j**.
O **LangGraph** permite orquestrar fluxos complexos (agentes que decidem quando consultar o grafo).

### LlamaIndex
Outro framework de destaque (MIT). Traz o **`KnowledgeGraphIndex`** e, mais recentemente, o
**`PropertyGraphIndex`**, que facilitam construir e consultar *property graphs* (grafos com
propriedades nos nós e arestas) com poucas linhas de código, integrando grafo + busca vetorial.

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%2010.png" alt="Logos / posicionamento dos principais frameworks" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura 10 — Logos / posicionamento dos principais frameworks.</strong></figcaption>
</figure>

A **Figura 10** ajuda a situar as ferramentas. Microsoft GraphRAG, LangChain e LlamaIndex atuam na **camada de orquestração** (constroem e consultam o grafo), enquanto Neo4j, Nebula e outros atuam na **camada de armazenamento** (guardam o grafo). Na prática, combinam-se: um framework de orquestração + um banco de grafos + um banco vetorial.

## 3.2 Bancos de grafos

- **Neo4j** — o banco de grafos mais popular do mercado. Usa a linguagem de consulta **Cypher** e
  oferece o pacote oficial **`neo4j-graphrag`** (Python) para montar pipelines GraphRAG.
  Há edição community open-source (GPL) e o core sob Apache 2.0.
- **NebulaGraph** — banco de grafos **distribuído** (Apache 2.0), projetado para **grande escala**
  (bilhões de nós/arestas).
- **Memgraph** — banco de grafos **em memória** (open core), foco em **tempo real** e *streaming*.
- **Apache AGE** — extensão que adiciona suporte a grafos (com a linguagem **openCypher**) **dentro do
  PostgreSQL** (Apache 2.0). Útil para quem já usa Postgres e não quer um banco separado.

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%2011.png" alt="Modelo de grafo de propriedades (property graph)" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura 11 — Modelo de grafo de propriedades (property graph).</strong></figcaption>
</figure>

A **Figura 11** mostra o modelo de dados usado pela maioria dos bancos de grafos: tanto nós quanto arestas carregam **propriedades** (atributos). Isso é mais rico que um grafo simples e permite consultas como "quais alugueis com valor acima de R$ 2.000 começaram este ano".

## 3.3 Bibliotecas e plataformas complementares

| Ferramenta | Licença | Destaque |
|---|---|---|
| **txtai** | Apache 2.0 | Embeddings + grafo semântico em uma lib leve, fácil de prototipar |
| **Cognee** | Apache 2.0 | Pipeline de "memória" com grafos de conhecimento para agentes |
| **Graphiti** (Zep) | Apache 2.0 | Grafos de conhecimento **temporais** (com noção de tempo/versão) |
| **R2R** (SciPhi) | MIT | RAG "pronto para produção" com suporte a grafo de conhecimento |
| **Haystack** (deepset) | Apache 2.0 | Framework de RAG modular, com componentes de grafo |

## 3.4 Bancos vetoriais (busca semântica complementar)

Como muitos sistemas GraphRAG são **híbridos**, costumam usar também um banco vetorial:
**Qdrant**, **Weaviate**, **Milvus**, **pgvector** (extensão do PostgreSQL) e **FAISS** (biblioteca da
Meta para busca de vetores).

## 3.5 Pilha (stack) típica de um projeto GraphRAG

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%2012.png" alt="Arquitetura de referência de uma stack GraphRAG" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura 12 — Arquitetura de referência de uma stack GraphRAG.</strong></figcaption>
</figure>

A **Figura 12** monta a "receita" de um sistema GraphRAG real. Da base ao topo: as **fontes** (documentos), o **banco de grafos** + **banco vetorial** (armazenamento), o **framework** (Microsoft GraphRAG / LangChain / LlamaIndex) que orquestra a indexação e a consulta, e o **LLM** (OpenAI, Gemini, Claude ou modelos locais via Ollama) que extrai entidades e gera as respostas. A escolha de cada peça depende de escala, custo e se a empresa já possui parte da infra.

---

# Assunto 4 — Aplicações do mundo real

O GraphRAG é especialmente valioso em domínios onde **as relações entre os dados são tão importantes
quanto os dados em si**, e onde perguntas exigem cruzar muitas fontes. A seguir, os principais casos.

## 4.1 Saúde e biomedicina

A medicina é um dos campos mais beneficiados. Sintomas, doenças, genes, proteínas, medicamentos e
artigos científicos formam uma teia gigantesca de relações. Grafos de conhecimento biomédicos (como o
**PrimeKG**, de Harvard) conectam dezenas de milhares de entidades. Aplicações:
- **Descoberta de medicamentos** (*drug discovery* e *drug repurposing*): "qual fármaco já aprovado
  atua sobre o gene associado a esta doença rara?" — uma pergunta *multi-hop* típica.
- **Apoio ao diagnóstico:** relacionar sintomas a possíveis condições e exames.

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%2013.png" alt="Grafo de conhecimento biomédico" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura 13 — Grafo de conhecimento biomédico.</strong></figcaption>
</figure>

A **Figura 13** ilustra por que a saúde adota GraphRAG: a resposta clinicamente útil quase nunca está em um único artigo — ela emerge de **conexões** entre doença, gene e fármaco espalhadas por milhares de publicações, exatamente o que o grafo torna percorrível.

## 4.2 Serviços financeiros e antifraude

Bancos e fintechs modelam contas, transações, dispositivos e pessoas como grafo para detectar
**anéis de fraude** e **lavagem de dinheiro**. Padrões suspeitos — várias contas "diferentes" que na
verdade compartilham um mesmo dispositivo ou endereço — aparecem como **estruturas no grafo** que a
busca vetorial jamais perceberia. O GraphRAG permite a um analista perguntar em linguagem natural:
"mostre as conexões entre esta conta sinalizada e contas já marcadas como fraudulentas".

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%2014.png" alt="Detecção de fraude por análise de grafo" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura 14 — Detecção de fraude por análise de grafo.</strong></figcaption>
</figure>

Na **Figura 14**, o padrão de fraude é **visualmente óbvio no grafo** (um anel de contas interligadas), mas seria invisível numa lista de transações isoladas. É o argumento central a favor de representar dados financeiros como grafo.

## 4.3 Jurídico e compliance

Escritórios e departamentos jurídicos lidam com acervos enormes de contratos, leis e jurisprudência.
O GraphRAG conecta **cláusulas, partes, obrigações e legislação aplicável**, permitindo:
- responder perguntas de **síntese** ("quais os principais riscos contratuais neste portfólio?");
- rastrear **dependências** ("quais contratos são afetados se esta lei mudar?").

## 4.4 Suporte ao cliente e bases de conhecimento internas

Empresas de tecnologia usam grafos sobre tíquetes de suporte, documentação e histórico de incidentes.
A **LinkedIn**, por exemplo, relatou (Xu et al., 2024) o uso de um grafo de conhecimento sobre tíquetes
de suporte, **recuperando casos relacionados** (e não apenas textualmente similares), o que **reduziu o
tempo médio de resolução** de chamados. É um caso público e bem documentado de GraphRAG aplicado.

## 4.5 Pesquisa empresarial (*enterprise search*)

É o caso de uso que a própria Microsoft destaca para o seu GraphRAG: dar a uma empresa a capacidade de
fazer **perguntas globais sobre seu acervo privado** — relatórios, e-mails, wikis — do tipo "quais os
temas e tendências recorrentes nos últimos 12 meses?". A busca **global** com resumos de comunidade é
o que torna isso viável sem estourar a janela de contexto do LLM.

## 4.6 Recomendação e e-commerce

Grafos de usuários, produtos e interações geram **recomendações explicáveis**: "recomendamos o produto
Y **porque** você comprou X, que costuma ser usado em conjunto". A explicação vem do **caminho no
grafo**, aumentando a confiança do usuário.

## 4.7 Mercado imobiliário

O domínio imobiliário é naturalmente um grafo: **imóveis, proprietários, locatários, corretores,
contratos, propostas e empresas** são entidades fortemente relacionadas. Um GraphRAG poderia responder
perguntas relacionais hoje difíceis, como:
- *"Quais proprietários têm imóveis parados há mais de 90 dias atendidos pelo corretor Y?"*
- *"Quais contratos de aluguel administrados pela filial Z vencem no próximo trimestre e cujos
  locatários já atrasaram pagamento?"*

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%2015.png" alt="Grafo de conhecimento imobiliário" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura 15 — Grafo de conhecimento imobiliário.</strong></figcaption>
</figure>

A **Figura 15** mostra como o negócio imobiliário "vira grafo" de forma natural — imóveis e pessoas são entidades fortemente relacionadas. Por ser um domínio altamente relacional, é um candidato natural ao GraphRAG, em que perguntas que cruzam várias relações são respondidas percorrendo as arestas do grafo.

---

# Apêndice — RAG vetorial x GraphRAG

A forma mais comum de RAG na prática é o **RAG vetorial clássico**, em que os documentos são
convertidos em **embeddings** e armazenados em um banco de dados vetorial; na consulta, recuperam-se
os trechos mais **similares** e os entrega ao LLM. O fluxo típico é: *embed → armazena vetor → busca
por similaridade → gera resposta*. O **GraphRAG** substitui (ou complementa) essa etapa de
recuperação por um **grafo de conhecimento**. O comparativo abaixo resume as principais diferenças.

## Comparativo

| Critério | RAG vetorial | GraphRAG |
|---|---|---|
| Estrutura de dados | Vetores (*chunks* isolados) | Grafo de entidades e relações |
| Perguntas "locais" (1 fato) | Excelente | Bom |
| Multi-hop (cruzar vários fatos) | Fraco | Excelente |
| Perguntas globais/síntese | Limitado | Forte (resumos de comunidade) |
| Explicabilidade das fontes | Trechos recuperados | Caminho no grafo (nós/arestas) |
| Custo de indexação | Baixo | Alto (LLM extrai entidades/relações) |
| Complexidade de infraestrutura | Menor | Maior (exige banco de grafos) |

<figure style="text-align:center; margin:1em 0; page-break-inside:avoid;">
  <img src="images/FIGURA%2016.png" alt="Comparativo visual RAG vetorial x GraphRAG" style="max-width:100%; max-height:10cm;" />
  <figcaption><strong>Figura 16 — Comparativo visual RAG vetorial x GraphRAG.</strong></figcaption>
</figure>

A **Figura 16** sintetiza toda a atividade: o RAG vetorial recupera por **semelhança**, o GraphRAG recupera por **relação**. Para o RAG vetorial, o melhor caminho de evolução costuma ser o modelo **híbrido** — manter o banco vetorial (ex.: Qdrant, Weaviate, Milvus) e adicionar um grafo de conhecimento por cima, usando cada um onde é mais forte.

**Conclusão prática:** para perguntas diretas e factuais, o RAG vetorial é mais simples e barato e
continua adequado. Quando as perguntas passam a exigir **relações** (multi-hop) ou **visão global** de
um grande acervo, o GraphRAG (ou o híbrido) justifica o custo adicional de indexação e infraestrutura.

---

# Referências bibliográficas


1. LEWIS, P. et al. *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks.* NeurIPS, 2020.
   arXiv:2005.11401.
2. EDGE, D. et al. *From Local to Global: A Graph RAG Approach to Query-Focused Summarization.*
   Microsoft Research, 2024. arXiv:2404.16130.
3. MICROSOFT RESEARCH. *GraphRAG: A new approach for discovery using complex information.* Blog, 2024.
   Disponível em: <https://www.microsoft.com/en-us/research/blog/graphrag-new-tool-for-complex-data-discovery/>.
4. MICROSOFT. *GraphRAG (repositório oficial).* GitHub. Disponível em: <https://github.com/microsoft/graphrag>.
5. TRAAG, V. A.; WALTMAN, L.; VAN ECK, N. J. *From Louvain to Leiden: guaranteeing well-connected
   communities.* Scientific Reports, v. 9, 2019.
6. NEO4J. *What is GraphRAG?* e pacote `neo4j-graphrag`. Disponível em:
   <https://neo4j.com/labs/genai-ecosystem/graphrag/>.
7. LLAMAINDEX. *Property Graph Index — Documentação.* Disponível em: <https://docs.llamaindex.ai/>.
8. LANGCHAIN. *Graph Transformers / GraphRAG — Documentação.* Disponível em:
   <https://python.langchain.com/docs/>.
9. XU, X. et al. *Retrieval-Augmented Generation with Knowledge Graphs for Customer Service Question
   Answering.* (Estudo de caso LinkedIn), 2024. arXiv:2404.17723.
10. CHANDRASEKARAN, D.; MAGO, V. *Knowledge Graphs: An Information Retrieval perspective* / surveys de
    Knowledge Graphs aplicados a LLMs, 2023–2024.

> *Observação: confira e atualize as URLs e os números de arXiv no momento da entrega, pois links e
> versões podem mudar.*

