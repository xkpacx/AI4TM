# Building and querying a knowledge graph

This is the text companion to the knowledge graph pipeline notebook. **Time**: about 45-60 minutes, most of it spent waiting on model calls. **Cost**: a few cents at most on Gemini's cheapest models — the notebook makes on the order of 15-20 small calls total across extraction, entity resolution, and querying.

This week's overview described the shape of the problem: a question that crosses two departments usually has an answer that already exists, scattered across documents that were never written to talk to each other. This piece works through that scenario in miniature, built around a fictional outdoor-gear company, **Fernwood Outfitters**, invented specifically for this exercise so there's real content to build a graph from without touching anything that needs protecting.

---

## The scenario

Ten short documents, four types, all invented. Two product pages (a **TrailRunner 32L Backpack** and an **Alpine Rain Shell**), a returns-and-shipping policy, a customer-segments reference, two sets of meeting notes, and four support tickets. The tickets describe real customers running into two different problems on the same product: a **zipper failure** (a manufacturing defect) and a **shipping delay** tied to a promotional campaign called the Summer Trail Sale. Fernwood's own policy treats those two problems completely differently — a manufacturing defect earns a full refund and a free return, a late shipment earns a 20% refund or a free return, and a discount at checkout doesn't change either remedy.

No single document answers the question a support lead actually needs answered: which customers who complained about the TrailRunner backpack during the Summer Trail Sale are entitled to what, under which policy clause, and did the campaign's own messaging make things worse? The product page doesn't know about tickets. The policy doesn't know which customers it applies to. The meeting notes don't know the shipping SLA was breached until a person reads a ticket and a policy side by side. That gap between documents is exactly what a knowledge graph closes, not by adding new information, but by making the relationships between documents into something a query can walk across.

---

## Extracting entities and relationships

A knowledge graph stores **nodes** (entities — a customer, a product, a policy clause) and **edges** (typed, directed relationships between them — a customer *files* a ticket, a ticket is *about* a product). Turning a paragraph of prose into that structure is exactly the kind of task a language model is suited for: read the text, decide what the entities are, decide how they relate, and return the result as structured data instead of another paragraph of prose.

Left completely unconstrained, this drifts. One document produces a relationship type called `RELATES_TO`, another produces `IS_CONNECTED_WITH`, and the result technically has edges but no two of them are reliably the same kind of edge. The fix is the same one a database designer would reach for: define the schema first, and instruct the model to extract into it rather than invent its own. This week's exercise fixes eight node types (customer, product, ticket, campaign, segment, issue type, policy clause, company) and nine relationship types, each with a defined direction — a customer files a ticket, a ticket is about a product, a policy clause governs an issue type, and so on.

This is a real design tradeoff, not a formality. A fixed schema keeps the graph queryable and consistent, but it means the model can only extract what the schema anticipated — a relationship type nobody thought to define gets forced into the nearest existing one, or dropped entirely. Production knowledge-graph pipelines usually start narrower than they'd like and widen the schema deliberately as new document types show up, rather than letting the model freelance from the start.

---

## Merging the duplicates it produces

Extraction run separately over ten documents produces the same real-world entity under several different names: the product page calls it the **TrailRunner 32L Backpack**, one ticket calls it the **TrailRunner backpack**, and one customer, Maria Ibarra, shows up as **M. Ibarra** in her follow-up ticket. Left unmerged, the graph looks complete while missing exactly the connections a real question would need — a query for "everyone who filed a ticket about the TrailRunner 32L Backpack" would miss the ticket that used the shorter name, because nothing connects the two strings as the same node.

The course notebook handles this in two passes, cheapest first. A **normalization** pass lowercases, strips punctuation, and expands a small set of known abbreviations — catching "M. Ibarra" only because the alias is written down in advance, which is exactly the case a hand-written rule can't generalize beyond. Then an **embedding similarity** pass compares entity names as vectors, merging any two names of the same node type whose vectors sit close enough together — catching "TrailRunner backpack" and "TrailRunner 32L Backpack," which share no normalized string in common but mean the same product. Restricting the comparison to entities of the same type is what keeps a product from ever merging into a segment just because the phrasing happens to look similar.

Neither pass is exact. A normalization rule can merge two genuinely different customers who happen to share a nickname; a similarity threshold set too loose merges near-misses that shouldn't merge, and set too tight leaves real duplicates apart. Production systems usually add a third pass — a person reviewing the merge list before it's treated as final — which is part of what the overview meant by the human cost of keeping a graph's structure honest.

---

## Loading and querying in Cypher

