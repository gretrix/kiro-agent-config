# DRY Quick Reference Card

## 🎯 Core Principle
**Before creating anything new, ask: "Does this already exist?"**

---

## ✅ Quick Checklist

### Creating a Component?
- [ ] Search `app/components/quiz/` 
- [ ] Search `app/components/partner/`
- [ ] Check `CODE_REUSABILITY_AUDIT.md`
- [ ] Can I reuse or extend existing?

### Writing Business Logic?
- [ ] Check `lib/` directory
- [ ] Check `lib/validation/`
- [ ] Check `lib/quiz/` and `lib/partner/`
- [ ] Can I use existing utilities?

### Defining Types?
- [ ] Check `app/types/`
- [ ] Can I extend a base type?
- [ ] Are these fields duplicated elsewhere?

---

## 📚 Shared Components Library

| Component | Location | Use For |
|-----------|----------|---------|
| QuizProgress | `app/components/quiz/QuizProgress.tsx` | Progress bars in multi-step flows |
| GoogleAddressAutocomplete | `app/components/quiz/GoogleAddressAutocomplete.tsx` | Address input fields |
| QuestionCard | `app/components/quiz/QuestionCard.tsx` | Rendering different question types |

---

## 🔧 Shared Utilities

| Utility | Location | Use For |
|---------|----------|---------|
| Validation | `lib/validation/` | Form validation |
| Conditional Logic | `lib/quiz/conditionalLogic.ts` | Show/hide based on answers |
| Database Queries | `lib/db/` | CRUD operations |

---

## 🚩 Red Flags (Possible Duplication)

- Copy-pasting code from another file
- "This is similar to [other feature]..."
- Writing validation from scratch
- Creating progress indicators
- Implementing form state management
- Writing database CRUD operations

---

## ✨ Green Flags (Good DRY)

- Importing shared components
- Extending base types
- Using shared utilities
- Configuring generic components
- Documenting flow-specific code

---

## 📖 Full Documentation

- **Audit:** `CODE_REUSABILITY_AUDIT.md`
- **Steering:** `.kiro/steering/dry-principles.md`
- **Requirements:** `.kiro/specs/code-reusability-audit/requirements.md`
- **Summary:** `DRY_IMPLEMENTATION_SUMMARY.md`

---

## 💡 Remember

**Reuse > Extend > Create**

1. First, try to reuse existing code
2. If not exact match, try to extend/configure it
3. Only create new if truly unique
4. Always document why code is flow-specific
