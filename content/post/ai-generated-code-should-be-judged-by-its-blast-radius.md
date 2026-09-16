---
title: "LLM-Generated Code Should Be Judged by Its Blast Radius"
date: 2026-09-11T03:00:00+00:00
lastmod: 2026-09-16
slug: ai-generated-code-should-be-judged-by-its-blast-radius
tags: ["llm-assisted-coding", "software-engineering", "agents", "code-review", "production-systems"]
description: "If code generation is faster than review, teams should limit output by blast radius, require author ownership, and verify independent behavior before merge."
---

I like coding agents. I use them a lot. But I do not judge their output by how fast they type, I judge it by what breaks if they are wrong.

That is the blast radius question.

A script I run once on local data has small blast radius. A service that handles real users, money, or medical data has large blast radius. Those two cases should not go through the same review bar.

| Dimension | What I ask |
| --- | --- |
| Impact | If this fails, who gets hurt and how badly? |
| Exposure | How many users or systems can hit the failure? |
| Reversibility | Can we detect and undo the damage quickly? |
| Lifetime | Is this temporary code or long-lived production logic? |

If blast radius is high, verification has to be high. If blast radius is low, move fast.

## The main failure mode

Most teams do not fail because a Large Language Model (LLM) writes bad syntax. They fail because they accept changes no one can explain.

I keep seeing the same pattern:

| Pattern in PR | Risk it introduces |
| --- | --- |
| Big generated diff with little context | Reviewers miss behavior changes |
| New helper code where good code already exists | Duplicate logic and future drift |
| Tests that mock security boundaries | False confidence |
| Cross-module edits without clear ownership | Hidden coupling |

An LLM can produce 1,000 lines for a problem that needed 50. If the team cannot explain why each piece exists, the extra code is debt on day one.

## My bar before review starts

I treat review readiness as the author's job, not the reviewer's job.

Before I ask for domain review, I want this evidence:

| Requirement | Evidence |
| --- | --- |
| One clear purpose | Linked issue and user-visible problem |
| Author self-review | Author confirms they reviewed and trimmed the diff |
| Explainable approach | Why this design, what alternatives were skipped |
| Relevant checks | Test results for behavior that matters |
| Real dependencies | Docs for any new service, endpoint, or package |

This does not ban LLM use. It sets ownership.

If I find multiple major issues early, like missing ownership context, unverified security boundaries, or unrelated edits mixed together, I stop review, explain the pattern, and send it back. That keeps the queue healthy.

## Tests need independent expectations

If the same generation loop writes code and rewrites tests, you can get agreement without correctness.

I want important expectations defined independently first, then implementation validated against them.

For example, if unauthorized requests must never reach an operation, I want at least one test that exercises the real boundary. If every test mocks the boundary away, a passing suite means very little.

## Where I use LLM review

I still use LLMs in review. They are good at triage:

| Good use | Human decision still needed |
| --- | --- |
| Summarize large diffs | Verify summary against real patch |
| Flag suspicious lines | Decide if issue is real and important |
| Suggest missing tests | Confirm expected behavior |

I do not treat agreement between two LLMs as proof. Shared context can produce shared mistakes.

## Practical policy for small teams

If your team is drowning in generated PRs, I would do this first:

| Action | Why |
| --- | --- |
| Gate PR admission on readiness evidence | Review time goes to real changes |
| Cap in-flight review load | Generation pace matches verification capacity |
| Separate mechanical edits from behavior changes | Review focus improves |
| Track rework and incident-linked merges | Quality trend becomes visible |

The key point is simple: when generation outruns verification, reduce generation. The queue is already telling you the safe operating limit.

Fast code generation is useful. But in production systems, the winning move is still careful judgment about risk, scope, and ownership.
