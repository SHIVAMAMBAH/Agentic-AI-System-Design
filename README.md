# <div align = "center">**System Design Document**</div>

## Agentic Reasoning System

**Challenge:** Saptang Labs – Machine Learning Challenge  
**Author:** Turing Machines  
**Date:** October 2025  

---

## 1.  Introduction

### 1.1 Purpose

This document outlines the design and architecture of an **Agentic Reasoning System (ARS)** — an AI system capable of autonomously decomposing, planning, executing, and verifying solutions for logic-based reasoning tasks.
Unlike monolithic large language models, ARS performs **structured multi-step reasoning** by integrating lightweight models, symbolic tools, and rule-based planning.

### 1.2 Scope

The system aims to:

* Decompose complex logic problems into solvable subproblems.
* Dynamically select the most appropriate solver or tool.
* Execute subtasks sequentially or in parallel.
* Verify intermediate and final results.
* Produce transparent, human-readable reasoning traces.

The final deliverable is a modular, interpretable pipeline optimized for **accuracy, transparency, and reproducibility**.

---

## 2.  Objectives

| Objective                      | Description                                                                               |
| ------------------------------ | ----------------------------------------------------------------------------------------- |
| **Problem Decomposition**      | Identify subcomponents and logical relations within complex problems.                     |
| **Tool Selection**             | Match subproblems with suitable solvers (symbolic, numeric, code-based).                  |
| **Execution**                  | Perform computations, symbolic manipulations, or simulations.                             |
| **Verification**               | Check for consistency, dimensional correctness, or logical coherence.                     |
| **Reasoning Trace Generation** | Maintain a full record of all reasoning steps, justifications, and verification outcomes. |

---

## 3.  Restrictions

To ensure innovation in system design rather than LLM dependency:

* **Prohibited:** GPT-4, GPT-5, Claude-3, Gemini Ultra, and equivalent reasoning-heavy APIs.
* **Permitted:**

  * Small or base open models (e.g., *Phi-3-mini, Mistral-7B, Llama-3-8B-Instruct*)
  * Symbolic tools: *SymPy, Z3, PrologPy, MiniKanren*
  * Algorithmic and rule-based reasoning components

---

## 4.  System Architecture

### 4.1 Overview

The system follows a **four-phase architecture** integrating planning, reasoning, and verification:

```
┌──────────────────────────┐
│     Input Interface       │
│ (Natural Language Query)  │
└────────────┬──────────────┘
             ▼
┌──────────────────────────┐
│  1. Problem Decomposer    │
│  - LLM hybrid(T5-small)
│  - Generates subproblems  │
└────────────┬──────────────┘
             ▼
┌──────────────────────────┐
│  2. Planner & Tool Mapper │
│  - Builds reasoning graph │
│  - Assigns solvers/tools  │
└────────────┬──────────────┘
             ▼
┌──────────────────────────┐
│  3. Executor & Verifier   │
│  - Runs subtasks          │
│  - Cross-verifies results │
└────────────┬──────────────┘
             ▼
┌──────────────────────────┐
│  4. Reasoning Trace Gen.  │
│  - Logs all steps         │
│  - Produces final answer  │
└──────────────────────────┘
```

---

## 5.  Module Descriptions

### 5.1 Problem Decomposer

**Goal:** Convert a raw natural language problem into atomic subproblems.
**Techniques:**

* we are using T5-small and training it on GSM8K and is available at https://raw.githubusercontent.com/openai/grade-school-math/refs/heads/master/grade_school_math/data/train.jsonl

**Output Example:**

```json
{"question": "Natalia sold clips to 48 of her friends in April, and then she sold half as many clips in May. How many clips did Natalia sell altogether in April and May?",
"answer": "Natalia sold 48/2 = <<48/2=24>>24 clips in May.\nNatalia sold 48+24 = <<48+24=72>>72 clips altogether in April and May.\n#### 72"}
```

---

### 5.2 Planner & Tool Mapper

**Goal:** Assign each subproblem to the most efficient solving mechanism.

| Subproblem Type | Tool/Method                   | Example                       |
| --------------- | ----------------------------- | ----------------------------- |
| Arithmetic      | Internal calculator           | `200 / (60+40)`               |
| Algebraic       | SymPy symbolic solver         | Solve for x in `2x + 3 = 7`   |
| Logical         | Rule-based inference / Prolog | Deduce from premises          |
| Algorithmic     | Python code executor          | Simulation or iteration tasks |

**Output Example:**

```json
{
  "plan": [
    {"step": "relative_speed", "tool": "calculator"},
    {"step": "meeting_time", "tool": "symbolic_solver"}
  ]
}
```

---

### 5.3 Executor & Verifier

**Goal:** Execute subtasks, verify results, and check intermediate consistency.

**Verification Strategies:**

* **Redundant evaluation:** Use both numeric and symbolic solvers.
* **Tolerance-based check:** `abs(result_1 - result_2) < ε`.
* **Dimensional analysis:** Ensure unit consistency.
* **Logic equivalence:** Validate logical expressions via Z3 or truth tables.

