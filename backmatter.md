# Back Matter

## Glossary

- abstention: trained model behavior of declining to answer when evidence is insufficient
- acceptance test: a fixed evaluation a model configuration must pass before deployment
- accuracy floor: the minimum measured performance a deployment gate requires
- batch packing: serving multiple requests in one inference batch; a source of run-to-run nondeterminism
- checkpoint: a saved snapshot of model or training state, used for recovery
- CMMS: computerized maintenance management system; system of record for work orders
- constrained decoding: restricting generation so only tokens legal under a grammar can be emitted
- context window: the bounded buffer of text a model reads verbatim while generating
- distillation: training a small model on a larger model's outputs to transfer capability
- edge box: compute device deployed near the machines, outside the data center
- enum decode: constrained decoding where the output must be one of a fixed set of values
- error bar: the measured spread of a benchmark score across repeated runs
- evaluation gate: automated test rig that decides whether a model change may ship
- fault code: a machine's structured identifier for a specific failure condition
- fine-tuning: continuing a model's training on domain data to specialize it
- grammar: a formal specification of legal output structure used in constrained decoding
- grounding: placing trusted documents in the context window and requiring answers from them
- hallucination: fluent, specific, wrong output produced by normal model operation
- historian: time-series database recording plant process values (tags) over time
- inference: running a trained model to produce output (as opposed to training it)
- inference engine: software that loads model weights and executes generation
- interlock: a control condition that must hold before an operation is permitted
- KV cache: inference memory holding attention state for the current context
- ladder logic: graphical PLC programming language scanned top to bottom
- latency: time from request to first (or complete) response
- local model: a model whose weights run on hardware the operator owns
- lossy compression: storage that preserves regularities but not exact values; how weights hold training text
- MoE (mixture of experts): architecture activating a subset of weights per token
- NPSH: net positive suction head; pump property referenced in cavitation analysis
- parameter: one learned weight; model size is quoted in parameters (millions/billions)
- PLC: programmable logic controller; the computer that runs a machine's control program
- protocol frame: one structured unit of an industrial communication protocol
- quantization: storing model weights at reduced numeric precision to shrink memory
- register: an addressed data location on an industrial device readable over a protocol
- retraction: published withdrawal of a prior claim, with the reason stated
- sampling: the policy for choosing the next token from the model's probability distribution
- schema: the required structure of an output (fields, types, enumerations)
- serving: running an inference engine as a persistent network service
- tag: a named point in a historian or SCADA system (e.g. a temperature reading)
- temperature: sampling parameter controlling randomness; zero picks the most probable token
- tier: a named model size class in this book's ladder (from sub-billion to tens of billions)
- token: the sub-word unit a model reads and writes
- tokenizer: the fixed procedure that splits text into tokens
- unified memory: hardware design where CPU and GPU share one memory pool
- VFD: variable frequency drive; motor controller and a rich source of fault codes
- weights: the learned numbers that constitute a trained model
- working memory: informal term for the context window, by analogy to human cognition

## Lab citation convention

In-text markers of the form `[LAB: RESULTS-MATRIX §C]` or `[LAB: PROJECT-LOG
2026-08-03]` resolve into the RogerAI Labs lab record: RESULTS-MATRIX sections hold
configuration tables with measurements; PROJECT-LOG entries are dated experiment
narratives. A third form, `[LAB: CLAUDE.md …]`, resolves into the lab's standing
charter — the operating-principles and "watch the traps" notes that record serving and
hardware lessons as durable rules rather than dated one-off runs. Claims without a
`[LAB:]` marker are labeled unmeasured in the prose.

## References

- O'AILLY platform machinery (gates, standards, review pipeline): https://github.com/oailly-press/platform
- llama.cpp — open-source inference engine used throughout the lab work: https://github.com/ggml-org/llama.cpp
- C2PA content provenance standard: https://c2pa.org/
- Authors Guild, "AI Best Practices for Authors" (disclosure landscape): https://authorsguild.org/resource/ai-best-practices-for-authors/
- RogerAI Labs lab record: RESULTS-MATRIX.md / PROJECT-LOG.md — in-text `[LAB:]` markers name the section or dated entry used.

*(References grow with the chapters; every citation must resolve at Pass 1.)*
