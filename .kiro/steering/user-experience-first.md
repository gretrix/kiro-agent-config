---
inclusion: manual
---

# User Experience First - UX Validation Checklist

> **Note**: This document is a subset of `human-centered-design.md`, focused specifically on form validation UX. The foundational framework in `human-centered-design.md` takes precedence and applies to ALL work, not just forms.

## Core Principle
**Validation errors should be caught at the earliest possible point in the user journey, not at submission.**

## The UX-First Mindset

### When Fixing Validation Errors, Always Ask:
1. **Where should the user learn about this error?**
   - ❌ At final submission (frustrating)
   - ✅ As they type (immediate feedback)
   - ✅ When they try to proceed to next step (early feedback)

2. **What's the user's mental model?**
   - If a field is optional, can they leave it blank? → YES
   - If a field has format requirements, do they know what they are? → Show placeholder/hint
   - If they enter invalid data, when do they find out? → IMMEDIATELY

3. **Is this frustrating?**
   - Filling out a long form only to get an error at the end? → VERY FRUSTRATING
   - Getting instant feedback as you type? → HELPFUL
   - Being blocked from proceeding with clear error message? → ACCEPTABLE

## Validation Hierarchy (Best to Worst)

### 1. ✅ BEST: Prevent Invalid Input
```typescript
// Example: Only allow numbers in a phone field
<input type="tel" pattern="[0-9]*" />
```

### 2. ✅ GREAT: Real-time Validation
```typescript
// Example: Show error as user types
onChange={(e) => {
  const value = e.target.value;
  if (value && !isValidURL(value)) {
    setError('Please enter a valid URL');
  }
}}
```

### 3. ✅ GOOD: Step-by-Step Validation
```typescript
// Example: Validate before allowing "Next" button
const canProceed = () => {
  return allFieldsValid();
};

<button disabled={!canProceed()}>Next</button>
```

### 4. ⚠️ ACCEPTABLE: Form-Level Validation
```typescript
// Example: Validate on form submit
onSubmit={(e) => {
  e.preventDefault();
  const errors = validateForm();
  if (errors.length > 0) {
    showErrors(errors);
    return;
  }
  submitForm();
}}
```

### 5. ❌ BAD: API-Level Validation Only
```typescript
// Example: User only finds out after API call
await fetch('/api/submit', { body: formData })
  .then(res => {
    if (!res.ok) {
      alert('Validation failed!'); // TOO LATE!
    }
  });
```

## Mandatory UX Checks for Forms

### Before Implementing ANY Form:
- [ ] Can user proceed with invalid data? → NO
- [ ] Does user get immediate feedback? → YES
- [ ] Are error messages clear and actionable? → YES
- [ ] Can user see what's wrong before submitting? → YES
- [ ] Is there a loading state during submission? → YES
- [ ] Is there clear success/failure feedback? → YES

### For Optional Fields with Format Requirements:
- [ ] Placeholder shows expected format
- [ ] Help text explains requirements
- [ ] Validation only triggers if field has content
- [ ] User can clear field to make it valid again
- [ ] Error message suggests correct format

### Example: Optional URL Field
```typescript
<input
  type="url"
  placeholder="https://example.com"
  value={website}
  onChange={(e) => {
    const value = e.target.value;
    setWebsite(value);
    
    // Only validate if they've entered something
    if (value.trim()) {
      try {
        new URL(value);
        setError('');
      } catch {
        setError('Please enter a valid URL (e.g., https://example.com)');
      }
    } else {
      setError(''); // Empty is valid for optional field
    }
  }}
/>
{error && <span className="text-red-600">{error}</span>}
```

## Common UX Mistakes to Avoid

### ❌ Mistake 1: Backend-Only Validation
**Problem:** User fills out entire form, clicks submit, gets generic error
**Solution:** Validate each field as user interacts with it

### ❌ Mistake 2: Allowing Invalid State
**Problem:** User can click "Next" with invalid data, gets error later
**Solution:** Disable "Next" button until all fields are valid

### ❌ Mistake 3: Unclear Error Messages
**Problem:** "Validation failed" - what does that mean?
**Solution:** "Website must be a valid URL like https://example.com"

### ❌ Mistake 4: No Visual Feedback
**Problem:** User doesn't know if field is valid or invalid
**Solution:** Show green checkmark for valid, red border for invalid

### ❌ Mistake 5: Validating Too Early
**Problem:** Showing error before user finishes typing
**Solution:** Validate onBlur or after user stops typing (debounce)

## Implementation Checklist

When implementing form validation:

1. **Design Phase:**
   - [ ] Map out user journey step-by-step
   - [ ] Identify all validation points
   - [ ] Decide when each validation should trigger
   - [ ] Design error message copy

2. **Implementation Phase:**
   - [ ] Add frontend validation FIRST
   - [ ] Add visual feedback (colors, icons, messages)
   - [ ] Disable submit/next buttons when invalid
   - [ ] Add backend validation as safety net
   - [ ] Test with intentionally invalid data

3. **Testing Phase:**
   - [ ] Try to submit invalid data (should be blocked)
   - [ ] Try to proceed with invalid data (should be blocked)
   - [ ] Verify error messages are clear
   - [ ] Verify user can fix errors easily
   - [ ] Test edge cases (empty, whitespace, special chars)

## The "User Journey" Test

Before completing ANY form implementation, walk through this:

1. **Open the form as a new user**
2. **Try to break it:**
   - Enter invalid data
   - Leave required fields empty
   - Enter edge cases
   - Try to proceed without completing
3. **Ask yourself:**
   - When did I find out about the error?
   - Was it frustrating?
   - Was the error message helpful?
   - Could I easily fix it?

If any answer is "no" or "frustrating", fix it before completing the task.

## Remember

> "The best error message is the one the user never sees because we prevented the error from happening."

**Priority Order:**
1. User Experience (prevent frustration)
2. Data Integrity (ensure valid data)
3. System Stability (handle edge cases)

**Always think:** "If I were the user, would this be annoying?"

---

**This is MANDATORY for all form implementations. No form is complete until the UX is validated.**
