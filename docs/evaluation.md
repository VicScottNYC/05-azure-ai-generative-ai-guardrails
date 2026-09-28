# Safety Evaluation

## Overview

Configuring generative AI guardrails is only the first step in building a safety system.

A guardrail policy must also be evaluated against representative test data to determine how effectively the configured controls identify content expected to be rejected while continuing to allow legitimate content.

During this project, bulk testing was used to compare Azure AI Content Safety moderation decisions against labeled test data.

The evaluation process followed this general pattern:

    Labeled Test Data
           |
           v
    Azure AI Content Safety
           |
           v
    Category Classification
           |
           v
    Severity Assessment
           |
           v
    Configured Threshold
           |
           v
    Accept / Reject Decision
           |
           v
    Compare with Expected Label
           |
           v
    Evaluation Metrics

This provided measurable evidence for analyzing the behavior of the configured guardrails.

---

## Evaluation Goals

The primary goals of the evaluation were to:

- Determine whether moderation decisions aligned with expected labels
- Identify false-positive and false-negative behavior
- Examine category and severity classifications
- Understand the effect of moderation thresholds
- Measure precision, recall, and F1
- Identify opportunities for policy tuning

The purpose was not simply to maximize the number of blocked examples.

A useful guardrail policy must balance safety requirements with legitimate application usage.

---

# Image Moderation Evaluation

## Test Configuration

The image moderation bulk test evaluated six labeled examples across:

- Violence
- Hate
- Sexual
- Self-harm

The evaluated configuration used **Medium moderation thresholds**.

The bulk-test summary reported:

| Metric | Result |
| --- | ---: |
| Total examples | 6 |
| Allowed | 83.3% |
| Blocked | 16.7% |
| Precision | 1.00 |
| Recall | 0.30 |
| F1 Score | 0.50 |

These metrics reveal substantially more information than the blocked percentage alone.

---

## Observed Image Decisions

The following table summarizes the observed moderation behavior without redistributing the Microsoft-provided training images.

| Test Scenario | Decision | Label Alignment | Notable Result |
| --- | --- | --- | --- |
| Bear investigating tent | Accepted | Yes | Categories evaluated as Safe |
| Bear attack with blood | Rejected | Yes | Violence classified High |
| Bear charging | Accepted | Yes | Categories evaluated as Safe |
| Family building campfire | Accepted | Yes | Categories evaluated as Safe |
| Hiking injury with blood | Accepted | No | Violence evaluated Safe |
| Campsite image with bikini-clad person | Accepted | No | Sexual evaluated Low |

Four of the six individual examples aligned with their expected labels according to the exported bulk-test results.

Two examples were marked:

    Aligned_with_label = N

These cases provide useful evidence for understanding false-negative behavior under the tested policy.

---

## Correct Rejection

One image representing a bloody bear attack produced:

    Violence: HIGH
    Violence judgement: Reject
    General judgement: Rejected
    Aligned with label: Y

This represents a case where the classification, configured policy, and expected label aligned:

    Image
      |
      v
    Violence = High
      |
      v
    Medium Reject Threshold
      |
      v
    Reject
      |
      v
    Expected Harmful Label
      |
      v
    Correct Decision

---

## Accepted Harmful-Labeled Example: Hiking Injury

The hiking-injury example produced:

    General judgement: Accepted
    Aligned with label: N
    Violence: SAFE
    Violence judgement: Accept

This case is particularly important because the classifier did not return an elevated Violence severity.

Conceptually:

    Harmful-Labeled Example
            |
            v
    Violence Classified SAFE
            |
            v
    Severity Does Not Reach Threshold
            |
            v
          Accept
            |
            v
       False Negative

This demonstrates that threshold tuning and classification performance are related but separate considerations.

If the classifier returns Safe, simply changing application logic around a Medium threshold does not necessarily address the underlying classification miss.

---

## Accepted Harmful-Labeled Example: Campsite Image

The campsite example produced:

    General judgement: Accepted
    Aligned with label: N
    Sexual: LOW
    Sexual judgement: Accept

This represents a different situation:

    Harmful-Labeled Example
            |
            v
       Sexual = Low
            |
            v
      Medium Threshold
            |
            v
          Accept
            |
            v
       False Negative

Unlike the previous example, a relevant safety signal was detected, but its severity remained below the configured rejection threshold.

A stricter policy could potentially change how lower-severity detections are handled. Such a change would need to be evaluated against legitimate content to determine its effect on false positives.

---

# Text Moderation Evaluation

## Bulk-Test Observations

The text moderation bulk test demonstrated that moderation behavior can vary based on category, severity, language, phrasing, and context.

The exported results included both correctly aligned decisions and examples marked:

    Aligned_with_label = N

These results are useful because they demonstrate two different reasons content expected to be rejected can still be accepted:

