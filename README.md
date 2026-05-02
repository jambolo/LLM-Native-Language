# LLM Native Language (LNL)

LNL is a compact, unambiguous, token-efficient language designed for LLMs operating as reasoners, agents, and domain specialists. It targets LLM-internal reasoning, inter-agent communication, and domain-specific annotation - contexts where natural language is too verbose, ambiguous, and semantically under-specified.

## Motivation

English is poorly suited for LLM-to-LLM and LLM-internal use:

* **Token inefficiency** -- articles, auxiliaries, and redundant syntax consume tokens without adding semantic content
* **Ambiguity** -- pronoun-heavy, context-dependent, relies on pragmatic inference that is unreliable across model instances
* **No structured semantics** -- conditionality, sequence, causality, and confidence are expressed in prose, forcing models to parse rather than decode
* **No dialect separation** -- internal reasoning, agent commands, and domain knowledge share the same surface form

## Goals

* Reduce token count vs. equivalent English by ≥40%
* Eliminate syntactic ambiguity in all core constructs
* Encode semantic relationships (sequence, causation, conditionality, confidence) in grammar, not vocabulary
* Support distinct dialects with shared base grammar
* Remain parseable by current LLMs without fine-tuning (bootstrap phase)
* Enable lossless round-trip translation to/from English for human inspection

## Dialects

LNL defines three dialects that share the base grammar but diverge in lexicon and pragmatics:

**Internal Reasoning Dialect (IRD)** -- single-model scratchpad, chain-of-thought, hypothesis tracking. Adds `$` (fact), `?` (hypothesis), `%%` (working memory), `>>rev` (revision), `DEAD` (dead end).

**Agent-to-Agent Dialect (A2A)** -- task delegation, capability negotiation, result passing, error signaling between LLM agents. Messages are typed (`REQ`, `RES`, `ERR`, `ACK`, `NEG`) with explicit sender/receiver, IDs, TTL, and priority.

**Domain-Specific Dialect (DSD)** -- namespaced vocabulary extensions for fields like code (`code:`), medicine (`med:`), law (`law:`), finance (`fin:`). Any base operator can be used with namespaced tokens; domains may define shorthand macros that expand to base LNL.

## Comparison

| Approach           | Token efficiency | Ambiguity | Structured semantics | LLM-native |
|--------------------|------------------|-----------|----------------------|------------|
| English prose      | Low              | High      | None                 | No         |
| JSON/XML           | Medium           | Low       | Partial              | No         |
| Formal logic (FOL) | Medium           | Very low  | High                 | No         |
| Pseudocode         | Medium           | Medium    | Partial              | Partial    |
| **LNL**            | **High**         | **Low**   | **High**             | **Yes**    |

## Status

Draft v0.1 -- bootstrap phase. Grammar is enforced by convention; formal EBNF spec and tooling are planned. See `design.md` for full design document including open questions, tradeoffs, and next steps.
