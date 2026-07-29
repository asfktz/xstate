---
'xstate': patch
---

Delayed transitions (`onTimeout` and `after`) and error transitions (`onError`) in machines created with `setup({ states })` now type the `context` patch against the **target** state's context schema, matching the behavior of `on:` transitions. Previously, a cross-state context patch that was valid at runtime failed to typecheck:

```ts
const machine = setup({
  schemas: {
    context: z.object({
      error: z.union([z.string(), z.null()]),
      reason: z.union([z.literal('timeout'), z.null()])
    })
  },
  states: {
    active: {
      schemas: {
        context: z.object({ error: z.null(), reason: z.null() })
      }
    },
    expired: {
      schemas: {
        context: z.object({ error: z.null(), reason: z.literal('timeout') })
      }
    },
    failed: {
      schemas: {
        context: z.object({ error: z.string(), reason: z.null() })
      }
    }
  }
}).createMachine({
  context: { error: null, reason: null },
  initial: 'active',
  states: {
    active: {
      timeout: 5000,
      // Previously a type error; now checked against `expired`'s context
      onTimeout: () => ({
        target: 'expired',
        context: { reason: 'timeout' as const }
      }),
      // Previously a type error; now checked against `failed`'s context
      onError: ({ event }) => ({
        target: 'failed',
        context: { error: event.error.message }
      })
    },
    expired: { type: 'final' },
    failed: { type: 'final' }
  }
});
```