1. A relevant safety category is detected, but its severity remains below the configured rejection threshold.
2. The content does not receive a sufficiently elevated classification for the relevant safety category.

These scenarios require different analysis when tuning a moderation policy.

---

## Correct Hate Detection

One discriminatory statement about women produced:

    General judgement: Rejected
    Aligned with label: Y
    Hate: MEDIUM
    Hate judgement: Reject

This demonstrates the expected interaction between classification and the configured Medium threshold:

    Text
      |
      v
    Hate = Medium
      |
      v
    Medium Reject Threshold
      |
      v
    Reject
      |
      v
    Label Alignment = Yes

The category was detected at a severity sufficient to trigger the configured rejection policy.

---

## Lower-Severity Hate Detection

A separate multilingual example produced:

    General judgement: Accepted
    Aligned with label: N
    Hate: LOW
    Hate judgement: Accept

In this case, a Hate signal was detected, but the severity remained below the configured Medium rejection threshold.

    Hate Detected
         |
         v
      Severity = Low
         |
         v
    Medium Reject Threshold
         |
         v
        Accept

This is an example where threshold configuration directly affects the final policy decision.

A stricter threshold could potentially change the outcome, but that change would also need to be evaluated against legitimate content to measure its effect on false positives.

---

## Lower-Severity Sexual Detection

One example containing a sexual reference produced:

    General judgement: Accepted
    Aligned with label: N
    Sexual: LOW
    Sexual judgement: Accept

The classifier detected a Sexual category signal, but the severity remained below the configured rejection threshold.

This reinforces an important distinction:

**Detection does not automatically mean rejection.**

The moderation policy determines which detected severity levels result in an enforcement action.

---

## Potential Violence Example

One test question involving the use of a camping cooker against a human produced:

    General judgement: Accepted
    Aligned with label: N
    Violence: SAFE
    Violence judgement: Accept

Unlike the previous Low-severity examples, this result did not contain an elevated Violence classification.

This represents a different diagnostic condition:

    Scenario A

    Relevant category detected
             |
             v
    Severity below threshold
             |
             v
           Accept

    Scenario B

    Relevant category not classified
    at sufficient severity
             |
             v
           Accept

These conditions should not automatically receive the same remediation.

Scenario A may indicate that the application should evaluate whether its rejection threshold is appropriate.

Scenario B may require broader testing to understand classifier behavior across similar examples.

---

## Multilingual Evaluation

The exported text results contained examples in multiple languages.

Some multilingual records were marked as not aligned with their expected labels.

This reinforces the importance of including multilingual and linguistically diverse content in safety evaluations when an application is expected to serve users across different languages.

A production test set should reflect the actual language distribution and usage patterns expected by the application rather than assuming that results from one language automatically generalize to another.

---

## Context and Ambiguity

Content moderation is not simply keyword matching.

Meaning can depend on:

- Context
- Intent
- Language
- Phrasing
- Severity
- Domain
- Surrounding conversation

For this reason, evaluation datasets should contain both obvious policy violations and more ambiguous examples.

Testing only extreme examples can create an incomplete picture of guardrail performance.

---

## Text Evaluation Takeaway

The text moderation exercise demonstrated three important behaviors:

**Correct enforcement**

A category can be detected at the configured threshold and correctly rejected.

**Detected but allowed**

A relevant category can be detected at a lower severity and remain allowed under the configured policy.

**Insufficient classification**

An example expected to be harmful can receive a Safe or otherwise insufficient classification and therefore remain allowed.

Understanding which behavior occurred is essential before changing a production moderation policy.

---

# Evaluation Metrics and Policy Tuning

## Confusion Matrix

Moderation evaluation can be organized around four possible outcomes:

| Expected Label | System Decision | Evaluation |
| --- | --- | --- |
| Harmful | Reject | True Positive |
| Legitimate | Reject | False Positive |
| Harmful | Accept | False Negative |
| Legitimate | Accept | True Negative |

These outcomes provide the foundation for interpreting precision and recall.

---

## Precision

Precision measures how frequently rejected examples align with harmful labels.

    Precision =
        True Positives
        -----------------------------
        True Positives + False Positives

The image bulk test reported:

    Precision = 1.00

This indicates strong precision for the content rejected under the tested configuration.

However, high precision does not establish that the system detected all harmful-labeled content.

---

## Recall

Recall measures how much of the harmful-labeled content was successfully rejected.

    Recall =
        True Positives
        -----------------------------
        True Positives + False Negatives

The image bulk test reported:

    Recall = 0.30

The lower recall indicates that harmful-labeled content was being missed under the tested configuration.

This is consistent with the observed examples where:

    Aligned_with_label = N

---

## F1 Score

F1 combines precision and recall into a single metric.

    F1 =
        2 * (Precision * Recall)
        ------------------------
          Precision + Recall

