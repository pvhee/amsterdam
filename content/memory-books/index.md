---
title: "Memory Books: How Knowledge Graphs Give Runbooks a Brain"
date: 2026-04-10
tags: [BLOK, Cutover, knowledge-graph, AI, runbooks, memory, agentic-AI]
---

What if your runbooks could remember?

Not just store steps in a sequence, but actually *remember* — the way a senior engineer remembers that a particular failover always trips on the third dependency, or that the last time someone ran this migration on a Friday it took twice as long because of the batch jobs. That kind of memory. The kind that lives in people's heads and walks out the door when they leave.

This is the problem at the heart of enterprise IT operations, and it is one that the convergence of **knowledge graphs**, **agentic AI**, and **structured runbook models** is finally beginning to solve. We are calling this concept **Memory Books** — runbooks that don't just execute, but learn, remember, and reason.

## Runbooks Are Already Memory — They Just Don't Know It

Cutover CTO Kieran Gutteridge has been articulating this for a while now. In his writing on agentic AI and the runbook task model, he describes Cutover's directed graph architecture not just as a workflow engine but as a *foundation for intelligent systems*. His key insight: a runbook task model based on a directed graph — with clear task dependencies, sequencing, and data passing between steps — is structurally similar to what you need for an agentic AI to reason about complex operations.

> "The ability of AI agents to take action, learn, and solve complex problems in an explainable and trusted way offers a significant opportunity to accelerate IT operations and drive greater efficiency." — Kieran Gutteridge, CTO, Cutover

Gutteridge's work on using AWS Bedrock and generative AI to auto-generate runbooks from unstructured data sources demonstrates the first half of this vision: you can take scattered documentation, incident reports, and tribal knowledge and *synthesize* them into structured, executable processes. Cutover's team showed they could use Amazon Bedrock and Anthropic's Claude to rapidly produce runbooks and generate next-best-action suggestions for everything from cloud migrations to cyber disaster recovery.

But generation is only the beginning. The real question is: once a runbook has been executed — once it has encountered reality — what does it *learn*?

## From Runbooks to Memory Books

Traditional runbooks are static. They capture what someone knew at the time of writing. They degrade. They go stale. When the infrastructure changes, the runbook doesn't know. When a team member discovers a workaround during a 3 AM incident, that knowledge lives in a Slack thread, not in the runbook.

A **Memory Book** is different. It is a runbook backed by a knowledge graph — a living, temporal, interconnected representation of everything the organisation knows about its operations.

Here is how it works:

### 1. Episodic Memory — What Happened

Every execution of a runbook becomes an episode. Not just "step 3 completed in 45 seconds" but the full context: who ran it, what the environment state was, which steps were skipped or modified on the fly, what anomalies were observed. This is analogous to what the Zep/Graphiti architecture calls the **episode subgraph** — raw events annotated with timestamps, preserving high-fidelity records of what actually occurred.

In Cutover's model, this maps naturally to the execution data that already flows through their platform. Every task in the directed graph generates telemetry. The difference is treating that telemetry not as logs to be archived but as *episodes to be learned from*.

### 2. Semantic Memory — What We Know

From those episodes, entities and relationships emerge. The knowledge graph extracts patterns: "This database migration step typically takes 12 minutes in production but only 3 minutes in staging." "When team X runs this runbook, they always add a manual verification step after step 7." "The last three times this recovery procedure ran, it failed at the DNS propagation check."

This is **semantic memory** — extracted, generalised knowledge that transcends any single execution. It is what transforms a runbook from a static document into a living knowledge base.

### 3. Temporal Awareness — What Changed and When

This is where knowledge graphs particularly shine. Unlike a flat document, a knowledge graph can represent *when* something was true. Zep's architecture explicitly tracks both event time and ingestion time for every node and edge, with validity intervals on relationships. When a fact is superseded — when the infrastructure changes, when a procedure is updated — the old knowledge isn't deleted. It is *timestamped and preserved*.

For enterprise operations, this is critical. You need to know not just what the current procedure is, but what it *was* when that incident happened six months ago. You need to understand how your operational knowledge has evolved. A Memory Book gives you that for free.

### 4. Community Memory — What We Collectively Understand

At the highest level, the knowledge graph clusters related entities and patterns into communities — high-level domain summaries that represent the organisation's collective understanding of how its systems behave. This is the kind of knowledge that currently lives only in the heads of your most experienced engineers: "Our payment systems are most fragile during month-end processing" or "Failovers in region X always cascade to the monitoring stack."

## The Agentic Layer: Memory Books in Action

This is where Gutteridge's vision of agentic AI becomes concrete. Cutover's open-sourced MCP (Model Context Protocol) server — announced in mid-2025 — allows AI agents to query operational data and take action on runbooks and tasks using natural language. The MCP server is the *interface*; the knowledge graph is the *brain*.

When an AI agent is embedded in a Memory Book:

- **Before execution**, it can query the knowledge graph: "What happened the last three times this runbook was run? What went wrong? What workarounds were applied?" It enters the operation *with context*, not cold.
- **During execution**, it can reason about the current state against historical patterns: "Step 5 is taking longer than the 95th percentile of previous executions. The last time this happened, it was because of a connection pool exhaustion in the downstream service. Should we investigate?"
- **After execution**, it writes back to the graph. The episode is recorded. New entities and relationships are extracted. The Memory Book *grows*.

This is what Cutover describes as the "human-machine approach" — AI that doesn't replace human expertise but *augments* it by making institutional knowledge available to everyone, regardless of experience level. In their framing, automated runbooks act as "living knowledge repositories, capturing the collective expertise of IT teams and making it readily available."

The Memory Book takes that further: it doesn't just capture expertise. It *connects* it, *timestamps* it, and makes it *queryable*.

## Why This Matters Now

Three things have converged to make Memory Books possible:

1. **Structured runbook models** like Cutover's directed graph task model provide the scaffolding — the clear definition of dependencies, sequencing, and data flow that both humans and AI agents can reason about.

2. **Temporal knowledge graphs** like Graphiti/Zep provide the memory infrastructure — the ability to incrementally build, update, and query a living representation of operational knowledge without batch recomputation.

3. **Agentic AI and MCP** provide the interface — AI agents that can read from and write to the knowledge graph, reason about context, and take action within the guardrails of the runbook structure.

The result is an operational memory system that gets *better* with every execution. Every incident response, every migration, every failover adds to the graph. Knowledge that used to be lost — in retired employees' heads, in archived Slack threads, in outdated Confluence pages — is captured, connected, and kept alive.

## The Path Forward

We are not proposing a theoretical architecture. Every component of this exists today. Cutover's platform already orchestrates complex enterprise operations with a directed graph model. Knowledge graph engines like Graphiti already build temporally-aware entity graphs from unstructured data in real time. The Model Context Protocol already provides the standard for AI agents to interact with operational systems.

The work ahead is integration: connecting the execution telemetry of enterprise runbooks to knowledge graph infrastructure, and giving AI agents the ability to read and write to that graph as part of the operational workflow.

Kieran Gutteridge is right that runbook task models are a "promising foundation for building agentic AI systems." The next step is giving those systems memory. Not logs. Not documentation. *Memory* — structured, temporal, queryable, and alive.

That is what a Memory Book is. And it is how we stop losing what we know.
