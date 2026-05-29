# Spec: `retrieve()`

**File:** `retriever.py`
**Status:** Spec incomplete — fill in all blank fields before implementing

---

## Purpose

Given a user's natural language query, find the most relevant chunks from the vector store using semantic similarity search. Return them ranked by relevance so that `generate_response()` can use them as context.

---

## Input / Output Contract

**Inputs:**

| Parameter   | Type  | Description                                                                |
| ----------- | ----- | -------------------------------------------------------------------------- |
| `query`     | `str` | The user's natural language question                                       |
| `n_results` | `int` | Maximum number of chunks to return (default: `N_RESULTS` from `config.py`) |

**Output:** `list[dict]`

Each dict in the returned list must contain exactly these keys:

| Key          | Type    | Description                                                   |
| ------------ | ------- | ------------------------------------------------------------- |
| `"text"`     | `str`   | The chunk text                                                |
| `"game"`     | `str`   | The game name this chunk came from                            |
| `"distance"` | `float` | Cosine distance score — lower means more similar to the query |

Results should be ordered from most to least relevant (lowest to highest distance). Returns an empty list `[]` if the collection contains no documents.

---

## Design Decisions

_Complete the fields below before writing any code. Use your AI tool in Plan or Ask mode to help you reason through what belongs here — but the decisions are yours._

---

### Query approach

_Describe how you will use `_collection.query()` to find relevant chunks. What arguments will you pass, and why?_

````
For collections.query we have to pass the query and the return size here in the config it looks to be 3```

---

### Return structure

*Sketch out what one item in your return list looks like as a concrete example. Where does each field come from in the query results?*

````

After the .\_collections.query embedds the user query -
it return JSON like response:
{
"ids":["game1","game2","game3"]
documents:[blah blah, blah blah, blah blah]
metadatas : [
{"game":"Catan"}, {"game": ..}
...
Distance : [
[[0,12,025,...]]
] ]

}
Here distnace the lowe rthe better means they are closer

```

---

### Handling the nested result structure

*`_collection.query()` returns nested lists. Describe what index you need to access to get the actual list of results for a single query, and why the nesting exists.*

```

We need to do result["id"][0] to get the list of chunk texts -
It is nested since we can pass in multple quieries and for query 1 its [0] in tihs example we have 1 query but what if we had 3 queries then chroma makes it easy for us wiht:
query[0].documetns getting top 3 for query 1
query[1].docuemnts getting top 3 for ...
...

```

---

### Relevance threshold

*Will you filter out results above a certain distance score, or return all `n_results` regardless of how relevant they are? What are the tradeoffs of each approach?*

```

Currently we are returning the top3 and we do not have a standard/threshold

```

---

### Edge cases

*How does your implementation behave when: (a) the collection is empty, (b) the query matches no chunks well, (c) the query matches chunks from multiple games?*

```

If collection is empty then it reuturns and empty array
b) technicailyl it just return the three best matches thoguh they will be very high distnace unless we create a threshold or standard for it.
c) it returns from mutlple games but limits to top 3 still

```

---

## Implementation Notes

*Fill this in after implementing, before moving to Milestone 3.*

**Test query and top result returned:**

```

Query: [your test query]
Top result game: [game name]
Distance score: [score]
Does it make sense? [yes / no / explain]

```

**One thing about the query results that surprised you:**

```

Nothin really eveything was expected considering how everythign is structured

```

```
