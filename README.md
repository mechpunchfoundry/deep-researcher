# Deep Researcher

A self-hosted, privacy-first AI research pipeline that turns a topic into a structured, sourced research output — without sending your data anywhere you don't control.

Built entirely on local infrastructure using n8n, Ollama, and PostgreSQL.

---

## What it does

You give it a topic. It searches the web, evaluates what it finds, reads the most relevant sources in full, extracts meaningful insights, analyses them, and delivers a coherent written output — all without any external AI API calls.

The whole process runs on your own hardware.

---

## How it works

The pipeline runs in five phases:

**1. Gather**
Generates a set of search queries tailored to the research mode, runs them against a self-hosted SearXNG instance, then assesses coverage and runs additional gap-filling searches if needed. The number of gap analysis rounds is configurable.

**2. Triage**
Scores every result for relevance using a local LLM. Results below a mode-specific threshold are discarded. What remains is an ordered list of the most useful sources.

**3. Extract**
Fetches each source in full via a self-hosted Trafilatura API. Sends the full article text to the LLM with a mode-aware prompt to extract meaningful insights. For long articles, a vector embedding subsystem identifies the most relevant passages before extraction.

**4. Analyse**
An LLM agent works across the extracted insights, querying by relevance score, cross-referencing sources, and recording analytical conclusions. The depth of analysis is shaped by the research mode.

**5. Synthesise**
Combines extraction insights and analytical conclusions into a coherent written output, routed to the appropriate destination based on output type.

---

## Research modes

The pipeline supports six modes, each with its own prompting behaviour, triage threshold, and analytical depth:

| Mode | Purpose |
|------|---------|
| `survey` | Broad overview of a topic |
| `targeted` | Focused answer to a specific question |
| `deep_dive` | Exhaustive treatment of a subject |
| `benchmarking` | Comparative analysis across options |
| `speculative` | Exploratory, forward-looking research |
| `technical` | Code, architecture, and implementation focus |

---

## Output types

Results are routed to one of three destinations depending on what you asked for:

- **Ideas** — creative directions, possibilities, and inspiration
- **Technical notes** — implementation detail, code patterns, architecture
- **Research library** — structured reference material for ongoing use

---

## Architecture

The pipeline is built as modular reusable sub-workflows in n8n rather than a single monolithic flow. Each phase is a standalone component that accepts a `job_id` and `phase_tag`, making individual phases independently testable and replaceable.

Job state is tracked throughout in PostgreSQL. Every search result, triage score, fetched page, extracted insight, and analytical conclusion is persisted — meaning the pipeline can be interrupted, inspected, and resumed at any point.

```
Orchestrator
├── Query Generator        (reusable)
├── Search & Store         (reusable)
├── Triage                 (reusable)
├── Extract                (reusable)
├── Distil                 (single-use)
├── Analyse                (single-use)
└── Synthesise & Deliver   (single-use)
```

---

## Stack

| Component | Purpose |
|-----------|---------|
| n8n | Workflow orchestration |
| Ollama | Local LLM inference (Mistral Nemo) |
| nomic-embed-text | Vector embeddings for long documents |
| PostgreSQL | Job tracking and persistent storage |
| SearXNG | Self-hosted metasearch |
| Trafilatura | Full article extraction API |
| Docker | Containerisation |
| Tailscale | Secure remote access |

All inference is local. No data leaves the network.

---

## Infrastructure

Runs on a self-hosted Intel i5 / 16GB RAM server on Ubuntu, accessible remotely via Tailscale. No GPU required — the pipeline is designed around CPU-friendly models that produce strong results at modest hardware requirements.

---

## Background

This project grew out of automation and tooling work done managing large-scale game testing operations, where the problem of quickly gathering, evaluating, and synthesising information across dozens of concurrent projects was a daily reality. Deep Researcher is the more deliberate, fully realised version of that instinct — built properly, from scratch, with privacy and local control as first principles.

---

## Status

Actively developed. Currently refactoring from a monolithic 111-node workflow into the modular sub-workflow architecture described above.# deep-researcher
