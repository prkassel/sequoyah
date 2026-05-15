# Sequoyah Project

https://education.nationalgeographic.org/resource/sequoyah-and-creation-cherokee-syllabary/

Born around 1770, Sequoyah was a self-taught polymath—a silversmith, blacksmith, and linguist who spoke only Cherokee and could read nothing. After witnessing the strategic disadvantage of those without a written language during the War of 1812, he spent 12 years refining an 85-character syllabary. The Cherokee Nation formally adopted it in 1825, and within a few years it powered the *Cherokee Phoenix*—the first bilingual newspaper in U.S. history. His work achieved nearly 100% literacy for the Cherokee Nation within 25 years and eventually inspired 21 other scripts for 65 languages worldwide.

The Sequoyah Project is an open-source initiative to return creative and financial sovereignty to the individual through "Non-Custodial AI" — showing that capable AI is already within reach for anyone, without subscriptions, API costs, or the environmental burden of routing every task through a data center.

---

## Architecture: The Sovereign Strategy

The system should operate on a clear governance hierarchy to ensure the user remains in control:

* **The Director (User):** Responsible for high-level intent, vision, and final approval.
* **The Production Assistant (PA):** A stateless execution engine that provides the "recipe" (logic) while the user's local hardware does the "cooking" (execution).
* **The State Gate:** A "Minimum-Required-Chat UX" protocol that halts execution to request missing variables via interactive fields, eliminating "hallucination chatter" and unnecessary cloud costs.



## Core Features

### 1. Reflective Self-Learning

The project implements **Instruction Hardening** to evolve without manual prompt engineering:

* **Heuristic Log:** A local SQLite database tracks the "Delta"—the exact difference between the AI’s initial draft and the Director’s final edits.
* **Automated Synthesis:** When frequency analysis detects recurring patterns in edits, the system generates a "Protocol Update" for approval to permanently harden instructions.



### 2. High Utility Density

The aim is AI that works on hardware within reach of a typical chatbot user, without a cloud account:

* **Local Inference:** Optimized for Small Language Models (SLMs) such as Qwen 2.5 ($3\text{B}$ parameters) or Llama 3.2 ($3\text{B}$–$8\text{B}$ parameters).
* **E-Waste Compatibility:** Targeted for low-power hardware like a Raspberry Pi 5, 2017 MacBook Pro or a realistic baseline, to ensure accessibility as a universal utility.
* **Agentic Firmware:** Runs on a minimalist Just Enough Operating System (JeOS) like Alpine Linux ($<100\text{MB}$) to eliminate telemetry and GUI overhead.
