### Dataset for Decomposer is GSM8K
- **Dessciption**: 8.5K high-quality, step-by-step grade school math problems.
- **Why it's useful**: Gold-standard for reasoning decomposition. Use only the question → reasoning steps (not the final answer) pairs.

### Dataset for Tool Mapper is MathQA
- **Description**: Math problems labeled by operation type (e.g., addition, ratio, geometry).
- **Why it's useful**: Use labels to train tool selection logic (e.g., choose SymPy if symbolic).

### Verification & Logical Reasoning is ProofWriter
- **Description**: Natural language deduction problems with reasoning chains.
- **Why it's useful**: Train verifier to check logical consistency.
