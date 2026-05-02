# LLM-Native Language (LNL) — Design Document

**Status:** Draft v0.1
**Date:** 2026-05-02

---

## 1. Motivation

Natural language (English) is the de facto medium for LLM prompts, reasoning traces, and inter-agent communication. It is well-suited for human consumption but poorly suited for LLM-to-LLM and internal reasoning use:

- **Token inefficiency.** Articles, auxiliaries, hedges, and redundant syntax consume tokens without adding semantic content.
- **Ambiguity.** English is context-dependent, pronoun-heavy, and relies on pragmatic inference that may be unreliable across model instances and contexts. For example, model A says 'it' referring to X, model B parses 'it' as Y.
- **No structured semantics.** Concepts like conditionality, sequence, causality, and confidence are expressed in prose, forcing models to parse rather than decode.
- **No dialect separation.** Internal reasoning, agent commands, and domain knowledge share the same surface form, creating confusion about register and intent.

LNL is a compact, unambiguous, token-efficient language designed for LLMs operating as reasoners, agents, and domain specialists. It is emitted and consumed by models and agents. It is the language they use to reason and communicate.

---

## 2. Goals

| Priority | Goal |
|----------|------|
| G1 | Reduce token count vs. equivalent English (amount TBD pending benchmark) |
| G2 | Eliminate syntactic ambiguity in all core constructs (criterion TBD) |
| G3 | Encode semantic relationships (sequence, causation, conditionality, confidence) in grammar, not vocabulary (comprehensive list is TBD) |
| G4 | Support distinct dialects with shared base grammar. (initial list of supported dialects is TBD) |
| G5 | Initially remain parseable by current LLMs without fine-tuning (bootstrap phase) |
| G6 | Semantic preservation in round-trip translation to/from English for human inspection |
| G7 | Error recovery — Malformed LNL produces partial parse with error markers, never silent misinterpretation. |

---

## 3. Non-Goals

- Human readability as a primary concern. Not a replacement for human-facing output.
- Natural language generation for end users
- Replacing domain-specific formal languages (SQL, code, math notation) — LNL wraps and annotates them, does not replace them
- A fixed vocabulary — lexicons are dialect-extensible, but lexicons closed within a dialect version; new terms require version bump.
- A programming language
- A knowledge representation
- A serialization format

---

## 4. Design Principles

**P1 — Symbolic over verbal.** Relationships expressed as operators, not words. `A -> B` not "A leads to B."

**P2 — Mandatory explicitness.** No unresolved or implicit referents. Every entity named or tagged on first use, referenced by tag thereafter.

**P3 — Layered structure.** Three layers in every statement:

1. *Slot* — the semantic role (agent, action, object, condition, result)
2. *Token* — the content filling that slot
3. *Qualifier* — optional confidence, scope, or modality marker

**P4 — Grammar encodes semantics.** Operators carry meaning that cannot be expressed by word choice alone. A reader (human or LLM) must not need world knowledge to parse structure.

**P5 — Dialects share base, diverge in lexicon and pragmatics.** All dialects parse with the same grammar engine. Dialect-specific vocabulary is namespaced.

**P6 — Compact defaults, explicit expansion.** Default omissions are defined. A bare verb implies the active present tense agent-action form. Deviations require explicit markers.

---

## 5. Core Grammar

### 5.1 Operators

| Operator | Meaning | English equivalent |
|----------|---------|-------------------|
| `->` | Sequence / then | "leads to", "then", "results in" |
| `=>` | Causal implication | "therefore", "causes" |
| `?:` | Conditional | "if … then" |
| `\|` | Disjunction | "or" |
| `&` | Conjunction | "and" |
| `!` | Action / imperative | "do", "execute", "perform" |
| `~` | Negation | "not", "no", "never" |
| `@` | Agent designator | "performed by", "assigned to" |
| `#` | Tag / label | entity or concept reference |
| `^` | Confidence / certainty | followed by 0.0–1.0 or L/M/H |
| `*` | Scope / context | "within", "given", "in the context of" |
| `<-` | Dependency / requires | "depends on", "requires" |
| `<<` | Input | data or context flowing in |
| `>>` | Output | data or result flowing out |
| `//` | Comment / annotation | human-readable aside, ignored by parser |

