---
name: superpowers
description: Collection of development workflow skills for Claude Code. Includes brainstorming, debugging, code review, test-driven development, plan execution, and more. Use as a meta-skill that delegates to specific sub-skills based on the task. Triggered by any development task that could benefit from structured workflows.
---

# Superpowers

A collection of development workflow skills that enhance Claude Code's capabilities. Each sub-skill handles a specific phase of development.

## Available Sub-Skills

| Sub-skill | When to use |
|-----------|-------------|
| `brainstorming` | Before any creative work - features, components, design |
| `dispatching-parallel-agents` | 2+ independent tasks without shared state |
| `executing-plans` | Executing written implementation plans |
| `finishing-a-development-branch` | Implementation complete, ready to integrate |
| `requesting-code-review` | Before merging, verify work meets requirements |
| `subagent-driven-development` | Executing plans with independent sub-tasks |
| `systematic-debugging` | Any bug, test failure, or unexpected behavior |
| `test-driven-development` | Before writing implementation code |
| `verification-before-completion` | Before claiming work is complete |
| `writing-plans` | Multi-step tasks, before touching code |

Each sub-skill has its own SKILL.md in its respective directory with detailed instructions.
