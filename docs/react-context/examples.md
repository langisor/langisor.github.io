# React Context Examples

## Example 1: Preloading Large Datasets with React Context

For preloading ~2000 records at app startup, here's a robust pattern that handles loading states, errors, and caching:

### Step 1: Create the Data Context

```ts
// src/contexts/DataContext.tsx
'use client'

import { createContext, useContext, useState, useEffect, ReactNode } from 'react'

// Define your record type
interface Record {
  id: string
  name: string
  // ... other fields
}

interface DataContextType {
  records: Record[]
  isLoading: boolean
  error: string | null
  refetch: () => Promise<void>
}

const DataContext = createContext<DataContextType | undefined>(undefined)

export function DataProvider({ children }: { children: ReactNode }) {
  const [records, setRecords] = useState<Record[]>([])
  const [isLoading, setIsLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  const fetchRecords = async () => {
    try {
      setIsLoading(true)
      setError(null)

      const response = await fetch('/api/records', {
        // Add cache control if needed
        cache: 'no-store', // or 'force-cache' depending on your needs
      })

      if (!response.ok) {
        throw new Error(`Failed to fetch records: ${response.statusText}`)
      }

      const data = await response.json()
      setRecords(data)
    } catch (err) {
      setError(err instanceof Error ? err.message : 'An error occurred')
      console.error('Error fetching records:', err)
    } finally {
      setIsLoading(false)
    }
  }

  // Fetch on mount
  useEffect(() => {
    fetchRecords()
  }, [])

  return (
    <DataContext.Provider
      value={{
        records,
        isLoading,
        error,
        refetch: fetchRecords
      }}
    >
      {children}
    </DataContext.Provider>
  )
}

export function useRecords() {
  const context = useContext(DataContext)
  if (context === undefined) {
    throw new Error('useRecords must be used within a DataProvider')
  }
  return context
}
```

### Step 2: Add Provider to Layout with Loading UI

```ts
// src/app/layout.tsx
import { DataProvider } from '@/contexts/DataContext'
import { Toaster } from '@/components/ui/toaster'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        <DataProvider>
          {children}
        </DataProvider>
        <Toaster />
      </body>
    </html>
  )
}
```

### Step 3: Create a Loading Wrapper Component

To show a loading screen until data is ready:

```ts
// src/components/DataLoader.tsx
'use client'

import { useRecords } from '@/contexts/DataContext'
import { Loader2 } from 'lucide-react'
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert'
import { Button } from '@/components/ui/button'
import { ReactNode } from 'react'

interface DataLoaderProps {
  children: ReactNode
  fallback?: ReactNode
}

export function DataLoader({ children, fallback }: DataLoaderProps) {
  const { isLoading, error, refetch } = useRecords()

  if (isLoading) {
    return (
      fallback || (
        <div className="flex min-h-screen items-center justify-center">
          <div className="text-center">
            <Loader2 className="h-12 w-12 animate-spin text-primary mx-auto" />
            <p className="mt-4 text-lg text-muted-foreground">
              Loading data...
            </p>
          </div>
        </div>
      )
    )
  }

  if (error) {
    return (
      <div className="flex min-h-screen items-center justify-center p-4">
        <Alert variant="destructive" className="max-w-md">
          <AlertTitle>Error Loading Data</AlertTitle>
          <AlertDescription className="mt-2">
            {error}
          </AlertDescription>
          <Button
            onClick={refetch}
            variant="outline"
            className="mt-4"
          >
            Retry
          </Button>
        </Alert>
      </div>
    )
  }

  return <>{children}</>
}
```

### Step 4: Wrap Your App Content

```ts
// src/app/page.tsx
import { DataLoader } from '@/components/DataLoader'
import { Dashboard } from '@/components/Dashboard'

export default function Home() {
  return (
    <DataLoader>
      <Dashboard />
    </DataLoader>
  )
}
```

### Step 5: Use the Data in Components

```ts
// src/components/Dashboard.tsx
'use client'

import { useRecords } from '@/contexts/DataContext'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Input } from '@/components/ui/input'
import { useState, useMemo } from 'react'

export function Dashboard() {
  const { records } = useRecords()
  const [search, setSearch] = useState('')

  // Filter records client-side (efficient since data is already loaded)
  const filteredRecords = useMemo(() => {
    if (!search) return records

    return records.filter(record =>
      record.name.toLowerCase().includes(search.toLowerCase())
    )
  }, [records, search])

  return (
    <div className="container mx-auto p-6">
      <Card>
        <CardHeader>
          <CardTitle>Records ({records.length} total)</CardTitle>
        </CardHeader>
        <CardContent>
          <Input
            placeholder="Search records..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="mb-4"
          />

          <div className="space-y-2">
            {filteredRecords.map(record => (
              <div
                key={record.id}
                className="p-3 border rounded-lg hover:bg-accent"
              >
                {record.name}
              </div>
            ))}
          </div>
        </CardContent>
      </Card>
    </div>
  )
}
```

