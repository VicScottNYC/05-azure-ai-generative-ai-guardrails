# Guardrail Architecture

## Overview

Generative AI guardrails should be implemented as a layered architecture rather than as a single moderation control.

This project examines how Azure AI Content Safety and related Microsoft Foundry safety capabilities can be positioned around a generative AI application to evaluate user input, protect model interactions, inspect generated output, and enforce application-specific safety policies.

The conceptual architecture is:

    User / Application
            |
            v
    +---------------------------+
    |     Input Guardrails      |
    |---------------------------|
    | Content Moderation        |
    | Custom Blocklists         |
    | Prompt Shields            |
    +---------------------------+
            |
            v
    +---------------------------+
    |    Generative AI Model    |
    +---------------------------+
            |
            v
    +---------------------------+
    |     Output Guardrails     |
    |---------------------------|
    | Content Moderation        |
    | Groundedness Evaluation   |
    | Application Policy        |
    +---------------------------+
            |
            v
       Allow / Block

The individual capabilities address different risks and should not be treated as interchangeable controls.

---

## 1. User and Application Layer

The process begins when a user or application submits content to a generative AI system.

Depending on the solution, this content might include:

- User prompts
- Chat messages
- Documents
- Images
- Retrieved enterprise knowledge
- Application-generated context

Before this information is passed to a generative AI model, safety controls can be applied according to the application's risk profile.

---

## 2. Input Guardrails

Input guardrails evaluate content before model inference.

Their purpose is to identify content or instructions that violate the application's safety or security policy.

### Content Moderation

Azure AI Content Safety can classify content across safety categories such as:

- Hate
- Violence
- Sexual
- Self-harm

The classification result provides information that an application can use when making a policy decision.

The exercises demonstrated that classification alone does not determine whether content should be blocked.

The application must evaluate the detected severity against its configured moderation threshold.

Conceptually:

    Input
      |
      v
    Classification
      |
      v
    Severity
      |
      v
    Threshold Comparison
      |
      +----> Below threshold ----> Allow
      |
      `----> At/above threshold -> Reject

---

## 3. Custom Blocklists

Category-based moderation may not cover every organization-specific restriction.

Custom blocklists provide an additional mechanism for detecting terms or expressions that an organization wants to control according to its own policies.

This creates another policy layer alongside general safety classification.

For example, an application may need controls for:

- Organization-specific prohibited terminology
- Domain-specific harmful expressions
- Known abusive phrases
- Terms that require additional review

Blocklists should complement broader content classification rather than replace it.

---

## 4. Prompt Shields

Generative AI systems face risks that differ from conventional harmful-content classification.

An input may appear relatively harmless from a content-moderation perspective while still attempting to manipulate the AI system.

Prompt Shields are designed to help identify adversarial prompt attacks.

This distinction is important:

    Content Moderation
           |
           +--> What kind of potentially harmful
                content is present?

    Prompt Shields
           |
           +--> Is an instruction attempting to
                manipulate or attack the AI system?

A layered AI security architecture can use both controls because they address different threat categories.

---

## 5. Generative AI Model

After input passes the application's required controls, the request can be processed by the generative AI model.

Passing input validation does not mean the model's output should automatically be returned to the user.

Generated content can require its own evaluation.

This creates a second guardrail boundary after inference.

---

## 6. Output Guardrails

Output guardrails inspect or evaluate generated responses before they are returned to the user or downstream application.

Potential controls include:

- Content moderation
- Groundedness evaluation
- Application-specific policy enforcement

This creates a pattern such as:

    Approved Input
          |
          v
       AI Model
          |
          v
    Generated Response
          |
          v
    Output Evaluation
          |
          +----> Pass ----> Return response
          |
          `----> Fail ----> Block / handle

---

## 7. Groundedness

Groundedness addresses a different problem from harmful-content moderation.

For applications that provide trusted source material, groundedness evaluation can help determine whether a generated response is supported by that material.

This is particularly relevant for retrieval-augmented generation and enterprise knowledge systems.

Conceptually:

    Trusted Sources
          |
          v
    Retrieved Context ----+
                          |
                          v
    User Prompt ------> AI Model
                          |
                          v
                   Generated Answer
                          |
                          v
                  Groundedness Check
                          |
                    +-----+-----+
                    |           |
                 Supported   Unsupported

A response can therefore be safe from a content-moderation perspective while still being insufficiently supported by the supplied source material.

The two risks require different controls.

---

## 8. Policy Enforcement

Safety services provide signals that applications can use when making decisions.

The application still needs an enforcement policy.

A simplified decision pipeline is:

    Classification
          |
          v
       Severity
          |
          v
    Policy Threshold
          |
          v
    Application Decision
          |
      +---+---+
      |       |
    Allow   Reject

The exercises demonstrated why this distinction matters.

Changing a threshold changes application behavior even when the underlying content classification remains the same.

---

## 9. Evaluation and Feedback

A production guardrail architecture should include an evaluation loop.

    Guardrail Configuration
             |
             v
        Test Dataset
             |
             v
      Moderation Results
             |
             v
    Compare Against Labels
             |
             v
    Precision / Recall / F1
             |
             v
       Analyze Errors
             |
             v
       Tune Policy
             |
             +--------> Repeat

This converts guardrail configuration from a one-time administrative task into an engineering lifecycle.

Evaluation can reveal:

- False positives
- False negatives
- Overly restrictive thresholds
- Insufficient detection coverage
- Categories requiring different policy treatment

---

## 10. Defense in Depth

No single guardrail capability addresses every generative AI risk.

A stronger architecture combines multiple controls:

    +--------------------------------------+
    |        Application Security          |
    +--------------------------------------+
                     |
    +--------------------------------------+
    | Prompt / Input Protection            |
    | - Prompt Shields                     |
    | - Input validation                   |
    +--------------------------------------+
                     |
    +--------------------------------------+
    | Content Safety                       |
    | - Hate                               |
    | - Violence                           |
    | - Sexual                             |
    | - Self-harm                          |
    | - Custom blocklists                  |
    +--------------------------------------+
                     |
    +--------------------------------------+
    | Model / Grounding Layer              |
    | - Generative AI model                |
    | - Trusted source context             |
    +--------------------------------------+
                     |
    +--------------------------------------+
    | Output Evaluation                    |
    | - Content moderation                 |
    | - Groundedness                       |
    | - Application policy                 |
    +--------------------------------------+
                     |
    +--------------------------------------+
    | Monitoring and Evaluation            |
    | - Test datasets                      |
    | - Precision                          |
    | - Recall                             |
    | - F1                                 |
    | - Policy tuning                      |
    +--------------------------------------+

This defense-in-depth approach allows different controls to address different AI safety and security risks.

---

## Key Architecture Takeaway

The primary architectural lesson from this project is that **guardrails are not a single filter placed in front of an AI model**.

A mature generative AI solution can require controls across the entire request lifecycle:

**Input → Threat Detection → Content Classification → Policy Enforcement → Model Inference → Output Evaluation → Groundedness → Monitoring → Continuous Tuning**

The appropriate combination of controls depends on the application's data, users, business requirements, and risk profile.