---
inclusion: manual
---

# DRY Coding Principles - Fortune Leo Project

## Core Principle
**Don't Repeat Yourself (DRY)**: Every piece of knowledge must have a single, unambiguous, authoritative representation within the system.

## Mandatory Checks Before Implementation

### 1. Component Creation Checklist
Before creating ANY new component, you MUST:

- [ ] Search `app/components/quiz/` for similar UI patterns
- [ ] Search `app/components/partner/` for similar functionality
- [ ] Check if a shared component already exists
- [ ] Review `CODE_REUSABILITY_AUDIT.md` for known patterns
- [ ] Ask: "Can I reuse or extend an existing component?"

**Example:** Before creating a new progress indicator, check if `QuizProgress.tsx` can be reused.

### 2. Business Logic Checklist
Before implementing business logic, you MUST:

- [ ] Check `lib/` directory for existing utilities
- [ ] Look for similar validation in `lib/validation/`
- [ ] Look for similar conditional logic in `lib/quiz/` or `lib/partner/`
- [ ] Check if the pattern exists in another flow
- [ ] Consider creating a shared utility if logic is generic

**Example:** Before writing email validation, check if it exists in `lib/validation/`.

### 3. Type Definition Checklist
Before defining new types, you MUST:

- [ ] Check `app/types/` for existing similar types
- [ ] Look for base types that can be extended
- [ ] Consider if this type shares fields with other types
- [ ] Use TypeScript extension (`extends`) rather than duplication
- [ ] Document why flow-specific types are necessary

**Example:** Instead of duplicating `id`, `createdAt`, `updatedAt` fields, extend a `BaseEntity` type.

### 4. API Route Checklist
Before creating API routes, you MUST:

- [ ] Check existing routes for similar patterns
- [ ] Look for shared validation middleware
- [ ] Look for shared error handling patterns
- [ ] Consider extracting common logic to utilities
- [ ] Use consistent response formats

**Example:** Use shared `withValidation` and `withErrorHandling` wrappers.

## Shared Component Library

### Current Shared Components
These components are PROVEN to work across multiple flows:

1. **QuizProgress** (`app/components/quiz/QuizProgress.tsx`)
   - Used by: Quiz flows, Partner intake
   - Props: `currentQuestion`, `totalQuestions`
   - Purpose: Progress bar for multi-step flows

2. **GoogleAddressAutocomplete** (`app/components/quiz/GoogleAddressAutocomplete.tsx`)
   - Used by: Quiz flows
   - Purpose: Address input with Google Places API
   - Can be used: Anywhere address input is needed

### Shared Utilities

1. **Validation** (`lib/validation/`)
   - Partner schemas
   - Common validation patterns

2. **Conditional Logic** (`lib/quiz/conditionalLogic.ts`, `lib/partner/conditionalLogic.ts`)
   - Show/hide logic based on answers
   - Dependency checking

3. **Database Queries** (`lib/db/`)
   - CRUD operations
   - Transaction management

## Implementation Patterns

### Multi-Step Forms
When implementing multi-step forms:
- Use `QuizProgress` for progress indication
- Follow the state management pattern from `QuizContainer` or `PartnerIntakeContainer`
- Implement auto-save to localStorage
- Handle next/back navigation consistently
- Scroll to top on step change

### Form Fields
When rendering form fields:
- Check `QuestionCard.tsx` for existing field types
- Reuse field rendering logic where possible
- Consider creating a unified `FormField` component for new field types

### Validation
When validating user input:
- Use shared validation utilities
- Don't write custom email/phone/URL validators
- Extend existing validation schemas

## When Duplication is Acceptable

Duplication is acceptable ONLY when:

1. **Performance Critical**: Shared code would introduce unacceptable performance overhead
2. **Highly Specialized**: The code is so specific to one flow that generalization would make it more complex
3. **Temporary**: You're prototyping and plan to refactor later (document this!)
4. **Different Evolution**: The two pieces of code will evolve independently

**You MUST document the reasoning when duplicating code.**

## When Creating New Abstractions

When you create a shared utility, wrapper, or pattern:

1. **Prove it works**: Migrate at least one existing consumer to use the new abstraction in the same commit
2. **Create a reference**: The migrated route becomes the canonical example for future routes
3. **Never ship dead abstractions**: If nothing uses it yet, it's dead code that confuses future decisions about "which pattern is correct"

This prevents the project from accumulating competing patterns that coexist without resolution.

## Response Template

When I (Kiro) receive a request, I will:

1. **First**, check for existing implementations:
   ```
   "Let me check if we have existing components/utilities for this..."
   ```

2. **Then**, report findings:
   ```
   "I found [component/utility] that handles [similar functionality].
   We can reuse/extend it for this use case."
   ```
   OR
   ```
   "I didn't find existing implementations. I'll create a new [component/utility]
   that can be reused across [potential use cases]."
   ```

3. **Finally**, implement with reusability in mind:
   - Use generic prop names
   - Avoid hard-coded values
   - Make components configurable
   - Document usage examples

## Refactoring Guidelines

When refactoring to reduce duplication:

1. **Preserve Functionality**: All existing features must continue working
2. **Test Thoroughly**: Run all affected tests
3. **Update Gradually**: Use feature flags if needed
4. **Document Changes**: Update audit document
5. **Remove Old Code**: Delete duplicate implementations after migration

## Audit Maintenance

The `CODE_REUSABILITY_AUDIT.md` file tracks:
- Known duplication opportunities
- Consolidation progress
- Priority matrix
- Action plans

**Update this file whenever:**
- New duplication is identified
- Consolidation is completed
- New shared components are created

## Quick Reference

### Before ANY implementation:
1. Search for existing code
2. Check the audit document
3. Consider reusability
4. Document decisions

### Red Flags (Indicates Possible Duplication):
- "This is similar to [other feature]..."
- Copy-pasting code from another file
- Writing validation logic from scratch
- Creating progress indicators
- Implementing form state management
- Writing CRUD database queries

### Green Flags (Good DRY Practice):
- Importing shared components
- Extending base types
- Using shared utilities
- Configuring generic components
- Documenting why code is flow-specific

---

**Remember:** The goal is not zero duplication, but *intentional* code organization where duplication is the exception, not the rule, and is always documented.