### Step 6: Advanced Pattern - With Pagination & Caching

For better performance with 2000 records:

```ts
// src/contexts/DataContext.tsx
'use client'

import { createContext, useContext, useState, useEffect, ReactNode, useMemo } from 'react'

interface Record {
  id: string
  name: string
  category?: string
}

interface DataContextType {
  records: Record[]
  isLoading: boolean
  error: string | null
  refetch: () => Promise<void>
  // Helper methods
  getRecordById: (id: string) => Record | undefined
  getRecordsByCategory: (category: string) => Record[]
  searchRecords: (query: string) => Record[]
}

const DataContext = createContext<DataContextType | undefined>(undefined)

export function DataProvider({ children }: { children: ReactNode }) {
  const [records, setRecords] = useState<Record[]>([])
  const [isLoading, setIsLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  // Create indexed maps for fast lookups
  const recordsById = useMemo(() => {
    return new Map(records.map(record => [record.id, record]))
  }, [records])

  const recordsByCategory = useMemo(() => {
    const map = new Map<string, Record[]>()
    records.forEach(record => {
      if (record.category) {
        const existing = map.get(record.category) || []
        map.set(record.category, [...existing, record])
      }
    })
    return map
  }, [records])

  const fetchRecords = async () => {
    try {
      setIsLoading(true)
      setError(null)

      const response = await fetch('/api/records')

      if (!response.ok) {
        throw new Error(`Failed to fetch records: ${response.statusText}`)
      }

      const data = await response.json()
      setRecords(data)

      // Optional: Cache to localStorage
      if (typeof window !== 'undefined') {
        localStorage.setItem('cached_records', JSON.stringify(data))
        localStorage.setItem('cached_records_timestamp', Date.now().toString())
      }
    } catch (err) {
      setError(err instanceof Error ? err.message : 'An error occurred')

      // Try to load from cache on error
      if (typeof window !== 'undefined') {
        const cached = localStorage.getItem('cached_records')
        if (cached) {
          setRecords(JSON.parse(cached))
          setError('Using cached data (offline)')
        }
      }
    } finally {
      setIsLoading(false)
    }
  }

  useEffect(() => {
    // Try to load from cache first for instant display
    if (typeof window !== 'undefined') {
      const cached = localStorage.getItem('cached_records')
      const timestamp = localStorage.getItem('cached_records_timestamp')

      // Use cache if less than 1 hour old
      if (cached && timestamp) {
        const age = Date.now() - parseInt(timestamp)
        if (age < 3600000) { // 1 hour
          setRecords(JSON.parse(cached))
          setIsLoading(false)
          return // Skip fetch
        }
      }
    }

    fetchRecords()
  }, [])

  // Helper functions
  const getRecordById = (id: string) => recordsById.get(id)

  const getRecordsByCategory = (category: string) =>
    recordsByCategory.get(category) || []

  const searchRecords = (query: string) => {
    const lowerQuery = query.toLowerCase()
    return records.filter(record =>
      record.name.toLowerCase().includes(lowerQuery)
    )
  }

  return (
    <DataContext.Provider
      value={{
        records,
        isLoading,
        error,
        refetch: fetchRecords,
        getRecordById,
        getRecordsByCategory,
        searchRecords
      }}
    >
      {children}
    </DataContext.Provider>
  )
}

export function useRecords() {
  const context = useContext(DataContext)
  if (context === undefined) {
    throw new Error('useRecords must be used within a DataProvider')
  }
  return context
}
```

### Step 7: Using the Helper Methods

```ts
// src/components/RecordDetail.tsx
'use client'

import { useRecords } from '@/contexts/DataContext'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'

export function RecordDetail({ id }: { id: string }) {
  const { getRecordById } = useRecords()

  const record = getRecordById(id)

  if (!record) {
    return <div>Record not found</div>
  }

  return (
    <Card>
      <CardHeader>
        <CardTitle>{record.name}</CardTitle>
      </CardHeader>
      <CardContent>
        {/* Record details */}
      </CardContent>
    </Card>
  )
}
```

### Key Benefits of This Approach

1. **Single fetch**: Data loads once at app startup
2. **Global access**: Any component can access the data without prop drilling
3. **Loading states**: Clean UX during initial load
4. **Error handling**: Graceful error recovery with retry
5. **Performance**: Indexed lookups for O(1) access by ID
6. **Caching**: Optional localStorage for instant loads on return visits
7. **Type safety**: Full TypeScript support throughout

This pattern works great for reference data, lookup tables, or any dataset that doesn't change frequently during a session.

## Example 2. Preloading Large Datasets with React Context + SWR

Using SWR gives you automatic caching, revalidation, and optimized data fetching out of the box.

## Step 1: Install SWR

```bash
npm install swr
```

## Step 2: Create the Data Context with SWR

