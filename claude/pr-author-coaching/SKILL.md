---
name: pr-author-coaching
description: "Coach one or more GitHub engineers using their recent pull-request history. Use for coaching-style analysis: collecting a bounded sample of recent PRs by author, reading human and AI review comments, inspecting the historical PR code state, and aggregating recurring strengths and recurring problems into evidence-backed guidance."
---

# PR Author Coaching

Read [../../workflows/pr-author-coaching.md](../../workflows/pr-author-coaching.md) before starting.

Follow the shared workflow:

- resolve the exact repo and PR sample first
- assess each PR as a bounded historical review-plus-code-quality unit
- stay local by default and use child sessions or subagents only when the sample is large enough to justify parallelism
- normalize recurring issues with the shared coaching rubric before aggregating them
- return a coaching-ready report in session unless the user explicitly asks for a draft artifact
