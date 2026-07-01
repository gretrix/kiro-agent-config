---
inclusion: manual
---

# Human-Centered Design - Mandatory Validation Framework

## Foundational Principle

**Technology exists to serve humans. If a human cannot understand it, feel confident using it, and know what to do next, we have not finished the job.**

This is not a "nice to have." This is the definition of done.

## The Human Equation

Every piece of work must satisfy this equation:

```
Perceived Value > Perceived Cost
```

Where:
- **Perceived Value** = Does the user understand what they gain?
- **Perceived Cost** = Confusion, frustration, time spent figuring things out, anxiety

If the cost (confusion, frustration) exceeds the value (accomplishing their goal), the feature has failed—regardless of whether the code works.

## This Applies to EVERYTHING

Not just "UI features." Everything:
- Admin dashboards (admins are humans too)
- API error messages (developers reading them are humans)
- Configuration screens (operators are humans)
- Status displays (anyone monitoring is human)
- Log output (someone will read it)
- Email notifications (recipients are humans)
- Documentation (readers are humans)

**If a human will ever see it, read it, or interact with it, this framework applies.**

---

## PHASE 1: Before Building - Define the Human

Before writing any code, explicitly answer these questions. Document the answers.

### 1.1 Who Is the Target User?

Not "user" or "admin." Be specific:
- **Role**: What is their job/responsibility?
- **Goal**: What are they trying to accomplish? (Not "use the feature" but the real-world outcome)
- **Context**: When/why are they using this? (Rushed? Careful? Stressed? Routine?)
- **Frequency**: First-time? Daily? Occasional?

### 1.2 What Is Their Technical Range?

Every user group has a spectrum. Design for the FULL range:

| Dimension | Least | Most |
|-----------|-------|------|
| Technical knowledge | "What's an API?" | "Show me the raw JSON" |
| Domain expertise | "What's a foreclosure?" | "I've processed 10,000 notices" |
| System familiarity | "First time here" | "I use this daily" |
| Patience/time | "I have 30 seconds" | "I'll read every detail" |
| Confidence | "Am I doing this right?" | "I know exactly what I want" |

**The interface must work for EVERYONE in this range, not just the middle.**

### 1.3 What Are Their Emotional States?

Users aren't robots. They have feelings that affect how they perceive the interface:
- **Anxiety**: "Did I break something?" "Is this working?"
- **Frustration**: "Why is this so complicated?" "Where is the thing I need?"
- **Confusion**: "What does this mean?" "What should I do?"
- **Impatience**: "Just let me do the thing"
- **Satisfaction**: "That was easy" "I accomplished my goal"

**Design to minimize negative emotions and maximize positive ones.**

### 1.4 Document the Human Profile

Before building, create a brief profile:

```markdown
## Human Profile: [Feature Name]

**Primary User**: [Role and responsibility]
**Goal**: [What they're trying to accomplish in the real world]
**Context**: [When/why they're using this]

**Technical Range**:
- Least technical: [Description]
- Most technical: [Description]

**Emotional Considerations**:
- Primary concern: [What might worry them]
- Success feeling: [What should they feel when done]
```

---

## PHASE 2: During Building - Design for Humans

### 2.1 The First-Glance Test

For every screen, component, or output, a first-time user with zero context must be able to answer:

| Question | If Answer is "No" or "Unclear" |
|----------|-------------------------------|
| What is this for? | Add clear heading/description |
| What is the current state/status? | Add status indicators with plain language |
| Is everything working or is something wrong? | Add clear success/error states |
| What should I do next? | Add clear call-to-action or guidance |
| What do these terms mean? | Simplify terminology or add explanations |

**If ANY answer is unclear, the design is incomplete.**

### 2.2 Zero Mental Gymnastics Rule

**Users should NEVER have to:**
- Leave the site to understand something (Google a term, look up a conversion)
- Do mental math or conversions (UTC to local time, bytes to MB, etc.)
- Remember technical formats (cron syntax, regex, date formats)
- Cross-reference information from multiple places
- Guess what will happen when they click something

**The system handles ALL translations invisibly:**

| User Input | System Responsibility |
|------------|----------------------|
| Time selection | Accept in user's local timezone, convert to server timezone internally |
| Date display | Always show in user's local timezone with relative context ("Tomorrow at 2am") |
| File sizes | Show human-readable (2.5 MB, not 2621440 bytes) |
| Durations | Show human-readable ("2 hours 15 minutes", not 8100 seconds) |
| Technical formats | Provide UI that generates them (schedule picker → cron, not cron input) |
| Currency | Show in user's currency with proper formatting |
| Numbers | Use locale-appropriate formatting (1,234.56 vs 1.234,56) |