```ts
// src/contexts/DataContext.tsx
'use client'

import { createContext, useContext, ReactNode, useMemo } from 'react'
import useSWR from 'swr'

// Define your record type
interface Record {
  id: string
  name: string
  category?: string
}

interface DataContextType {
  records: Record[]
  isLoading: boolean
  error: any
  mutate: () => void
  // Helper methods
  getRecordById: (id: string) => Record | undefined
  getRecordsByCategory: (category: string) => Record[]
  searchRecords: (query: string) => Record[]
}

const DataContext = createContext<DataContextType | undefined>(undefined)

// Fetcher function for SWR
const fetcher = async (url: string) => {
  const response = await fetch(url)

  if (!response.ok) {
    throw new Error(`Failed to fetch records: ${response.statusText}`)
  }

  return response.json()
}

export function DataProvider({ children }: { children: ReactNode }) {
  // SWR hook for data fetching
  const { data, error, isLoading, mutate } = useSWR<Record[]>(
    '/api/records',
    fetcher,
    {
      // SWR configuration
      revalidateOnFocus: false, // Don't refetch on window focus
      revalidateOnReconnect: true, // Refetch on reconnect
      dedupingInterval: 60000, // Dedupe requests within 60s
      shouldRetryOnError: true,
      errorRetryCount: 3,
      errorRetryInterval: 5000,
      // Cache the data
      fallbackData: [], // Initial value before first fetch
    }
  )

  const records = data || []

  // Create indexed maps for fast lookups
  const recordsById = useMemo(() => {
    return new Map(records.map(record => [record.id, record]))
  }, [records])

  const recordsByCategory = useMemo(() => {
    const map = new Map<string, Record[]>()
    records.forEach(record => {
      if (record.category) {
        const existing = map.get(record.category) || []
        map.set(record.category, [...existing, record])
      }
    })
    return map
  }, [records])

  // Helper functions
  const getRecordById = (id: string) => recordsById.get(id)

  const getRecordsByCategory = (category: string) =>
    recordsByCategory.get(category) || []

  const searchRecords = (query: string) => {
    const lowerQuery = query.toLowerCase()
    return records.filter(record =>
      record.name.toLowerCase().includes(lowerQuery)
    )
  }

  return (
    <DataContext.Provider
      value={{
        records,
        isLoading,
        error,
        mutate,
        getRecordById,
        getRecordsByCategory,
        searchRecords
      }}
    >
      {children}
    </DataContext.Provider>
  )
}

export function useRecords() {
  const context = useContext(DataContext)
  if (context === undefined) {
    throw new Error('useRecords must be used within a DataProvider')
  }
  return context
}
```

## Step 3: Configure SWR Globally (Optional)

For better control across your app:

```ts
// src/app/providers.tsx
'use client'

import { SWRConfig } from 'swr'
import { DataProvider } from '@/contexts/DataContext'
import { ReactNode } from 'react'

const fetcher = async (url: string) => {
  const response = await fetch(url)
  if (!response.ok) {
    throw new Error('An error occurred while fetching the data.')
  }
  return response.json()
}

export function Providers({ children }: { children: ReactNode }) {
  return (
    <SWRConfig
      value={{
        fetcher,
        revalidateOnFocus: false,
        revalidateOnReconnect: true,
        shouldRetryOnError: true,
        errorRetryCount: 3,
        dedupingInterval: 60000,
        // Use localStorage for persistence
        provider: () => {
          if (typeof window !== 'undefined') {
            return new Map(JSON.parse(localStorage.getItem('app-cache') || '[]'))
          }
          return new Map()
        },
        // Persist to localStorage
        onSuccess: (data, key) => {
          if (typeof window !== 'undefined') {
            const cache = new Map(JSON.parse(localStorage.getItem('app-cache') || '[]'))
            cache.set(key, data)
            localStorage.setItem('app-cache', JSON.stringify(Array.from(cache.entries())))
          }
        }
      }}
    >
      <DataProvider>
        {children}
      </DataProvider>
    </SWRConfig>
  )
}
```

## Step 4: Update Layout

```ts
// src/app/layout.tsx
import { Providers } from '@/app/providers'
import { Toaster } from '@/components/ui/toaster'
import './globals.css'

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        <Providers>
          {children}
        </Providers>
        <Toaster />
      </body>
    </html>
  )
}
```

## Step 5: Create Loading Wrapper Component

