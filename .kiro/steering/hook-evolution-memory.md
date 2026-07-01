---
inclusion: fileMatch
fileMatchPattern: '**/hooks/**,**/steering/**'
---
# Hook Evolution Memory

## Principle
Before modifying any hook or steering rule, read `.kiro/hooks/CHANGELOG.md` first. It contains the history of why each change was made, what was tried before, and what lessons were learned. Changing a hook without understanding its history risks reverting a fix or repeating a mistake.

## When Changing Hooks or Steering
1. Read the CHANGELOG to understand prior evolution
2. Make the change
3. Append a new entry to the CHANGELOG with: what changed, why, and the lesson

## When the User Complains About Agent Behavior
That complaint is a signal that a hook or steering rule needs updating. Document the complaint verbatim in the CHANGELOG entry so future sessions can see the exact feedback that drove the change.
