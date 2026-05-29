# Spec: `generate_response()`

**File:** `generator.py`
**Status:** Spec incomplete — fill in all blank fields before implementing

---

## Purpose

Given a user query and a list of retrieved rule chunks, generate a response that directly answers the question using only the retrieved text as context. The response must be grounded — it should not draw on the model's general knowledge of board games, only on what was retrieved.

---

## Input / Output Contract

**Inputs:**

| Parameter          | Type         | Description                                                                             |
| ------------------ | ------------ | --------------------------------------------------------------------------------------- |
| `query`            | `str`        | The user's original question                                                            |
| `retrieved_chunks` | `list[dict]` | Ranked list of chunks from `retrieve()`, each with `"text"`, `"game"`, and `"distance"` |

**Output:** `str`

A plain string containing the response to show the user. The response should:

- Answer the question using only the retrieved rule text
- Identify which game the answer comes from
- Acknowledge clearly when the answer is not found in the loaded rules

Returns a fallback string (not an error) when `retrieved_chunks` is empty.

---

## Design Decisions

_Complete the fields below before writing any code. Use your AI tool in Plan or Ask mode to help you reason through what belongs here — but the decisions are yours._

---

### Context formatting

_How will you format the retrieved chunks before passing them to the LLM? Describe the structure — not the code. Consider: will you label chunks by game? Include distance scores? Separate chunks with delimiters?_

```
The retrieved chunks are formatted into a single context string before being passed to the LLM. Each chunk is labeled with its game name in brackets (e.g. [Catan]) so the model can attribute rules to the correct game and avoid mixing rules across games. Chunks are separated by a --- delimiter so boundaries are clear. Distance scores are not included in the prompt — they're used internally for ranking and logging only, and would add noise for the LLM. The final prompt sends this context block followed by the user's original query, plus a system instruction telling the model to answer using only the provided context.
```

---

### System prompt — grounding instruction

_Write the exact system prompt instruction you will use to prevent the model from answering beyond the retrieved text. This is the most important design decision in this function._

```
You are a board game rules assistant. Answer the user's question using ONLY the
rule text provided in the context below. Do not use any outside knowledge about
board games, even if you think you know the answer.

If the provided context does not contain enough information to answer the
question, say so clearly — do not guess or fill in gaps from memory.

Always state which game your answer comes from, using the game name labeled in
the context (e.g. "According to the Catan rules...").
context:[...]
```

---

### System prompt — citation instruction

_Write the exact instruction you will use to tell the model to identify which game its answer comes from._

```
State which game the answer comes from, using the game name labeled in brackets
in the context. Phrase it naturally, e.g. "According to the Catan rules, ..." —
and if chunks from more than one game are used, name the game for each part.
```

---

### Fallback behavior

_What should the response say when the answer isn't found in the loaded rule books? Write the exact fallback message._

````

I couldn't find anything in the loaded rule books to answer that. Try
rephrasing your question, or make sure the relevant game's rules have
been loaded.

```

---

### Handling low-relevance chunks

_`retrieved_chunks` may include chunks with high distance scores (weak relevance). Will you filter these out before building context, pass them all in, or handle them another way? What are the tradeoffs?_

````

Two options. (1) Threshold-filter: drop chunks above a distance cutoff before building context. Pro: cleaner context, and if everything is filtered out we can return the fallback without an LLM call. Con: cosine distances aren't intuitive, the cutoff is brittle and dataset-dependent, and low distance means on-topic, not contains the answer — so a threshold can wrongly discard a good chunk or keep a topical-but-useless one. (2) Pass everything, let the LLM judge: rely on the grounding system prompt to ignore weak chunks and say "not found" when none answer the question. Pro: no magic number, more robust, and the LLM handles irrelevant context well with a strong prompt. Con: slightly noisier prompt and always pays for the call. Decision: pass everything — we only return 3 chunks (noise is minimal), it stays consistent with the no-threshold retriever, and grounding is already handled in the system prompt.

```

---

### Message structure

_Describe how you will structure the messages list for the API call — what goes in the system message vs. the user message?_

```

The messages list uses two roles. The system message holds everything that stays constant across calls: the assistant's role, the grounding instruction (answer only from the provided context; say so if the answer isn't there), and the citation instruction (name the game the answer comes from). The user message holds what changes each call: the formatted context (the 3 retrieved chunks, labeled by game) followed by the user's question. The split follows the principle that the system message defines how to behave while the user message carries this request's data — which is why the query and chunks go in the user message, not the system prompt.

```

---

## Implementation Notes

_Fill this in after implementing and testing._

**Test query and response:**

```

Query: What happens if you roll a 7 in Catan?
Response: In the game "Catan", when a 7 is rolled no resources are produced.
Every player with more than 7 cards discards half (rounded down);
the roller moves the robber and steals one resource.
Correctly grounded? Maybe (since chunking is bit naive)
Cited the right game? yes ("In the game 'Catan'...")

```

**One thing you changed from your original spec after seeing the actual output:**

```

My spec assums a correct, cited answer meant the response was grounded. After
seeing the output, I realized naive chunking means I can't be sure the
retrieved chunk contained the _complete_ rule — a correct answer might partly
come from the model's training. I'd switch to semantic chunking to trust
grounding more.

```

```