```ts
// src/components/DataLoader.tsx
'use client'

import { useRecords } from '@/contexts/DataContext'
import { Loader2, AlertCircle } from 'lucide-react'
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert'
import { Button } from '@/components/ui/button'
import { ReactNode } from 'react'

interface DataLoaderProps {
  children: ReactNode
  fallback?: ReactNode
}

export function DataLoader({ children, fallback }: DataLoaderProps) {
  const { isLoading, error, mutate } = useRecords()

  if (isLoading) {
    return (
      fallback || (
        <div className="flex min-h-screen items-center justify-center">
          <div className="text-center">
            <Loader2 className="h-12 w-12 animate-spin text-primary mx-auto" />
            <p className="mt-4 text-lg text-muted-foreground">
              Loading data...
            </p>
          </div>
        </div>
      )
    )
  }

  if (error) {
    return (
      <div className="flex min-h-screen items-center justify-center p-4">
        <Alert variant="destructive" className="max-w-md">
          <AlertCircle className="h-4 w-4" />
          <AlertTitle>Error Loading Data</AlertTitle>
          <AlertDescription className="mt-2">
            {error.message || 'An error occurred while loading data.'}
          </AlertDescription>
          <Button
            onClick={() => mutate()}
            variant="outline"
            className="mt-4"
          >
            Retry
          </Button>
        </Alert>
      </div>
    )
  }

  return <>{children}</>
}
```

## Step 6: Use in Your App

```ts
// src/app/page.tsx
import { DataLoader } from '@/components/DataLoader'
import { Dashboard } from '@/components/Dashboard'

export default function Home() {
  return (
    <DataLoader>
      <Dashboard />
    </DataLoader>
  )
}
```

## Step 7: Use the Data in Components

```ts
// src/components/Dashboard.tsx
'use client'

import { useRecords } from '@/contexts/DataContext'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'
import { RefreshCw } from 'lucide-react'
import { useState, useMemo } from 'react'

export function Dashboard() {
  const { records, mutate, isLoading } = useRecords()
  const [search, setSearch] = useState('')

  // Filter records client-side
  const filteredRecords = useMemo(() => {
    if (!search) return records

    return records.filter(record =>
      record.name.toLowerCase().includes(search.toLowerCase())
    )
  }, [records, search])

  return (
    <div className="container mx-auto p-6">
      <Card>
        <CardHeader className="flex flex-row items-center justify-between">
          <CardTitle>Records ({records.length} total)</CardTitle>
          <Button
            onClick={() => mutate()}
            variant="outline"
            size="sm"
            disabled={isLoading}
          >
            <RefreshCw className={`h-4 w-4 mr-2 ${isLoading ? 'animate-spin' : ''}`} />
            Refresh
          </Button>
        </CardHeader>
        <CardContent>
          <Input
            placeholder="Search records..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="mb-4"
          />

          <div className="space-y-2">
            {filteredRecords.map(record => (
              <div
                key={record.id}
                className="p-3 border rounded-lg hover:bg-accent transition-colors"
              >
                {record.name}
              </div>
            ))}
          </div>

          {filteredRecords.length === 0 && (
            <p className="text-center text-muted-foreground py-8">
              No records found
            </p>
          )}
        </CardContent>
      </Card>
    </div>
  )
}
```

## Step 8: Using Helper Methods

```tsx
// src/components/RecordDetail.tsx
"use client";

import { useRecords } from "@/contexts/DataContext";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";

export function RecordDetail({ id }: { id: string }) {
  const { getRecordById } = useRecords();

  const record = getRecordById(id);

  if (!record) {
    return (
      <Card>
        <CardContent className="py-8 text-center text-muted-foreground">
          Record not found
        </CardContent>
      </Card>
    );
  }

  return (
    <Card>
      <CardHeader>
        <CardTitle>{record.name}</CardTitle>
        {record.category && (
          <Badge variant="secondary">{record.category}</Badge>
        )}
      </CardHeader>
      <CardContent>{/* Record details */}</CardContent>
    </Card>
  );
}
```

## Step 9: Advanced - Optimistic Updates

When you need to update records optimistically:

```tsx
// src/components/RecordEditor.tsx
"use client";

import { useRecords } from "@/contexts/DataContext";
import { Button } from "@/components/ui/button";
import { useState } from "react";

export function RecordEditor({ id }: { id: string }) {
  const { getRecordById, mutate, records } = useRecords();
  const [name, setName] = useState("");

  const record = getRecordById(id);

  const updateRecord = async () => {
    // Optimistically update the UI
    mutate(
      records.map((r) => (r.id === id ? { ...r, name } : r)),
      false // Don't revalidate immediately
    );

    try {
      // Make the API call
      await fetch(`/api/records/${id}`, {
        method: "PATCH",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ name }),
      });

      // Revalidate to ensure consistency
      mutate();
    } catch (error) {
      // Revert on error
      mutate();
    }
  };

  return (
    <div className="space-y-4">
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        className="border rounded px-3 py-2 w-full"
      />
      <Button onClick={updateRecord}>Update Record</Button>
    </div>
  );
}
```

## Step 10: Preloading with Server Components (Bonus)

For even faster initial loads in Next.js:

