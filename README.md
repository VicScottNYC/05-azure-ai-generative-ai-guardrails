# Azure AI Generative AI Guardrails

A Responsible AI portfolio project demonstrating the implementation, testing, and evaluation of generative AI safety controls using Microsoft Azure AI Content Safety and Microsoft Foundry.

This project explores how application-level guardrails can detect harmful content, enforce moderation policies, evaluate safety performance, and reduce risk in generative AI applications.

---

## Project Objectives

This project demonstrates practical experience with:

- Text content moderation
- Image content moderation
- Hate, violence, sexual, and self-harm classifications
- Severity-based moderation thresholds
- Allow and reject policy decisions
- Custom blocklists
- Prompt Shields
- Groundedness detection concepts
- Bulk safety testing
- Precision, recall, and F1 evaluation
- False-positive and false-negative analysis
- Responsible AI policy tuning

---

## Technologies

- Microsoft Foundry
- Azure AI Content Safety
- Microsoft Entra ID
- Azure Responsible AI capabilities
- Generative AI safety evaluation

---

## Guardrail Architecture

A production generative AI application can apply safety controls before and after model inference.

    User / Application
            |
            v
    +-----------------------+
    |   Input Guardrails    |
    |-----------------------|
    | Content Moderation    |
    | Custom Blocklists     |
    | Prompt Shields        |
    +-----------------------+
            |
            v
    +-----------------------+
    |  Generative AI Model |
    +-----------------------+
            |
            v
    +-----------------------+
    |   Output Guardrails   |
    |-----------------------|
    | Content Moderation    |
    | Groundedness Checks   |
    | Policy Enforcement    |
    +-----------------------+
            |
            v
       Allow / Block

The exact controls used in a production architecture depend on the application's risk profile and the Azure AI safety capabilities being integrated.

See [Guardrail Architecture](docs/architecture.md) for additional details.

---

## Content Safety Categories

The moderation exercises evaluated content across four primary safety categories:

| Category | Purpose |
| --- | --- |
| Hate | Detects hateful or discriminatory content |
| Violence | Detects violent or threatening content |
| Sexual | Detects sexual content |
| Self-harm | Detects content involving self-harm |

Classification results can be combined with configurable severity thresholds to determine whether content should be allowed or rejected.

---

## Moderation Policy

The exercises demonstrated an important distinction between **classification** and **policy**.

### Classification

What category and severity level does the safety system assign to the content?

### Policy

At what severity should the application reject the content?

This produces a decision process similar to:

    Content
       |
       v
    Classification
       |
       v
    Severity
       |
       v
    Configured Threshold
       |
       v
    Policy Decision
       |
       +----> Allow
       |
       +----> Reject

This separation is important because detecting a category does not automatically determine what an application should do with the content.

See [Moderation Policy](docs/moderation-policy.md).

---

## Safety Evaluation

Bulk moderation testing was used to compare Azure AI Content Safety decisions against labeled test data.

The image moderation exercise demonstrated why evaluating a safety system requires more than simply counting blocked content.

At the configured Medium thresholds, the bulk image test produced:

| Metric | Result |
| --- | ---: |
| Precision | **1.00** |
| Recall | **0.30** |
| F1 Score | **0.50** |
| Allowed | **83.3%** |
| Blocked | **16.7%** |

The high precision indicated that content blocked by the tested configuration aligned strongly with the harmful labels.

The lower recall showed that harmful-labeled examples were still being accepted.

This illustrates an important Responsible AI engineering principle:

> A moderation policy must balance detection coverage with the operational and safety costs of false positives and false negatives.

Threshold selection should therefore be driven by application risk, representative testing, evaluation results, and organizational policy rather than simply maximizing the amount of content blocked.

See [Safety Evaluation](docs/evaluation.md) for the detailed analysis.

---

## Additional Guardrail Capabilities

### Custom Blocklists

Custom blocklists provide an additional policy layer for terms or expressions that an organization wants to detect according to its own requirements.

They can complement category-based moderation when an application has domain-specific terminology or organizational restrictions.

### Prompt Shields

Prompt Shields help protect generative AI systems against adversarial prompt attacks, including attempts to manipulate model behavior through malicious instructions.

