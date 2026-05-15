# Sequoyah Project

https://education.nationalgeographic.org/resource/sequoyah-and-creation-cherokee-syllabary/

Born around 1770, Sequoyah was a self-taught polymath—a silversmith, blacksmith, and linguist who spoke only Cherokee and could read nothing. After witnessing the strategic disadvantage of those without a written language during the War of 1812, he spent 12 years refining an 85-character syllabary. The Cherokee Nation formally adopted it in 1825, and within a few years it powered the *Cherokee Phoenix*—the first bilingual newspaper in U.S. history. His work achieved nearly 100% literacy for the Cherokee Nation within 25 years and eventually inspired 21 other scripts for 65 languages worldwide.

Project Sequoyah is an open-source initiative to make every AI interaction count — so that a well-structured workspace on a small, local model delivers reliable results instead of expensive correction loops.

---

## Introduction

AI access isn't the problem anymore. Anyone can open a chatbot. The problem is that using AI poorly is expensive — and only large enterprise organizations can afford to do it. When a corporation burns thousands of tokens on hallucinated outputs and multi-message correction loops, it's a rounding error. When an individual does it, that's real money or real time lost, often worse than doing the work manually.

The difference between AI that frustrates you and AI that works isn't the model — it's the workspace. Today, extracting real value requires massive scaffolding: thousands of words of documentation and hard-coded constraints. That demands a level of technical wrangling that regular users shouldn't have to deal with.

Sequoyah's core premise is that **the workspace is the product, not the model.** A well-configured local 3B model will outperform an unconstrained frontier model for your daily, recurring tasks — because the workspace prevents the waste that makes AI feel unreliable.

Sequoyah makes small models genuinely useful by eliminating the guesswork that burns tokens and kills trust. Everything in this project serves that goal. If a feature doesn't make the tool more reliable, more efficient, or easier for the average user, it doesn't belong here.

---

## Architecture: The Sovereign Strategy

The system operates on a clear governance hierarchy to ensure the user remains in control:

* **The Director (User):** Responsible for high-level intent, vision, and final approval.
* **The Production Assistant (PA):** A stateless execution engine that provides the "recipe" (logic) while the user's local hardware does the "cooking" (execution).
* **The State Gate:** A protocol that halts execution to request missing variables via interactive fields, rather than guessing and hallucinating. This is the single highest-impact feature for reducing wasted tokens and frustrating correction loops.

---

## Core Features

### 1. Guided Workspace Scaffolding

The number one reason people fail with AI tools is that they start a blank session with no structure and expect the model to read their mind. Sequoyah should make it trivially easy to build the workspace that makes a model useful.

* **Project Onboarding:** A new user should be able to describe their project in their own words — what it is, how they work, and what they expect from the model — and get a workspace that's ready to use from the first session.
* **Constraint Templates:** Pre-built starting points for common use cases (writing, coding, data analysis, creative work) that encode the patterns that experienced users have already discovered. Not generic prompt libraries — opinionated constraints that prevent the most common failure modes.
* **Plain Language:** The configuration interface should never require the user to understand prompt engineering, token budgets, or model internals. If the user can describe what they want in their own words, the system should translate that into effective model constraints.

### 2. The State Gate (Divergence Prevention)

Most wasted time and cost with AI comes from the model silently making assumptions when it doesn't have enough information, then the user spending multiple rounds trying to get it back on track. The State Gate makes this structurally impossible.

* **Halt and Ask:** When the PA encounters ambiguity or missing context, it stops and asks — via structured fields, not open-ended chat — before proceeding. No guessing, no hallucinated defaults.
* **Minimum-Required-Chat:** The interaction model is designed to minimize back-and-forth. Instead of a conversational loop, the PA presents what it needs, the Director fills it in, and execution resumes. Fewer round trips means fewer tokens burned and less opportunity for divergence.
* **Scope Lock:** Once the Director has approved a direction, the PA cannot silently deviate from it. If execution hits a point that requires a change in scope, the system halts for re-approval rather than improvising.