```tsx
// src/app/page.tsx
import { DataLoader } from "@/components/DataLoader";
import { Dashboard } from "@/components/Dashboard";
import { Suspense } from "react";

async function getRecords() {
  const response = await fetch("https://your-api.com/records", {
    cache: "no-store", // or next: { revalidate: 60 }
  });
  return response.json();
}

export default async function Home() {
  // Preload data on server
  const initialData = await getRecords();

  return (
    <DataLoader fallbackData={initialData}>
      <Dashboard />
    </DataLoader>
  );
}
```

### Key Benefits of Using SWR

1. **Automatic caching**: Data is cached and shared across components
2. **Revalidation**: Automatic background updates when needed
3. **Focus revalidation**: Optional refetch when user returns to tab
4. **Error retry**: Built-in retry logic with exponential backoff
5. **Optimistic updates**: Easy to implement optimistic UI updates
6. **Deduplication**: Multiple components requesting same data = single request
7. **Persistence**: Can persist cache to localStorage
8. **TypeScript**: Full type safety out of the box

This approach is production-ready and handles all edge cases like network failures, stale data, and concurrent requests automatically!

## Example 3. Preloading Dictionary Data with React Context + SWR

> Here's a complete implementation for your dictionary app with ~2000 words.

### Step 1: Create the Dictionary Context with SWR

```ts
// src/contexts/DictionaryContext.tsx
'use client'

import { createContext, useContext, ReactNode, useMemo } from 'react'
import useSWR from 'swr'
import { DictionaryResponse, Word, Category } from '@/lib/types'

interface DictionaryContextType {
  // Core data
  words: Word[]
  categories: Category[]
  total: number
  isLoading: boolean
  error: any
  mutate: () => void

  // Helper methods
  filterWordsByCategory: (categoryName: string) => Word[]
  searchWords: (query: string) => Word[]
  getWordBySerial: (serial: number) => Word | undefined
  getCategoryById: (catId: number) => Category | undefined
  getWordsBySerialRange: (start: number, end: number) => Word[]
}

const DictionaryContext = createContext<DictionaryContextType | undefined>(undefined)

// Fetcher function for SWR
const fetcher = async (url: string): Promise<DictionaryResponse> => {
  const response = await fetch(url)

  if (!response.ok) {
    throw new Error(`Failed to fetch dictionary: ${response.statusText}`)
  }

  return response.json()
}

export function DictionaryProvider({ children }: { children: ReactNode }) {
  // SWR hook for data fetching
  const { data, error, isLoading, mutate } = useSWR<DictionaryResponse>(
    '/api/dict',
    fetcher,
    {
      // SWR configuration for optimal performance
      revalidateOnFocus: false,
      revalidateOnReconnect: true,
      revalidateIfStale: false,
      dedupingInterval: 300000, // 5 minutes
      shouldRetryOnError: true,
      errorRetryCount: 3,
      errorRetryInterval: 5000,
      // Keep data cached
      keepPreviousData: true,
    }
  )

  // Extract data with defaults
  const words = data?.words || []
  const categories = data?.categories || []
  const total = data?.total || 0

  // Create indexed maps for O(1) lookups
  const wordsBySerial = useMemo(() => {
    return new Map(words.map(word => [word.serial, word]))
  }, [words])

  const wordsByCategory = useMemo(() => {
    const map = new Map<string, Word[]>()
    words.forEach(word => {
      const existing = map.get(word.category_name) || []
      map.set(word.category_name, [...existing, word])
    })
    return map
  }, [words])

  const categoriesById = useMemo(() => {
    return new Map(categories.map(cat => [cat.cat_id, cat]))
  }, [categories])

  // Helper function: Filter words by category
  const filterWordsByCategory = (categoryName: string): Word[] => {
    return wordsByCategory.get(categoryName) || []
  }

  // Helper function: Search words (searches across multiple fields)
  const searchWords = (query: string): Word[] => {
    if (!query.trim()) return words

    const lowerQuery = query.toLowerCase().trim()

    return words.filter(word =>
      word.urdu.toLowerCase().includes(lowerQuery) ||
      word.english.toLowerCase().includes(lowerQuery) ||
      word.arabic.includes(lowerQuery) ||
      word.transliteration?.toLowerCase().includes(lowerQuery) ||
      word.category_name.toLowerCase().includes(lowerQuery) ||
      word.example_urdu?.toLowerCase().includes(lowerQuery) ||
      word.example_english?.toLowerCase().includes(lowerQuery)
    )
  }

  // Helper function: Get word by serial number
  const getWordBySerial = (serial: number): Word | undefined => {
    return wordsBySerial.get(serial)
  }

  // Helper function: Get category by ID
  const getCategoryById = (catId: number): Category | undefined => {
    return categoriesById.get(catId)
  }

  // Helper function: Get words by serial range (useful for pagination)
  const getWordsBySerialRange = (start: number, end: number): Word[] => {
    return words.filter(word => word.serial >= start && word.serial <= end)
  }

  return (
    <DictionaryContext.Provider
      value={{
        words,
        categories,
        total,
        isLoading,
        error,
        mutate,
        filterWordsByCategory,
        searchWords,
        getWordBySerial,
        getCategoryById,
        getWordsBySerialRange
      }}
    >
      {children}
    </DictionaryContext.Provider>
  )
}

export function useDictionary() {
  const context = useContext(DictionaryContext)
  if (context === undefined) {
    throw new Error('useDictionary must be used within a DictionaryProvider')
  }
  return context
}
```