**Implementation Pattern:**
```typescript
// BAD: User has to know UTC offset
<input type="time" /> // Saves as-is, displays as-is

// GOOD: System handles timezone invisibly
<input type="time" /> // Input in user's TZ, convert to server TZ on save
                      // Convert back to user's TZ on display
```

**Validation Question:** "Does the user need to know ANYTHING technical to use this correctly?"
- If YES → Redesign to hide the complexity
- If NO → Proceed

### 2.3 Terminology Audit

For every label, status, message, and button:

| Check | Bad Example | Good Example |
|-------|-------------|--------------|
| Would a non-developer understand? | "Partial" | "Stopped early - 15 of 159 counties processed" |
| Does it explain "so what?" | "Parse failures: 0" | "All notices processed successfully" |
| Is there a simpler word? | "Propagate configuration" | "Apply settings" |
| Does it guide action? | "Error occurred" | "Connection failed. Check your internet and try again." |
| Is jargon necessary? | "Cron expression" | "Schedule" (with advanced option for cron) |

**Rule: If you have to explain what something means, the label is wrong.**

### 2.4 The "So What?" Test

Every piece of information displayed must answer: **"So what? Why should I care?"**

| Data Point | Without "So What?" | With "So What?" |
|------------|-------------------|-----------------|
| "15/159 counties" | User thinks: "Is that good? Bad? Why?" | "15 of 159 counties processed. Stopped due to rate limiting. Will resume automatically in 1 hour." |
| "Captcha Rate: N/A" | User thinks: "What does N/A mean here?" | "No captchas encountered this run" |
| "Status: Partial" | User thinks: "Partial what? Is this a problem?" | "Partially complete - some counties couldn't be reached. See details below." |

### 2.5 Emotional Design Checklist

| State | User Should Feel | Design Must Include |
|-------|------------------|---------------------|
| Success | Confident, accomplished | Clear success message, what was achieved, what happens next |
| Failure | Informed, not blamed, knows how to recover | What went wrong (plain language), why it happened, how to fix it |
| Warning | Alert but not alarmed, knows what to do | What to be aware of, whether action is needed, what happens if ignored |
| Loading | Patient, informed | What's happening, rough time estimate if possible |
| Empty state | Guided, not confused | Why it's empty, what to do to populate it |

### 2.6 Range Accommodation

Design must work for the full user range:

**For the least technical user:**
- [ ] Can they complete the task without understanding technical details?
- [ ] Are there plain-language explanations for everything?
- [ ] Is the happy path obvious and easy to follow?
- [ ] Are errors explained in terms they understand?

**For the most technical user:**
- [ ] Can they access detailed information if they want it?
- [ ] Are there "advanced" options that don't clutter the simple path?
- [ ] Can they see raw data/logs if needed?
- [ ] Is efficiency prioritized for power users?

**For the first-time user:**
- [ ] Is onboarding implicit in the design?
- [ ] Are there hints/tooltips for non-obvious features?
- [ ] Is the purpose of each element clear?
- [ ] Can they succeed without reading documentation?

**For the daily user:**
- [ ] Are common actions quick to access?
- [ ] Is there keyboard navigation for efficiency?
- [ ] Are defaults sensible for repeat use?
- [ ] Does the interface remember their preferences?

---

## PHASE 3: Before Completing - Validate with Humans in Mind

### 3.1 The Stranger Test

Pretend you are a stranger who has never seen this system. Walk through the feature and document:

1. **First impression**: What do I think this is for?
2. **Orientation**: Do I know where I am and what I can do?
3. **Terminology**: Are there any words I don't understand?
4. **Next action**: Is it obvious what I should do?
5. **Confidence**: Do I feel confident or anxious?

**If any answer reveals confusion, fix it before marking complete.**

### 3.2 The Edge Case Communication Test

For every possible state, verify the user experience:

