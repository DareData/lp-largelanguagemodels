# Lesson 02: Chunking Strategies

### Cutting documents into pieces that retrieve well

A 500-page annual report or loan prospectus cannot be handed to the model for every question, and a single vector cannot represent it well either, so the system cuts documents into pieces small enough to be about one thing and large enough to make sense on their own. Chunking looks like a minor preprocessing detail and behaves like a core design decision, because the same corpus, embedding model and index can give very different answers depending on where the cuts fall.

This lesson explains why chunking exists, compares the main strategies, shows how to treat tables, lists and FAQ pairs, presents two techniques that put lost context back, and gives an evidence-based way to choose, since published results disagree.

<p align="center">
  <img src="../images/l3-02-01.png" alt="Chunks">
</p>

### Learning Objectives

By the end of this lesson, you will be able to:

- **Explain** why documents are chunked and how chunk size trades precision against context.
- **Compare** fixed, recursive, semantic, parent-child, proposition and hierarchical chunking.
- **Choose** a chunk unit for FAQ pairs, tables, lists and code.
- **Apply** contextual retrieval and late chunking, and measure the effect of a chunking choice with an evidence-hit test.

---

### Lesson Content

#### Why we cut at all, and how small is too small

Three constraints force the cut. The first is the **input limit of embedding models**, one of the main limitations of RAG and much tighter than that of language models: all three of OpenAI's embedding models accept at most 8,192 tokens per input ([OpenAI embeddings guide](https://developers.openai.com/api/docs/guides/embeddings)), about a dozen pages of text, while OpenAI's newest language models accept context windows of about 1.05 million tokens (as of October 2026). A 500-page report simply does not fit into one embedding call.

The second is that an embedding model **compresses its whole input into one vector** of fixed size: 1,536 numbers for `text-embedding-3-small` and 3,072 for `text-embedding-3-large`. That is far more room than the three dimensions we can picture, but it is not infinite, so the more topics one input covers, the more they are averaged together. A chunk that mixes returns, delivery and gift cards gets a vector that sits somewhere between the three topics, so it is not particularly close to a question about any of them and may rank below chunks that cover only one.

The third is the **space and cost of the prompt**. Every retrieved passage occupies part of the model's context window, is billed per token and adds latency, so sending the few passages that answer the question is cheaper and faster than sending whole documents, and leaves room for instructions and conversation history. More text is also not automatically better: [Liu et al. (2023)](https://arxiv.org/abs/2307.03172) found that accuracy was highest when relevant information sat at the start or end of the input and dropped when it was in the middle, even for models built for long contexts. That study tested 2023 models, which are old by language model standards, so treat its exact pattern with caution; more recent work still finds degradation with length, for example NoLiMa ([Modarressi et al., 2025](https://arxiv.org/abs/2502.05167)), where 11 of 13 models claiming at least 128K tokens of context fell below half of their short-context performance at 32K tokens once the question and the relevant passage shared few words.

The opposite risk is chunks so small that they lose their context. A chunk can be embedded however short it is, but a fragment such as "The renewal fee is 20 EUR", cut away from the heading that says it is about passports, lands far from the other passport chunks and from a question like "how much does it cost to renew my passport?", so it is simply not found. Chunk size therefore trades staying on one topic against keeping enough context to land near the right questions, and it is best measured in tokens, because tokens are what embedding models limit and prompts spend. There is no universal best size, which is why the last section of this lesson is about how to measure.

| Chunk size | Where its vector lands | What goes wrong |
| :-- | :-- | :-- |
| Too small | Far from related chunks and questions, because the context that says what it is about is missing | The chunk is not found; "The renewal fee is 20 EUR" is never retrieved for a passport question |
| Too large | Between several topics, close to none of them | The chunk ranks low, and when it is found it fills the prompt with unrelated text |
| Balanced | Near the questions about its topic | Nothing systematic; the right size for your corpus is found by measuring |

#### Strategies compared