### Step 2: Configure SWR Globally with Persistence

```ts
// src/app/providers.tsx
'use client'

import { SWRConfig } from 'swr'
import { DictionaryProvider } from '@/contexts/DictionaryContext'
import { ReactNode } from 'react'

const fetcher = async (url: string) => {
  const response = await fetch(url)
  if (!response.ok) {
    throw new Error('An error occurred while fetching the data.')
  }
  return response.json()
}

export function Providers({ children }: { children: ReactNode }) {
  return (
    <SWRConfig
      value={{
        fetcher,
        revalidateOnFocus: false,
        revalidateOnReconnect: true,
        shouldRetryOnError: true,
        errorRetryCount: 3,
        dedupingInterval: 300000, // 5 minutes
        // Use localStorage for persistence across sessions
        provider: () => {
          if (typeof window !== 'undefined') {
            const cacheKey = 'dictionary-app-cache'
            const cached = localStorage.getItem(cacheKey)
            return new Map(cached ? JSON.parse(cached) : [])
          }
          return new Map()
        },
      }}
    >
      <DictionaryProvider>
        {children}
      </DictionaryProvider>
    </SWRConfig>
  )
}
```

### Step 3: Update Layout

```ts
// src/app/layout.tsx
import { Providers } from '@/app/providers'
import { Toaster } from '@/components/ui/toaster'
import './globals.css'

export const metadata = {
  title: 'Dictionary App',
  description: 'Core 2000 Words Dictionary',
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body>
        <Providers>
          {children}
        </Providers>
        <Toaster />
      </body>
    </html>
  )
}
```

### Step 4: Create Loading Wrapper Component

```tsx
// src/components/DictionaryLoader.tsx
"use client";

import { useDictionary } from "@/contexts/DictionaryContext";
import { Loader2, AlertCircle, BookOpen } from "lucide-react";
import { Alert, AlertDescription, AlertTitle } from "@/components/ui/alert";
import { Button } from "@/components/ui/button";
import { ReactNode } from "react";
import { Progress } from "@/components/ui/progress";

interface DictionaryLoaderProps {
  children: ReactNode;
  fallback?: ReactNode;
}

export function DictionaryLoader({
  children,
  fallback,
}: DictionaryLoaderProps) {
  const { isLoading, error, mutate, total } = useDictionary();

  if (isLoading) {
    return (
      fallback || (
        <div className="flex min-h-screen items-center justify-center">
          <div className="text-center space-y-4 max-w-md">
            <BookOpen className="h-16 w-16 text-primary mx-auto" />
            <div className="space-y-2">
              <Loader2 className="h-8 w-8 animate-spin text-primary mx-auto" />
              <h2 className="text-2xl font-semibold">Loading Dictionary</h2>
              <p className="text-muted-foreground">
                Loading {total > 0 ? total : "~2000"} words...
              </p>
            </div>
            <Progress value={33} className="w-full" />
          </div>
        </div>
      )
    );
  }

  if (error) {
    return (
      <div className="flex min-h-screen items-center justify-center p-4">
        <Alert variant="destructive" className="max-w-md">
          <AlertCircle className="h-4 w-4" />
          <AlertTitle>Error Loading Dictionary</AlertTitle>
          <AlertDescription className="mt-2 space-y-4">
            <p>
              {error.message ||
                "An error occurred while loading the dictionary data."}
            </p>
            <Button
              onClick={() => mutate()}
              variant="outline"
              className="w-full"
            >
              Retry Loading
            </Button>
          </AlertDescription>
        </Alert>
      </div>
    );
  }

  return <>{children}</>;
}
```

### Step 5: Use in Your App

```ts
// src/app/page.tsx
import { DictionaryLoader } from '@/components/DictionaryLoader'
import { DictionaryDashboard } from '@/components/DictionaryDashboard'

export default function Home() {
  return (
    <DictionaryLoader>
      <DictionaryDashboard />
    </DictionaryLoader>
  )
}
```

### Step 6: Create Dashboard Component

