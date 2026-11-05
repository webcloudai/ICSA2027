# ICSA 2027 — focused architectural-validation artifacts

This directory accompanies the paper **Engineering Observable and Reliable Multi-Agent AI Systems for Feedback-Driven Improvement** (the ICSA submission). It contains **recorded outputs of controlled, non-production experiments**, not source code, a runnable implementation, or production dataset. The validation claims and limitations are described in the paper's **Focused Architectural Validation** section. The original execution environment and its dependencies are not distributed here.

## Contents and relation to the paper

| Folder | Paper experiment | Contents | What to inspect |
| --- | --- | --- | --- |
| `causal_reconstruction/` | Causal reconstruction (I1); comparison table and figure | Three standalone HTML visualizations and corresponding per-scenario `traces/` JSON exports | Compare the explicit activation-predecessor `links` with span parentage and timestamps under sequential, fan-out/fan-in, and branched fan-out/fan-in execution. |
| `contract_definition_versioning/` | Executable-definition versioning (I5); V1–V4 table | Five ordered pairs of contract-definition snapshots and associated Proposal Selector activation evidence | Compare the top-level `contract.version` against the activation's `contract_version`; inspect changed projection and interface identities. |
| `retrospective_diagnosis/` | Retrospective diagnosis of silent behavioral failures | One expected and one unexpected invocation; each includes an HTML execution report, operational traces, and selected semantic evidence and definition snapshots | Follow the proposal selector and Judge evidence. In the unexpected invocation the Proposal Selector yields an empty proposal list; examine the input action ordering and compare the effective definition versions. |
| `typed_control-flow_closure/` | Typed control-flow closure / structured exceptional-path recording (I4) | A Judge failure-evidence JSON record | Inspect `stage`, `failure_type`, `failure_message`, invocation/activation identity, predecessor link, inputs, bounds, definition identity, and absent semantic output. |

### 1. Causal reconstruction

Open these self-contained HTML visualizations in a browser:

- `01_sequential_flow.html` — sequential routing execution.
- `02_fan-out_in_flow.html` — parallel fan-out and join.
- `03_fan-out_in_branches_flow.html` — branched parallel execution and join.

Their associated JSON traces are stored in `traces/01_sequential_flow/`, `traces/02_fan-out_in_flow/`, and `traces/03_fan-out_in_branches_flow/`, respectively. Individual files represent spans or log events; use the shared trace/invocation identifiers to group records. For node spans, inspect `parentSpanId`, time bounds, `attribites` (the spelling in these captured exports), `evidence.activation_id`, and `links`. The links encode activation-level predecessors, which should not be inferred from temporal proximity alone. The HTML views are convenient visual aids; the JSON exports are the inspectable records. The comparison of reconstruction effort in the paper is a qualitative assessment, not a timing benchmark.

### 2. Executable-definition versioning

Files are prefixed `00` through `04` to denote the ordered test cases. Each case includes:

- `NN_contract_AgentRouter__RouterActionSelector.json`: effective executable-definition snapshot (not merely the formal contract tuple).
- `NN_proposal_selector_<activation-id>.json`: activation-local semantic evidence referencing that effective identity via `contract_version`.

`00` is the baseline. `01` (V1) changes the input projection; `02` (V2) reverts it; `03` (V3) changes the input interface; `04` (V4) changes the bounds projection/bound object. In particular, confirm that the identity at `02` equals `00`, while `01`, `03`, and `04` differ from `00`. Compare individual projection `version` fields as well as the top-level definition `version`. Each pair is from a separate invocation. These are selected snapshots, not complete execution traces for those invocations.

### 3. Retrospective diagnosis

`invocation_with_expected_results/` and `invocation_with_unexpected_results/` each contain:

- `report_<invocation-id>.html` — standalone visual execution report.
- `traces/*.json` — exported operational spans and log events.
- `evidences/*.json` — selected Proposal Selector and Judge activation evidence plus their effective definition snapshots.

The expected execution presents feasible actions in order `CANNED`, `REJECT`, `Agent_Spec001`, produces a proposal and obtains Judge authorization for `pCANNED`. The unexpected execution presents the same action alternatives in order `REJECT`, `Agent_Spec001`, `CANNED`; the Proposal Selector emits no proposals, and the Judge takes its no-proposals fallback (`pNO_PROPOSALS`). The compared Proposal Selector and Judge activation records refer to the same respective executable-definition identities across the two invocations. These records support the stated **ordering-dependent behavior** diagnosis; they do not by themselves provide source-level proof of the precise coding defect, nor do they show that all other possible environmental effects were ruled out.

### 4. Typed control-flow closure / exceptional path

`selected_proposals_judge_978f718e-6525-4316-b564-59169f9062f4.json` captures the injected Judge logic exception as a failure record (`stage: logic`, `failure_type: ComponentExecutionError`, `output: null`). It preserves the execution boundary's evidence even though no normal semantic decision was produced. **This folder does not include the full downstream operational trace or the corresponding definition snapshot**, so it independently demonstrates the recorded failure fields, not the separate claim that the Controller executed no unauthorized action. The paper reports that observation from the controlled experiment.

## Reading and interpretation notes

- All artifacts represent **sample executions**, not a statistically representative evaluation or a benchmark. No user feedback, training, or production use is evaluated here.
- The JSON is exported implementation evidence, not a standardized interchange schema. In operational-span exports, the key `attribites` is intentionally retained as recorded. Some log-event exports use `attributes` instead. Timestamp and status representations may also differ across exports.
- `evidence.invocation_id`, `evidence.node_id`, and `evidence.activation_id` in trace records allow activation correlation; semantic-evidence JSON commonly also includes `invocation_id`, `node_id`, `activation_id`, `causal_predecessors`, and `contract_version`. Some contract snapshot files have `evidence.node_id` identifying the enclosing agent rather than the `node_id` of the semantic component; use `node_id` and the linked version when matching definitions.
- Definition snapshots and semantic-evidence records are **curated subsets**. Some referenced predecessors or definitions are outside a given subfolder; lack of a local counterpart does not mean the execution lacked that activation or definition.
- The HTML files embed visualization dependencies and can be opened locally; their size is dominated by the embedded plotting library. To independently inspect captured evidence, use the JSON records rather than treating diagram geometry or the report's rendering as authoritative.
- These materials enable *inspection* of recorded observations, not re-execution of the experiments. They are not a replacement for publicly released implementation code or an automated reproduction script.

## Minimal inspection workflow

1. Read the corresponding experiment paragraph and table in the paper.
2. Open the HTML report where provided to find the relevant nodes and execution path.
3. Open the associated JSON evidence and/or trace files; match invocation and activation identities before comparing records.
4. For versioning, compare `contract.version` in each `NN_contract_*.json` with `contract_version` in the corresponding `NN_proposal_selector_*.json`.
5. For retrospective diagnosis, compare the Proposal Selector's `input.feasible_actions` and `output.proposals`, then the Judge's `input.proposals` and `output.decision` across the two invocations.

