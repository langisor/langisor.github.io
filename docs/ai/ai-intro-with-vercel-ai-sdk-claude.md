# Complete AI SDK Setup and Workflow

## Step 1: Install Dependencies

```bash
# Create a new Next.js project if you don't have one
npx create-next-app@latest my-ai-app --typescript

cd my-ai-app

# Install the core AI SDK
npm install ai

# Install provider packages (install only what you use)
npm install @ai-sdk/openai          # For GPT models
npm install @ai-sdk/anthropic       # For Claude
npm install @ai-sdk/google          # For Gemini

# Install zod for schema validation (optional but recommended)
npm install zod
```

## Step 2: Set Up Environment Variables

Create a `.env.local` file in your project root:

```env
# Get these API keys from the respective provider websites
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
GOOGLE_GENERATIVE_AI_API_KEY=...

# Never commit this file to version control
# Add to .gitignore: .env.local
```

## Step 3: Understanding the Request Flow

When a user interacts with your AI-powered application, here's what happens behind the scenes:

**The User Side**: Your React component (like ChatUI) captures user input and sends it to an API route via a fetch request. The user sees a loading state while waiting for the response.

**The Server Side**: Your Next.js API route receives the request. It imports the model and provider, then calls either `generateText`, `streamText`, or another template function. The result is processed and sent back to the client.

**The Model Side**: Your request goes to the provider's servers, which route it to the actual model. The model generates tokens and sends them back to your server. If you're using streaming, tokens arrive individually and are forwarded to the client immediately, creating a real-time typing effect.

**Back to the User**: The client receives tokens and displays them. If there's an error at any step, it's caught and displayed to the user gracefully.

This flow is why streaming is so important—without it, the user stares at a loading spinner while the API generates a 500-token response, which might take 5-10 seconds. With streaming, tokens start appearing after 200 milliseconds, and the user feels like the system is thinking in real-time.

## Step 4: Building Your First Feature - A Simple Q&A Bot

### Server Side: Create the API Route

Create `app/api/qa/route.ts`:

```typescript
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai';

// Choose your model once here
const model = openai('gpt-4o');

export async function POST(request: Request) {
  const { question } = await request.json();

  // Validate input
  if (!question || question.trim().length === 0) {
    return Response.json(
      { error: 'Question is required' },
      { status: 400 }
    );
  }

  try {
    // Call the model
    const result = await generateText({
      model: model,
      prompt: question,
      system: 'You are a helpful assistant. Answer questions clearly and concisely.',
      temperature: 0.7,
      maxTokens: 500,
    });

    return Response.json({
      answer: result.text,
      usage: {
        inputTokens: result.usage.promptTokens,
        outputTokens: result.usage.completionTokens,
      },
    });
  } catch (error) {
    console.error('Error:', error);
    return Response.json(
      { error: 'Failed to generate answer' },
      { status: 500 }
    );
  }
}
```

### Client Side: Create a Component

Create `app/components/QABot.tsx`:

```typescript
'use client';

import { useState } from 'react';

export default function QABot() {
  const [question, setQuestion] = useState('');
  const [answer, setAnswer] = useState('');
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');

  async function handleAsk() {
    // Reset state
    setAnswer('');
    setError('');
    setLoading(true);

    try {
      // Call your API route
      const response = await fetch('/api/qa', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ question }),
      });

      if (!response.ok) {
        const data = await response.json();
        throw new Error(data.error || 'Failed to get answer');
      }

      const data = await response.json();
      setAnswer(data.answer);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
    } finally {
      setLoading(false);
    }
  }

  return (
    <div className="max-w-2xl mx-auto p-6">
      <h1 className="text-2xl font-bold mb-6">Q&A Bot</h1>

      <div className="space-y-4">
        <input
          type="text"
          value={question}
          onChange={(e) => setQuestion(e.target.value)}
          placeholder="Ask me anything..."
          onKeyPress={(e) => e.key === 'Enter' && handleAsk()}
          disabled={loading}
          className="w-full p-3 border rounded-lg disabled:bg-gray-100"
        />

        <button
          onClick={handleAsk}
          disabled={loading || !question.trim()}
          className="w-full px-4 py-2 bg-blue-500 text-white rounded-lg disabled:bg-gray-400"
        >
          {loading ? 'Thinking...' : 'Ask'}
        </button>

        {error && (
          <div className="p-4 bg-red-100 text-red-700 rounded-lg">
            {error}
          </div>
        )}

        {answer && (
          <div className="p-4 bg-green-100 text-gray-800 rounded-lg">
            <strong>Answer:</strong>
            <p className="mt-2">{answer}</p>
          </div>
        )}
      </div>
    </div>
  );
}
```

## Step 5: Understanding Decision Points