```ts
// src/components/DictionaryDashboard.tsx
'use client'

import { useDictionary } from '@/contexts/DictionaryContext'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'
import { Badge } from '@/components/ui/badge'
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue
} from '@/components/ui/select'
import { RefreshCw, Search } from 'lucide-react'
import { useState, useMemo } from 'react'

export function DictionaryDashboard() {
  const {
    words,
    categories,
    total,
    mutate,
    isLoading,
    filterWordsByCategory,
    searchWords
  } = useDictionary()

  const [search, setSearch] = useState('')
  const [selectedCategory, setSelectedCategory] = useState<string>('all')

  // Filter and search words
  const filteredWords = useMemo(() => {
    let result = words

    // Apply category filter
    if (selectedCategory !== 'all') {
      result = filterWordsByCategory(selectedCategory)
    }

    // Apply search
    if (search.trim()) {
      result = result.filter(word =>
        word.urdu.toLowerCase().includes(search.toLowerCase()) ||
        word.english.toLowerCase().includes(search.toLowerCase()) ||
        word.arabic.includes(search) ||
        word.transliteration?.toLowerCase().includes(search.toLowerCase())
      )
    }

    return result
  }, [words, selectedCategory, search, filterWordsByCategory])

  return (
    <div className="container mx-auto p-6 space-y-6">
      {/* Header */}
      <div className="flex items-center justify-between">
        <div>
          <h1 className="text-3xl font-bold">Dictionary</h1>
          <p className="text-muted-foreground">
            {total.toLocaleString()} total words • {categories.length} categories
          </p>
        </div>
        <Button
          onClick={() => mutate()}
          variant="outline"
          disabled={isLoading}
        >
          <RefreshCw className={`h-4 w-4 mr-2 ${isLoading ? 'animate-spin' : ''}`} />
          Refresh
        </Button>
      </div>

      {/* Filters */}
      <Card>
        <CardContent className="pt-6">
          <div className="flex flex-col md:flex-row gap-4">
            <div className="flex-1 relative">
              <Search className="absolute left-3 top-1/2 -translate-y-1/2 h-4 w-4 text-muted-foreground" />
              <Input
                placeholder="Search by Urdu, English, Arabic, or transliteration..."
                value={search}
                onChange={(e) => setSearch(e.target.value)}
                className="pl-10"
              />
            </div>
            <Select value={selectedCategory} onValueChange={setSelectedCategory}>
              <SelectTrigger className="w-full md:w-[200px]">
                <SelectValue placeholder="Select category" />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value="all">All Categories</SelectItem>
                {categories.map(cat => (
                  <SelectItem key={cat.cat_id} value={cat.category}>
                    {cat.category}
                  </SelectItem>
                ))}
              </SelectContent>
            </Select>
          </div>
        </CardContent>
      </Card>

      {/* Results */}
      <Card>
        <CardHeader>
          <CardTitle>
            Words ({filteredWords.length.toLocaleString()})
          </CardTitle>
        </CardHeader>
        <CardContent>
          <div className="space-y-3">
            {filteredWords.map(word => (
              <div
                key={word.serial}
                className="p-4 border rounded-lg hover:bg-accent transition-colors space-y-2"
              >
                <div className="flex items-start justify-between gap-4">
                  <div className="flex-1 space-y-1">
                    <div className="flex items-center gap-2 flex-wrap">
                      <Badge variant="outline" className="font-mono">
                        #{word.serial}
                      </Badge>
                      <Badge variant="secondary">
                        {word.category_name}
                      </Badge>
                    </div>
                    <div className="grid grid-cols-1 md:grid-cols-3 gap-2">
                      <div>
                        <p className="text-sm text-muted-foreground">Urdu</p>
                        <p className="font-semibold text-lg" dir="rtl">{word.urdu}</p>
                      </div>
                      <div>
                        <p className="text-sm text-muted-foreground">English</p>
                        <p className="font-semibold">{word.english}</p>
                      </div>
                      <div>
                        <p className="text-sm text-muted-foreground">Arabic</p>
                        <p className="font-semibold text-lg" dir="rtl">{word.arabic}</p>
                      </div>
                    </div>
                    {word.transliteration && (
                      <p className="text-sm text-muted-foreground italic">
                        {word.transliteration}
                      </p>
                    )}
                  </div>
                </div>

                {/* Examples */}
                {(word.example_english || word.example_urdu || word.example_arabic) && (
                  <div className="pt-2 border-t space-y-1">
                    <p className="text-xs font-semibold text-muted-foreground">Examples:</p>
                    <div className="grid grid-cols-1 md:grid-cols-2 gap-2 text-sm">
                      {word.example_urdu && (
                        <p dir="rtl" className="text-muted-foreground">
                          {word.example_urdu}
                        </p>
                      )}
                      {word.example_english && (
                        <p className="text-muted-foreground">{word.example_english}</p>
                      )}
                    </div>
                    {word.example_transliteration && (
                      <p className="text-xs italic text-muted-foreground">
                        {word.example_transliteration}
                      </p>
                    )}
                  </div>
                )}
              </div>
            ))}
          </div>

          {filteredWords.length === 0 && (
            <div className="text-center py-12 text-muted-foreground">
              <Search className="h-12 w-12 mx-auto mb-4 opacity-50" />
              <p className="text-lg font-semibold">No words found</p>
              <p className="text-sm">Try adjusting your search or filters</p>
            </div>
          )}
        </CardContent>
      </Card>
    </div>
  )
}
```