**Cypher** is Neo4j's query language, the graph equivalent of SQL, and its core statement for loading data is `MERGE` rather than `CREATE`. `CREATE` always adds a new node or edge, even if an identical one already exists — run a `CREATE`-based load twice and the graph doubles. `MERGE` checks whether a matching node or edge already exists and only creates it if it doesn't, which makes the load safe to re-run without duplicating anything, useful the first time a bug in the extraction step gets fixed and the graph needs reloading.

The notebook works through three queries, increasing in how many hops (relationship traversals) each needs. One hop finds every ticket about a specific product. Two hops finds every customer who filed one of those tickets. The third query is the actual payoff, and it needs three relationship types sourced from three different original documents in a single traversal — customer to ticket, ticket to product, ticket to issue type, and issue type to the policy clause that governs it:

> Running that query over this week's ten documents returns Maria Ibarra's ticket with issue type "manufacturing defect," governed by the policy's manufacturing-defects clause, and David Chen's ticket, on the same product, with issue type "shipping delay," governed by the shipping SLA clause instead. Same product, two different remedies, and no single document in the original set states that mapping for these two specific customers.

---

## What a graph answers that a table struggles with, and where a table still wins

That payoff query is the case for a graph: the question needed a variable, unknown-in-advance number of hops across data that started in four unrelated documents. A relational table answers a fixed-shape question fast because someone decided the shape, the columns, in advance. A graph answers a question shaped like "follow this relationship, then whatever it's connected to, then whatever that's connected to" without anyone having pre-decided how many hops the question would need — which is exactly what this week's question needed, and would have required a new table or a new join every time the question's shape changed even slightly.

That advantage doesn't hold in the other direction. A question like "what's the average order value for the Weekend Hikers segment this quarter" is an aggregation over many rows of the same shape, a `GROUP BY` in SQL or its Cypher equivalent, and a warehouse table built for that column answers it faster and more cheaply than a graph traversal would, because the graph has to walk relationships to reach data a table would already have sitting in a single indexed column. The honest framing from this week's overview holds here directly: decide which structure fits by the shape of the query actually needed, not by which technology is newer. A team that puts every table into a graph "for flexibility" pays a real cost in query speed and infrastructure for questions a table would have answered directly.

---

## GraphRAG: using the graph as a model's source of facts

**RAG** (retrieval-augmented generation) is the general pattern behind this: before asking a model to answer, retrieve the specific information the answer needs and hand it to the model as context, rather than trusting whatever the model already carries from training. The retrieval step is usually a similarity search over document chunks. **GraphRAG** swaps that retrieval step for a graph traversal — instead of handing the model a paragraph that might contain the answer, hand it the actual subgraph, the specific nodes and relationships, the question depends on.

The difference matters most exactly where the earlier section did: a question that needs several connected facts assembled from different places. A chunk-similarity search retrieves whichever passages read as topically similar to the question, which answers *what a passage is about*, not *which entities connect to which* — a graph traversal retrieves the second thing directly.

The course notebook implements this by asking the model to write the Cypher query itself, given the schema — a real, named technique usually called **text-to-Cypher**. That's a genuinely useful pattern and one worth treating carefully: a model-written query that runs unchecked against a live database is a real risk if the query could modify data, so the notebook's retrieval function refuses to execute anything containing a write keyword (`CREATE`, `MERGE`, `DELETE`, `SET`, `REMOVE`) before it ever reaches the database. This is the same instinct Session 3 covered under a different name — validate anything a model produces before it gets to act, rather than trusting output because it happens to be syntactically well-formed.

---

## Checking groundedness

An answer that reads confidently is not the same as an answer actually supported by what was retrieved. The point of GraphRAG is that the model should only be asserting what the retrieved subgraph contains, so a useful check, even a rough one, is whether the entity names in the answer actually appear among the entity names retrieved. This won't catch every kind of error — a model can misstate a *relationship* between two entities that were both genuinely retrieved — but it catches the most damaging failure: an answer that names a customer, product, or policy clause that was never in the retrieved subgraph at all, which means the model filled a gap from training data or invention rather than from the graph itself.

Worth testing directly rather than assuming it holds: what happens when the graph genuinely can't answer a question. Asked about Fernwood's revenue this quarter, a fact with no node anywhere in this schema, the model should say so plainly rather than guessing at a number. That's the behavior the notebook's answer prompt is written to push toward, and confirming it actually holds, rather than assuming a prompt worked because it reads reasonably, is part of the exercise.

---

## Does this actually improve on the alternatives?

Groundedness checking works without knowing the real answer in advance — it only asks whether the model stayed inside what it was given. The course notebook goes a step further, asking a stronger question that's only answerable here because Fernwood Outfitters was invented for this exercise and the correct answer is fully known in advance: does GraphRAG produce a more correct answer than the alternatives actually would have?

