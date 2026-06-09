# RAG Thyroid

Sistema de consulta a documentos científicos e técnicos sobre doenças da tireoide e hipoparatireoidismo, baseado em RAG (Retrieval-Augmented Generation).

## O problema

Este projeto nasceu do contato próximo com a realidade de pacientes com doenças da tireoide e hipoparatireoidismo, uma condição crônica que exige acompanhamento contínuo e manejo cuidadoso dos níveis de cálcio no organismo.

Ao longo dessa jornada, é comum surgirem dúvidas sobre sintomas, medicações e exames. No entanto, muitas das informações disponíveis estão distribuídas entre artigos científicos, protocolos clínicos, bulas e diretrizes técnicas, frequentemente em linguagem especializada ou em outros idiomas.

Foi dessa necessidade que surgiu a ideia do projeto: criar um sistema baseado em RAG capaz de consultar documentos científicos e técnicos sobre tireoide e hipoparatireoidismo e gerar respostas fundamentadas nessas fontes.

A proposta **não é substituir o acompanhamento médico**, realizar diagnósticos ou indicar tratamentos, mas servir como uma ferramenta de apoio ao acesso à informação baseada em evidências. Para pacientes, isso pode facilitar a compreensão da própria condição e dos materiais disponíveis. Para profissionais de saúde, a mesma arquitetura pode agilizar a consulta a protocolos, artigos e documentos técnicos relevantes.

## Como funciona

O pipeline está implementado no notebook [`rag-documents.ipynb`](rag-documents.ipynb) e segue estas etapas:

1. **Ingestão** — PDFs da pasta `documents/` são carregados com LangChain (`PyPDFLoader`).
2. **Fragmentação** — O texto é dividido em chunks de 600 caracteres com overlap de 150.
3. **Classificação temática** — Cada chunk recebe metadados de tópico (ex.: `hypoparathyroidism`, `thyroid_nodule`, `medication_vitamin_d`).
4. **Indexação** — Os chunks são convertidos em embeddings (`text-embedding-3-small`) e armazenados no ChromaDB (`chroma_db/`).
5. **Consulta** — Perguntas em linguagem natural disparam busca semântica (top 4 chunks) e o modelo `gpt-4o-mini` gera uma resposta em português com base no contexto recuperado, citando as fontes utilizadas.

## Documentos indexados

| Arquivo | Tema principal |
|---|---|
| `protocolo-clinico-e-diretrizes-terapeuticas-hipoparatireoidismo.pdf` | Hipoparatireoidismo |
| `doenca_nodular_da_tireoide-diagnostico.pdf` | Nódulos tireoidianos |
| `Investigating_the_thyroid_nodule.pdf` | Nódulos tireoidianos |
| `Pediatric Thyroid Nodule.pdf` | Nódulos tireoidianos (pediátrico) |
| `The Oncologist - 2008 - Yeung - Management of the Solitary Thyroid Nodule.pdf` | Nódulos tireoidianos |
| `Thyroid Nodules.pdf` / `Thyroid Nodules-2.pdf` | Nódulos tireoidianos |
| `bula_sigmatriol.pdf` / `Bula-Sigmatriol-Profissional-Consulta-Remedios.pdf` | Calcitriol (Sigmatriol) |
| `fixare.pdf` | Suplemento (cálcio + D3 + K2) |
| `Bula-clortalidona-Profissional.pdf` | Clortalidona |

## Pré-requisitos

- Python 3.10+
- Chave de API da [OpenAI](https://platform.openai.com/)

## Instalação

```bash
pip install langchain langchain-community langchain-openai chromadb pypdf
```

## Uso

1. Abra o notebook `rag-documents.ipynb`.
2. Configure sua chave da OpenAI (variável de ambiente ou célula dedicada no notebook — **nunca versione a chave**).
3. Execute as células em ordem para carregar os PDFs, indexar os chunks e criar a cadeia RAG.
4. Faça perguntas via `qa_chain.invoke("sua pergunta aqui")`.

Exemplo:

```python
resposta = qa_chain.invoke(
    "Sinto às vezes dores nas mãos, mesmo tomando cálcio. "
    "Isso pode indicar desequilíbrio de cálcio?"
)
print(resposta["result"])

for doc in resposta["source_documents"]:
    print(doc.metadata["source"], doc.metadata.get("page"))
```

## Estrutura do projeto

```
rag-thyroid/
├── documents/          # PDFs fonte (artigos, bulas, protocolos)
├── chroma_db/          # Banco vetorial persistido (gerado pelo notebook)
├── rag-documents.ipynb # Pipeline completo de ingestão e consulta
└── README.md
```

## Stack

- [LangChain](https://python.langchain.com/) — orquestração do pipeline RAG
- [ChromaDB](https://www.trychroma.com/) — armazenamento vetorial
- [OpenAI](https://openai.com/) — embeddings (`text-embedding-3-small`) e geração (`gpt-4o-mini`)
- [PyPDF](https://pypi.org/project/pypdf/) — leitura de PDFs

## Aviso importante

Este sistema é uma ferramenta de **apoio informativo**. As respostas são geradas a partir de trechos dos documentos indexados e podem conter imprecisões ou estar desatualizadas. **Não utilize este projeto para diagnóstico, prescrição ou decisões clínicas.** Sempre consulte um profissional de saúde qualificado.
