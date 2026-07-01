---
description: Ensures all changes are validated before being marked complete
inclusion: auto
---

# Validate Before Done

## Principle
Nothing is complete until it's proven working in the real environment. "It compiles" and "the test passed" are not the same as "a user can actually do this successfully."

## The Standard
Before saying something is done, you should be able to answer YES to: "If a real person used this right now, would it work?" — not in theory, but proven by actually triggering the code path.

## Multi-tenant Awareness
This is a white-label application. Features that work on one domain may break on another. Validate from the perspective of users on different domains when the change involves URLs, redirects, sessions, branding, or any user-facing path.
