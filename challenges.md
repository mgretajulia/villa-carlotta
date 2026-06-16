---
layout: default
title: Villa Carlotta
---

<div class="site-nav">
  <a href="index.html">⭐ Home</a>
  <a href="topic.html">⭐ Topic</a>
  <a href="methodology.html">⭐ Methodology</a>
  <a href="sparql.html">⭐ SPARQL & Results</a>
  <a href="gaps.html">⭐ Identifying Gaps</a>
  <a href="prompts.html">⭐ LLM Prompts</a>
  <a href="rdf.html">⭐ RDF Triples</a>
  <a href="conclusion.html">⭐ Conclusion</a>
</div>

<p align="center">
  <img src="https://placehold.co/400x250?text=Challenges+%28400x250%29" alt="Challenges" width="400">
</p>

# Challenges Faced During the Project

In the development of our KE4H project on
[**Villa Carlotta**](https://w3id.org/arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03),
we ran into several challenges related to semantic technologies, data selection, and prompt engineering. Here is how
we tackled each of them.

---

### 🧩 Understanding How SPARQL Works

At the beginning, we found it difficult to understand how SPARQL works and how to formulate accurate, precise
queries, since the language requires strict syntax and logic.

**✅ Solution:**
We studied existing SPARQL patterns and practiced using key keywords such as `UNION`, `OPTIONAL`, `FILTER`, `BIND`
and `VALUES`, which gradually improved our confidence in writing more complex queries.

---

### 🏛️ Picking the Right IRI Among Many Results

[Query 1](sparql.html) returned **11 different IRIs** all labeled "Villa Carlotta", and [Query 2](sparql.html) added
two more under `ArchitecturalOrLandscapeHeritage`. Choosing the *right* IRI to use as the subject of our new triples
was not obvious.

**✅ Solution:**
We standardized on
[`Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03`](https://w3id.org/arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03),
since it is the IRI that ArCo explicitly classifies as `ArchitecturalOrLandscapeHeritage` — the closest match to
"the villa as a whole" — and it is also the IRI that turned out to link the modern and historical names together (see
our [Breakthrough Analysis](sparql.html)).

---

### 🔍 The "Villa Sommariva" Name Confusion

Once we discovered that Villa Carlotta was historically known as "Villa Sommariva" (and, even earlier, "Villa
Clerici"), our searches suddenly returned **37 results instead of 11** — many of which were ambiguous or only
loosely related to our topic.

**✅ Solution:**
We used `UNION` and `BIND` to tag each result with its source ("Property" vs "Subject"), then manually filtered for
the entities that also appeared in our original Query 1 results — this is how we found the single cross-over IRI that
connects both names.

---

### 💬 Getting Useful Answers from LLMs

Across our five prompts (see [LLM Prompts](prompts.html)), the LLMs often gave us **valid but generic** triples —
typically a single long description — instead of the more granular, multi-entity triples we were hoping for.
[Gemini](https://gemini.google.com/) in particular frequently returned **"no results"** when we tested its generated
queries on Yasgui and DBpedia.

**✅ Solution:**
We compared all four LLMs (Copilot, ChatGPT, Gemini, Claude AI) side by side for every missing information item, and
combined the **best parts of each answer**. For example, ChatGPT's "Title resource" pattern (Missing Information #3)
and Gemini's multi-entity modeling (Missing Information #4) turned out to be the most useful results overall, while
Claude AI was used to independently fact-check the historical claims behind each triple.

---

### 🤖 Shortcomings of Each LLM

Looking back across our five prompts (see [LLM Prompts](prompts.html)), each LLM showed a different pattern of
problems — and not always a consistent one.

- **Copilot** generated SPARQL `CONSTRUCT` queries for our first three prompts that *looked* syntactically correct
  but consistently returned **empty results** — no valid triples — even after we switched from a few-shot to a
  chain-of-thought prompting strategy. Then, in prompts 4 and 5, Copilot suddenly started producing **working
  queries with valid triples**, suggesting it gradually adapted to the ArCo vocabulary and our course materials.
- **ChatGPT** was by far the most reliable: across all three prompting techniques, it consistently produced
  **accurate `CONSTRUCT` queries and valid triples**, with minimal hallucination and stable, ontology-aligned
  outputs.
- **Gemini** repeatedly **hallucinated**, producing overly long responses and invalid or non-executable queries for
  prompts 1–3. In prompt 4 it finally generated a valid `CONSTRUCT` query (confirmed in Yasgui) — but it returned
  **24 triples with many different predicates**, making it hard to tell which ones were actually relevant. By
  prompt 5, it had regressed back to hallucinated, unusable output.

**✅ Solution:**
We treated each LLM's output as a *draft* rather than a final answer — verifying every generated query in Yasgui and
DBpedia before trusting it, and re-running queries with a different prompting technique whenever a model produced
empty or hallucinated results.

| LLM | Shortcomings / Problems Encountered | Behaviour Across Prompts |
|---|---|---|
| Copilot | Correct-looking queries but empty results; no valid triples in early prompts; slow adaptation to ontology | Prompts 1–3: non-functional queries. Prompts 4–5: valid queries and triples after contextual adaptation |
| ChatGPT | Very few shortcomings; minimal hallucination | Consistently produced valid queries and triples across all prompting techniques |
| Gemini | Frequent hallucinations; invalid queries; overly long outputs; too many heterogeneous triples; regression in prompt 5 | Prompts 1–3: invalid or hallucinated results. Prompt 4: valid query but excessive triples. Prompt 5: regression to hallucination |

---

### 🖼️ Choosing and Sizing Images for the Website

Since this is a student project, we don't yet have our own photos of Villa Carlotta, its art collection, or its
gardens.

**✅ Solution:**
We used placeholder images (from [placehold.co](https://placehold.co/)) throughout the site, with the recommended
pixel size written directly in each image's URL and alt text (e.g. `800x450`). This makes it easy for the team to
swap in real photos later — just replace the `https://placehold.co/...` link with the path to your own image of the
same size.

---

### 🖥️ Technical and Infrastructure Issues

Working with several AI tools and the ArCo SPARQL endpoint at the same time meant we were sometimes affected by
downtimes or slow responses, especially when multiple team members were running queries at once.

**✅ Solution:**
We spread our queries out over time and re-ran any query that timed out or returned an unexpected empty result before
concluding that a gap was real.

---

Each of these challenges helped us gain deeper insight into semantic web technologies, LLM prompting strategies, and
the complexities of enriching digital cultural heritage data.

<hr>
<div class="page-footer-nav">
  <a href="rdf.html">← Previous</a>
  <a href="conclusion.html">Next →</a>
</div>
