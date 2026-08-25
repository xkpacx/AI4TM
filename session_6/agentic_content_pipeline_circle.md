# Hands-on: build a search AI agent

*Building an agentic content pipeline*

This is the text companion to the agentic content pipeline notebook. **Time**: about 40-50 minutes, most of it spent waiting on model calls. **Cost**: a few cents at most on Gemini's cheapest models: the notebook makes on the order of 25-35 small calls total.

The earlier lesson this week turned a search performance export into a ranked table of content gaps: real questions, backed by real query volume, that the site's existing pages don't adequately answer. This piece works through what happens next: handing that table to an **agent**, in the definition this course has used since Week 2: a model given tools and a loop.

---

## What the agent is actually given

Three tools, and nothing else. `list_gaps` ranks the flagged gaps by demand. `get_cluster_detail` returns one cluster's member queries, its intent, its query shape, and its current best-matching page. `get_page_content` returns that page's title, meta description, and H1. All three are read-only functions over data already sitting in the notebook: none of them writes anything, and none of them reaches the open internet.

That restriction matters for what comes later. Everything the agent's final answer says should trace back to one of these three calls. A tool set built this narrow makes that check possible; a wider one, a live competitor-content lookup, a CMS draft-save function, would still be a legitimate agent, but each addition is a new thing a reviewer has to trust or verify, not just a new capability for free.

---

## Writing the loop by hand

The Gemini SDK can build a tool's schema automatically from a plain Python function's type hints and docstring, and run the entire call-execute-repeat cycle behind one function call. The notebook doesn't use that shortcut. The schema for each tool is declared explicitly, automatic function calling is switched off, and the loop, send the task, read the response for a function call, execute it, feed the result back, repeat, is written out as ordinary Python.

That choice is the whole point of the exercise stated in this week's overview: naming the tools, the stopping condition, and the point where a human reads the output. A loop hidden inside an SDK call can't be named, only assumed. The version in the notebook has two visible stopping conditions instead of one implicit one: the model can decide, on its own, that it has enough to answer without asking for another tool, or a hard cap on the number of calls (`MAX_TOOL_CALLS`) can end the loop first, for the run where the model's own judgment about being finished doesn't arrive. Both conditions show up directly in the transcript the notebook prints, under a `stopped_by` value that's worth reading on an actual run rather than assuming in advance.

---

## What a typical run looks like

Given one task, find the highest-demand gap, look at it, check the page it currently gets matched to, and write a grounded recommendation, a working run tends to call `list_gaps` first, because there's nothing yet to reason about, then `get_cluster_detail` on whichever cluster came back highest-demand, then `get_page_content` on that cluster's best-matching page, and stops there. At that point it has a cluster, its real queries, its real intent, and the existing page's actual title and meta description on hand, enough to write something grounded without inventing anything.

> Read the transcript your own run actually produces rather than taking this description on faith. The point of writing the loop out instead of hiding it inside the SDK is exactly so that sequence is checkable, not assumed.

---

## Where a human still has to read this

Nothing this agent produces reaches a CMS on its own, for a specific, nameable reason, not caution alone. The agent's tools are read-only fetches from real numbers, but its final recommendation is still a generation step, and a generation step can misstate what its own tools returned even when every underlying fact was real. Session 5 raised the same concern about GraphRAG answers under the name **groundedness**: an answer that reads confidently is not the same as an answer actually supported by what was retrieved.

The check a reviewer owes this output is narrow and fast: does the cluster label, the score, and the page the recommendation names actually match what `get_cluster_detail` and `get_page_content` returned in the transcript. That takes a minute, and it catches the failure that matters most: a confident recommendation built on a query, a score, or a page that was never actually retrieved, rather than one drawn from what the tools genuinely returned.

---

## From one recommendation to several

The same loop, run once per gap instead of once total, produces a lightweight content-opportunity memo for each gap above a chosen size rather than one deep dive. Each run is bounded independently, and each one still owes the same human check before anything in it gets acted on. Scaling an agent's reach, in other words, doesn't reduce how much a human has to verify: it multiplies the number of outputs that need the same check applied to each one.

---

## What gaps to carry forward

The tool set here is deliberately narrow, which is part of what makes its output checkable: a production version would likely need more tools, and each one widens what a reviewer has to trust. The loop bound is a fixed number chosen for this exercise, not derived from anything about how many calls a real task actually needs; a real deployment would want that number reasoned about directly, the same way this week's earlier lesson treated `min_cluster_size` as a judgment call rather than a default. And the groundedness check described above catches a recommendation that misstates its own retrieved facts, though it doesn't catch a recommendation that stays faithful to those facts while the facts themselves, further upstream in the gap table, were wrong. This is the same layered-trust problem every agentic pipeline built on top of an earlier pipeline inherits, not a flaw unique to this notebook, and naming it here is more useful than discovering it the first time a stakeholder acts on a recommendation that was faithful to bad data.

The next lesson this week takes one gap from this same table and goes deeper on a single output type, a full landing page brief, and turns the "a human should check this" instinct from this piece into something actually measured.
