# Introduction to AI with  [**`Vercel AI SDK`**](https://ai-sdk.dev/docs/foundations/overview)

**Vercel AI SDK** designed to feel like "React for AI"—it abstracts away the messy parts of streaming and API calls so you can focus on the UI.

Here are the basic foundations you requested, followed by a guide to getting started.

---

## 1. The Three Pillars: Models, Providers, and Templates

Think of these as the "Engine," the "Gas Station," and the "Blueprint" for your application.

| Term | Analogy | Description | Examples |
| --- | --- | --- | --- |
| **Model** | The Brain | The actual Large Language Model (LLM) that processes text and generates logic. | `gpt-4o`, `claude-3-5-sonnet`, `gemini-1.5-pro` |
| **Provider** | The Host | The service that provides an API to access those models. They handle the billing and infrastructure. | OpenAI, Anthropic, Google, AWS Bedrock |
| **Template** | The Blueprint | Pre-built starter kits provided by Vercel to help you skip the "boilerplate" setup (auth, DB, styling). | Chatbot Starter, RAG (Knowledge Base) Template |

---

## 2. Core Architecture in Next.js

The SDK is split into two main parts:

1. **AI SDK Core (`ai`):** Used on the **Server** (API routes/Server Actions). It talks to the AI providers.
2. **AI SDK UI (`@ai-sdk/react`):** Used on the **Client**. It provides hooks like `useChat` to manage the UI state and handle streaming automatically.

---

## 3. Implementation Guide: Building a Basic Chat

### Step 1: Installation

Install the core SDK and the provider of your choice (we'll use OpenAI for this example).

```bash
npm install ai @ai-sdk/openai @ai-sdk/react zod

```

### Step 2: Set up your API Route

Create a route handler in your Next.js App Router. This is where the "heavy lifting" happens.

**`app/api/chat/route.ts`**

```typescript
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';

// Allow streaming responses up to 30 seconds
export const maxDuration = 30;

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = await streamText({
    model: openai('gpt-4o'), // The Provider (openai) + Model (gpt-4o)
    messages,
  });

  return result.toDataStreamResponse();
}

```

### Step 3: Create the Frontend

Use the `useChat` hook to automatically handle the input state, the message history, and the streaming text.

**`app/page.tsx`**

```tsx
'use client';

import { useChat } from '@ai-sdk/react';

export default function Chat() {
  const { messages, input, handleInputChange, handleSubmit } = useChat();

  return (
    <div className="flex flex-col w-full max-w-md py-24 mx-auto stretch">
      {messages.map(m => (
        <div key={m.id} className="whitespace-pre-wrap mb-4">
          <strong className={m.role === 'user' ? 'text-blue-500' : 'text-green-500'}>
            {m.role === 'user' ? 'User: ' : 'AI: '}
          </strong>
          {m.content}
        </div>
      ))}

      <form onSubmit={handleSubmit}>
        <input
          className="fixed bottom-0 w-full max-w-md p-2 mb-8 border border-gray-300 rounded shadow-xl text-black"
          value={input}
          placeholder="Say something..."
          onChange={handleInputChange}
        />
      </form>
    </div>
  );
}

```

---

## 4. Why this approach is powerful

* **Zero State Management:** You don't need `useState` for the messages; `useChat` handles it.
* **Automatic Streaming:** The text "types out" in real-time without you having to write complex `ReadableStream` logic.
* **Easy Swapping:** If you want to switch to Anthropic, you just change `openai('gpt-4o')` to `anthropic('claude-3-5-sonnet')` in your API route.

Would you like me to show you how to implement **Tools** (so the AI can actually "do" things like fetch data or send emails)?

[A Complete Guide To Vercel's AI SDK](https://www.youtube.com/watch?v=mojZpktAiYQ)

This video provides a practical, deep-dive walkthrough of the AI SDK's core concepts and demonstrates how to build real-world AI features in Next.js applications.
