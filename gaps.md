---
layout: default
title: Villa Carlotta
---

<div class="site-nav">
  <a href="index.html">⭐ Home</a>
  <a href="topic.html">⭐ Topic</a>
  <a href="methodology.html">⭐ Methodology</a>
  <a href="sparql.html">⭐ SPARQL & Results</a>
  <a href="prompts.html">⭐ LLM Prompts</a>
  <a href="rdf.html">⭐ RDF Triples</a>
  <a href="challenges.html">⭐ Challenges</a>
  <a href="conclusion.html">⭐ Conclusion</a>
</div>

# How We Identified the Information Gaps

Our queries on the [SPARQL & Results](sparql.html) page confirmed that
[**Villa Carlotta**](https://w3id.org/arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03)
(`CO160-00021_R03`) is represented in ArCo as an `ArchitecturalOrLandscapeHeritage`. We then ran a series of `ASK`
queries to confirm — with a simple **TRUE/FALSE** answer — three specific gaps in this record.

## Phase 1 — The Committent / Patronage Gap

Historically, a grand estate like Villa Carlotta didn't just appear: it was shaped by powerful patrons. It was

- **originally commissioned by the Marquis Giorgio Clerici** in the late 1600s,
- **drastically enriched by the art collector Giovanni Battista Sommariva** in the early 1800s, and
- **later gifted to Princess Charlotte of Prussia** (hence its modern name, "Carlotta").

In a complete knowledge graph, these historical figures should be explicitly linked to the property IRI as creators
or patrons. We wanted to check whether ArCo is missing these connections.

### Step A — The Discovery Query

First, we inspected one of our Villa Carlotta property IRIs from Query 1 to see which predicates it currently uses to
describe its history or origin. We used `ORDER BY` to sort the predicates alphabetically, so we could scan the list
with **CTRL+F** for keywords such as "patron", "committent", "Clerici" and "Sommariva".

```sparql
PREFIX arco: <https://w3id.org/arco/ontology/arco/>

SELECT DISTINCT ?predicate ?object
WHERE {
  # one of our verified Villa Carlotta property IRIs from Query 1
  <https://w3id.org/arco/resource/HistoricOrArtisticProperty/0300053557> ?predicate ?object .
}
ORDER BY ?predicate
```

### Result

![Discovery query result](https://placehold.co/800x400?text=Discovery+query+result+%28800x400%29)

We got a long list of predicates and objects, but found **no information connecting this property to Villa Carlotta
or to its patrons** (Clerici, Sommariva).

### Step B — The ASK Validation Query

Based on this manual scan, we ran an exact `ASK` query to confirm the gap across the villa's primary identity, testing
the predicate `a-cd:hasCommittent`.

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX a-cd: <https://w3id.org/arco/ontology/context-description/>

ASK WHERE {
  ?culturalProperty a <https://w3id.org/arco/ontology/arco/HistoricOrArtisticProperty> ;
                    rdfs:label "Villa Carlotta"@it ;
                    a-cd:hasCommittent ?committent .
}
```

### Result

<div class="callout">
<strong>FALSE</strong> — we have officially identified <strong>Gap #1: Missing Patronage Data</strong>.
</div>

We will need to use an LLM to extract the historical records of Clerici and Sommariva so we can prepare our
enrichment triples (see the [LLM Prompts](prompts.html) and [RDF Triples](rdf.html) pages).

---

## Phase 2 — The Art Collection / Component Gap

Villa Carlotta is not just an architectural shell — it is internationally famous as a museum housing elite
neoclassical masterworks, most notably original sculptures by **Antonio Canova** (such as *Palamede*) and
**Bertel Thorvaldsen**.

In the ArCo ontology, structural pieces or physical collections inside a building are often connected using
"containment" predicates. If these famous art pieces exist in ArCo but aren't explicitly declared as components of
the villa, a semantic user has no way to ask: *"What art pieces are inside Villa Carlotta?"*

### Step A — The Benchmarking Query

To figure out how ArCo connects indoor artwork collections to a main building, we searched ArCo for assets explicitly
labeled "Canova" and looked for predicates related to containment or location.

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX arco: <https://w3id.org/arco/ontology/arco/>

SELECT DISTINCT ?artwork ?predicate ?palace
WHERE {
  ?artwork a arco:HistoricOrArtisticProperty ;
           rdfs:label ?label ;
           ?predicate ?palace .

  FILTER(REGEX(?label, "Canova", "i"))
  FILTER(REGEX(STR(?predicate), "component|location|site|holding", "i"))
}
LIMIT 20
```

### Result

![Benchmarking query result](https://placehold.co/800x400?text=Benchmarking+query+result+%28800x400%29)

We discovered that ArCo has **no single, dedicated predicate** for "housing" museum items. Instead it uses several
predicates such as `arco:isCulturalPropertyComponentOf`, `hasCulturalPropertyAddress`, and
`hasTimeIndexedTypedLocation`.

### Step B — The ASK Validation Query for Villa Carlotta

We chose `arco:isCulturalPropertyComponentOf` for our investigation. We then verified whether Villa Carlotta's main
IRI is missing incoming connections from its famous resident artworks.

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX arco: <https://w3id.org/arco/ontology/arco/>

ASK WHERE {
  ?artwork a arco:HistoricOrArtisticProperty ;
           rdfs:label ?artworkLabel .

  # Checking if any artwork claims to be a component of Villa Carlotta
  ?artwork arco:isCulturalPropertyComponentOf <https://dati.cultura.gov.it/lodview-arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03> .

  FILTER(REGEX(?artworkLabel, "Canova", "i"))
}
```

### Result

<div class="callout">
<strong>FALSE</strong> — we have identified <strong>Gap #2: Disconnected Art Collection</strong>.
</div>

This means the graph treats the villa and its famous artworks as completely isolated entities, which we will need to
semantically bridge.

---

## Phase 3 — The Botanical Garden / Landscape Gap

Our exploratory results in [Query 5](sparql.html#query6) briefly surfaced a mention of *"Giardini"* (Gardens).
Villa Carlotta is renowned for its **8-hectare botanical garden**, featuring large collections of azaleas,
rhododendrons and rare trees. In cultural heritage ontologies, landscapes and historic gardens have specific
architectural relationships to the primary villa complex.

### The Verification Query

We checked whether the main Villa Carlotta property node is explicitly linked to a botanical garden asset, or
whether the garden is just floating as a separate text label.

```sparql
PREFIX arco: <https://w3id.org/arco/ontology/arco/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

ASK WHERE {
  <https://dati.cultura.gov.it/lodview-arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03> ?relation ?garden .
  ?garden rdfs:label ?gardenLabel .
  FILTER(REGEX(?gardenLabel, "giardino|giardini", "i"))
}
```

### Result

<div class="callout">
<strong>FALSE</strong> — we have verified <strong>Gap #3: Missing Landscape/Garden Linkage</strong>.
</div>

The complex spatial relationship between the historical palace and its landscape architecture is completely
unrepresented in ArCo.

---

## Conclusion

Our investigation shows that
**[Villa Carlotta](https://dati.cultura.gov.it/lodview-arco/resource/Lombardia/ArchitecturalOrLandscapeHeritage/CO160-00021_R03)**
is indeed represented in the Italian Ministry of Culture's official RDF dataset and ontology network — but:

- there are **no links to its historical owners** (Clerici, Sommariva, Princess Charlotte of Prussia),
- there are **no links to its art collection** (Canova, Thorvaldsen and others), and
- there is **no connection between its botanical garden and the main property**.

We also confirmed there is currently **no connection between the modern name "Villa Carlotta" and its historical name
"Villa Sommariva"**.

At the next stage, we used LLMs to verify this data and to draft RDF triples that could close these gaps — see the
[LLM Prompts](prompts.html) and [RDF Triples](rdf.html) pages.

<hr>
<div class="page-footer-nav">
  <a href="sparql.html">← Previous</a>
  <a href="prompts.html">Next →</a>
</div>
