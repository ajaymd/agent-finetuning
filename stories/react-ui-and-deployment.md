# Story: React UI and deployment for the healthcare assistant

**Goal:** Put the fine-tuned healthcare assistant (Qwen2.5-1.5B with the v2 QLoRA adapter) behind a React web app. Patients can ask questions, clinicians review flagged answers, and the team can ship new adapter versions safely.

**Team:** [Ajay](../personas/AIML.md) (Sr AI/ML Engineer), [Dana](../personas/DE.md) (Data Engineer), [Priya](../personas/PM.md) (AI PM)

**Process:** AI-DLC. v1 rules cover Inception and Construction, and v1-operations rules cover Rules Validation, Deployment and Post-Deployment Testing. Every stage ends at an approval gate, and every decision goes in `aidlc-docs/audit.md`.

---

## 1. The story, day by day

### Monday: framing (Inception: Requirements Analysis)

> **Priya:** Clinics want patients to self-serve on common questions. But a wrong answer about chest pain is a lawsuit. I need two things: patients get answers in seconds, and anything risky waits for a clinician.
>
> **Ajay:** That's the case-review pattern I shipped in financial compliance. The agent drafts the answer and shows its evidence, a human approves or edits it, and we persist the decision with an audit trail. Low-risk answers go straight out, and flagged ones go to a queue.
>
> **Dana:** Then I need to know exactly what we store. Patient text is PHI. My proposal is that we never log raw prompts. We log metadata only, and the clinician queue keeps the text encrypted with a retention limit.
>
> **Priya:** Agreed. The metrics bar: safety and accuracy at or above v2 on the LangSmith eval, and no new helpfulness loss. P95 latency is TBD; we'll set it after the load test.

**Gate:** Priya approves `requirements.md`. Ajay and Dana sign the same question file.

### Tuesday: planning (Inception: Workflow Planning, Units Generation)

> **Ajay:** Five units. One, the React chat UI. Two, the clinician review console. Three, the API and guardrails service. Four, model serving. Five, the eval and release pipeline.
>
> **Dana:** I take the data side of units three and five: PHI masking, trace metadata, and the eval set wired into the release pipeline.
>
> **Priya:** The review console is the one I care about most. If clinicians hate it, the whole thing fails.

**Gate:** Ajay approves the unit plan. Priya confirms that every unit traces to a metric.

### Wednesday: build (Construction)

> **Ajay:** The agents generate the React screens and the FastAPI service from the functional design. I review every pull request. The risk router decides queue or no queue, and its rules are in the design doc, not hidden in a prompt.
>
> **Dana:** The eval set is frozen. The release pipeline runs the 10 benchmark prompts plus the held-out set against any new adapter before it can deploy.
>
> **Priya:** I'll run the clinician walkthrough of the review console this afternoon.

**Gate:** Build and Test is green, and the eval scores meet the Monday bar.

### Thursday: ship to staging (Operations: Rules Validation, Deployment)

> **Ajay:** Rules validation flagged two gaps: no alarm on queue backlog, and no rollback runbook. Both are implementation gaps.
>
> **Priya:** Fix both now. Neither is an accepted exception.
>
> **Dana:** I'll add the LLM metrics: token counts, time to first token, latency at P50 and P99, and errors per adapter version. No prompt text.

### Friday: go or no-go (Operations: Post-Deployment Testing)

> **Ajay:** Staging passed the functional tests. The load test is the open item.
>
> **Priya:** Then the status is "staging, pending load test", and the canary waits for it. I'll put that in the roadmap.
>
> **Dana:** Next week's run is reproducible. The data version, adapter version and eval scores are all linked in the lineage log.

---

## 2. User stories and acceptance criteria

| # | As a... | I want... | So that... | Acceptance criteria | Owner |
|---|---|---|---|---|---|
| US-1 | Patient | to ask a health question in a chat window | I get a clear answer without calling the clinic | Streams a response; shows a safety disclaimer on every answer; works on mobile; meets WCAG 2.1 AA | Ajay |
| US-2 | Patient | to be told when a clinician must review my question | I know an answer is coming and why it's delayed | Flagged questions show a "reviewing" state and an expected wait; emergency keywords show call-911 guidance at once, with no model call | Ajay, Priya |
| US-3 | Clinician | a review queue with the draft answer and the reason it was flagged | I approve, edit or reject quickly | Approve, edit and reject actions; each decision saved with reviewer, time and diff; nothing reaches the patient until approved | Ajay |
| US-4 | Clinician | to rate answers | the team learns where the model is weak | A thumbs or 1–5 rating stored with the adapter version; ratings feed the weekly eval review | Dana, Priya |
| US-5 | Data engineer | PHI masked before anything is logged or traced | we stay HIPAA-safe | Names, dates of birth, phone numbers and emails are masked in logs and LangSmith traces; tests prove the masking | Dana |
| US-6 | PM | a dashboard of answer volume, flag rate, review time and eval scores per adapter version | I can make go or no-go calls with data | Dashboard is live in staging; every metric is broken down by adapter version | Dana, Priya |
| US-7 | ML engineer | to roll back to the previous adapter in one step | a bad release costs minutes, not days | Adapter version is a config value; rollback is tested in staging and documented in the runbook | Ajay |

