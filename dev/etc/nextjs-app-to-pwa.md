# How to make your Nextjs App working offline

> Making your Next.js App Router app work offline, especially when hosted on Netlify, primarily involves implementing a **Service Worker** to cache assets and provide fallback content. This process turns your app into a **Progressive Web Application (PWA)**.

Since you're using the App Router and `next.config.ts`, the most streamlined way to handle the complex Service Worker configuration is by using a specialized package like **`@ducanh2912/next-pwa`**.

Here's the step-by-step approach:

## 1\. Install and Configure `next-pwa`

You'll use the `@ducanh2912/next-pwa` package, which is actively maintained and has good support for the Next.js App Router and TypeScript.

### Installation

Install the package using your package manager:

```bash
npm install @ducanh2912/next-pwa
# or
yarn add @ducanh2912/next-pwa
```

### Configure `next.config.ts`

Modify your `next.config.ts` to wrap your existing configuration with the PWA setup. This handles generating and registering the Service Worker.

```typescript
// next.config.ts
import type { NextConfig } from "next";
// Import the PWA plugin wrapper
import withPWA from "@ducanh2912/next-pwa";

// Configure PWA options
const pwaConfig = withPWA({
  dest: "public", // Directory where Service Worker files will be output
  register: true, // Automatically registers the Service Worker on the client
  skipWaiting: true, // Activates the new Service Worker immediately on update
  disable: process.env.NODE_ENV === "development", // Disable PWA in dev mode
  cacheOnFrontEndNav: true, // Cache pages visited during client-side navigation
  aggressiveFrontEndNavCaching: true, // Cache aggressively for faster offline navigation
  // You can add more Workbox options here for fine-grained caching
});

// Your main Next.js configuration
const nextConfig: NextConfig = {
  // Your other Next.js config options...
};

// Export the wrapped configuration
export default pwaConfig(nextConfig);
```

-----

## 2\. Create a Web App Manifest

The Web App Manifest (`manifest.json` or `manifest.ts`) is a required file for a PWA. It defines the app's metadata (name, icons, display mode) for when a user installs it.

### Create `app/manifest.ts`

The App Router allows you to dynamically generate the manifest using a `manifest.ts` file in your `app` directory.

```typescript
// app/manifest.ts
import { MetadataRoute } from 'next'

export default function manifest(): MetadataRoute.Manifest {
  return {
    name: 'Your App Name',
    short_name: 'ShortName',
    description: 'My offline-ready Next.js App',
    start_url: '/',
    display: 'standalone', // Makes it feel more like a native app
    background_color: '#ffffff',
    theme_color: '#000000',
    icons: [
      {
        src: '/icon-192x192.png', // Place these icons in your public/ folder
        sizes: '192x192',
        type: 'image/png',
      },
      {
        src: '/icon-512x512.png',
        sizes: '512x512',
        type: 'image/png',
      },
    ],
  }
}
```

### Add Icons

Make sure you create the necessary icon files (e.g., `icon-192x192.png`, `icon-512x512.png`) and place them in your **`public/`** directory.

-----

## 3\. Configure a Fallback Page (Optional but Recommended)

For a better offline experience, you should create a simple fallback page that loads when a user tries to access a page that hasn't been cached.

### Create the Offline Page

Create a file like **`app/offline/page.tsx`** (or `app/_offline/page.tsx`, depending on the package version/convention) that is entirely a Client Component and relies only on assets that are guaranteed to be cached by the Service Worker.

```tsx
// app/offline/page.tsx
'use client'
import React from 'react';

export default function OfflinePage() {
  return (
    <div style={{ padding: '20px', textAlign: 'center' }}>
      <h1>You Are Offline 😞</h1>
      <p>It looks like you're not connected to the internet.</p>
      <p>The content for this page wasn't loaded while you were online.</p>
      {/* You can add a button to try navigating back or home */}
    </div>
  );
}
```

### Update `next.config.ts` for Fallbacks

You'll need to specify this fallback in your `next.config.ts` to instruct the Service Worker to use it for navigation requests that fail when offline. Add a `fallbacks` section to your PWA config:

```typescript
// next.config.ts (inside the pwaConfig object)
// ...
workboxOptions: {
  // ... other workbox options
  fallbacks: {
    // Use the route to your offline page
    document: '/offline', 
  }
},
// ...
```

-----

## 4\. Netlify Deployment

Netlify's Next.js adapter should automatically handle the Service Worker file generated in the `public/` directory after the build.

### Build and Deploy

1.  **Run a production build** locally to test and see the generated PWA files:
    ```bash
    npm run build
    ```
    You should see the Service Worker files (`sw.js`, etc.) generated inside your `public` directory.
2.  **Commit and push** your changes to your Git repository.
3.  **Netlify** will automatically run the build and deploy your app, including the PWA files.

### Testing Offline Functionality

After deployment:

1.  Visit your deployed app in your browser (must be a secure context, Netlify uses HTTPS).
2.  Navigate to a few pages to ensure they are cached by the Service Worker.
3.  Open the browser **DevTools** (F12) and go to the **Application** tab.
4.  Select the **Service Workers** section and check the **Offline** box.
5.  Try navigating to one of the pages you just visited—it should load\!
6.  Try navigating to a page you haven't visited; it should load your `/offline` fallback page.
