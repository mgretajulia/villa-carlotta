---
layout: default
title: Villa Carlotta
---

<div class="site-nav">
  <a href="index.html"> Home</a>
  <a href="topic.html">🏛️ Topic</a>
  <a href="methodology.html">⚒️ Methodology</a>
  <a href="sparql.html">📊 SPARQL & Results</a>
  <a href="gaps.html">🔍 Identifying Gaps</a>
  <a href="rdf.html">🔗 RDF Triples</a>
  <a href="challenges.html">⚠️ Challenges</a>
  <a href="conclusion.html">✅ Conclusion</a>
</div>

# Using LLMs to Verify and Produce New Knowledge


For the three gaps identified on the [Identifying Gaps](gaps.html) page — plus two extra pieces of missing
information about the art collection and the artworks that depict the villa — we asked **four LLMs**:
[Copilot](https://copilot.microsoft.com/), [ChatGPT](https://chat.openai.com/),
[Gemini](https://gemini.google.com/) and [Claude AI](https://claude.ai/), using three prompting techniques:
**zero-shot**, **few-shot** and **chain-of-thought**.

Copilot, ChatGPT and Gemini were asked to propose new RDF predicates and SPARQL `CONSTRUCT` queries for each
missing piece of information (see the [RDF Triples](rdf.html) page for the results). Claude AI was used
differently: as an independent **verification layer**, asked to confirm or fact-check the historical claims
before we turned them into triples.

---

## 1️⃣ Missing Information #1 — The Historical Owners of Villa Carlotta

### Few-shot Prompting Technique

We gave the model the exact IRI of Villa Carlotta and asked it to choose (or propose) the correct predicate from the
ArCo ontology.

> **Prompt:** Could you create an RDF triple for the following sentence: *"Villa Carlotta has been owned by:
> Giorgio Clerici (original builder), Giovanni Battista Sommariva (collector and renovator), Princess Charlotte of
> Prussia (modern namesake)"*?
> Note that: Villa Carlotta's IRI is
> `https://w3id.org/arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03`. For the predicate,
> find the correct predicate in the ArCo ontologies or propose a new one if it is not present. "Villa Carlotta" has
> to be put as a literal.

### [Copilot](https://copilot.microsoft.com/)

![Copilot answer](assets/llm-PROMTS/copilot-1.png)

**Considerations:** Copilot chose the general-purpose predicate `arco-core:description` and combined all three
owners into a single literal sentence. This is safe and valid, but it doesn't model the three owners as separate,
queryable entities.

### [ChatGPT](https://chat.openai.com/) — follow-up zero-shot prompt

After Copilot's answer, we asked ChatGPT a simple zero-shot follow-up: *"can you provide a QUERY?"*

![ChatGPT answer](assets/home/llm-PROMTS/chat-gpt-1.png)

**Considerations:** ChatGPT proposed a brand-new predicate, `ex:historicalOwners`, and built a full `CONSTRUCT`
query around it. This is more descriptive than a plain text blob, but it uses a placeholder `ex:` namespace that
would still need to be mapped onto a real ArCo (or new) ontology term.

### [Gemini](https://gemini.google.com/)

When we ran the generated query on Yasgui and DBpedia, **Gemini's query returned no results**.

### [Claude AI](https://claude.ai/) — verification with a zero-shot prompt

We separately asked Claude a zero-shot prompt: *"Provide a complete historical overview of the owners of Villa
Carlotta (Tremezzo, Lake Como). Include dates of ownership, historical context, and how each owner influenced the
development of the villa."*

**Considerations:** Claude gave a concise, fact-checked historical summary, confirming the same three owners:
**Giorgio Clerici** built the villa (1690–1740), **Giovanni Battista Sommariva** purchased it in 1801 and turned
it into a museum, and **Princess Charlotte of Prussia** received it in 1843, giving the villa its modern name.
Claude did not propose a new RDF predicate, but its independent confirmation gave us confidence in the
`ex:historicalOwners` triple produced by ChatGPT.

---

## 2️⃣ Missing Information #2 — Linking the Art Collection to Villa Carlotta

### Chain-of-Thought Prompting Technique

Villa Carlotta houses neoclassical masterpieces that are historically tied to its old name, "Villa Sommariva". We
asked the LLMs to reason step by step about how to connect them.

> **Prompt:** Create a triple using ArCo ontology that links these artworks to Villa Carlotta: Antonio Canova —
> *Palamede*, *Maddalena Penitente*, *Amore e Psiche* (replica); Bertel Thorvaldsen — *Triumph of Alexander*;
> Francesco Hayez — *Portrait of a Woman*; Pelagio Palagi — neoclassical decorative panels. You should know that
> these artworks are linked to the old name of the villa, "Villa Sommariva". Please also provide the SPARQL query.
> This is the IRI of Villa Carlotta:
> `https://w3id.org/arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03`. Think step by step.

### [Copilot](https://copilot.microsoft.com/)

No results were returned from the generated query.

### [ChatGPT](https://chat.openai.com/)

ChatGPT reused the same approach as for Missing Information #1, proposing a new predicate,
`arco-lite:hasHistoricalCollection`, to link Villa Carlotta to a literal description of the collection.

### [Gemini](https://gemini.google.com/)

Gemini produced very long triples, but again **no results from the tested query**.

### [Claude AI](https://claude.ai/) — verification with a few-shot prompt

We gave Claude two worked examples before asking it for a complete list of
artworks connected to Villa Carlotta through its owners. 
Few Shot Prompting

Examples provided to the model:

Request 1: 
“List artworks owned by Giovanni Battista Sommariva.” 

Reply 1:“ Sommariva owned major neoclassical artworks by Canova, Thorvaldsen, Hayez, Palagi”

Request 2: 
“Explain which artworks from Sommariva’s collection were located in Villa Carlotta.” 

Reply 2:“ Canova, Thorvaldsen, Hayez, Palagi’s artworks were displayed in Villa Sommariva (now Villa Carlotta).”|

Final Answer: “Sommariva owned major neoclassical artworks by Canova, Thorvaldsen, Hayez, Palagi. These artworks were displayed in Villa Sommariva (now Villa Carlotta)”
Task: “Provide a complete list of artworks connected to Villa Carlotta through its owners.*) 

**Considerations:** Claude gave a shorter but highly accurate list, confirming **Antonio Canova** (*Palamede*,
*Maddalena Penitente*, *Amore e Psiche* replica) and **Bertel Thorvaldsen** (*Triumph of Alexander*), alongside
works by **Francesco Hayez** and **Pelagio Palagi** — all collected under the villa's former name, "Villa
Sommariva". As with Missing Information #1, Claude did not generate a SPARQL query, but its answer confirmed that
the facts behind ChatGPT's `arco-lite:hasHistoricalCollection` triple are accurate.

**Considerations:** none of the four LLMs managed to produce triples that linked the *individual* artworks (Canova,
Thorvaldsen, Hayez, Palagi) as separate entities — all of them collapsed the collection into one descriptive object.
This remains an open area for future, more targeted prompting.

---

## 3️⃣ Missing Information #3 — Connecting the Three Names of the Villa

Villa Carlotta has been known under three names over the centuries: **Villa Clerici**, **Villa Sommariva**, and
**Villa Carlotta**. We asked the LLMs to model these as one architectural structure.

> **Prompt:** "Villa Carlotta"'s old names are Villa Clerici and Villa Sommariva. Knowing that Villa Carlotta's IRI is
> `https://w3id.org/arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03`, create a triple based
> on ArCo ontology that links the three names together as referring to the same architectural structure. Generate the
> SPARQL query too.

### [Copilot](https://copilot.microsoft.com/)

![Copilot query](assets/home/llm-PROMPTS/copilot-2.png)

Copilot proposed using `arco-core:hasAlternativeDenomination`, but the resulting query returned **no results** from
either Yasgui or DBpedia.

### [ChatGPT](https://chat.openai.com/)

![ChatGPT query](assets/home/llm-PROMPTS/chat-gpt-2.png)

ChatGPT took a different, more structured approach: it created **three separate "Title" resources** — one for each
name — and linked Villa Carlotta to each of them with `a-cd:hasTitle`, while giving each Title resource a literal
name via `l0:name`. This query **successfully produced 6 triples** (see the [RDF Triples](rdf.html) page for the
full result).

### [Gemini](https://gemini.google.com/)

No results from the provided query.

### [Claude AI](https://claude.ai/) — verification with a chain-of-thought prompt

We separately asked Claude to *"reason step-by-step: are Villa Clerici, Villa Sommariva, and Villa Carlotta
historically the same building? Provide historical evidence, ownership transitions, and naming chronology."*

**Considerations:** Claude gave a cautious, fact-checked confirmation that all three names refer to the same
architectural structure — **Villa Clerici** (original 17th-century name), **Villa Sommariva** (renamed under
Sommariva's ownership), and **Villa Carlotta** (modern name after Princess Charlotte). This independent
confirmation reassured us that ChatGPT's "Title resource" triples below were historically accurate, not just
syntactically valid.

**Considerations:** ChatGPT's "Title resource" pattern was the only one of the four that produced a working,
queryable result — and it nicely mirrors how ArCo already models alternative titles for other cultural properties.

---

## 4️⃣ Missing Information #4 — Linking Artworks that Depict Villa Carlotta

Several historical artworks depict the villa itself (rather than being part of its indoor collection). We asked the
LLMs to connect these depictions to the villa.

> **Prompt:** The following artworks depicting Villa Carlotta are missing from ArCo. We want to create RDF triples
> based on ArCo ontologies in order to relate these works to the villa:
> - 19th-century landscape paintings of Lake Como featuring the villa, by **Giuseppe Bisi**
> - Engravings of Villa Sommariva from travel books, by **Ludwig Bechstein**
> - Sketches made by visiting artists: **Jean-Joseph-Xavier Bidauld**, **Giuseppe Bisi**, and **Alessandro Sanquirico**
>   (two sketches)
>
> Always provide the SPARQL `CONSTRUCT` query. Let's think step by step.

### [Copilot](https://copilot.microsoft.com/)

![Copilot Turtle](assets/home/llm-PROMPTS/copilot-3.png)

Copilot generated a `CONSTRUCT` query that links six new artwork IRIs to Villa Carlotta via `arco-dd:hasSubject`.

### [ChatGPT](https://chat.openai.com/)

![ChatGPT query 1](assets/home/llm-PROMPTS/chat-gpt-3.png)

ChatGPT proposed its own set of "CulturalProperty" IRIs (one per artwork/sketch group) and linked each of them to
Villa Carlotta with `a-cd:hasSubject`. It then offered a **second** query that instead adds one combined
`l0:description` literal to Villa Carlotta, summarizing all of these depictions in a single sentence.

### [Gemini](https://gemini.google.com/)

![Gemini result](assets/home/llm-PROMPTS/gemini-3.png)

Gemini's answer was by far the **richest and most structured**: it created a separate entity for each artwork/sketch
group (e.g. `BisiLandscapePaintings`, `BechsteinEngravings`, `BidauldSketches`, `BisiSketches`, `SanquiricoSketches`),
a separate `Agent` entity for each author (Giuseppe Bisi, Ludwig Bechstein, Jean-Joseph-Xavier Bidauld, Alessandro
Sanquirico), and connected everything with `rdfs:label`, `arco-cd:hasAuthor`, `rdf:type`, and `arco-cd:hasSubject`.

**Considerations:** for this task, Gemini's structured, multi-entity modeling was clearly the most useful starting
point for real enrichment — see the full triple list on the [RDF Triples](rdf.html) page.

---

## 5️⃣ Missing Information #5 — The Botanical Gardens

> **Prompt:** The Botanical Gardens of Villa Carlotta have the following information, which is missing from ArCo:
> garden size — 8 hectares; over 150 species of rhododendrons; over 40 species of azaleas; ancient sequoias,
> camellias, and a bamboo grove; garden developed during the 19th century. Help us create one RDF triple which
> contains all of this information based on ArCo ontologies, and provide the SPARQL query.

### [Copilot](https://copilot.microsoft.com/)

![Copilot garden triple](https://placehold.co/700x400?text=Copilot+garden+triple+%28700x400%29)

Copilot combined all the garden facts into a single literal and attached it to Villa Carlotta with
`arco-core:description`, using a `CONSTRUCT` query with `BIND(true AS ?x)`.

### [ChatGPT](https://chat.openai.com/)

![ChatGPT garden triple](https://placehold.co/700x400?text=ChatGPT+garden+triple+%28700x400%29)

ChatGPT took the same "single descriptive literal" approach, but used the predicate `l0:description` instead.

### [Gemini](https://gemini.google.com/)

Gemini only produced a `SELECT`-style query asking ArCo for an existing `arco-core:description` of the gardens —
which, as expected, returned **no results** from either Yasgui or DBpedia.

---

## 📋 Summary: What All Four LLMs Found (Missing from ArCo)

| Category | Present in ArCo? | Found via LLMs? |
|---|---|---|
| Owners of the villa | No | Yes |
| Artworks owned by the villa's owners | No | Yes |
| Confirmation that the 3 names = 1 villa | No | Yes |
| Artworks related to the villa | No | Yes |
| Botanical gardens | No | Yes |

## 📊 General Considerations

- **Copilot** and **ChatGPT** both tend to fall back on a single descriptive literal (`arco-core:description` /
  `l0:description`) whenever the missing information is a block of free text, such as the owners list or the garden
  description.
- **ChatGPT** was the only model that consistently proposed *new, more granular* predicates (e.g.
  `ex:historicalOwners`, `arco-lite:hasHistoricalCollection`) and the "Title resource" pattern for Missing
  Information #3.
- **Gemini** struggled when asked to retrieve information that genuinely does not exist in ArCo (returning "no
  results" three times out of five), but produced by far the **most detailed and structured** triples for Missing
  Information #4, where multiple new entities needed to be created.
- **Claude AI** was used as an independent verification layer rather than a triple-generator: across zero-shot,
  few-shot and chain-of-thought prompts, it consistently gave concise, fact-checked confirmations of the owners,
  the art collection, and the equivalence of the villa's three names — without proposing new RDF predicates or
  SPARQL queries.
- Across all five tasks, **few-shot** and **chain-of-thought** prompts consistently produced more structured and
  reusable answers than plain **zero-shot** prompts.
- For a closer look at *why* Copilot and Gemini struggled in the first place, see the
  [Shortcomings of Each LLM](challenges.html#shortcomings-of-each-llm) section on the Challenges page.

<hr>
<div class="page-footer-nav">
  <a href="gaps.html">← Previous</a>
  <a href="rdf.html">Next →</a>
</div>
