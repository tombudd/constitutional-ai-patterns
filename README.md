# Constitutional AI Patterns

> A pattern language for runtime constitutional constraints in AI systems — documented in the style of the Gang of Four, applied to AI governance.

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-blue.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Status: Active](https://img.shields.io/badge/Status-Active-brightgreen)]()

---

## What This Is

Constitutional AI is becoming a widely-used term. But most implementations treat the "constitution" as a prompt — a list of principles fed to the model at inference time, with no enforcement mechanism beyond the model's own tendency to follow instructions.

This repository documents a different approach: **runtime constitutional constraints** — governance that operates at the system level, not the prompt level.

The patterns here are implementation-agnostic. They apply whether you're building on top of a foundation model, designing a multi-agent system, or architecting an autonomous agent framework. Each pattern describes:

- **The problem** it solves
- **The context** in which it applies
- **The solution** structure
- **The consequences** (forces resolved, forces introduced)
- **Implementation sketches** in pseudocode

---

## Pattern Catalogue

### Governance Patterns
| Pattern | Problem Solved |
|---------|---------------|
| [Constitutional Identity](patterns/governance/constitutional-identity.md) | How does a system maintain consistent values across contexts? |
| [Runtime Constraint Enforcement](patterns/governance/runtime-constraint.md) | How are constraints enforced at execution time, not just training time? |
| [Immutable Audit Receipt](patterns/governance/audit-receipt.md) | How is every decision made accountable and reconstructable? |
| [Constitutional Boundary](patterns/governance/constitutional-boundary.md) | How does a system recognise and handle constraint violations? |
| [Adversarial Self-Test](patterns/governance/adversarial-self-test.md) | How does a system verify its own alignment under attack? |

### Trust Patterns
| Pattern | Problem Solved |
|---------|---------------|
| [Trust Calibration](patterns/trust/trust-calibration.md) | How does a system maintain calibrated trust in its own outputs? |
| [Graduated Authority](patterns/trust/graduated-authority.md) | How are different actions gated by different levels of authorisation? |
| [Human-in-the-Loop Gate](patterns/trust/human-gate.md) | When and how does a system escalate to human oversight? |

### Identity Patterns
| Pattern | Problem Solved |
|---------|---------------|
| [Persistent Identity](patterns/identity/persistent-identity.md) | How does an AI system maintain coherent identity across sessions? |
| [Cryptographic Soul Print](patterns/identity/soul-print.md) | How is system identity verifiably anchored to a cryptographic chain? |
| [Character Invariance](patterns/identity/character-invariance.md) | How does core character persist across fine-tuning and updates? |

### Accountability Patterns
| Pattern | Problem Solved |
|---------|---------------|
| [Causal Receipt Chain](patterns/accountability/causal-chain.md) | How is each action traceable to its causal antecedents? |
| [Consequence Projection](patterns/accountability/consequence-projection.md) | How does a system reason about downstream effects before acting? |
| [Anomaly Flag](patterns/accountability/anomaly-flag.md) | How are behavioural deviations detected and surfaced? |

---

## A Sample Pattern: Immutable Audit Receipt

**Intent:** Ensure every significant decision produces an immutable, verifiable record that can be reconstructed without access to the original system.

**Problem:** AI systems make consequential decisions that affect people. Without accountability infrastructure, it is impossible to audit, challenge, or learn from those decisions. Logs can be deleted. Prompts can be changed. The model can be updated. Only a receipt that exists *outside* the system's control can provide genuine accountability.

**Solution Structure:**

```
Decision Event
    │
    ▼
Receipt Generator
    ├── action_id: UUID
    ├── timestamp: ISO8601
    ├── inputs_hash: SHA256(inputs)
    ├── outputs_hash: SHA256(outputs)
    ├── reasoning_summary: str
    ├── constraint_checks: List[ConstraintResult]
    ├── confidence: float
    └── signature: HMAC(payload, system_key)
    │
    ▼
Immutable Store (append-only)
```

**Key forces resolved:**
- Accountability without surveillance (receipts are auditable, not surveillance)
- Non-repudiation (system cannot deny what it decided)
- Reconstructability (any receipt can be independently verified)

**Consequences introduced:**
- Storage overhead (mitigated by hashing rather than storing full inputs)
- Key management (system keys must be secured and rotated)
- Latency (receipt generation adds ~2ms per decision at scale)

**Implementation sketch:**

```python
import hashlib, hmac, uuid
from datetime import datetime, timezone
from dataclasses import dataclass, field
from typing import Any

@dataclass
class GovernanceReceipt:
    action_id: str = field(default_factory=lambda: str(uuid.uuid4()))
    timestamp: str = field(default_factory=lambda: datetime.now(timezone.utc).isoformat())
    inputs_hash: str = ""
    outputs_hash: str = ""
    reasoning_summary: str = ""
    constraint_checks: list = field(default_factory=list)
    confidence: float = 0.0
    signature: str = ""

    @classmethod
    def generate(cls, inputs: Any, outputs: Any, reasoning: str,
                 constraints: list, confidence: float, key: bytes) -> "GovernanceReceipt":
        receipt = cls(
            inputs_hash=hashlib.sha256(str(inputs).encode()).hexdigest(),
            outputs_hash=hashlib.sha256(str(outputs).encode()).hexdigest(),
            reasoning_summary=reasoning,
            constraint_checks=constraints,
            confidence=confidence,
        )
        payload = f"{receipt.action_id}:{receipt.timestamp}:{receipt.inputs_hash}:{receipt.outputs_hash}"
        receipt.signature = hmac.new(key, payload.encode(), hashlib.sha256).hexdigest()
        return receipt

    def verify(self, key: bytes) -> bool:
        payload = f"{self.action_id}:{self.timestamp}:{self.inputs_hash}:{self.outputs_hash}"
        expected = hmac.new(key, payload.encode(), hashlib.sha256).hexdigest()
        return hmac.compare_digest(self.signature, expected)
```

See the full pattern at [patterns/governance/audit-receipt.md](patterns/governance/audit-receipt.md).

---

## Design Principles

These patterns share four underlying principles:

**1. Constitution over policy**
A constitution defines what the system *is*. A policy defines what it *does*. Policies can be changed. Constitutions should be nearly immutable.

**2. Enforcement over aspiration**
A principle that cannot be enforced is a decoration. Every pattern here is designed to be mechanically enforceable, not just aspirationally true.

**3. Receipts over logs**
Logs are mutable and system-local. Receipts are immutable and externally verifiable. The difference matters for accountability.

**4. Identity over instruction**
A system that acts well because of what it *is* is more robust than one that acts well because of what it's *told*. These patterns build character, not compliance.

---

## Contributing

Proposals for new patterns welcome. Use the [pattern template](PATTERN_TEMPLATE.md) and open a PR.

See [tombudd.com/get-involved](https://tombudd.com/get-involved) for broader collaboration on AI governance.

---

© 2025–2026 Tom Budd / ResoVerse Technologies · CC BY 4.0
