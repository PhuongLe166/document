# How to Build a Custom Agent Framework with PI

PI is a TypeScript toolkit for building AI agents. It’s a monorepo of packages:
- `pi-ai`: handles LLM communication across providers
- `pi-agent-core`: adds the agent loop with tool calling
- `pi-coding-agent`: a full coding agent with built-in tools, session persistence, and extensibility
- `pi-tui`: provides a terminal UI for building CLI interfaces

## The stack

![PI architecture](_static/pi-architecture.png)

Each layer adds capability. Use as much or as little as you need.
- `pi-ai`: Call any LLM through one interface. Anthropic, OpenAI, Google, Bedrock, Mistral, Groq, xAI, OpenRouter, Ollama, and more. Streaming, completions, tool definitions, cost tracking
- `pi-agent-core`: Wraps `pi-ai` into an agent loop. You define tools, the agent calls the LLM, executes tools, feeds results back, and repeats until done
- `pi-coding-agent`: The full agent runtime. Built-in file tools (read, write, edit, bash), JSONL session persistence, context compaction, skills, and an extension system
- `pi-tui`: Terminal UI library with differential rendering. Markdown display, multi-line editor with autocomplete, loading spinners, and flicker-free screen updates

## Prerequisites

- Node.js 20+
- An API key from at least one provider (Anthropic, OpenAI, Google, etc.)

## Setup

```
mkdir pi-agent && cd pi-agent
npm init -y
npm install @mariozechner/pi-ai @mariozechner/pi-agent-core @mariozechner/pi-coding-agent @mariozechner/pi-tui chalk
npm install -D typescript @types/node tsx
```

Set your API key:

```
export ANTHROPIC_API_KEY=sk-ant-...
# or
export OPENAI_API_KEY=sk-...
```

### Layer 1: pi-ai
**Your first LLM call**

Create `basics.ts`
```
import { getModel, completeSimple } from "@mariozechner/pi-ai";

async function main() {
  const model = getModel("anthropic", "claude-opus-4-5");

  const response = await completeSimple(model, {
    systemPrompt: "You are a helpful assistant.",
    messages: [
      { role: "user", content: "What is the capital of France?", timestamp: Date.now() }
    ],
  });

  // response is an AssistantMessage
  for (const block of response.content) {
    if (block.type === "text") {
      console.log(block.text);
    }
  }

  console.log(`\nTokens: ${response.usage.totalTokens}`);
  console.log(`Stop reason: ${response.stopReason}`);
}

main();
```
Run it:
```
npx tsx basics.ts
```
- `getModel`: looks up a model by provider and ID from PI’s built-in catalog of 2000+ models
- `completeSimple`: sends the messages and returns the full `AssistantMessage` when the model is done
- `response`: has a `.content` array of typed blocks - text, thinking, or toolCall plus `.usage` for token counts and `.stopReason` for why the model stopped ("stop", "toolUse", "length", "error", "aborted") 

**Streaming**

`completeSimple` waits for the full response. For real-time output, use `streamSimple`:
```
import { getModel, streamSimple } from "@mariozechner/pi-ai";

async function main() {
  const model = getModel("anthropic", "claude-opus-4-5");

  const stream = streamSimple(model, {
    systemPrompt: "You are a helpful assistant.",
    messages: [
      { role: "user", content: "Explain how TCP works in 3 sentences.", timestamp: Date.now() }
    ],
  });

  for await (const event of stream) {
    switch (event.type) {
      case "text_delta":
        process.stdout.write(event.delta);
        break;
      case "done":
        console.log(`\n\nTokens: ${event.message.usage.totalTokens}`);
        break;
      case "error":
        console.error("Error:", event.error.errorMessage);
        break;
    }
  }
}

main();
```

- Every provider has its own streaming format - Anthropic, OpenAI, and Google all do it differently
- `streamSimple` normalizes them into a single set of events: start, text_start, text_delta, text_end, thinking_start/delta/end, toolcall_start/delta/end, done, and error
- Write your streaming handler once, and it works with any provider. For most use cases, you only care about text_delta (the text chunks) and done (the final message)

