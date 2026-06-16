# Trabalho Final — Estruturas de Dados (N3)

Repositório com a **Atividade Avaliativa N3** da disciplina de **Estruturas de Dados**, do 4º período
do curso de **Sistemas de Informação** da **UNEMAT — Universidade do Estado de Mato Grosso**.

A atividade consiste em uma **pesquisa teórica sobre Grafos**, abordando duas aplicações práticas da
estrutura de dados de grafos:

1. **Sistemas de Recomendação baseados em Grafos**
2. **Sistemas RAG (Geração Aumentada por Recuperação) baseados em Grafos — GraphRAG**

Para cada tema, o texto cobre: o que é, como funciona, frameworks e bibliotecas open-source
disponíveis e aplicações do mundo real — com figuras ilustrativas (explicadas no texto) e referências
bibliográficas.

## Informações

| | |
|---|---|
| **Aluno** | Angel Rafael Souza Da Silva |
| **Instituição** | UNEMAT — Universidade do Estado de Mato Grosso |
| **Curso** | Sistemas de Informação — 4º Período |
| **Disciplina** | Estruturas de Dados — Atividade Avaliativa N3 |
| **Professor** | Me. Cides S. Bezerra |

- **`duplo/`** — versão combinada, com os dois temas em **um único PDF** (formato recomendado pelo
  enunciado: "entregar um único PDF").
- **`Sistemas de Recomendação baseados em Grafos/`** e **`Sistemas GraphRAG/`** — cada tema em um
  arquivo separado, caso a entrega seja dividida por assunto.

Cada pasta contém o documento em **Markdown** (`.md`) — fonte editável — e o **PDF** gerado a partir
dele, além das imagens (`.png`) usadas nas figuras.

## Como o PDF é gerado

Os PDFs foram gerados a partir dos arquivos Markdown via conversão para HTML (com `marked`) e
impressão headless pelo navegador (Google Chrome `--print-to-pdf`). Alternativamente, é possível usar
o `pandoc`:

```bash
pandoc graphrag.md -o graphrag.pdf --pdf-engine=xelatex -V geometry:margin=2cm
```

> As imagens são referenciadas de forma relativa dentro de cada pasta; por isso o `.md`, o `.pdf` e
> os `.png` devem permanecer juntos no mesmo diretório.