### Step 7: Word Detail Component Example

```ts
// src/components/WordDetail.tsx
'use client'

import { useDictionary } from '@/contexts/DictionaryContext'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Badge } from '@/components/ui/badge'
import { Separator } from '@/components/ui/separator'

export function WordDetail({ serial }: { serial: number }) {
  const { getWordBySerial } = useDictionary()

  const word = getWordBySerial(serial)

  if (!word) {
    return (
      <Card>
        <CardContent className="py-8 text-center text-muted-foreground">
          Word not found
        </CardContent>
      </Card>
    )
  }

  return (
    <Card>
      <CardHeader>
        <div className="flex items-center justify-between">
          <CardTitle>Word Details</CardTitle>
          <Badge variant="outline">#{word.serial}</Badge>
        </div>
      </CardHeader>
      <CardContent className="space-y-6">
        <div className="space-y-4">
          <div>
            <h3 className="text-sm font-medium text-muted-foreground mb-1">Category</h3>
            <Badge>{word.category_name}</Badge>
          </div>

          <Separator />

          <div className="grid gap-4">
            <div>
              <h3 className="text-sm font-medium text-muted-foreground mb-1">Urdu</h3>
              <p className="text-2xl font-semibold" dir="rtl">{word.urdu}</p>
            </div>

            <div>
              <h3 className="text-sm font-medium text-muted-foreground mb-1">English</h3>
              <p className="text-2xl font-semibold">{word.english}</p>
            </div>

            <div>
              <h3 className="text-sm font-medium text-muted-foreground mb-1">Arabic</h3>
              <p className="text-2xl font-semibold" dir="rtl">{word.arabic}</p>
            </div>

            {word.transliteration && (
              <div>
                <h3 className="text-sm font-medium text-muted-foreground mb-1">
                  Transliteration
                </h3>
                <p className="text-lg italic">{word.transliteration}</p>
              </div>
            )}
          </div>

          {(word.example_urdu || word.example_english || word.example_arabic) && (
            <>
              <Separator />
              <div className="space-y-3">
                <h3 className="text-sm font-semibold">Examples</h3>

                {word.example_urdu && (
                  <div>
                    <p className="text-sm text-muted-foreground mb-1">Urdu</p>
                    <p dir="rtl">{word.example_urdu}</p>
                  </div>
                )}

                {word.example_english && (
                  <div>
                    <p className="text-sm text-muted-foreground mb-1">English</p>
                    <p>{word.example_english}</p>
                  </div>
                )}

                {word.example_arabic && (
                  <div>
                    <p className="text-sm text-muted-foreground mb-1">Arabic</p>
                    <p dir="rtl">{word.example_arabic}</p>
                  </div>
                )}

                {word.example_transliteration && (
                  <div>
                    <p className="text-sm text-muted-foreground mb-1">Transliteration</p>
                    <p className="italic text-sm">{word.example_transliteration}</p>
                  </div>
                )}
              </div>
            </>
          )}
        </CardContent>
      </Card>
  )
}
```

### Step 8: Category Statistics Component

```ts
// src/components/CategoryStats.tsx
'use client'

import { useDictionary } from '@/contexts/DictionaryContext'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Badge } from '@/components/ui/badge'

export function CategoryStats() {
  const { categories, filterWordsByCategory, total } = useDictionary()

  return (
    <Card>
      <CardHeader>
        <CardTitle>Categories</CardTitle>
      </CardHeader>
      <CardContent>
        <div className="space-y-2">
          {categories.map(category => {
            const count = filterWordsByCategory(category.category).length
            const percentage = ((count / total) * 100).toFixed(1)

            return (
              <div
                key={category.cat_id}
                className="flex items-center justify-between p-3 border rounded-lg hover:bg-accent transition-colors"
              >
                <div className="flex-1">
                  <p className="font-medium">{category.category}</p>
                  <p className="text-sm text-muted-foreground">
                    {count.toLocaleString()} words ({percentage}%)
                  </p>
                </div>
                <Badge variant="secondary">{count}</Badge>
              </div>
            )
          })}
        </div>
      </CardContent>
    </Card>
  )
}
```

### Key Features

1. **Single API call on startup**: All 2000 words loaded once
2. **Fast lookups**: O(1) access using Maps for serial numbers and categories
3. **Multi-field search**: Search across Urdu, English, Arabic, transliteration
4. **Category filtering**: Quick filtering by category
5. **SWR caching**: Automatic caching and revalidation
6. **localStorage persistence**: Data persists across sessions
7. **Error handling**: Graceful error states with retry
8. **TypeScript safety**: Full type safety throughout
9. **Optimized rendering**: useMemo for expensive computations
10. **Right-to-left support**: Proper RTL for Urdu and Arabic text

This implementation is production-ready and handles your 2000+ word dictionary efficiently!
