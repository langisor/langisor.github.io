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

<iframe width="560" height="315" src="https://www.youtube.com/embed/mojZpktAiYQ?si=U6hBSuUlO8FTn-1K" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

**Summarize the video:**

This comprehensive tutorial by Matt Pocock provides a complete guide to Vercel's AI SDK, an essential tool for building and shipping AI applications (0:00). The video covers a wide range of functionalities offered by the SDK, including:

    Understanding the AI SDK: The AI SDK simplifies interactions with large language models (LLMs) by providing a unified API, reducing the need for extensive glue code when switching between different LLM providers like OpenAI or Anthropic (1:20). It offers helpful features for common use cases such as streaming text, structured outputs, tool calling, and agents (2:00).
    Core Functionalities:
        Generating Text: Demonstrates how to use generateText for simple text generation (3:23).
        Streaming Text: Explains streamText for receiving text token by token, useful for real-time UI updates (4:58).
        System Prompts: Shows how to use system prompts to define the AI's role and instructions, ensuring consistent behavior (6:02).
        Hot-Swapping Models: Highlights the flexibility of swapping between different LLM models by treating them as language model types (6:52).
        Preserving Chat History: Details how to maintain conversation context using the CoreMessage type and messages array (7:57).
        Running Local Models: Explains how to connect the AI SDK to locally running models using createOpenAICompatible (12:38).
    Advanced Features:
        Structured Outputs: Demonstrates extracting structured data from LLMs using Zod schemas and generateObject (13:49).
        Streaming Structured Outputs: Shows how to stream structured objects as they are being generated using streamObject for real-time updates (16:21).
        Building a Classifier: Illustrates classifying text into categories (e.g., positive, negative, neutral) using generateObject with an enum output (18:07).
        Describing Images: Explains how to generate alt text for images by passing image data or URLs to the LLM (19:36).
        Data Extraction from PDFs: Shows how to extract structured information from PDF files using the AI SDK (23:10).
        Embeddings & Vector Databases: Introduces how to create and use embeddings for semantic similarity comparisons, useful for search and categorization (26:04).
        Tool Calling: Explores how LLMs can interact with external tools by describing the tools they want to call, allowing for more powerful applications (30:08).
        Building an Agent: Demonstrates creating an agentic loop where tool results are fed back into the LLM, enabling it to take multiple steps and ground itself in real-world information, reducing hallucinations (33:46).