**Output Example:**

```json
{
  "execution_results": {
    "relative_speed": "100 km/h",
    "meeting_time": "2 hours"
  },
  "verification": "passed"
}
```

---

### 5.4 Reasoning Trace Generator

**Goal:** Produce human-readable, auditable reasoning logs.

**Trace Structure:**

```json
{
  "trace": [
    "Step 1: Identified relative motion problem.",
    "Step 2: Computed relative speed = 60 + 40 = 100 km/h.",
    "Step 3: Calculated meeting time = 200 / 100 = 2 hours.",
    "Step 4: Verified result using symbolic solver. Verification passed."
  ],
  "final_answer": "2 hours"
}
```

---

## 6.  Tool Registry

| Tool Name           | Type       | Library          | Capability                           |
| ------------------- | ---------- | ---------------- | ------------------------------------ |
| **SymPy**           | Symbolic   | `sympy`          | Algebraic & calculus-based reasoning |
| **Z3 Solver**       | Logical    | `z3-solver`      | Logic and constraint satisfaction    |
| **Python Executor** | Code       | `exec()` sandbox | Algorithmic subtask execution        |
| **NumPy/Math**      | Numeric    | `numpy`, `math`  | Fast computation and array logic     |
| **MiniProlog**      | Rule-based | `prologpy`       | Deductive inference tasks            |

---

## 7. 📊 Data Flow

1. **Input:** Problem in text form.
2. **Decomposition:** Identify structure and subproblems.
3. **Planning:** Select sequence and tools.
4. **Execution:** Perform calculations/symbolic solutions.
5. **Verification:** Validate correctness and consistency.
6. **Reasoning Trace:** Construct human-readable explanation.
7. **Output:** Final answer + reasoning trace.

---

## 8.  Example Walkthrough

**Input:**

> "A box contains 5 red, 3 blue, and 2 green balls. If one ball is drawn at random, what is the probability it is not green?"

**Pipeline Trace:**

1. **Decompose:**

   * Identify total balls = 5 + 3 + 2 = 10.
   * Identify favorable = not green → 8.
   * Apply probability formula P = favorable / total.
2. **Select Tools:**

   * Use symbolic/numeric calculator.
3. **Execute:**

   * P = 8 / 10 = 0.8.
4. **Verify:**

   * Alternate check via complementary probability: 1 – (2/10) = 0.8 → matches.
5. **Generate Trace:**

   ```
   Step 1: Count total balls = 10.
   Step 2: Not green = 8.
   Step 3: Probability = 8/10 = 0.8.
   Step 4: Verified by complement rule.
   Final Answer: 0.8
   ```

---

## 9.  Implementation Plan

| Phase       | Deliverable         | Description                                          |
| ----------- | ------------------- | ---------------------------------------------------- |
| **Phase 1** | Core architecture   | Implement decomposer, planner, and executor modules. |
| **Phase 2** | Tool integration    | Add symbolic, logical, and numerical solvers.        |
| **Phase 3** | Verification logic  | Implement redundancy checks and tolerance metrics.   |
| **Phase 4** | Reasoning trace     | Develop structured and human-readable logging.       |
| **Phase 5** | Dataset integration | Evaluate and fine-tune on benchmark dataset.         |

---

## 10.  Evaluation Metrics

| Metric                 | Definition                            |
| ---------------------- | ------------------------------------- |
| **Accuracy**           | Correct final answers on dataset      |
| **Verification Score** | % of outputs validated successfully   |
| **Interpretability**   | Clarity of reasoning trace            |
| **Modularity**         | Ease of extension to new tool types   |
| **Reproducibility**    | Ease of running pipeline from scratch |

---

## 11.  Innovation Highlights

* Hybrid **rule-based + symbolic + neural** reasoning.
* Self-verifying computation via dual-tool crosschecks.
* Transparent **reasoning graph** instead of hidden chains.
* Designed for explainability and scientific rigor.

---

## 12.  Folder Structure

```
agentic_reasoning_system/
├── decomposer/
│   ├── rule_based.py
│   └── llm_based.py
├── planner/
│   └── tool_selector.py
├── executor/
│   ├── symbolic_solver.py
│   ├── logic_solver.py
│   └── calculator.py
├── verifier/
│   └── consistency_checker.py
├── trace/
│   └── trace_logger.py
├── datasets/
│   └── logic_problems.json
├── main.py
└── README.md
```

---

## 13.  Future Extensions

* Integration with **graph-based reasoning memory** (storing solved subpatterns).
* **Adaptive tool learning:** system updates tool-selection heuristics from past success rates.
* Expansion to **multi-agent reasoning**: planner + verifier + critic agents.

---

## 14.  Conclusion

This **Agentic Reasoning System** bridges symbolic, algorithmic, and lightweight neural reasoning to produce reliable, interpretable, and verifiable solutions to logical problems.
It emphasizes *planning, transparency,* and *modularity* — fulfilling the core objectives of the Saptang Labs Machine Learning Challenge while adhering to its restrictions on pre-trained reasoning-heavy LLMs.

---