### 3. Portable Workspaces

One practical path for people who can't afford ongoing cloud API costs: use a large model to build and validate your workspace, then run it locally on a small model. Sequoyah should make this transition seamless.

* **Model-Agnostic Configuration:** The workspace configuration should not be coupled to any specific model or provider. The same project file that works with a cloud API should work with a local 3B model — the system adapts the prompting strategy to the model's capabilities, but the user's intent and constraints stay the same.
* **Complexity Budget:** Small models have tight context windows. The system should automatically manage prompt complexity — surfacing the most relevant constraints for the current task rather than dumping the entire project configuration into every request. This is critical for making workspaces that were developed with large models functional on small ones.
* **Local-First, Cloud-Optional:** Optimized for Small Language Models (SLMs) such as Qwen 2.5 (3B parameters) or Llama 3.2 (3B–8B parameters) running on hardware people already own — a Raspberry Pi 5, a 2017 MacBook Pro, or similar. Cloud APIs are supported but never required.

---

## Exploratory: Automated Workspace Improvement

The following is an exploratory direction — not a committed roadmap. The question it tries to answer: **can the workspace get better on its own over time, without requiring the user to manually tune it?**

The idea is that every time a user corrects the PA's output, that correction contains signal about what the workspace is missing. If the system can detect recurring corrections and propose configuration updates, users who can't or won't maintain their own workspace documentation still get a tool that improves with use.

This is how that might work, broken into components that have been pressure-tested against the real constraints of the target hardware.

### Deterministic Pattern Learning

Rather than asking a 3B model to reflect on its own outputs (which it cannot do reliably), pattern detection is handled by a deterministic script outside the model entirely.

* A local Python script uses `difflib.SequenceMatcher` for word-level delta mapping between the PA's drafts and the Director's final edits stored in a local SQLite database (the Heuristic Log).
* When the same edit pattern passes a frequency threshold, a rigid template proposes a configuration update as a structured rule — no natural language generation involved.

### Prompt Space Management

Small models have tight context windows. Every token in the system prompt needs to earn its place.

* A strict hard cap (e.g., 10 automated instruction slots) governed by a Least Frequently Used (LFU) eviction policy keeps the prompt lean.
* When a new rule displaces an old one, the system outputs a clean diff to the CLI for the Director's review and approval.
* A `rejected_patterns` log prevents the system from repeatedly proposing rules the Director has already declined, with a cooldown period before re-evaluation.

### Idle-Time Processing

When the user is inactive, the system could run pattern analysis and maintenance as a low-priority background routine.

* **CPU scheduling:** `nice -n 19` ensures the Linux scheduler automatically yields cycles to the foreground the moment the user returns.
* **Database concurrency:** SQLite WAL mode with `wal_autocheckpoint=1000` allows the background routine to read while the foreground writes, preventing lock contention.
* **Environment hygiene:** Standard low-overhead maintenance (SQLite `VACUUM`, clearing inference engine temp state) runs via the same idle loop. Basic housekeeping, not a major feature.

### Summary

| Component | Naive Approach | Production Architecture |
| --- | --- | --- |
| **Data Synthesizer** | SLM reads logs and writes its own prompt rules. | Deterministic Python script using `difflib.SequenceMatcher` for word-level delta mapping. |
| **Prompt Space Optimizer** | SLM evaluates semantic contradictions and merges rules. | Strict 10-slot instruction cap with LFU eviction, subject to Director approval. |
| **System Loop Feedback** | Present rules endlessly until accepted. | `rejected_patterns` log with cooldown to prevent proposal spam. |
| **Idle Execution** | Kill background tasks to free CPU on wake. | `nice -n 19` scheduling; SQLite WAL mode with `wal_autocheckpoint=1000`. |
| **Environment Hygiene** | Major optimization to reverse hardware degradation. | Low-overhead shell utilities via cron for basic cache and database cleaning. |

> **Note:** This section is exploratory. Whether automated pattern detection meaningfully improves a workspace without human curation is an open question. The point of this project is to find out — and to share what we learn.
