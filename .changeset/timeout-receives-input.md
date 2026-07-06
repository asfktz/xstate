---
'xstate': patch
---

The `timeout` delay function and `onTimeout` transition now receive the target state's `input`.

```ts
const machine = setup({
  states: {
    active: {
      schemas: { input: z.object({ duration: z.number() }) }
    }
  }
}).createMachine({
  initial: 'idle',
  states: {
    idle: {
      on: {
        activate: ({ event }) => ({
          target: 'active',
          input: { duration: event.duration }
        })
      }
    },
    active: {
      timeout: ({ input }) => input.duration,
      onTimeout: ({ input }, enq) => {
        enq.raise({ type: 'timedOut', duration: input.duration });
        return { target: 'idle' };
      }
    }
  }
});
```
