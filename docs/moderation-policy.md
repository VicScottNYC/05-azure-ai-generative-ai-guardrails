# Moderation Policy

## Overview

A content safety system does more than identify potentially harmful content. An application must determine how classification results translate into an operational policy.

This project explored how Azure AI Content Safety classifications, severity levels, moderation thresholds, and application decisions work together to determine whether content is accepted or rejected.

The core policy pipeline is:

    Content
       |
       v
    Safety Classification
       |
       v
    Category + Severity
       |
       v
    Configured Threshold
       |
       v
    Policy Decision
       |
       +----> Accept
       |
       `----> Reject

This separation between **classification** and **policy enforcement** is fundamental to designing effective generative AI guardrails.

---

## Safety Categories

The moderation exercises evaluated four primary content safety categories:

| Category | Purpose |
| --- | --- |
| Hate | Identifies hateful or discriminatory content |
| Violence | Identifies violent or threatening content |
| Sexual | Identifies sexual content |
| Self-harm | Identifies content involving self-harm |

A single piece of content can be evaluated independently across these categories.

For example, content may receive a higher severity classification for Violence while remaining Safe for Hate, Sexual, and Self-harm.

---

## Severity Levels

The exercises exposed moderation results using severity levels such as:

- Safe
- Low
- Medium
- High

Severity provides the signal used by the moderation policy.

A simplified interpretation is:

| Severity | General Interpretation |
| --- | --- |
| Safe | No detected content requiring action under the evaluated category |
| Low | Lower-severity detection |
| Medium | More significant detection that may trigger stricter policies |
| High | Higher-severity detection likely to require intervention |

Severity classification should not be confused with the application's final decision.

The application policy determines what to do with the classification.

---

## Classification vs. Enforcement

Consider content classified as:

    Category: Violence
    Severity: Medium

The classification describes what the safety system detected.

The moderation threshold determines what happens next.

Conceptually:

    Violence = Medium
           |
           v
    Compare with configured
    Violence threshold
           |
        +--+--+
        |     |
      Below  Meets/exceeds
        |     |
      Allow  Reject

This means the same classification can result in different application behavior when different policies are configured.

---

## Threshold-Based Policy

The exercises used configurable thresholds for the individual safety categories.

A stricter policy can reject content at a lower severity.

A more permissive policy can allow lower-severity detections and reject only content that reaches a higher threshold.

This introduces an important engineering tradeoff:

    Stricter Threshold
           |
           +--> Potentially catches more harmful content
           |
           `--> May increase false positives

    More Permissive Threshold
           |
           +--> May reduce unnecessary blocking
           |
           `--> May increase false negatives

The appropriate threshold therefore depends on application risk rather than on a universal setting.

---

## Category-Specific Policies

Not every application needs identical thresholds for every category.

Conceptually, a policy might evaluate categories independently:

    Hate --------> Threshold A
    Violence ----> Threshold B
    Sexual ------> Threshold C
    Self-harm ---> Threshold D

This allows organizations to adapt moderation behavior to:

- Application audience
- Business requirements
- Regulatory considerations
- Data sensitivity
- User population
- Organizational risk tolerance
- Expected content

The exercises demonstrated category-level threshold configuration rather than treating all harmful-content categories as a single binary filter.

---

## Policy Decision Logic

The generated moderation logic examined during the exercise demonstrated the underlying threshold concept.

At a high level:

    if detected_severity >= configured_reject_threshold:
        Reject
    else:
        Accept

The exact service representation of severity and thresholds is implementation-specific, but the engineering principle remains the same:

**Classification produces a safety signal. Policy determines the action.**

---

## Custom Blocklists

Severity-based moderation does not necessarily capture every organization-specific requirement.

Custom blocklists can supplement category-based moderation by detecting terms or expressions that require special handling.

Conceptually:

    Incoming Content
          |
          +--------> Category Moderation
          |
          +--------> Blocklist Evaluation
                          |
                          v
                    Match / No Match
                          |
                          v
                    Policy Decision

Blocklists can be useful when an organization has domain-specific restrictions that are not represented sufficiently by general safety categories.

They should be treated as an additional policy control rather than a replacement for semantic content moderation.

---

## Prompt Attack Protection

Prompt attacks represent a different problem from harmful-content classification.

For example, a prompt could attempt to override system instructions without containing content that receives a high Hate, Violence, Sexual, or Self-harm severity.

This creates two separate questions:

    Content Safety:
    "Is harmful content present?"

    Prompt Security:
    "Is this instruction attempting to manipulate
     the AI system?"

Prompt Shields can therefore complement content moderation as part of a defense-in-depth architecture.

---

## False Positives

A **false positive** occurs when a system blocks or flags content that should have been accepted according to the expected label or policy.

False positives can affect:

- User experience
- Legitimate business workflows
- Support volume
- Application usability
- Trust in automated moderation

An excessively restrictive policy can increase this risk.

For some applications, however, accepting additional false positives may be preferable to allowing high-risk harmful content.

The correct tradeoff depends on the use case.

---

## False Negatives

A **false negative** occurs when content expected to be rejected is accepted by the tested configuration.

False negatives are especially important in safety-sensitive applications because harmful content can pass through the guardrail.

The bulk tests performed during this project demonstrated this problem directly.

Some harmful-labeled examples were accepted under the configured Medium thresholds.

This reinforced an important lesson:

> Enabling content moderation does not guarantee that every harmful example will be detected and blocked.

Guardrail performance must be measured.

---

## Precision

Precision answers:

> Of the examples the system rejected as harmful, how many aligned with the harmful labels?

Conceptually:

    Precision =
        True Positives
        -----------------------------
        True Positives + False Positives

High precision indicates that when the tested system blocks content, those decisions frequently align with the harmful labels.

High precision alone does not prove that the system detects all harmful content.

---

## Recall

Recall answers:

> Of the examples labeled as harmful, how many did the tested system successfully reject?

Conceptually:

    Recall =
        True Positives
        -----------------------------
        True Positives + False Negatives

Low recall indicates that harmful-labeled examples are being missed.

This is why precision and recall should be considered together.

---

## F1 Score

F1 combines precision and recall into a single metric.

Conceptually:

    F1 =
        2 * (Precision * Recall)
        ------------------------
          Precision + Recall

A system with excellent precision but poor recall will therefore receive a lower F1 score.

This makes F1 useful when evaluating the balance between the two metrics.

---

## Observed Evaluation Tradeoff

During the image moderation bulk test, the configured policy produced:

| Metric | Result |
| --- | ---: |
| Precision | 1.00 |
| Recall | 0.30 |
| F1 Score | 0.50 |
| Allowed | 83.3% |
| Blocked | 16.7% |

The result demonstrates an important policy tradeoff.

The configuration showed high precision for the content it rejected, but substantially lower recall.

In practical terms, the tested configuration was selective about blocking content but did not reject all examples labeled as harmful in the exercise dataset.

That is precisely why moderation policies require evaluation and tuning.

---

## Threshold Tuning

A responsible threshold-tuning process can follow this cycle:

    Define Risk Requirements
             |
             v
      Configure Thresholds
             |
             v
       Run Test Dataset
             |
             v
       Measure Results
             |
             v
    Analyze FP / FN Cases
             |
             v
      Adjust Thresholds
             |
             v
          Re-test
             |
             +-------> Repeat

Threshold changes should be supported by evaluation evidence rather than changed simply to produce a desired test outcome.

This was particularly relevant during the exercises because observed moderation behavior needed to be interpreted using both the configured threshold and the returned severity classification.

---

## Production Policy Considerations

A production moderation policy should consider more than a single benchmark score.

Relevant considerations include:

- Who uses the application?
- What types of content are expected?
- What happens if harmful content is missed?
- What happens if legitimate content is incorrectly blocked?
- Are minors or other higher-risk populations involved?
- Does the application operate in a regulated environment?
- Is the system public-facing or internal?
- Can rejected content be reviewed or appealed?
- Should certain categories use stricter thresholds?
- How will moderation performance be monitored over time?

The answers influence how an organization balances safety, usability, and operational requirements.

---

## Human Review and Escalation

Automated moderation does not eliminate the need for human judgment in every scenario.

Depending on application risk, a policy can include an escalation path:

    Content
       |
       v
    Automated Evaluation
       |
       +----> Clearly Allowed ----> Continue
       |
       +----> Clearly Rejected ---> Block
       |
       `----> Ambiguous / High Risk
                     |
                     v
                Human Review

This is particularly useful when the cost of an incorrect automated decision is high.

---

## Continuous Evaluation

Guardrail policy should not be considered finished after initial deployment.

Application behavior, user behavior, model capabilities, organizational requirements, and threat patterns can change.

A mature process therefore includes:

**Configure → Test → Measure → Analyze → Tune → Monitor → Repeat**

This converts moderation from a static configuration into an ongoing Responsible AI engineering practice.

---

## Key Policy Takeaway

The primary lesson from this project is:

**The moderation model provides safety signals; the application policy determines how those signals affect users and system behavior.**

Effective guardrails therefore require both:

1. Reliable safety classification.
2. Deliberate policy design backed by measurable evaluation.

Neither component is sufficient by itself.