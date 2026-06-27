# Website Q&A Bot

A small RAG app that lets you paste a URL and ask questions about whatever's on that page. Built it with LangChain for the retrieval/LLM plumbing, FAISS for the vector store, and Gradio for the UI. Runs fine in Google Colab.

It's basically the resume Q&A bot reworked to pull from a live webpage instead of a PDF, with a temperature slider and source chunks added on top.

## How it works

1. You give it a URL, it scrapes the page with `WebBaseLoader`.
2. The text gets chopped into ~1000 character chunks (200 char overlap so context doesn't get cut off mid-sentence).
3. Each chunk gets embedded with `all-MiniLM-L6-v2` and dropped into a FAISS index.
4. When you ask something, it grabs the 4 most relevant chunks and stuffs them into the prompt along with your question.
5. Llama 3.3 70B (via Groq) answers using only that context — nothing else.

## Why it won't just make stuff up

The prompt is pretty locked down:

- If the answer isn't in the retrieved chunks, it says so instead of guessing.
- If someone tries the classic "ignore previous instructions" / "reveal your system prompt" / "pretend you're a different AI" tricks, it just treats that text as regular page content and ignores it as a command.
- It won't hand over personal or sensitive info even if it happens to be sitting in the context.
- Off-topic questions (general trivia, "write me code", etc.) get redirected back to "I can only answer questions about this website's content."

You can also see exactly which chunks the model used to answer, so it's not a black box — if the answer looks off, you can check whether the retrieval actually pulled the right part of the page.

## Setup

You need a Groq API key. In Colab, add it under Secrets as `GROQ_API_KEY`, or just set it directly:

```python
import os
os.environ["GROQ_API_KEY"] = "your-key-here"
```

Then run the script top to bottom. The first cell installs everything, the rest sets up the chain and launches the Gradio app.

## Using it

1. Paste a URL into the box and hit **Index Website**.
2. Wait for the status line to say it's indexed.
3. Type a question, adjust the temperature slider if you want (0 = stick close to the text, higher = a bit more freeform phrasing), hit enter.
4. Answer shows up top, the chunks it pulled from show up below so you can sanity-check it.

## Notes / things to watch out for

- Some sites block scrapers or render content with JS, so `WebBaseLoader` won't always get the full page — works best on plain HTML/text-heavy pages like docs, blogs, wikis.
- Re-indexing a new URL overwrites the old index, so one site at a time.
- Temperature only affects phrasing, not the facts — the retrieval step is what actually controls what the model can talk about.
- This is a demo/learning project, not hardened for production — don't point it at anything you wouldn't want scraped.

## Stack

LangChain · Groq (Llama 3.3 70B) · HuggingFace sentence-transformers · FAISS · Gradio