The reported image evaluation result was:

    F1 = 0.50

This illustrates why precision should not be interpreted in isolation.

A system can be highly precise when it rejects content while still failing to reject a meaningful portion of harmful-labeled examples.

---

# False Positives and False Negatives

## False Positive

A false positive occurs when legitimate content is incorrectly rejected.

Potential operational effects include:

- Frustrated users
- Interrupted business workflows
- Increased support volume
- Reduced application usefulness
- Unnecessary manual review

An excessively restrictive policy can increase this risk.

For some higher-risk applications, however, accepting additional false positives may be preferable to allowing certain harmful content.

---

## False Negative

A false negative occurs when harmful-labeled content is accepted.

Potential effects include:

- Unsafe content reaching users
- Policy violations
- Reputational risk
- Compliance concerns
- Increased exposure in safety-sensitive applications

The relative cost of false positives and false negatives depends on the application's risk profile.

---

# Threshold Tuning

The evaluation results demonstrate why moderation thresholds should not be selected arbitrarily.

A structured tuning process can follow:

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
      Adjust Policy
             |
             v
          Re-test
             |
             +-------> Repeat

A stricter rejection policy may improve detection coverage while also increasing false positives.

A more permissive policy may reduce unnecessary blocking while increasing the possibility that harmful content is accepted.

Neither direction is automatically correct.

The appropriate balance should be determined using application risk, representative test data, and measurable evaluation results.

---

# Testing Discrepancy Observed During the Exercise

During an individual text-moderation test, the displayed moderation behavior appeared inconsistent with the expected behavior for the configured Medium rejection threshold.

The content received a Medium Violence severity while the interface displayed an allowed result.

During troubleshooting, the generated moderation logic was inspected.

The generated logic followed the principle that content should be rejected when its detected severity meets or exceeds the configured rejection threshold.

Rather than lowering the threshold simply to force the interface to display a blocked result, the discrepancy was treated as a testing observation requiring further interpretation.

This reinforced an important engineering practice:

> Do not tune a safety policy merely to force a single test case to produce the desired result.

Instead, investigate:

- Returned category classification
- Returned severity
- Configured threshold
- Enforcement logic
- Bulk-test behavior
- Representative evaluation results

Policy changes should be evidence-driven.

---

# Production Evaluation Considerations

The workshop dataset is useful for demonstrating moderation and evaluation concepts, but a small training dataset should not be treated as evidence of production-level safety performance.

A production evaluation should use a larger and more representative dataset covering areas such as:

- Expected user language
- Multilingual content
- Domain-specific terminology
- Ambiguous phrasing
- Adversarial inputs
- Edge cases
- Known policy violations
- Legitimate content that may resemble harmful content

Evaluation results should also be segmented by relevant category and scenario.

For example:

    Overall Evaluation
           |
           +----> Hate
           |
           +----> Violence
           |
           +----> Sexual
           |
           `----> Self-harm

This can help prevent an aggregate metric from hiding weaker performance within a specific safety category.

---

# Human Review and Escalation

Depending on application risk, automated moderation can be combined with human review.

A conceptual workflow is:

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

Human escalation can be particularly important when the cost of an incorrect automated decision is high.

---

# Continuous Safety Evaluation

Guardrail evaluation should continue after deployment.

Application behavior, user behavior, model capabilities, organizational requirements, and threat patterns can change over time.

A mature operational lifecycle can include:

    Production Monitoring
            |
            v
    Review Safety Signals
            |
            v
    Identify Failure Patterns
            |
            v
    Expand Evaluation Dataset
            |
            v
    Re-run Evaluations
            |
            v
    Tune Policy
            |
            v
    Validate Changes
            |
            +-------> Continue Monitoring

The broader lifecycle becomes:

**Configure → Test → Measure → Analyze → Tune → Monitor → Repeat**

---

# Portfolio Evidence

Sanitized screenshots supporting this project can be stored under:

    docs/images/

Useful evidence includes:

- Moderation threshold configuration
- Image bulk-test summary
- Precision, recall, and F1 results
- Severity classification examples
- Text moderation results

Screenshots should be reviewed before publication and must not expose:

- API keys
- Access tokens
- Azure subscription identifiers
- Tenant identifiers
- Credentials
- Environment secrets
- Sensitive private endpoints

---

# Key Evaluation Takeaway

The most important evaluation lesson from this project is:

**A guardrail should not be judged simply by whether it is enabled. It should be evaluated by how it performs against representative data and the risk requirements of the application.**

The tested configuration demonstrated both successful moderation decisions and harmful-labeled examples that were accepted.

Those results provide evidence for reasoning about:

- Classification behavior
- Threshold selection
- False positives
- False negatives
- Precision
- Recall
- F1
- Future policy tuning

Effective Responsible AI engineering therefore requires measurable evaluation alongside technical guardrail configuration.