Three approaches, run on the same question: no retrieval at all (the model answers from nothing but the question itself), standard vector similarity RAG (embed every document, embed the question, retrieve the top few most similar documents, hand them to the model as context), and the GraphRAG traversal built earlier. The question needs facts from two different tickets and the policy document at once: for each customer who reported a problem with the TrailRunner 32L Backpack, what kind of issue did they report, and what does Fernwood's policy entitle them to as a result? The correct answer, established directly from the source documents: Maria Ibarra reported a manufacturing defect, David Chen reported a shipping delay. Those two issue types are what the policy actually keys its remedy off of, so a customer's issue type is the fact that determines whether the rest of an answer can be right at all — and it's a cleanly checkable fact in a way remedy wording isn't, since Fernwood's policy happens to phrase both remedies using the same words, "full refund" and "free return." **"Manufacturing defect" and "shipping delay" share no words, so attributing the wrong one to a customer is unambiguous, and worse than an incomplete answer: it states something specific and wrong rather than simply leaving a gap.**

No retrieval has nothing to draw on and cannot be right beyond a lucky guess, since Fernwood Outfitters, both customers, and both tickets exist nowhere outside this course's documents — not in any model's training data. Vector similarity RAG depends on two separate things going right: whether the documents actually needed made it past a fixed top-k cutoff (a real operational constraint — no production system hands a model its entire corpus on every question), and, even once retrieved, whether the model correctly keeps two customers' facts apart while reading a pile of undifferentiated raw text. The policy document is a real point of exposure for the first failure: its text never mentions "TrailRunner" by name, so nothing in a similarity score ties it to this specific product's tickets the way an explicit `GOVERNS` edge in the graph does. GraphRAG is exposed to neither failure, because the graph already encodes which policy clause governs which issue type as a structural edge, independent of which words the source documents happen to share with each other.

> That's the actual mechanism behind "a graph improves factual accuracy." It isn't that a graph makes a model smarter — it's that a graph turns a fact-assembly step the model would otherwise have to infer from raw text into a deterministic database lookup performed before the model is ever asked to reason at all.

One question and one graph is a demonstration of that mechanism, not a benchmark, and the honest generalization is narrower than "GraphRAG always wins": any question whose answer depends on connecting facts across documents that don't share much wording is a question where similarity-based retrieval is structurally exposed to missing the connection, and a graph traversal isn't, because it was never relying on wording in the first place. Run the notebook's own comparison rather than taking this description on faith — the point of measuring three approaches side by side is to see the actual result, not to assume it in advance.

---

## What this costs

In model calls, this week's notebook makes roughly ten extraction calls (one per document), one embedding call per canonical entity group during merging, one Cypher-generation call plus one answer call per GraphRAG question, and the three-way comparison adds another ten embedding calls (one per document, reused for the vector RAG baseline) plus a handful of answer calls across the three approaches. None of that is expensive on a lightweight model — the token cost guide from Session 1 covers the method for checking (`count_tokens` against a per-million-token price) and its own note that pricing changes, so check [AI Studio](https://aistudio.google.com) or the relevant provider's current pricing page rather than trusting a number written on a fixed date. What scales here is call *count*, not call *size* — a real internal-documentation set of a few thousand documents means a few thousand extraction calls, which is where the estimate needs doing before a real run, not after.

In human work, which is the cost easiest to undercount: someone has to define the schema before extraction starts, done once here but revisited every time a new document type shows up, review the merge decisions entity resolution makes (the similarity threshold in the notebook is a judgment call, not a fact), and re-run extraction whenever a source document changes, since nothing in this pipeline keeps the graph in sync with its sources automatically. A graph accurate the day it was built and never touched again degrades the same way a spreadsheet does when nobody keeps updating it — except a stale graph looks structurally complete right up until someone traverses the specific edge that's now wrong.

---

## What gaps to carry forward

The entity resolution in this exercise used one similarity threshold tuned by eye on ten documents, which will not hold unchanged at real-document-set scale. The text-to-Cypher step trusts the model to write a correct query and only guards against a destructive one, not a wrong-but-safe one that retrieves the wrong subgraph without any error to flag it. The groundedness check catches missing entities, not misstated relationships between entities that were genuinely retrieved. And the three-way comparison is a demonstration of one mechanism on one question, not a benchmark — it doesn't establish a general margin by which GraphRAG outperforms the alternatives, only the specific structural reason it can, on a question shaped like this one. None of these are simplifications unique to this exercise — each is a real, open problem in production GraphRAG systems, and worth knowing going in rather than discovering the first time a stakeholder asks a question the graph answers confidently and wrongly.