| State | What User Sees | What User Understands | What User Does Next |
|-------|---------------|----------------------|---------------------|
| Loading | [Describe] | [What they think is happening] | [Wait? Refresh? Leave?] |
| Empty | [Describe] | [Why it's empty] | [How to populate] |
| Success | [Describe] | [What was accomplished] | [What to do next] |
| Partial success | [Describe] | [What worked, what didn't, why] | [How to complete/retry] |
| Failure | [Describe] | [What went wrong, why] | [How to fix/recover] |
| Error | [Describe] | [What the error means] | [Specific recovery steps] |

### 3.3 The Demographic Walkthrough

Walk through the feature as each user type:

**As the least technical user:**
- Can I complete my goal without understanding how it works?
- Do I feel confident or confused?
- Would I need to ask someone for help?

**As the most technical user:**
- Can I get the details I need?
- Is there unnecessary hand-holding slowing me down?
- Can I access advanced features easily?

**As a first-time user:**
- Do I understand what this is for?
- Can I figure out how to use it without instructions?
- Do I know if I succeeded?

**As a daily user:**
- Can I do my common tasks quickly?
- Is there unnecessary friction for routine actions?
- Does the interface respect my expertise?

### 3.4 The Emotional Audit

For each interaction point, identify the likely emotional response:

| Interaction | Likely Emotion | Is This Acceptable? | If No, How to Fix |
|-------------|---------------|---------------------|-------------------|
| [Action 1] | [Emotion] | [Yes/No] | [Fix] |
| [Action 2] | [Emotion] | [Yes/No] | [Fix] |

**Acceptable emotions**: Confident, informed, satisfied, efficient, clear
**Unacceptable emotions**: Confused, anxious, frustrated, lost, blamed

---

## PHASE 4: Documentation - The Human Validation Report

For every feature, create a brief validation report:

```markdown
## Human Validation Report: [Feature Name]

### Human Profile
- **User**: [Who]
- **Goal**: [What they're trying to accomplish]
- **Technical range**: [Least to most technical]

### First-Glance Test Results
- Purpose clear: [Yes/No - if No, what was done]
- Status clear: [Yes/No - if No, what was done]
- Next action clear: [Yes/No - if No, what was done]

### Terminology Audit
- Terms simplified: [List any changes made]
- Jargon removed: [List any changes made]

### Range Accommodation
- Least technical user: [Can complete task? Yes/No]
- Most technical user: [Has needed details? Yes/No]
- First-time user: [Can succeed without help? Yes/No]
- Daily user: [Efficient for routine use? Yes/No]

### Emotional Audit
- Primary emotion at completion: [What user should feel]
- Negative emotions mitigated: [List what was addressed]

### Edge Cases Verified
- [List each state and how it's communicated]
```

---

## Integration with Other Steering Documents

This document is **foundational**. Other steering documents handle specific aspects:

- `user-experience-first.md` → Form validation UX (subset of this)
- `automated-testing-workflow.md` → Functional testing (complements this)
- `test-before-complete.md` → E2E testing (complements this)

**Hierarchy:**
1. Human-Centered Design (this document) - Is it usable by humans?
2. UX Validation - Is the interaction smooth?
3. Functional Testing - Does the code work?
4. E2E Testing - Does the flow complete?

**A feature is not complete unless ALL levels pass.**

---

## The Definition of Done

A task is complete when:

1. ✅ **Code works** (functional testing)
2. ✅ **Tests pass** (automated testing)
3. ✅ **Build succeeds** (technical validation)
4. ✅ **Human can understand it** (first-glance test)
5. ✅ **Human knows what to do** (next-action clarity)
6. ✅ **Human feels confident** (emotional audit)
7. ✅ **Works for full user range** (demographic walkthrough)
8. ✅ **Edge cases communicate clearly** (state communication)

**If any of these fail, the task is not complete.**

---

## Quick Reference Checklist

Before marking ANY work complete:

### Minimum Viable Human Check
- [ ] Would a stranger understand what this is for?
- [ ] Would a stranger know what to do next?
- [ ] Would a stranger feel confident, not confused?
- [ ] Are all terms understandable without technical knowledge?
- [ ] **Zero Mental Gymnastics**: Does the user need to do ANY conversions, lookups, or calculations?
- [ ] **Zero Mental Gymnastics**: Are all times shown in the user's local timezone?
- [ ] Do all states (loading, empty, error, success) communicate clearly?
- [ ] Does success feel like success?
- [ ] Does failure explain how to recover?

### If ANY answer is "No" → Fix before completing.

---

## Remember

> "The best interface is one that doesn't require explanation."

> "If you have to tell the user what something means, the design has failed."

> "Technology that confuses humans is broken technology, regardless of whether the code works."

**Quality over speed. Do it right the first time.**

---

**This framework is MANDATORY for all work. No exceptions.**
