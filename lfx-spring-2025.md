# Not Just Another RAG Project: My LFX Mentorship at Vitess

**By Sakshi Kumar**

When I applied for the LFX Mentorship with Vitess, I walked in with a certain overconfidence. I thought I had “been there, done that” with RAG (Retrieval-Augmented Generation). After all, I had just spent two grueling months building a long-document RAG system for Inter IIT Tech Meet: burning the midnight oil, obsessing over retrieval strategies, and using every SoTA technique I could get my hands on.

So, naturally, I assumed this mentorship would just be a paid extension of what I already knew. A chance to execute some code, brush up a few pipelines, and tick the boxes.

Voila! How wrong can one be?

---

## The Humbling Start

In our initial meetings, I was enthusiastically recommending advanced RAG ideas: tree-based clustering for document retrieval, vector store filtering via metadata scoring and even self RAG.

My mentors, thankfully, kept me grounded.

Their advice was simple: start with a basic pipeline and iterate. The goal wasn’t to go for every technique I knew, but to design something that worked _robustly_ for Vitess use cases. Something stable, extensible, and maintainable: _actual software_.

For the uninitiated: **RAG** is a design pattern where you retrieve contextually relevant documents from a corpus (like docs, Slack threads, GitHub discussions) and then pass that context to an LLM for question answering. In our case, the goal was to build a chatbot that could answer CLI and Slack queries about Vitess by indexing its documentation.

---

## Wait, This Isn’t Just ML?

Turns out, this project was much less about LLMs and much more about building a real-world, production-grade retrieval system. We picked **Haystack** over the more mainstream **LangChain**, because it was modular, extensible, and dev-friendly at its core.

I ended up extending several of Haystack’s base classes to suit our use case, from extracting YAML frontmatter for metadata tagging, to adding debug-friendly wrappers at key stages in the pipeline. At some point, it hit me: _this wasn’t a machine learning project, it was a software development project with ML in it._

---

## Testing My Patience (And Code)

Rejecting a method wasn’t as easy as relying on instinct or experience. I had to prove it. I wrote **close to 100 test cases**, some contributed by my mentors too.

To do this well, I needed to understand **Vitess** itself, not just the chatbot interface. I had assumed this was a side project, isolated from core Vitess development. But I was wrong again. Understanding **Vitess’ sharded MySQL architecture**, and how CLI and docs map to real developer workflows, was key to designing a usable RAG system.

---

## Our Final Architecture

We slowly converged on a hybrid retrieval architecture. Here’s a breakdown of some of the coolest pieces:

- **Dynamic Data Extraction:**
  We didn’t rely on static files. Instead, markdown files were dynamically extracted from [vitess.io](https://vitess.io). First on demand, later using a config-based system with support for forced re-extraction flags.

- **Frontmatter Metadata:**
  Markdown files often had YAML metadata: titles, aliases, descriptions. I built a parser that persisted metadata across chunks (until a new frontmatter block was found). This metadata hugely improved contextual retrieval.

- **Pipeline Components:**

  - `TextConverter` → Turn file paths or byte streams into Haystack docs
  - `DocumentCleaner` → Strip irrelevant noise
  - `DocumentSplitter` → Chunk intelligently with size/overlap configs
  - `DocumentEmbedder` → Encode with our chosen embedding model
  - `BM25Retriever` + `EmbeddingRetriever` → Hybrid retrieval
  - `DocumentJoiner`, `Ranker` → Rank top documents

- **Storage:**
  Started with `InMemoryDocumentStore`, moved to `ChromaDB` for persistence and embedding version compatibility.

---

## Tooling & Developer Experience (DX)

This part surprised me the most - how much of my time was spent on DX:

- **Streamlit Web UI**
  A quick-and-dirty UI so stakeholders could play with queries and evaluate answers.

- **CLI Scripts**
  For running rapid pipeline iterations during local development.

- **FastAPI Backend**

  - Chatbot endpoint
  - Slack listener
  - Feedback collector to assess real-world performance

- **Custom Debugger**
  Added at four key pipeline stages. Lifesaver during metadata bugs and embedding mismatches.

---

## What I Learned (The Hard Way)

- **Factory Method Design**
  This was _tough_. In our first meeting on pipeline factories, I was blank. But my mentors patiently walked me through a working example, and from there, I slowly started to understand how it made everything more modular and immutable.

- **Config-Driven Architecture**
  Moving from hard-coded strings to config variables made the system significantly more stable and extensible. I always understood this in theory, but seeing just how much it could replace, even things I had unknowingly hard-coded, was eye-opening. It didn’t feel wrong at the time, but in hindsight, it clearly was. This was one of those lessons that felt more “software engineering” than “ML,” but it turned out to be a major takeaway.

- **Metadata Woes in Haystack**
  I spent _days_ wondering why my metadata component didn’t work, only to find out Haystack silently overrode it. Lesson learned: _read the docs_, and if needed, the source code.

- **Versioning Nightmares**
  Between Pydantic v2 and older embedding models, I kept running into dependency hell. I eventually built clean wrappers to isolate version conflicts.

- **Local LLM Realities**
  Explored GPU-less local inference (AutoGPTQ, AWQ), and learned the hard limits of tools like bitsandbytes. Didn’t implement, but it opened a whole new rabbit hole I now understand better.

---

## Final Thoughts

I came into this mentorship thinking I’d just plug in some embeddings, fetch a few docs, and wrap it all in a chatbot. Instead, I left with:

- A deeper appreciation for **production engineering**
- Actual battle scars from debugging haystack components
- A better grip on software design patterns and API thinking
- Respect for _how much work it takes to build something simple that actually works_

This wasn’t just an ML internship. It was an **end-to-end software build**, with real users, real data, and real constraints.

To my mentors - thank you for keeping me humble, supporting me through the learning curve, and encouraging me to go beyond what I thought was the "expected outcome." I walked in thinking I knew RAG. I walked out realizing how much I still have to learn.

---

## Looking Ahead

As the project now moves into its next phase, I’m excited to hand it off to the next GSoC mentee who will take it forward. I’ll definitely be around - staying in touch with this wonderful community, watching the RAG agent evolve with real user feedback, and (hopefully) seeing some of my early suggestions, like multi-cycle retrieval in a single RAG lookup, come to life in production. It’s been a deeply rewarding journey, and I look forward to contributing in whatever way I can as the story continues.

---
