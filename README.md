# Flow

This is a Type-Safe agnostic runtime Web Framework concept to handle Workflows and Schedules nativily, it was inspired by Hono.

## Features

- Workflows
- Failure detection
- Composability
- Interrupt
- Encryption
- Schedules
- Observabillitty

## Details

```tsx
import { App } from "flow";

const app = new App({
  retries: 3,
  telemetry: {
    logging: { forward: { level: "DEBUG" } },
  },
});

app
  .flow("firstFlow", [
    {
      name: "step1",
      retry: 3, // retry 3 times
      execute: async (ctx) => {
        ctx.set("flow", "secondFlow");
        await ctx.cancel();
        await ctx.invoke("cleanupDatabase");
      },
    },
    {
      name: "step2",
      execute: async (ctx) => {
        const flow = ctx.get("flow");
        console.log("Main Flow - Step 2: Message is", ctx.state.message);
        await ctx.invoke(flow);
      },
    },
  ])
  .flow("internalFlow", [
    {
      name: "step1",
      execute: async (ctx) => {
        ctx.state.message = "Hello from internalFlow!";
        console.log("Internal Flow - Step 1:", ctx.state.message);
      },
    },
    {
      name: "step2",
      execute: async (ctx) => {
        console.log("Internal Flow - Step 2: Message is", ctx.state.message);
      },
    },
  ])
  .flow("secondFlow", [
    {
      name: "step1",
      execute: async (ctx) => {
        ctx.state.message = "Hello from secondFlow!";
        console.log("Second Flow - Step 1:", ctx.state.message);
        await ctx.invoke("sendReports");
      },
    },
    {
      name: "step2",
      execute: async (ctx) => {
        console.log("Second Flow - Step 2: Message is", ctx.state.message);
        await ctx.invoke("internalFlow");
      },
    },
  ])
  .get("/first", async (ctx) => {
    await ctx.invoke("internalFlow");
    return ctx.json({ message: ctx.state.message });
  })
  .post("/second", async (ctx) => {
    return ctx.text(ctx.state.message);
  })
  .schedule("cleanupDatabase", { interval: "0 0 * * *" }, [
    {
      name: "step1",
      execute: async (ctx) => {
        console.log("Cleaning up database...");
      },
    },
  ])
  .schedule("sendReports", { interval: "1h" }, [
    {
      name: "step1",
      execute: async (ctx) => {
        console.log("Sending reports...");
      },
    },
  ])
  .schedule("autoStartTask", { interval: "5m", autoStart: true }, [
    {
      name: "step1",
      timeout: 5000,
      execute: async (ctx) => {
        console.log("Auto-start task running...");
      },
    },
  ]);

export default app;
```
