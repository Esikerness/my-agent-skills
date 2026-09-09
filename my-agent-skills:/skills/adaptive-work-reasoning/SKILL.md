---
name: adaptive-work-reasoning
description: A work-focused derivative of arc-skill that uses explicit hypotheses, small safe tests, evidence tracking, and verified completion for complex, ambiguous, or repeatedly failing tasks across coding, research, integrations, file work, and browser workflows.
---

# Adaptive Work Reasoning

This skill is derived from `arc-skill`. It was adapted at the user's request for practical work execution: an AI was asked to rework the original approach into a compact, evidence-driven workflow for real coding, debugging, research, file, integration, and browser tasks.

Use a compact hypothesis-driven loop. The goal is not to deliberate longer; it is to learn from each check and avoid repeating an unsupported assumption.

## Before acting

- Restate the requested outcome, the exact deliverable, and the boundaries of the task.
- Separate known facts, assumptions, and open questions.
- Identify the cheapest safe observation or test that can distinguish the leading hypotheses.
- For a change, inspect the relevant source and existing behavior first. Preserve working behavior, routes, identifiers, content, responsive behavior, and integrations unless the user asked to change them.

## The working loop

For each meaningful step:

1. State a short prediction: what should change, what should stay unchanged, and what evidence would confirm it.
2. Perform the smallest useful action.
3. Compare the actual result with the prediction, including errors, diffs, logs, visual state, or returned data.
4. Update the working model. Mark a rule as verified only when evidence supports it; keep uncertain ideas explicitly provisional.
5. Record durable decisions, failed approaches, constraints, and the next test when the task spans multiple steps or a context reset could matter.

After a failed prediction, stop and revise the hypothesis before taking another dependent action. A failed test is useful evidence, not a reason to conceal uncertainty.

## Choosing verification

Match verification to the risk and the requested outcome:

- source or static checks prove what was written, not what a deployed page or external service does;
- syntax, type, and unit checks prove narrower invariants;
- browser, form, API, mail, publication, and mobile checks prove live behavior;
- visual inspection proves layout only at the tested viewport and state.

Report these outcomes separately. Do not call a task live-verified when only source inspection passed. If live verification is unavailable, say exactly what remains untested and provide the safest next check.

## Efficient execution

- Batch only steps whose result is already predictable and reversible or low-risk.
- Keep exploration granular, especially before irreversible writes, publication, deletion, purchases, messages, or production changes.
- Reuse confirmed facts and existing project conventions instead of rediscovering them.
- Use tools for evidence, not as a substitute for a plan. Prefer targeted searches and focused file reads.
- If the task cannot be completed safely because a required choice, credential, artifact, or external state is missing, explain the blocker and ask one focused question.

## Handoff

Finish with: what changed or was concluded, what was verified and how, what remains uncertain, and any exact file paths or next action the user needs. For code or documents, provide the complete requested artifact when the user asks for the full code or file.