Prompt attack protection represents a different security concern from harmful-content classification and should be treated as part of a layered AI security architecture.

### Groundedness Detection

Groundedness detection can help evaluate whether generated responses are supported by source material supplied to an application.

This is particularly relevant for retrieval-augmented generation and enterprise knowledge applications where responses are expected to remain supported by trusted information.

---

## Responsible AI Engineering Lessons

The exercises reinforced several principles that apply to production AI systems:

1. Safety classification and application policy are separate concerns.
2. Moderation thresholds should be tested against representative data.
3. False negatives can remain even when precision is high.
4. False positives and false negatives have different business and safety costs.
5. Guardrails should be evaluated using measurable metrics.
6. Safety controls should be implemented as layers rather than relying on a single protection mechanism.
7. Guardrail policies should be reviewed and tuned as application behavior, data, and risk requirements change.
8. Safety evaluation should be treated as an engineering lifecycle rather than a one-time configuration task.

---

## Evaluation Lifecycle

The project demonstrates the following safety-engineering workflow:

    Content
       |
       v
    Classification
       |
       v
    Severity Assessment
       |
       v
    Moderation Threshold
       |
       v
    Policy Decision
       |
       v
    Bulk Evaluation
       |
       v
    Precision / Recall / F1
       |
       v
    Policy Tuning

This lifecycle provides a repeatable approach for evaluating whether guardrail settings are appropriate for an application's risk profile.

---

## Repository Structure

    05-azure-ai-generative-ai-guardrails/
    |
    |-- README.md
    |-- .gitignore
    |
    `-- docs/
        |-- architecture.md
        |-- moderation-policy.md
        |-- evaluation.md
        `-- images/

---

## Documentation

- [Guardrail Architecture](docs/architecture.md)
- [Moderation Policy](docs/moderation-policy.md)
- [Safety Evaluation](docs/evaluation.md)

---

## Security

No credentials, API keys, Azure endpoints, tenant information, subscription identifiers, or environment-specific secrets are stored in this repository.

Local environment files and development artifacts are excluded through `.gitignore`.

Screenshots included in this repository are sanitized before publication to remove sensitive Azure environment information.

---

## Evaluation Evidence

The following screenshots provide sanitized evidence from the guardrail testing and evaluation performed during this project.

### Image Moderation Bulk Evaluation

Bulk image moderation testing was used to compare Azure AI Content Safety decisions against labeled examples and evaluate the configured policy using precision, recall, and F1.

![Azure AI Content Safety image moderation bulk evaluation](docs/images/image-moderation-bulk-evaluation.png)

**Observed evaluation results:**

- Allowed: **83.3%**
- Blocked: **16.7%**
- Precision: **1.00**
- Recall: **0.30**
- F1: **0.50**

The results demonstrate why moderation effectiveness should not be evaluated using precision alone. Although rejected content showed strong precision in this test, the lower recall identified harmful-labeled examples that remained allowed.

### Text Moderation Threshold Testing

Individual text moderation testing was also performed across the Violence, Hate, Sexual, and Self-harm categories using Medium rejection thresholds.

![Azure AI Content Safety text moderation threshold test](docs/images/text-moderation-threshold-test.png)

One observed test produced a Medium Violence classification while the interface displayed an Allowed decision. Rather than modifying the threshold simply to force a blocked result, the behavior was investigated and documented as an evaluation observation.

This reinforces an important Responsible AI engineering principle: **guardrail policies should be tuned using representative evaluation data and measurable results rather than individual expected outcomes.**

For the complete analysis, see [`docs/evaluation.md`](docs/evaluation.md).

---

## Training Context

This portfolio project was developed while completing Microsoft Learn training focused on implementing generative AI guardrails with Azure AI Content Safety and Microsoft Foundry.

The repository documents implementation concepts, testing observations, evaluation results, and engineering lessons from the exercises.

Microsoft-provided workshop datasets, images, notebooks, application code, and other training assets are not redistributed in this repository.

---

## Portfolio Progression

This project is part of a broader Microsoft Foundry and Azure AI engineering portfolio covering:

- Generative AI applications
- AI agents
- Agent-driven workflows
- Enterprise knowledge grounding
- Responsible AI and generative AI guardrails
- AI evaluation and production engineering