### 5.2 Sentence Structure

Base form:

```text
[*scope] [#agent] [!action] [#object] [->|=>|?:] [#result] [^confidence]
```

All bracketed elements are optional except `!action` in imperative sentences and `#result` in causal chains.

Examples:

```text
#model_a !analyze #dataset_x -> #summary_1 ^H
#pipeline ?:#cache_hit => !return #cache_val | !compute -> !store
*code_review #reviewer !flag #func_auth ~secure ^M
```

### 5.3 Entity Tagging

- First use: full label with `#` prefix: `#user_query`
- Subsequent use in same context: short tag: `#uq`
- Cross-context reference: `#ctx:label` e.g. `#reasoning:hypothesis_3`

### 5.4 Confidence Markers

`^H` high | `^M` medium | `^L` low | `^0.85` numeric
Absence of `^` implies assertion (equivalent to `^H`).

### 5.5 Scope Blocks

Curly braces delimit scope blocks:

```text
*{task: summarize
  #doc_1 !extract #key_points
  #key_points !compress -> #summary ^M
}
```

---

## 6. Dialects

### 6.1 Internal Reasoning Dialect (IRD)

**Purpose:** Single-model scratchpad, chain-of-thought, planning, hypothesis tracking.

**Pragmatics:**

- Agent is always `self`; `@self` is the default and omitted
- Emphasis on hypothesis (`?`), action (`!`), fact (`$`), sequence (`->`)
- Supports revision markers: `~~` (retract prior), `>>rev` (revised conclusion)

**Additional operators:**

| Operator | Meaning |
|----------|---------|
| `$` | Established fact |
| `?` | Open question / hypothesis |
| `%%` | Working memory store |
| `>>rev` | Revision of prior conclusion |
| `DEAD` | Dead end / abandon branch |

**Example:**

```text
$ #input = classification_task
? #label correct ^L
! check #prior_context -> $ #prior = ~relevant
>>rev ? #label correct ^M
! emit #label
```

### 6.2 Agent-to-Agent Dialect (A2A)

**Purpose:** Task delegation, capability negotiation, result passing, error signaling between LLM agents.

**Pragmatics:**

- Both sender and receiver explicitly named
- Messages have type: `REQ` request | `RES` result | `ERR` error | `ACK` acknowledge | `NEG` negotiate
- Includes message ID and optional reply-to

**Structure:**

```text
MSG #<id> @<sender> -> @<receiver> <type>
  << <input_payload>
  >> <expected_output>
  ?:<condition>
  ^<confidence>
```

**Additional operators:**

| Operator | Meaning |
|----------|---------|
| `CAP?` | Capability query |
| `CAP:` | Capability declaration |
| `ERR:` | Error with code |
| `TTL:` | Time-to-live / deadline |
| `PRI:` | Priority level (1–5) |

**Example:**

```text
MSG #m042 @orchestrator -> @specialist_legal REQ
  << #contract_text_7
  >> #clause_risks[]
  TTL: 30s
  PRI: 2

MSG #m043 @specialist_legal -> @orchestrator RES #m042
  >> #clause_risks[3]
  ^M
```

### 6.3 Domain-Specific Dialect (DSD)

**Purpose:** Specialized vocabulary namespaces for fields such as code, medicine, law, finance. Extends base grammar with domain lexicons.

**Structure:** Domain namespace prefixes all domain terms: `code:`, `med:`, `law:`, `fin:`, etc.

**Principles:**

- Namespaced terms are defined in a domain lexicon file (separate from grammar spec)
- Any base operator can be used with domain-namespaced tokens
- Domain dialects may define shorthand macros that expand to base LNL

**Example (code domain):**

```text
*{code:review
  #fn_auth code:smell ^H => ! code:refactor
  #fn_auth <- #lib:jwt code:dep_risk ^M
  ! flag >> #review_report
}
```