When building with the AI SDK, you'll encounter several decision points. Let me explain the thinking behind each one.

### Should You Use generateText or streamText?

Use `generateText` when you need the complete response before displaying anything. This might be for data processing, where you need to analyze the full output. Most user-facing applications should use `streamText` because users expect to see responses appearing in real-time, just like ChatGPT.

### Which Model Should You Choose?

Consider three factors: capability needed, speed, and cost. For your Q&A bot, start with `gpt-4o-mini` because it's fast and cheap while being quite capable. If you find that responses aren't good enough, upgrade to `gpt-4o`. If you need the most reasoning power, consider `o1`. For comparing performance, try the same prompts with different models and measure output quality.

### What Should Your System Prompt Say?

The system prompt is instructions for the AI about how to behave. Generic instructions like "You are a helpful assistant" work but don't shape behavior much. Specific instructions like "You are a customer service representative. Always be polite, empathetic, and solve the customer's problem in 2-3 sentences" create consistent, predictable behavior. For your Q&A bot, specify what knowledge domain it covers and how detailed answers should be.

### Should You Implement Streaming?

Always stream responses to users. The difference between waiting 5 seconds for a full response and seeing text appear instantly is massive for user experience. Streaming is only slightly more complex to implement, and it's worth it.

## Step 6: Advanced Pattern - Using useChat for Conversations

The `useChat` hook is powerful because it manages conversation state for you. Here's why this matters: without the hook, you'd need to manually track all previous messages, send them with each new request, parse responses, append them to your state, manage loading indicators, and handle errors. The hook does all of this.

```typescript
'use client';

import { useChat } from 'ai/react';

export default function ChatBot() {
  // This single hook call gives you everything
  const { messages, input, handleInputChange, handleSubmit, isLoading } = useChat({
    api: '/api/chat', // Your API route
    initialMessages: [
      {
        id: '1',
        role: 'system',
        content: 'You are a helpful assistant',
      },
    ],
  });

  return (
    <div>
      {messages.map((m) => (
        <div key={m.id} className={m.role === 'user' ? 'user' : 'assistant'}>
          {m.content}
        </div>
      ))}
      <form onSubmit={handleSubmit}>
        <input
          value={input}
          onChange={handleInputChange}
          placeholder="Message..."
        />
      </form>
    </div>
  );
}
```

Your API route is simpler too:

```typescript
import { streamText } from 'ai';
import { openai } from '@ai-sdk/openai';

export async function POST(request: Request) {
  const { messages } = await request.json();

  // The hook sends the entire message history
  // You just pass it through
  const result = await streamText({
    model: openai('gpt-4o'),
    messages: messages,
  });

  return result.toAIStream();
}
```

The hook handles conversation history for you. When a user sends a new message, the hook includes all previous messages in the request, the server uses them as context, and the model generates responses based on the entire conversation.

## Step 7: Production Checklist

Before deploying your application, ensure you've covered these critical items:

Check that your API keys are in environment variables and never hardcoded anywhere in your source code. Environment variables should be different between development and production, allowing you to use free tier API keys for testing.

Implement proper error handling and logging. Users should see friendly error messages, while your application should log detailed errors for debugging. Monitor error rates and investigate spikes.

Add rate limiting so users can't spam your API and run up costs. Consider implementing user-level rate limiting using their IP or user ID. A reasonable limit might be 10 requests per minute per user.

Set appropriate timeouts. Don't let API requests hang indefinitely. Most applications timeout after 30-60 seconds.

Monitor token usage and costs. Log every API call's token consumption. If a user is using significantly more tokens than others, they might be abusing the feature, or you might have a bug.

Test with different models to understand their tradeoffs. Some models are faster, some are more capable, some are cheaper. A/B testing can reveal which models work best for your use case.

Implement caching where possible. If the same question is asked multiple times, cache the response instead of calling the API again. This saves money and improves response time.

Test error scenarios. What happens when the API is down? When rate limits are hit? When the user provides invalid input? Your application should handle all these gracefully.

## Step 8: Debugging and Optimization

When something isn't working, check these things in order. First, verify that your API key is correct and has the right permissions. Second, check that your prompts are clear and specific. Vague prompts produce vague responses. Third, monitor the actual requests and responses. Log both the prompt you're sending and the response you receive to understand what the model is doing. Fourth, experiment with different models and temperature settings. The same prompt might produce better results with a different model or temperature.

For optimization, focus first on costs. Token counting can reveal if you're sending unnecessarily long prompts or if certain features are token-heavy. Consider using cheaper models for simple tasks and powerful models only when needed. Focus second on user experience. Streaming is more important than a few milliseconds of latency. Rate limiting should prevent abuse without frustrating legitimate users.