You can also await the final message directly:
```
const stream = streamSimple(model, context);
const finalMessage = await stream.result(); // AssistantMessage
```

**Switching providers**

Switch to a different provider by changing the `getModel` call. The rest of your code stays the same.

```
// Just change this line - everything else stays the same
const model = getModel("anthropic", "claude-opus-4-5");
// const model = getModel("openai", "gpt-4o");
// const model = getModel("google", "gemini-2.5-pro");
// const model = getModel("groq", "llama-3.3-70b-versatile");

const stream = streamSimple(model, context);
```

Each provider needs its own API key set in the environment (ANTHROPIC_API_KEY, OPENAI_API_KEY, GEMINI_API_KEY, GROQ_API_KEY, etc.)

You can also define custom models for self-hosted endpoints:

```
import type { Model } from "@mariozechner/pi-ai";

const localModel: Model<"openai-completions"> = {
  id: "llama-3.1-8b",
  name: "llama-3.1-8b",
  api: "openai-completions",
  provider: "ollama",
  baseUrl: "http://localhost:11434/v1",
  reasoning: false,
  input: ["text"],
  cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
  contextWindow: 128000,
  maxTokens: 8192,
};

```

Under the hood, `pi-ai` uses the official provider SDKs (OpenAI SDK, Anthropic SDK, etc.). The api field determines which SDK handles the request - "openai-completions" routes through the OpenAI SDK, which is why it works with any OpenAI-compatible endpoint (Ollama, vLLM, Mistral, etc.).

API keys are resolved automatically from environment variables by provider name (OPENAI_API_KEY, ANTHROPIC_API_KEY, etc.) and passed to the SDK, which handles authentication. Ollama doesn’t require auth, so the example above works as-is. For a provider that needs a key, either set the matching environment variable or pass it directly:

```
const stream = streamSimple(localModel, context, {
  apiKey: "your-api-key",
});
```

**Thinking levels**

Models that support extended thinking (Claude, o3, Gemini 2.5) can be enabled via the `reasoning` option. It’s off by default.

```
const stream = streamSimple(model, context, {
  reasoning: "high", // "minimal" | "low" | "medium" | "high" | "xhigh"
});
```

When enabled, the stream emits `thinking_delta` events alongside `text_delta`.

### Layer 2: pi-agent-core

pi-ai lets you talk to LLMs. pi-agent-core lets the LLM talk back - with tools.


The `Agent` class runs the standard agent loop: send messages to the LLM, execute any tool calls it makes, feed results back, repeat until the model stops.

**Defining tools**

Tools use `TypeBox` schemas for type-safe parameter definitions:

```
import { Type } from "@mariozechner/pi-ai";
import type { AgentTool } from "@mariozechner/pi-agent-core";

const weatherParams = Type.Object({
  city: Type.String({ description: "City name" }),
});

const weatherTool: AgentTool<typeof weatherParams> = {
  name: "get_weather",
  label: "Weather",
  description: "Get the current weather for a city",
  parameters: weatherParams,
  execute: async (toolCallId, params, signal, onUpdate) => {
    // params is typed: { city: string }
    const temp = Math.round(Math.random() * 30);
    return {
      content: [{ type: "text", text: `${params.city}: ${temp}C, partly cloudy` }],
      details: { temp, city: params.city },
    };
  },
};
```

Define the schema as a standalone variable and pass it as the generic parameter to `AgentTool<typeof schema>` - this gives TypeScript the type information it needs to infer params correctly inside execute

Every tool has:
- name - identifier the LLM uses to call it
- label - human-readable display name
- description - tells the LLM when and how to use the tool
- parameters - TypeBox schema; validated with AJV before execution
- execute - runs when the LLM calls the tool; returns content (sent back to the LLM) and details (for your UI, not sent to the LLM)