# Personas

The three people who build the healthcare assistant together, and how they split the AI-DLC lifecycle.

| Persona | File | Owns | Leads in AI-DLC |
|---|---|---|---|
| Ajay Madkaiker, Sr AI/ML Engineer | [AIML.md](AIML.md) | How: model, serving, architecture, human-in-the-loop flow, go or no-go | Workflow Planning, Units Generation, Construction |
| Dana, Data Engineer | [DE.md](DE.md) | Data: training and eval sets, pipelines, lineage, PHI handling | Reverse Engineering (data audit), data units |
| Priya, AI PM | [PM.md](PM.md) | What and why: metrics, scope, clinician review, gap decisions | Requirements Analysis, Rules Validation, production gate |

Dana and Priya are composite personas, not real people.

Working rule: no gate closes until all three have answered the question file. Ajay breaks ties on *how*, and Priya breaks ties on *whether*.

The first story they work on together is the [React UI and deployment plan](../stories/react-ui-and-deployment.md).
