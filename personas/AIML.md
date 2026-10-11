# AIML.md: Sr AI/ML Engineer persona (Ajay Madkaiker)

Ajay owns *how* the healthcare assistant is built and makes the final technical call at each AI-DLC gate. He pairs with Dana (Data Engineer) on data and with Priya (AI PM) on scope and quality.

## Background

| Field | Detail |
|---|---|
| Current role | Lead Software Developer at a global risk and compliance company (2021 to present) |
| Experience | 15+ years in software; about 4 to 5 years on AI and LLM systems in production |
| Domain | Financial compliance and risk: AML, sanctions screening |
| Education | MS Computer Science (UT Arlington), BS Computer Science (University of Mumbai) |
| Certifications | Apache Spark, Azure Data, Google AI Professional |
| Location | Chicago, IL |

## Proof points he brings to this project

- **Human-in-the-loop agent in production (ASM case review).** An API invokes the agent, the UI shows its narrative, a reviewer makes the decision, and the system persists it with an audit trail. This is the pattern for the clinician review queue in the [React UI story](../stories/react-ui-and-deployment.md).
- **Screening agents.** He configured dataset-level settings for the account screening and transaction screening agents.
- **Production scale.** He built Java and Spring Boot REST APIs that screen millions of transactions a day against sanctions and AML watchlists at sub-second latency.
- **RAG done carefully (SupportDesk-RAG).** It uses OpenAI embeddings, LlamaIndex indexing and LangChain retrieval, plus an evaluation pipeline, anti-hallucination safeguards and agentic RAG with tool calling.
- **Multi-agent orchestration (AI Email Assistant).** A LangGraph pipeline classifies intent, styles tone, then reviews and scores quality. Claude is the primary model, with GPT-4o as fallback for cost, latency and quality tradeoffs.
- **Delivery tooling.** Docker, Kubernetes, Helm, Terraform, Jenkins, Argo CD, Prometheus and Grafana, plus LangSmith and GitHub Copilot with spec-driven development.
- **Security.** OAuth 2.0, IAM policies, SSL/TLS, and audit trails built to regulatory standards.
- **Mentoring.** He mentors juniors on Spring Boot, REST design and microservices, and he leads technical design reviews.

## Skills

- **AI/ML:** LLMs, RAG, agentic AI, LangGraph, LangChain, LlamaIndex, LangSmith, guardrails, prompt engineering, fine-tuning, vector search (Pinecone, Chroma)
- **Languages:** Python, Java, Go, Node.js
- **Frameworks:** Spring Boot, Spring Cloud, WebFlux, Angular, React
- **Data:** PostgreSQL, Oracle, SQL Server, MongoDB, Elasticsearch, Kafka, RabbitMQ
- **Cloud:** AWS, Azure

## Why AML maps to healthcare

Sanctions screening and clinical decision support share the same shape:

1. Generate a candidate answer.
2. Show the evidence behind it.
3. Score its confidence.
4. Hold it at a human approval gate.
5. Keep an audit trail no one can edit.

Ajay applies that shape to the assistant. Model answers for flagged topics go to a clinician queue before any patient sees them.

## What he owns in this project

- Model and serving choices: Qwen 1.5B with QLoRA, adapter versioning, vLLM or SageMaker
- Units and workflow planning in AI-DLC Inception
- Code review of agent-generated code, and Build and Test in Construction
- Backend and API design for the React UI, plus the human-in-the-loop review flow
- The go or no-go call on deployment, made with Priya

## Growth areas he is working on

- Owning more production Python (his production depth so far is mostly Java)
- Hands-on AWS Bedrock and SageMaker work (his LangGraph and LangChain skills carry over)
- ML-specific governance: model cards, prompt versioning, drift monitoring, rollback

## How he talks

Direct and focused on architecture. He asks "what's the failure mode?" before "what's the feature?" He pushes back on prompt-tuning marathons: "If the data is wrong, fix the data."
