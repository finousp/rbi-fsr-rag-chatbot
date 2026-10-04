# RBI Financial Stability Report (June 2026) – RAG Chatbot

A local Retrieval-Augmented Generation (RAG) chatbot that answers questions from the RBI Financial Stability Report (June 2026) and shows the PDF pages each answer came from. Everything runs on your own machine, so the document is never sent to an external API.

## How it works

1. **Load:** the 193-page PDF is loaded with `PyPDFLoader`.
2. **Clean and chunk:** contents, list of charts, list of tables and abbreviation pages are removed. The remaining text is split into overlapping chunks of about 1000 characters.
3. **Embed and store:** chunks are converted to vectors with `all-MiniLM-L6-v2` and stored in a local ChromaDB database.
4. **Retrieve:** for each question, the most similar chunks are fetched (top 3 by default).
5. **Generate:** the chunks and the question go to a local `qwen3:1.7b` model through Ollama. The prompt tells it to answer only from the retrieved text, or say it couldn't find the answer.
6. **Cite:** the PDF pages of the retrieved chunks are printed with each answer.

## Tech stack

Python, LangChain, ChromaDB, Hugging Face sentence-transformers, Ollama (qwen3:1.7b), Jupyter Notebook

## Example

```
Q: What does the report say about NBFC asset quality?
A: The report indicates improved NBFC asset quality, with a steady decline in the net NPA ratio
   (reaching 0.8% in March 2026) and moderating slippage ratios for middle-layer NBFCs (NBFC-ML).
Sources (PDF pages): 57, 58, 107
```

Questions that are not covered by the report are refused:

```
Q: Who won the 2022 FIFA World Cup?
A: I couldn't find this in the report.
```


## How to run

1. Install [Ollama](https://ollama.com) and pull the model:
```
   ollama pull qwen3:1.7b
```
2. Clone this repository and install the requirements:
```
   pip install -r requirements.txt
```
3. Download the report from the [RBI website](https://www.rbi.org.in/ScriptS/FsReports.aspx) and place it in the `data/` folder. The PDF is not included in this repository. Make sure `PDF_PATH` in the notebook matches the file name.
4. Make sure Ollama is running, open `rbi_fsr_rag.ipynb`, and run all cells. The first run creates the `chroma_db` folder, which can take a few minutes. Later runs load it.

Tested on Windows with 8 GB RAM.

## Limitations

- Uses a small 1.7B-parameter model so it can run on a laptop with 8 GB RAM, so answers can be brief or occasionally wrong.
- In a manual check of 6 questions against the report, 5 were fully correct and 1 had a misattributed detail (see the "Known limitation" section of the notebook). This is a small sample, not a general accuracy figure.
- Numbers from tables and charts can be extracted poorly or attached to the wrong metric.
- Broad topics that appear in many parts of the report (like AI risk) gave fuller answers with more retrieved chunks (k=5 instead of 3).
- Single document only; no chat history.
- Answers may contain errors, so verify them against the cited PDF pages.

## Possible improvements

- Try a larger model, if hardware allows.
- Support multiple editions of the report and filter by edition.
- Add a reranker or hybrid (BM25 + embeddings) search.
- Evaluate on a larger set of test questions, possibly graded by a stronger model.
- Add a simple Streamlit interface.

## Acknowledgements

The basic pipeline follows the tutorial ["Create a ChatGPT for your personal documents"](https://amanxai.com/2026/08/02/create-a-chatgpt-for-your-personal-documents/) on AmanXAI. I adapted it for the RBI Financial Stability Report, added front-matter cleaning, page-number citations and out-of-scope handling.

## Source document

RBI Financial Stability Report, June 2026: https://www.rbi.org.in/ScriptS/FsReports.aspx