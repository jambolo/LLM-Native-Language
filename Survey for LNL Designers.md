# LLM-Native Languages, Compressed Reasoning Formats, and Inter-Agent Protocols: A Survey for LNL Designers

## TL;DR

- **The space splits into three largely disjoint tracks**: (1) *latent/continuous* "neuralese" research (Coconut, CIPHER, Interlat, KVComm, Large Concept Models) where the medium is hidden states or embeddings, not text; (2) *compressed/structured discrete formats* aimed at off-the-shelf LLMs (LLMLingua, TOON, TypeChat/Guidance/LMQL, DSPy, Compressed CoT contemplation tokens); and (3) *agent-coordination wire protocols* (MCP, A2A, ACP, ANP, Agora, NLIP, FIPA-ACL legacy). LNL sits in track 2 but borrows semantic ambitions from track 1 and protocol-shape thinking from track 3.
- **LNL's exact niche — a discrete, off-the-shelf-parseable language with explicit semantic-relation operators, role/qualifier layering, mandatory referent disambiguation, English round-trip, and graceful partial-parse recovery — is not directly occupied** by any single project. The closest neighbors are LLMON (LLM-native markup), TOON (token-efficient JSON), Wang & Wen's "AI-centric language" position paper, and Attempto Controlled English. None combines all of LNL's properties.
- **However, individual LNL design choices recur widely**: token-efficient encoding (TOON, LLMLingua, gist tokens), unambiguous parsing (Lojban, ACE), explicit semantic roles/operators (FIPA-ACL performatives, Lojban evidentials, RDF), confidence/modality qualifiers (FIPA-ACL, ACE modality, Lojban attitudinals), and dialect-with-shared-grammar patterns (Agora's Protocol Documents). LNL is best framed as **a novel synthesis rather than a fundamentally new direction** — and it is largely orthogonal to (and competing with) the dominant industry bet on continuous latent reasoning.

---

## Key Findings

### 1. The field is bifurcating along a discrete-vs-continuous axis

The single most important framing for LNL is that **most of the cutting-edge "post-English" reasoning research has gone continuous**, not discrete. Meta's Coconut (2024) feeds last-hidden-states back as input embeddings; Compressed CoT (Cheng & Van Durme, 2024) generates variable-length continuous "contemplation tokens"; Pause/filler tokens (Goyal et al., 2024; Pfau et al., 2024) add inference-time computation slots; CIPHER (Pham et al., ICLR 2024) replaces token sampling with weighted-average embeddings for inter-LLM debate; Interlat (2025) and KVComm (2025) ship raw hidden states or KV caches between agents; Meta's Large Concept Models reason in SONAR sentence-embedding space.

LNL is explicitly discrete-token, off-the-shelf-parseable, and human-inspectable. **This is the minority position in current research but the majority position in deployed industry tooling**, where every shipping multi-agent system today still uses text-token messages.

### 2. Token-efficient discrete formats already exist and dominate practice

- **LLMLingua / LongLLMLingua / LLMLingua-2 (Microsoft Research, 2023–2024)**: prompt compression via small-model perplexity scoring; up to 20× compression with ~1% performance loss. Notably observed that GPT-4 has an *emergent ability* to recover compressed prompts that humans find cryptic — empirical evidence that LLMs accept dense non-English encodings.
- **TOON (Token-Oriented Object Notation, late 2025)**: schema-aware, indentation-based JSON replacement; reports ~30–60% token reduction and *higher* accuracy than JSON on uniform-array benchmarks (e.g., 70.1% vs 65.4%, 46% fewer tokens). Lossless round-trip with JSON. Closest direct analogue to LNL in spirit, though purely a serialization format.
- **Gist tokens (Mu et al., 2023)**: compress prompts up to 26× into learned soft tokens; *requires training*.
- **Selective-Context, FrugalPrompt, MultiTok**: assorted entropy- or attribution-based pruning approaches.

### 3. Structured-output / constrained-generation tooling is mature

- **TypeChat (Microsoft, 2023)**: TypeScript types as schema; LLM emits JSON validated against types; self-repair on validation failure.
- **LMQL (ETH Zurich)**: Python-embedded query language with `where` constraints, regex, datatype, and stopping conditions enforced via token masking during decoding.
- **Guidance (Microsoft)**: template-based constrained generation with deterministic templated tokens.
- **DSPy (Stanford/Databricks)**: declarative "programming, not prompting" with Signatures, Modules (`Predict`, `ChainOfThought`, `ReAct`), and gradient-free optimizers (BootstrapFewShot, MIPRO).
- **Outlines, SynCode, DOMINO**: CFG/regex-grammar-driven decoding with subword alignment.

These projects all use schema/grammar to constrain LLM *output format*. They do not provide a general-purpose communication language with built-in semantic-relation operators.

### 4. Agent communication protocols target plumbing, not language

The 2024–2025 explosion of agent protocols is fundamentally about **transport, discovery, and tool-calling**, not about the linguistic content of messages:

| Protocol | Owner | Date | Wire format | Scope |
|---|---|---|---|---|
| **MCP** (Model Context Protocol) | Anthropic | Nov 2024 | JSON-RPC | LLM ↔ tools/data |
| **A2A** (Agent2Agent) | Google → Linux Foundation | Apr 2025 | HTTP+SSE, gRPC; Agent Cards | Agent ↔ agent |
| **ACP** (Agent Communication Protocol) | IBM/BeeAI → merging into A2A | May 2025 | REST/HTTP | Agent ↔ agent |
| **ANP** (Agent Network Protocol) | Independent | 2024 | P2P, DID-based | Decentralized agent network |
| **NLIP** (Natural Language Interaction Protocol) | Ecma TC56 | Dec 2025 (ratified) | Multimodal envelope over HTTP/WebSocket/AMQP | Universal agent envelope |
| **LangChain Agent Protocol** | LangChain | 2024 | HTTP | Cross-framework agent interop |

All carry message *contents* as plain natural language or JSON. None defines the semantic encoding *inside* the message. **LNL would slot in at the payload layer**, orthogonal to (and complementary with) any of these.

### 5. Legacy ACLs solved many of LNL's design problems in the 1990s

- **KQML** and **FIPA-ACL** (1997, IEEE-standardized) are based on Searle's speech-act theory. FIPA-ACL defines ~22 *performatives* (`inform`, `request`, `confirm`, `query-if`, `not-understood`, `propose`, `agree`, `refuse`, …) — i.e., explicit grammatical encoding of communicative intent, exactly like LNL's grammar-level semantic operators.
- FIPA-SL content language supports modal operators (B for belief, U for uncertainty, I for intention) — direct analogues to LNL's confidence/modality qualifiers.
- These standards required shared ontologies and never achieved broad LLM-era adoption, but the design vocabulary is recyclable. LNL would benefit from explicit study of FIPA-ACL rather than reinventing performatives.

### 6. Constructed-language work for AI use is sparse but converging

- **Wang & Wen, "Building A Unified AI-centric Language System" (arXiv 2502.04488, ICLR 2025 workshop)** — position paper; argues for an unambiguous, regular, concise AI-centric language drawing from Esperanto/Lojban; proposes toy-language training experiments. **This is the single closest published precedent to LNL's framing.** It is a position paper, not an implementation.
- **Lojban**: predicate-logic-based, unambiguous grammar (one parse per sentence), evidentials and attitudinals (confidence/modality built into grammar), small regular vocabulary. Long history of theoretical AI-friendliness; community explicitly notes LLMs perform *poorly* on Lojban due to data sparsity — relevant warning for LNL's bootstrap-without-fine-tuning claim.
- **Attempto Controlled English (ACE)**: controlled English, deterministic mapping to first-order logic via Discourse Representation Structures; resolves anaphora, supports modality. This is essentially LNL's "round-trip-to-English" property already implemented, though biased toward human readability.
- **ConlangCrafter (2025)**: LLM-driven conlang generation; not for LLM communication but demonstrates LLM metalinguistic competence.
- **LLMON (arXiv 2603.22519)**: explicitly proposed *LLM-native markup language* using structural/semantic metadata to disambiguate instructions vs data inside prompts. Closest in *naming* and *spirit* to LNL but framed as markup, not as a language with its own grammar.

### 7. Reasoning-format research (CoT family) is adjacent but different

Chain-of-Thought, Tree-of-Thoughts, ReAct, Reflexion, Graph-of-Thoughts, Self-Refine all structure *what reasoning happens*, not *the encoding of the reasoning*. They use ordinary English. The structured-CoT literature only intersects LNL where it crosses into compressed/latent variants (Coconut, CCoT, Quiet-STaR, abstract-token CoT) — all of which require fine-tuning.

### 8. Emergent communication research is a cautionary tale

Pre-LLM emergent-communication work (Lazaridou & Baroni 2020 survey; Foerster, Mordatch, etc.) repeatedly produced opaque, non-compositional codes that worked but didn't generalize — the "emergent code" problem. Current LLM-era versions (CIPHER, Interlat, multi-agent neuralese) replicate these gains but inherit the same opacity costs. **A discrete, human-inspectable language like LNL is partially motivated as a safety/interpretability response to this trajectory** (cf. LessWrong "Reflections on Neuralese", which argues for keeping reasoning legible).

---

## Details: Comparative Map

| Project / class | Primary goal | Discrete or continuous | Off-the-shelf or fine-tune | Human-readable | Formal grammar | Status (May 2026) |
|---|---|---|---|---|---|---|
| **Coconut (Meta)** | Reasoning expressivity | Continuous (last hidden state) | Fine-tune | No | No | Active research; criticized for shortcut-learning (arXiv 2512.21711) |
| **Compressed CoT (Cheng & Van Durme)** | Reasoning + token efficiency | Continuous contemplation tokens | LoRA fine-tune | No | No | Active research |
| **Pause / Filler Tokens** | Test-time compute | Discrete placeholder | Best with pretraining | Trivially | No | Established result |
| **CIPHER** | Inter-LLM debate fidelity | Continuous (vocab-weighted embeddings) | Off-the-shelf (no weight changes) | Approx. via NN search | No | ICLR 2024; influential |
| **Interlat / KVComm / Q-KVComm** | Inter-agent efficiency | Continuous (hidden states / KV cache) | Some training | No | No | Active research (2025) |
| **Large Concept Models (Meta)** | Concept-level reasoning | Continuous (SONAR sentence embeddings) | Train from scratch | Via decoder | No | Released; modest adoption |
| **LLMLingua family** | Token efficiency | Discrete English (lossy) | Off-the-shelf | Marginally | No | Production-deployed |
| **Gist tokens (Mu)** | Prompt caching | Continuous learned tokens | Fine-tune | No | No | Established |
| **TOON** | Token-efficient structured data | Discrete | Off-the-shelf | Yes | Yes (spec'd) | Released late 2025; growing adoption |
| **TypeChat / LMQL / Guidance / Outlines / DSPy** | Structured output / programmability | Discrete | Off-the-shelf | Yes | Yes (TS types / CFG / regex / Python) | All actively maintained |
| **MCP** | LLM ↔ tool plumbing | Discrete (JSON-RPC) | Off-the-shelf | Yes | Yes (schema) | De facto standard 2025+ |
| **A2A** | Agent ↔ agent transport | Discrete (HTTP+SSE+JSON) | Off-the-shelf | Yes | Yes | Linux Foundation, v0.3, 150+ orgs |
| **ACP (IBM)** | Agent ↔ agent | Discrete (REST) | Off-the-shelf | Yes | Yes | Merging into A2A |
| **ANP** | Decentralized agent network | Discrete | Off-the-shelf | Yes | Yes | Niche |
| **NLIP (Ecma-430..434)** | Universal AI envelope | Discrete (CBOR/JSON over WebSocket/AMQP) | Off-the-shelf | Partially | Yes | Ratified Dec 2025 |
| **Agora (Marro et al., Oxford)** | Scalable LLM-network communication | Hybrid: structured routines + NL fallback + LLM-written routines | Off-the-shelf | Yes | Per Protocol Document | Research; ICLR submission |
| **FIPA-ACL / KQML** | Agent communication (legacy) | Discrete | N/A (pre-LLM) | Yes | Yes (speech-act based) | Legacy, mostly dormant |
| **Lojban** | Unambiguous human language | Discrete | LLMs handle poorly (sparse data) | To experts | Yes (LALR-parseable) | Niche community |
| **Attempto Controlled English (ACE)** | Controlled NL → FOL | Discrete | Off-the-shelf | Yes (looks like English) | Yes (deterministic to DRS/FOL) | Mature, niche |
| **LLMON (LLM-native markup)** | Structure/semantic metadata for prompts | Discrete | Off-the-shelf | Yes | Yes | Proposal stage (arXiv) |
| **Wang & Wen "AI-centric language"** | Unambiguous, efficient AI language | Discrete | TBD | Yes | Proposed | Position paper, no implementation |
| **CoT / ToT / ReAct / Reflexion** | Reasoning structure | Discrete English | Off-the-shelf | Yes | No | Standard practice |

---

## Where LNL Fits: Honest Assessment

### What LNL fills that's genuinely under-occupied

1. **A discrete general-purpose communication language with grammar-level semantic operators (causation, conditionality, sequence, confidence) parseable by current LLMs without fine-tuning.** TOON does serialization. JSON/YAML/XML have no semantic operators. FIPA-ACL has performatives but is dormant and pre-LLM. ACE has logic mapping but optimizes for human readability. LLMON adds metadata but isn't a language. **LNL's specific combination of operator-encoded relations + slot/token/qualifier three-layer + bootstrap-parseable is, as far as the literature shows, not currently published.**

2. **Mandatory referent disambiguation as a hard grammatical constraint.** Most projects punt on this (English does; ACE resolves anaphora during parsing but allows pronouns; FIPA-ACL relies on the content language). LNL's "no unresolved pronouns" rule is more aggressive than typical.

3. **Round-trip translatability + partial-parse error recovery + dialects sharing a base grammar.** This combination is unusual. Agora's Protocol Documents come closest to the dialect idea; ACE supports round-tripping; nobody does explicit error markers in a partial parse the way LNL proposes.

### Where LNL overlaps significantly with prior work

- **Token efficiency vs English**: TOON, LLMLingua, gist tokens, MultiTok, "Token Sugar" already establish that LLMs accept dense encodings. LNL is not pioneering this; it's joining a crowd.
- **Unambiguous grammar**: Lojban, ACE, controlled-NL literature have decades of head start. LNL should explicitly cite and differentiate.
- **Confidence/modality qualifiers**: FIPA-SL modal operators, ACE modality, Lojban attitudinals, evidential markers in many natural languages. Reusable design space.
- **Three-layer semantic-role encoding**: Resembles Frame Semantics (FrameNet), thematic-role labeling, RDF subject-predicate-object, and FIPA-ACL message structure (performative + content + parameters). Worth citing rather than presenting as novel.
- **Compact defaults + explicit expansion**: Mirrors XML default attributes, protobuf default values, and YAML anchors.

### Where LNL diverges from current research direction

- **Discrete-first while frontier research is going continuous.** Coconut, CIPHER, Interlat, LCMs all argue the future is sub-token. LNL is making the opposite bet: that interpretability, debuggability, and bootstrap-without-fine-tuning are worth the expressivity cost. This is defensible but should be argued explicitly. The Anthropic/safety community (e.g., LessWrong's neuralese discussions) supports this position.
- **No fine-tuning required.** This is rare among "LLM-targeted languages" — most (Coconut, Gist, CCoT, abstract-CoT) require training. LNL's bootstrap claim is most analogous to TOON, LLMLingua, and TypeChat, which all rely on the LLM's general schema-following ability. Plausible but should be empirically validated; LLMLingua-style work shows GPT-4 has emergent compressed-prompt comprehension while GPT-3.5 does not — frontier capability is required.
- **Not a programming language, KR, or serialization format.** This negative space is real but narrow. The risk is that LNL drifts toward one of these poles in implementation: toward serialization (becoming TOON), toward KR (becoming RDF/OWL/ACE), or toward DSL (becoming LMQL/DSPy signatures).

### Verdict on novelty

LNL's individual properties are each well-precedented. The **specific combination** — discrete + operator-encoded relations + slot/token/qualifier layering + dialect system + bootstrap-parseable + round-trip-with-English + partial-parse recovery — appears to be a new point in the design space. It is best characterized as **a thoughtful synthesis aimed at a specific niche (inter-LLM messaging that humans can audit), not a fundamental research breakthrough**. Its relevance depends heavily on whether the industry stays on discrete-token rails (LNL wins) or successfully moves to continuous neuralese (LNL becomes a legacy artifact alongside FIPA-ACL).

### Recommendations for LNL design work

1. **Read and cite explicitly**: Wang & Wen 2025 ("AI-centric language"), LLMON, FIPA-ACL/KQML literature, ACE, Lojban grammar, Agora (Marro et al.), TOON spec.
2. **Empirical bootstrap-parse claim is the highest-risk assumption.** Validate that frontier LLMs (Claude 4/5, GPT-5, Gemini 3) can parse LNL zero-shot at the accuracy LNL needs. LLMLingua's "GPT-4 emergent decompression" result suggests this is plausible only for top-tier models.
3. **Differentiate from TOON sharply.** TOON already owns "compact, schema-aware, lossless JSON for LLMs." LNL's pitch must lead with the semantic-relation grammar, not the token efficiency.
4. **Position vs MCP/A2A as complement, not competitor.** LNL is payload; A2A is transport. The combination is coherent and underspecified by anyone today.
5. **Have an answer for the neuralese trajectory.** If/when continuous inter-agent communication wins, what role does LNL play? (Plausible answer: human-auditable interface layer between latent regions, like assembly between high-level languages and machine code.)

---

## Caveats

- **Field is moving fast.** Several of the cited papers (Interlat, Q-KVComm, Compressed CoT variants, Agora, NLIP ratification, A2A v0.3) are from 2025 or later; the literature snapshot will date quickly.
- **Adoption signals can mislead.** MCP and A2A have hundreds of integrations, but this reflects standardization races, not validated technical superiority. ACP merging into A2A within one year of launch shows how volatile this is.
- **"Token efficiency" claims need scrutiny.** TOON's 30–60% reduction, LLMLingua's 20×, and gist tokens' 26× are all measured under specific tokenizers (mostly cl100k/o200k), specific datasets, and specific tasks. LNL should benchmark on its actual target workloads.
- **Coconut's reasoning gains have been challenged** (arXiv 2512.21711, "Do Latent Tokens Think?") for relying on shortcut learning rather than genuine reasoning. The continuous-reasoning literature is not as settled as marketing suggests.
- **FIPA-ACL's failure to scale** is a sobering precedent. Speech-act-theoretic languages with rich semantic vocabulary historically lost to simpler JSON-over-HTTP approaches. LNL needs to articulate why this time is different (plausible answer: LLMs eliminate the ontology-alignment problem that killed FIPA).
- **The "no fine-tuning required" claim is bootstrap-phase only**, per LNL's own description. Long-term, fine-tuned LNL competence may be needed; at that point the comparison shifts to Coconut/CCoT/gist-token territory, where LNL loses on raw efficiency.
- **No first-party industry research lab (Anthropic, OpenAI, DeepMind, Meta, xAI) appears to be publicly working on a discrete LLM-targeted language with LNL's specific properties.** They are working on continuous reasoning (Meta), agent protocols (Anthropic-MCP, Google-A2A), and structured outputs (everyone). This absence is either an opportunity or a signal that the major players don't see the niche as valuable; both interpretations are defensible and the question deserves explicit treatment in LNL's own design rationale.