Fixed windows of N tokens with a small overlap are the simple baseline, and they cut through sentences and sections. Recursive, structure-aware splitting tries headings first, then paragraphs and then sentences until pieces fit, and respects the structure the author gave the document, which makes it the practical default. Semantic chunking cuts where the embedding similarity between neighboring sentences drops, and parent-child retrieval searches small chunks but returns the larger parent section, separating what you match from what you send. 
> Two research results extend the range, and newer evidence tempers the first. Dense X Retrieval ([Chen et al., 2023](https://arxiv.org/abs/2312.06648)) proposes propositions, atomic statements that each carry one fact, as the retrieval unit, and found that indexing by propositions beat passage-level indexing on its benchmarks. A recent reproduction of chunking methods on shared benchmarks, [Zhou, Wang and Koopman (2026)](https://dl.acm.org/doi/abs/10.1145/3805712.3808575), found the opposite for ordinary search over a document collection: proposition-based segmentation consistently underperformed all other segmentation methods, and simple structure-based methods (fixed-size, sentence and paragraph splitting) outperformed model-guided ones. RAPTOR ([Sarthi et al., 2024](https://arxiv.org/abs/2401.18059)) builds a tree by repeatedly embedding, clustering and summarizing chunks from the bottom up, so a query can match a detail or a document-level summary, and it reported state-of-the-art results on multi-step question answering, including a 20-point absolute accuracy gain on QuALITY when paired with GPT-4. Both cost a model pass per document, and the tree is costly to refresh.

| Strategy | How it cuts | Strength | Weakness |
| :-- | :-- | :-- | :-- |
| Fixed size with overlap | Every N tokens, with a small overlap | Simple and predictable | Cuts through sentences and sections |
| Recursive, structure-aware | Headings first, then paragraphs, then sentences, until pieces fit | Respects the structure the author gave, the practical default | Needs structured input |
| Semantic | Cuts where the embedding similarity between neighboring sentences drops | Boundaries follow topics | Extra embedding cost without consistent gains over fixed-size chunking ([Qu, Tu and Bao, 2024](https://arxiv.org/abs/2410.13070)) |
| Parent-child | Searches small chunks and returns the larger parent section | Separates what you match from what you send | Two levels to store and manage |
| Propositions | Rewrites text into atomic statements, each carrying one fact | Beat passage-level indexing in [Chen et al. (2023)](https://arxiv.org/abs/2312.06648) | A model pass per document, and underperformed all other methods in a 2026 reproduction |
| Hierarchical summaries (RAPTOR) | Embeds, clusters and summarizes chunks into a tree | A query can match a detail or a document-level summary | A model pass per document, and costly to refresh |

Generic splitters assume prose, whereas real corpora also contain content with its own natural unit, and respecting that unit beats any size setting. An FAQ pair is one chunk, so that "how long do refunds take?" matches the pair directly. A table row travels with its caption and column headers, or is serialized into a sentence. A list stays with the sentence that introduces it, and code is split on syntax boundaries with the file path and signature as metadata. Every chunk should also inherit the metadata of its document and keep its position, so retrieval can filter and cite precisely and a neighbor can be fetched when an answer straddles a boundary.

| Content | Natural unit | Rule |
| :-- | :-- | :-- |
| FAQ pair | One question with its answer | One chunk, so that "how long do refunds take?" matches the pair directly |
| Table | A row | The row travels with its caption and column headers, or is serialized into a sentence |
| List | The list with its introducing sentence | The list stays with the sentence that introduces it |
| Code | A syntactic unit | Split on syntax boundaries, with the file path and signature as metadata |


#### Putting lost context back: contextual retrieval and late chunking

Cutting a document loses context: a chunk from the middle of an annual report that says "revenue grew 3% over the previous quarter" no longer says which company or which quarter, so it cannot be found for "how did Acme's revenue develop in Q2?". Two techniques put that context back, from opposite ends.

**Contextual retrieval** adds the context as text. Before indexing, a language model receives the whole document and one chunk and writes a short note, usually 50 to 100 tokens, that situates the chunk ("This chunk is from Acme Corp's Q2 2025 report, section on revenue"); the note is prepended to the chunk before it is embedded and keyword-indexed. In Anthropic's [report](https://www.anthropic.com/news/contextual-retrieval), the share of questions whose relevant chunk was missing from the top 20 fell from 5.7% to 3.7% with contextual embeddings (a 35% reduction), to 2.9% when contextual keyword search was added, and to 1.9% with reranking on top. Those are the vendor's own measurements on its own data, and the price is one model call per chunk at indexing time, which Anthropic estimated at about 1.02 USD per million document tokens using prompt caching.

**Late chunking** adds the context inside the embedding. Instead of embedding each chunk separately, a long-context embedding model reads the whole document (or as much as fits in its input limit) once, and the chunk vectors are formed afterwards from the token representations of each chunk's span, so every chunk vector has already "seen" its surroundings. [Günther et al. (2024)](https://arxiv.org/abs/2409.04701) report better results across retrieval tasks without additional training, and the method needs no extra model calls. The 2026 reproduction by [Zhou, Wang and Koopman (2026)](https://dl.acm.org/doi/abs/10.1145/3805712.3808575) confirmed a gain for search across a document collection but found that it degraded search for a passage within a single long document, so the benefit depends on the task. It also requires a long-context embedding model and still cannot see beyond that model's input window.

| Technique | Adds context through | Extra cost | Best for |
| :-- | :-- | :-- | :-- |
| Contextual retrieval | A generated note prepended to each chunk | One language model call per chunk at indexing | Chunks that say "it", "this policy" or "the company" without naming the subject |
| Late chunking | Embedding the document before splitting | A long-context embedding model, no extra calls | Documents that fit in the embedding model's input window |

#### Choosing a strategy

[Zhou, Wang and Koopman (2026)](https://dl.acm.org/doi/abs/10.1145/3805712.3808575), reproduced earlier chunking studies on shared benchmarks and found that the best strategy depends on the task: simple structure-based splitting did best for search across a collection of documents, a model-guided method did best for finding a passage inside one long document, and chunk size explained little of the difference. The reliable way to choose is to measure on your own corpus. Write 30 to 50 real questions, mark for each the passage that contains the answer (the evidence), index the corpus with each candidate strategy, and count how often the evidence appears in the top results. That share is the **evidence-hit rate**, and running it for each candidate is the evidence-hit test: for example, if recursive splitting finds the answer passage in the top 5 for 41 of 50 questions and semantic chunking for 39, recursive splitting wins on your data whatever the published results say. Change one setting at a time, keep the strategy that finds the evidence most often at an acceptable cost, and look at the questions that still miss, because they show which technique to try next.

#### In practice

Begin with recursive splitting by structure, heading prefixes and modest overlap, and build the evidence-hit test before trying anything else. Try the more expensive techniques only on the questions that fail, and compare their gain with their per-chunk cost. FAQ pairs, tables and lists usually deserve a custom unit whatever the general setting is. If answers keep missing a qualifier that sits one paragraph away, parent-child retrieval or contextual notes are the first things to try.

**Summary:**
- One vector per chunk: a chunk covering several topics lands between them, and a chunk without context lands far from related questions. Measure size in tokens and decide on your corpus.
- Recursive, structure-aware splitting is the default and did best in a recent reproduction; propositions, summary trees, contextual notes and late chunking are hypotheses with a cost, whose gains depend on the task.
- Match the unit to the content (FAQ pair, table row with headers, list with intro) and keep metadata and position on every chunk.
- Decide with an evidence-hit test (how often each strategy retrieves the passage that answers your real questions), and treat vendor-reported gains as such.

---

### References

- OpenAI, [Embeddings guide](https://developers.openai.com/api/docs/guides/embeddings) (accessed October 2026). Input limits and dimensions of OpenAI's embedding models.
- Liu et al. (2023), [Lost in the Middle](https://arxiv.org/abs/2307.03172). How models use long contexts; tested on 2023 models.
- Modarressi et al. (2025), [NoLiMa: Long-Context Evaluation Beyond Literal Matching](https://arxiv.org/abs/2502.05167).
- Chen et al. (2023), [Dense X Retrieval](https://arxiv.org/abs/2312.06648). Propositions as the retrieval unit.
- Zhou, Wang and Koopman (2026), [Beyond Chunk-Then-Embed: A Comprehensive Taxonomy and Evaluation of Document Chunking Strategies for Information Retrieval](https://dl.acm.org/doi/abs/10.1145/3805712.3808575) ([arXiv](https://arxiv.org/abs/2602.16974)). A reproduction of chunking strategies on shared benchmarks.
- Sarthi et al. (2024), [RAPTOR](https://arxiv.org/abs/2401.18059). Trees of recursive summaries.
- Anthropic (2024), [Introducing Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval). Context notes with BM25 and reranking.
- Günther et al. (2024), [Late Chunking](https://arxiv.org/abs/2409.04701). Embedding before splitting.
- Qu, Tu and Bao (2024), [Is Semantic Chunking Worth the Computational Cost?](https://arxiv.org/abs/2410.13070). A cost-benefit comparison.

### Further Reading

- [Lesson 01: Data Preparation](01-data-preparation.md) - the previous lesson.
- [Lesson 03: Embedding Models](03-embedding-models.md) - the next lesson.
