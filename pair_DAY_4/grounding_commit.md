# Grounding Commit — Day 4

**Artifact edited:** Week 11 — AI Sales Agent Evaluation Bench
**Specific location:** datasheet.md — contamination section

## What Changed

Before today, my datasheet.md contamination section listed three checks (n-gram overlap, embedding similarity, time-shift verification) and stated that the benchmark passed all three — implying the benchmark is uncontaminated. This claim was technically accurate but incomplete in a way that would not survive scrutiny from a careful reviewer.

After understanding the distinction between held-out partition contamination and base model pre-training contamination today, I made two concrete changes:

1. Renamed the section from "Contamination Checks" to "Contamination Controls and Limitations" to signal upfront that the section covers both what is controlled and what is not.

2. Added a "Limitations" paragraph at the end of the section with the following content: "The three checks above verify held-out partition integrity within this benchmark's data pipeline — they confirm that no tasks from the held-out set leaked into the training partition during our preparation. They do not address base model pre-training contamination. The Qwen base model was trained on internet-scale data whose full contents are not publicly disclosed. It is possible that tasks structurally similar to those in this benchmark appeared in Qwen's pre-training corpus. Membership inference attacks exist for probing this but require white-box model access and are not performed here. Evaluators should treat base model pre-training contamination as an open risk when interpreting benchmark results."

## Why This Matters

The original contamination section made an implicit completeness claim it could not support. The revised section is honest about the boundary of what the checks prove — which is a stronger and more defensible position than presenting contamination as fully solved.

## Commit Pointer

File: `datasheet.md`
Changes: Section renamed, limitations paragraph added distinguishing held-out partition integrity from base model pre-training contamination.
