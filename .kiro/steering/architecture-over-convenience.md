---
inclusion: manual
---
# Architecture Over Convenience

## Principle
When the data already tells you WHEN or WHERE something should happen, don't poll — listen. Choose the approach that matches the nature of the data, not the one that's fastest to code.

## Application
- If you know the exact future time an action should occur → use a timer/event, not a polling interval
- If you know which record changed → react to that change, don't scan the whole table
- If the system already emits a signal → subscribe to it, don't poll for its effect

## The Test
Before adding any periodic check, interval, or cron job, ask: "Does the system already know when this should happen?" If yes, schedule it directly rather than polling to discover it.

## When Polling IS Appropriate
- The trigger time is unknown and unpredictable (external system changes)
- Multiple independent actors can create work at any time (no single event source)
- The polling cost is negligible AND precision doesn't matter

Document the reasoning when choosing polling over event-driven design.
