---
layout: default
title: Villa Carlotta
---

<div class="site-nav">
  <a href="index.html">🏠 Home</a>
  <a href="topic.html">🏛️ Topic</a>
  <a href="methodology.html">⚒️ Methodology</a>
  <a href="sparql.html">📊 SPARQL & Results</a>
  <a href="gaps.html">🔍 Identifying Gaps</a>
  <a href="prompts.html">💬 LLM Prompts</a>
  <a href="challenges.html">⚠️ Challenges</a>
  <a href="conclusion.html">✅ Conclusion</a>
</div>

# Generating Our New RDF Triples

For all five pieces of missing information described on the [LLM Prompts](prompts.html) page, our main subject is
Villa Carlotta's IRI:

```
https://w3id.org/arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03
```

## 1. Triple Set — Historical Owners

[Copilot](https://copilot.microsoft.com/) suggested a simple descriptive triple:

```turtle
PREFIX arco-core: <https://w3id.org/arco/ontology/core/>

<https://w3id.org/arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03>
    arco-core:description
    "Villa Carlotta has been owned by Giorgio Clerici (original builder), Giovanni Battista Sommariva (collector and renovator), and Princess Charlotte of Prussia (modern namesake)." .
```

[ChatGPT](https://chat.openai.com/) instead proposed a new, dedicated predicate and the following `CONSTRUCT` query:

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX ex:   <https://example.org/ontology/>

CONSTRUCT {
  ?villa ex:historicalOwners ?owners .
}
WHERE {
  VALUES (?villa ?owners) {
    (
      <https://w3id.org/arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03>
      "Giorgio Clerici (original builder); Giovanni Battista Sommariva (collector and renovator); Princess Charlotte of Prussia (modern namesake)"@en
    )
  }
}
```

**Resulting triple:**

| Subject | Predicate | Object |
|---|---|---|
| `CO160-00021_R03` | `ex:historicalOwners` | "Giorgio Clerici (original builder); Giovanni Battista Sommariva (collector and renovator); Princess Charlotte of Prussia (modern namesake)"@en |

---

## 2. Triple Set — Art Collection Linked via "Villa Sommariva"

None of the three LLMs produced a triple that links the **individual** artworks (Canova's *Palamede*, Thorvaldsen's
*Triumph of Alexander*, etc.) as separate entities. The only usable output, from ChatGPT, reused the same
"description" pattern with a new predicate:

```turtle
<https://w3id.org/arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03>
    arco-lite:hasHistoricalCollection
    "Antonio Canova: Palamede, Maddalena Penitente, Amore e Psiche (replica); Bertel Thorvaldsen: Triumph of Alexander; Francesco Hayez: Portrait of a Woman; Pelagio Palagi: neoclassical decorative panels — collected under the villa's former name, Villa Sommariva." .
```

<div class="callout">
This triple set is the weakest of the five and is flagged as <strong>future work</strong>: a better model would give
each artwork its own IRI and link it to Villa Carlotta with <code>arco:isCulturalPropertyComponentOf</code> (see
Gap #2 on the <a href="gaps.html">Identifying Gaps</a> page).
</div>

---

## 3. Triple Set — The Three Names of the Villa

This was our most successful enrichment. [ChatGPT](https://chat.openai.com/) modeled each historical name as its own
**Title** resource, linked back to Villa Carlotta:

```sparql
PREFIX a-cd: <https://w3id.org/arco/ontology/context-description/>
PREFIX l0: <https://w3id.org/italia/onto/l0/>

CONSTRUCT {
  <https://w3id.org/arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03>
      a-cd:hasTitle ?title .

  ?title l0:name ?name .
}
WHERE {
  VALUES (?title ?name) {
    (<https://w3id.org/arco/resource/Title/VillaCarlotta>  "Villa Carlotta")
    (<https://w3id.org/arco/resource/Title/VillaClerici>   "Villa Clerici")
    (<https://w3id.org/arco/resource/Title/VillaSommariva> "Villa Sommariva")
  }
}
```

**Resulting triples (6):**

| # | Subject | Predicate | Object |
|---|---|---|---|
| 1 | `CO160-00021_R03` | `a-cd:hasTitle` | `Title/VillaCarlotta` |
| 2 | `CO160-00021_R03` | `a-cd:hasTitle` | `Title/VillaClerici` |
| 3 | `CO160-00021_R03` | `a-cd:hasTitle` | `Title/VillaSommariva` |
| 4 | `Title/VillaCarlotta` | `l0:name` | "Villa Carlotta" |
| 5 | `Title/VillaClerici` | `l0:name` | "Villa Clerici" |
| 6 | `Title/VillaSommariva` | `l0:name` | "Villa Sommariva" |

This directly closes the gap identified in [Query 4](sparql.html) (the missing "Villa Clerici" entity) and links it
to the cross-over entity found in our [Breakthrough Analysis](sparql.html).

---

## 4. Triple Set — Artworks That Depict Villa Carlotta

### Copilot's version

[Copilot](https://copilot.microsoft.com/) linked six new artwork IRIs to Villa Carlotta using `arco-dd:hasSubject`:

```sparql
PREFIX arco-dd: <https://w3id.org/arco/ontology/denotative-description/>

CONSTRUCT {
  ?artwork arco-dd:hasSubject
           <https://w3id.org/arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03> .
}
WHERE {
  VALUES ?artwork {
    <https://w3id.org/arco/resource/MovableProperty/GiuseppeBisi_LakeComoLandscape_VillaCarlotta>
    <https://w3id.org/arco/resource/MovableProperty/LudwigBechstein_Engraving_VillaSommariva>
    <https://w3id.org/arco/resource/MovableProperty/Sketch_Bidauld_VillaCarlotta>
    <https://w3id.org/arco/resource/MovableProperty/Sketch_GiuseppeBisi_VillaCarlotta>
    <https://w3id.org/arco/resource/MovableProperty/Sketch_Sanquirico_VillaCarlotta_1>
    <https://w3id.org/arco/resource/MovableProperty/Sketch_Sanquirico_VillaCarlotta_2>
  }
}
```

| # | Subject | Predicate | Object |
|---|---|---|---|
| 1 | `GiuseppeBisi_LakeComoLandscape_VillaCarlotta` | `arco-dd:hasSubject` | `CO160-00021_R03` |
| 2 | `LudwigBechstein_Engraving_VillaSommariva` | `arco-dd:hasSubject` | `CO160-00021_R03` |
| 3 | `Sketch_Bidauld_VillaCarlotta` | `arco-dd:hasSubject` | `CO160-00021_R03` |
| 4 | `Sketch_GiuseppeBisi_VillaCarlotta` | `arco-dd:hasSubject` | `CO160-00021_R03` |
| 5 | `Sketch_Sanquirico_VillaCarlotta_1` | `arco-dd:hasSubject` | `CO160-00021_R03` |
| 6 | `Sketch_Sanquirico_VillaCarlotta_2` | `arco-dd:hasSubject` | `CO160-00021_R03` |

### ChatGPT's version

[ChatGPT](https://chat.openai.com/) proposed its own set of `CulturalProperty` IRIs (one per artwork/sketch group)
and linked each of them to Villa Carlotta with `a-cd:hasSubject`:

```sparql
PREFIX a-cd: <https://w3id.org/arco/ontology/context-description/>

CONSTRUCT {
  ?artwork a-cd:hasSubject
      <https://w3id.org/arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03> .
}
WHERE {
  VALUES ?artwork {
    <https://w3id.org/arco/resource/CulturalProperty/LakeComoLandscapePainting>
    <https://w3id.org/arco/resource/CulturalProperty/VillaSommarivaEngraving>
    <https://w3id.org/arco/resource/CulturalProperty/SketchByBidauld>
    <https://w3id.org/arco/resource/CulturalProperty/SketchByBisi>
    <https://w3id.org/arco/resource/CulturalProperty/SketchBySanquirico>
  }
}
```

| # | Subject | Predicate | Object |
|---|---|---|---|
| 1 | `LakeComoLandscapePainting` | `a-cd:hasSubject` | `CO160-00021_R03` |
| 2 | `VillaSommarivaEngraving` | `a-cd:hasSubject` | `CO160-00021_R03` |
| 3 | `SketchByBidauld` | `a-cd:hasSubject` | `CO160-00021_R03` |
| 4 | `SketchByBisi` | `a-cd:hasSubject` | `CO160-00021_R03` |
| 5 | `SketchBySanquirico` | `a-cd:hasSubject` | `CO160-00021_R03` |

ChatGPT also offered a second, simpler query that instead adds **one combined `l0:description` literal** to Villa
Carlotta, summarizing all of these depictions in a single sentence:

```sparql
PREFIX l0: <https://w3id.org/italia/onto/l0/>

CONSTRUCT {
  <https://w3id.org/arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03>
      l0:description ?text .
}
WHERE {
  VALUES ?text {
    "Depicted in 19th-century Lake Como landscape paintings by Giuseppe Bisi, engravings of Villa Sommariva from travel books by Ludwig Bechstein, and sketches by Jean-Joseph-Xavier Bidauld, Giuseppe Bisi and Alessandro Sanquirico."
  }
}
```

### Gemini's version — the most detailed result

[Gemini](https://gemini.google.com/) modeled each artwork/sketch *group* and each *author* as its own entity:

| # | Subject | Predicate | Object |
|---|---|---|---|
| 1 | `BisiLandscapePaintings` | `rdfs:label` | "19th-century landscape paintings of Lake Como featuring Villa Carlotta" |
| 2 | `AlessandroSanquirico` | `rdfs:label` | "Alessandro Sanquirico" |
| 3 | `BechsteinEngravings` | `rdfs:label` | "Engravings of Villa Sommariva from travel books" |
| 4 | `GiuseppeBisi` | `rdfs:label` | "Giuseppe Bisi" |
| 5 | `SanquiricoSketches` | `arco-cd:hasAuthor` | `AlessandroSanquirico` |
| 6 | `BisiLandscapePaintings` | `arco-cd:hasAuthor` | `GiuseppeBisi` |
| 7 | `BisiSketches` | `arco-cd:hasAuthor` | `GiuseppeBisi` |
| 8 | `BidauldSketches` | `arco-cd:hasAuthor` | `JeanJosephXavierBidauld` |
| 9 | `BechsteinEngravings` | `arco-cd:hasAuthor` | `LudwigBechstein` |
| 10 | `BisiLandscapePaintings` | `rdf:type` | `arco-core:CulturalProperty` |
| 11 | `BechsteinEngravings` | `rdf:type` | `arco-core:CulturalProperty` |
| 12 | `BidauldSketches` | `rdf:type` | `arco-core:CulturalProperty` |
| 13 | `BisiSketches` | `rdf:type` | `arco-core:CulturalProperty` |
| 14 | `SanquiricoSketches` | `rdf:type` | `arco-core:CulturalProperty` |
| 15 | `BisiLandscapePaintings` | `arco-cd:hasSubject` | `CO160-00021_R03` |
| 16 | `BechsteinEngravings` | `arco-cd:hasSubject` | `CO160-00021_R03` |
| 17 | `BidauldSketches` | `arco-cd:hasSubject` | `CO160-00021_R03` |
| 18 | `BisiSketches` | `arco-cd:hasSubject` | `CO160-00021_R03` |
| 19 | `SanquiricoSketches` | `arco-cd:hasSubject` | `CO160-00021_R03` |
| 20 | `JeanJosephXavierBidauld` | `rdfs:label` | "Jean-Joseph-Xavier Bidauld" |
| 21 | `LudwigBechstein` | `rdfs:label` | "Ludwig Bechstein" |
| 22 | `SanquiricoSketches` | `rdfs:label` | "Sketches made by Alessandro Sanquirico" |
| 23 | `BisiSketches` | `rdfs:label` | "Sketches made by Giuseppe Bisi" |
| 24 | `BidauldSketches` | `rdfs:label` | "Sketches made by Jean-Joseph-Xavier Bidauld" |

---

## 5. Triple Set — Botanical Gardens

[Copilot](https://copilot.microsoft.com/) attached one combined description to Villa Carlotta:

```sparql
PREFIX arco-core: <https://w3id.org/arco/ontology/core/>

CONSTRUCT {
  <https://w3id.org/arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03>
      arco-core:description
      "Botanical gardens of Villa Carlotta: 8 hectares in size, containing over 150 species of rhododendrons, over 40 species of azaleas, ancient sequoias, camellias, a bamboo grove, and developed extensively during the 19th century." .
}
WHERE {
  BIND(true AS ?x)
}
```

[ChatGPT](https://chat.openai.com/) produced an equivalent triple using `l0:description` instead:

```sparql
PREFIX l0: <https://w3id.org/italia/onto/l0/>

CONSTRUCT {
  <https://w3id.org/arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03>
      l0:description ?description .
}
WHERE {
  VALUES ?description {
    "The Botanical Gardens of Villa Carlotta cover 8 hectares and include over 150 species of rhododendrons, over 40 species of azaleas, ancient sequoias, camellias, and a bamboo grove. The gardens were developed during the 19th century."
  }
}
```

| # | Subject | Predicate | Object |
|---|---|---|---|
| 1 | `CO160-00021_R03` | `arco-core:description` (Copilot) | "Botanical gardens of Villa Carlotta: 8 hectares in size, containing over 150 species of rhododendrons, over 40 species of azaleas, ancient sequoias, camellias, a bamboo grove, and developed extensively during the 19th century." |
| 2 | `CO160-00021_R03` | `l0:description` (ChatGPT) | "The Botanical Gardens of Villa Carlotta cover 8 hectares and include over 150 species of rhododendrons, over 40 species of azaleas, ancient sequoias, camellias, and a bamboo grove. The gardens were developed during the 19th century." |

[Gemini](https://gemini.google.com/) returned **no results** for this missing information.

---

## 🔗 Final Triples Summary

| Gap | Best predicate found | Status |
|---|---|---|
| Historical owners | `ex:historicalOwners` (proposed) | ✅ Usable triple produced |
| Art collection (Canova, Thorvaldsen...) | `arco-lite:hasHistoricalCollection` (proposed) | ⚠️ Weak — needs per-artwork IRIs |
| Three names (Carlotta / Clerici / Sommariva) | `a-cd:hasTitle` + `l0:name` | ✅ 6 triples produced |
| Depicting artworks | `arco-cd:hasSubject` / `arco-cd:hasAuthor` | ✅ 24 triples produced (Gemini) |
| Botanical gardens | `arco-core:description` / `l0:description` | ✅ Usable triple produced |

<hr>
<div class="page-footer-nav">
  <a href="prompts.html">← Previous</a>
  <a href="challenges.html">Next →</a>
</div>