**Example (medicine domain):**

```text
*{med:dx
  $ #pt med:sx[fever, cough, dyspnea]
  ? med:dx #covid ^M | med:dx #flu ^M
  ! med:order [med:test:pcr, med:test:cxr]
  ?:#pcr_pos => $ med:dx #covid ^H
}
```

---

## 7. Tradeoffs

| Decision | Chosen | Rejected alternatives | Rationale |
|----------|--------|-----------------------|-----------|
| Operator syntax | Symbolic (`->`, `=>`, `!`) | Keywords (`THEN`, `CAUSE`, `DO`) | Lower token count; unambiguous |
| Entity refs | `#tag` | Positional / implicit | Explicit refs prevent pronoun ambiguity |
| Confidence encoding | Inline `^` marker | Separate confidence layer | Keeps statements self-contained |
| Dialect separation | Namespace + pragma markers | Separate languages | Shared parser; easier cross-dialect reasoning |
| Scope blocks | `*{ }` | Indentation-based | Robust to tokenization artifacts |
| Human readability | Secondary | Primary | LNL is for LLMs; English bridge available |
| Grammar enforcement | Convention in bootstrap phase | Hard parser requirement | Current LLMs can learn from examples before tooling exists |

---

## 8. Comparison to Existing Approaches

| Approach | Token efficiency | Ambiguity | Structured semantics | LLM-native |
|----------|-----------------|-----------|----------------------|------------|
| English prose | Low | High | None | No |
| JSON/XML | Medium | Low | Partial (structure only) | No |
| Formal logic (FOL) | Medium | Very low | High | No |
| Pseudocode | Medium | Medium | Partial | Partial |
| **LNL** | **High** | **Low** | **High** | **Yes** |

LNL differs from formal logic by prioritizing LLM parseability and token economy over mathematical rigor. It differs from pseudocode by encoding semantics in grammar rather than relying on programming language conventions.

---

## 9. Open Questions

- **OQ1:** Optimal operator set — which operators should be core vs. dialect-only?
- **OQ2:** Handling of nested conditionals without visual complexity
- **OQ3:** Versioning and backward compatibility as the spec evolves
- **OQ4:** Evaluation methodology — how to measure ambiguity reduction empirically?
- **OQ5:** Whether a formal grammar (BNF/EBNF) should be written before or after working examples validate the design
- **OQ6:** Macro/expansion system scope — simple shorthands vs. full templating

---

## 10. Next Steps (when requested)

- Core grammar spec (EBNF + operator reference)
- IRD dialect spec with worked reasoning examples
- A2A dialect spec with message schema
- Domain lexicon format definition
- Token count benchmark vs. English equivalents

## 11. Design/Document Issues to Resolve

- [ ] P1 vs P5 tension. Symbolic operators (P1) are universal, but dialects (P5) extend lexicon — does dialect get new operators or only new tokens? §6.1 IRD adds $, ?, %%, >>rev, DEAD — that's new operators, contradicting "shared base grammar". Clarify: operators core-only OR dialects may extend.
- [ ] P3 three-layer model never used in §5. Slot/Token/Qualifier described but examples in §5.2 don't label slots. Either drop the layer abstraction or show it in grammar.
- [ ] P4 "no world knowledge to parse structure" — fine for syntax, but code:smell in §6.3 requires domain knowledge. Scope P4 to base grammar only.
- [ ] P6 "compact defaults" undefined. List the defaults explicitly (active voice, present tense, ^H confidence, @self agent in IRD). Reader can't apply principle without them.
- [ ] Missing principle: composability. Can statements nest? §5.5 shows scope blocks but principles silent on whether arbitrary nesting allowed.
- [ ] Missing principle: parser determinism. Single parse tree per input? Critical for G2.
- [ ] Motivation should preview goals in order (token → ambiguity → semantics → dialects). Currently mismatched.
- [ ] Add glossary: "dialect", "operator", "tag", "scope" used before defined.