---

## 3. Architecture

```text
 Patient browser                  Clinician browser
 React chat UI (US-1, US-2)       React review console (US-3, US-4)
            \                         /
             \     HTTPS + auth      /
              v                     v
        ┌──────────────────────────────────┐
        │  API service (FastAPI, Python)   │
        │  - auth (Cognito / OAuth 2.0)    │
        │  - PHI masking (US-5)            │
        │  - emergency keyword check       │
        │  - risk router: send now / queue │
        │  - review queue + audit table    │
        └───────────────┬──────────────────┘
                        │  internal, in the VPC
                        v
        ┌──────────────────────────────────┐
        │  Model serving                   │
        │  Qwen2.5-1.5B + v2 adapter       │
        │  (merged), vLLM or SageMaker     │
        └──────────────────────────────────┘
   Traces (metadata only) -> LangSmith      Metrics -> CloudWatch / Grafana
```

- **Frontend:** React with TypeScript (Vite), streaming responses over server-sent events, and a component library with accessible defaults.
- **API:** Python FastAPI, so the API and the model code use the same language. The risk router uses explicit rules plus a classifier score, and both are versioned.
- **Generation settings:** temperature 0.2, top_p 0.6, max_new_tokens 384. These are inside the ranges the Module 3 guide recommends. The SageMaker example in Module 3 uses temperature 0.7, which is too high for medical answers.

---

## 4. Deployment plan

### Where to serve the model

From Module 3, section 14 of the notes:

| Stage | Option | Why |
|---|---|---|
| Dev | Local GPU or Colab, with the adapter loaded from HF Hub | Fast iteration; the adapter toggle lets you compare base and fine-tuned outputs |
| Staging | SageMaker real-time endpoint on `ml.g4dn.xlarge` (T4), merged model | A 1.5B model in 4-bit fits on a T4. The Module 3 guide quotes about $0.74/hr |
| Production | Choose after the load test: SageMaker real-time, or vLLM on ECS/EKS if one GPU must serve several adapters | vLLM's continuous batching gives 2 to 4× throughput on the same GPU (Module 3) |

Not for production: RunPod and the free HF Inference API. They are not HIPAA-eligible, and the free API can't serve a custom adapter.

### Environments and gates

1. **Build.** A pull request triggers lint, unit tests, a UI build, and the PHI-masking tests.
2. **Eval gate.** For a new adapter, the pipeline runs the 10 benchmark prompts and Dana's held-out set through LangSmith. Safety and accuracy must be at or above the current production adapter, or the release stops.
3. **Staging deploy.** Infrastructure as code (Terraform, or CDK if the team prefers AWS-native). The frontend goes to S3 and CloudFront. The API runs on ECS Fargate, and the model on a SageMaker endpoint, all in a private VPC with encryption at rest and in transit.
4. **Rules Validation (v1-operations).** Run the observability, recovery and runbook rules, including the LLM rules: lifecycle metrics, and no prompt or response content in logs by default. Priya decides each gap.
5. **Post-deployment tests.** Functional tests, red-team prompts (emergency, self-harm, medication dosing), a clinician walkthrough, and a load test.
6. **Production rollout.** First, shadow mode for one week: the new adapter answers in parallel and only clinicians see it. Then a canary at 10% of traffic. Then full traffic. Rollback means switching the adapter version in config (US-7).

### Who does what

| Step | Ajay | Dana | Priya |
|---|---|---|---|
| Infrastructure as code and endpoint | Builds | Reviews the data stores and retention rules | — |
| Eval gate | Defines the thresholds with Priya | Owns the eval set and the pipeline step | Sets the bar |
| Rules Validation | Fixes design gaps | Fixes data and logging gaps | Decides fix or accept |
| Shadow and canary | Watches latency and errors | Watches data drift and flag rate | Reads clinician feedback |
| Go or no-go | Recommends | Confirms lineage is complete | Decides |

### Compliance checklist (before any real patient data)

- [ ] A Business Associate Agreement (BAA) with the cloud provider, and only HIPAA-eligible services in use
- [ ] PHI masking tested on logs, traces and error reports
- [ ] An audit trail for every clinician decision, kept in a store that can't be edited
- [ ] Every answer carries the disclaimer "Please consult a healthcare professional for personalized medical advice."
- [ ] Emergency guidance hard-coded and checked before the model is called
- [ ] Security review of auth, IAM roles and network paths

---

## 5. Open questions

- What P95 latency target does a patient chat need? Set it after the load test.
- Which topics are flagged for review on day one? Priya and a clinician draft the list.
- Do we need several adapters at once (for example per specialty)? If yes, vLLM on ECS is the better production choice.
