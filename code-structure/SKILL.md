---
name: code-structure
description: Use when multiple workflows duplicate the same operational logic, when deciding what belongs in actions vs shared services, or when refactoring repeated operational blocks across domain flows. Use when adding new features that share mechanics with existing ones.
---

# Service Layer Architecture

**Two-layer separation:** Actions orchestrate domain rules ("why/when"), service layer centralizes reusable mechanics ("how").

## When to Use

- Multiple callers need the same operation (sandbox creation, email sending, payment processing)
- Copy-pasting operational logic between action files
- Bug fix in one workflow doesn't propagate to others
- New feature shares mechanics with existing flows

**Don't use when:** Logic is truly domain-specific and used by one caller only.

## Core Pattern

```
Orchestration (Actions)                  Service Layer
├── business rules                       ├── reusable operations
├── state transitions                    ├── provider/SDK interactions
├── auth/ownership checks                ├── command execution details
├── failure classification               ├── health checks / readiness
├── retries / user-facing errors         └── structured results
└── calls service functions
```

**Rule:** "What this flow means" → actions. "How to do it reliably" → service layer.

## Quick Reference

| Principle | Do | Don't |
|---|---|---|
| API shape | Composable capability blocks | One giant "do everything" method |
| Inputs/outputs | Explicit params, structured returns | Hidden global state, reaching into DB |
| Migration | Extract one block, replace one caller, verify, then migrate rest | Refactor everything at once |
| Domain logic | Keep auth, policy, error classification in actions | Let service mutate domain state directly |
| Extraction trigger | Logic repeated across 2+ callers | Logic used once (over-abstraction) |

## Designing Service Functions

Each function should:
- Accept all required data as **explicit parameters**
- Return **structured outputs** (e.g., `{ ready, previewUrl, proxyPort }`)
- Never reach into database/state directly
- Make failure explicit (structured results, not swallowed errors)

## Migration Checklist

1. Write the flow in action code first
2. Mark repeated operational chunks across callers
3. Extract **only** repeated, non-domain chunks to service
4. Replace one caller → verify → replace remaining
5. Keep domain policy in actions
6. Run: typecheck, lint, confirm all flows work

## Anti-Patterns

| Anti-Pattern | Problem |
|---|---|
| **God service** | One huge function hides all control flow |
| **Leaky service** | Service mutates database tables directly |
| **Inconsistent API** | Different argument styles and error semantics |
| **Over-abstraction** | Extracting logic used by one caller |

## Example

```ts
// emailService.ts — shared mechanics
export async function sendWelcomeEmail(params: { to: string; name: string }) {
  const html = `<h1>Welcome ${params.name}</h1>`;
  await emailProvider.send(params.to, "Welcome", html);
}

// userSignup.ts — orchestration (owns WHEN)
if (user.marketingOptIn) {
  await sendWelcomeEmail({ to: user.email, name: user.name });
}

// adminInvite.ts — orchestration (different rule, same mechanic)
await sendWelcomeEmail({ to: invitee.email, name: invitee.name });
```

**Mental model:** New feature → write in action first → see repeated ops → extract to service → no repetition → keep in action.
