# Code Review Summary

## 1. Code Review Timing

Code review should be conducted at the **Pull Request (PR) stage**, before merging into the main branch.

### Recommended Timing
- After feature development is completed
- Before merging into main / develop branch
- Triggered when a PR is created

### Workflow

```text
Feature Branch → Create PR → Code Review → CI Pass → Merge
```

---

## 2. What Should Be Reviewed

Code review focuses on ensuring **code quality, correctness, and maintainability**.

### 2.1 Logic & Correctness
- Business logic correctness
- Edge cases handling
- Error handling

### 2.2 Code Quality
- Readability
- Naming conventions
- Code structure

### 2.3 Design & Architecture
- Proper layering (controller/service/repository)
- Reusability
- Separation of concerns

### 2.4 Performance & Security
- Inefficient queries or loops
- Potential security risks

### 2.5 Test Coverage
- Unit tests included
- Key logic covered

---

## 3. Who Should Review

### Required
- At least **1–2 developers** (peer review)

### Recommended
- Tech Lead (for complex features)
- Senior developers (for critical modules)

### Optional
- QA (for test-related validation)

---

## 4. Do We Need a Meeting?

### Default Approach (Recommended)
- ❌ No meeting required
- ✅ Use asynchronous review (PR comments)

### When Meeting is Needed
- Complex feature logic
- Major design decision
- Disagreement in review comments
- Knowledge sharing session

### Suggested Format (if needed)
- Duration: 1–2 hours
- Participants: Dev + Tech Lead
- Focus: design, logic, key implementation

---

## 5. Best Practices

- Review at **feature level**, not every commit
- Keep PR size manageable
- Combine:
  - Human review (logic/design)
  - CI checks (build/test/lint)
- Avoid mixing code review with business meetings (e.g. Sprint Review)

---

## 6. Summary

Code review is:
- A **mandatory quality gate** before merging code
- Performed at the **PR stage**
- Focused on **logic, quality, and design**
- Usually **asynchronous**, with meetings only when necessary

It works together with CI/CD to ensure production-ready code.
