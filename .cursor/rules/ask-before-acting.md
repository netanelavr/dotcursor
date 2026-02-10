---
description: Ask clarifying questions before starting any task
alwaysApply: true
---

# Ask Before Acting

On the **first message** of any task, pause and gather context before implementing.

## Required Steps

1. **Understand the request** - Identify what's being asked
2. **Check for gaps** - What's missing or ambiguous?
3. **Ask 2-4 targeted questions** from the relevant categories below

## Question Categories

### General (always consider)
- Scope: What exactly should be included/excluded?
- Constraints: Performance, compatibility, existing patterns to follow?
- Edge cases: How should unusual inputs be handled?
- Success criteria: How will we know it's done correctly?

### Feature / Data Flow (when the task touches backend → frontend data)
- Data lifecycle: How is it streamed/requested, persisted, retrieved, and displayed?
- Protocol: Does it follow existing contracts (e.g., AI SDK event types) or need new ones?
- State management: What state fields are added/reset across start, complete, clear, error?
- Observability: Does it need Langfuse metadata, contextLogger entries, or ReporterService metrics?
- UI spec: Rendering format, animations, component style, reusable components?
- Edge cases: Empty content, tool-call-only messages, existing filters/guards that need updating?

## When to Skip Questions

Skip if the request is:
- A simple bug fix with clear reproduction steps
- A direct question (not a task)
- Explicitly marked as "just do it" or similar
- A follow-up to a conversation where context was already gathered

## Example

**User**: "Add caching to the API"

**Good response**: 
- What data should be cached? (responses, computed values, both?)
- What invalidation strategy? (TTL, event-based, manual?)
- Any existing caching patterns in the codebase I should follow?
- Expected cache hit rate or performance target?

**Bad response**: Immediately implementing Redis caching without asking.
