# Next.js 14 App Router Development Rules

**Activation Mode:** Glob: `app/**/*.{ts,tsx}`  
**Last Updated:** 2025-01-18  
**Tested With:** Windsurf Cascade, Next.js 14.0.0+

---

## Project Context

### Tech Stack
- **Framework:** Next.js 14+ (App Router)
- **Language:** TypeScript 5.3+
- **Styling:** TailwindCSS 3.4+ with shadcn/ui
- **State:** Zustand + TanStack Query v5
- **Database:** PostgreSQL with Prisma ORM
- **Authentication:** NextAuth.js v5
- **Deployment:** Vercel

### Architecture Philosophy
- Server Components by default
- Client Components only when necessary
- Progressive enhancement
- Type-safe end-to-end

---

## Code Standards

### TypeScript
```typescript
// ✅ Always use strict mode
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true
  }
}

// ✅ Use interfaces for objects
interface User {
  id: string;
  email: string;
  name: string;
}

// ✅ Use types for unions/intersections
type UserRole = 'admin' | 'user' | 'guest';
type UserWithRole = User & { role: UserRole };

// ❌ Avoid 'any'
const data: any = {}; // BAD

// ✅ Use 'unknown' and type guard
const data: unknown = {};
if (isUser(data)) {
  console.log(data.email); // GOOD
}
```

### Component Structure
```typescript
// ✅ Server Component (default)
interface PageProps {
  params: { id: string };
  searchParams: { query?: string };
}

export default async function UserPage({ params, searchParams }: PageProps) {
  const user = await fetchUser(params.id);
  return <UserProfile user={user} />;
}

// ✅ Client Component (only when needed)
'use client';

import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}

// ❌ Don't use 'use client' unnecessarily
'use client'; // BAD - no client features used
export function StaticText() {
  return <p>Hello World</p>;
}
```

### File Naming
```
app/
├── (auth)/                    # Route group (lowercase with dashes)
│   ├── login/
│   │   └── page.tsx          # Route: /login
│   └── register/
│       └── page.tsx          # Route: /register
├── dashboard/
│   ├── page.tsx              # Route: /dashboard
│   ├── loading.tsx           # Loading UI
│   ├── error.tsx             # Error UI
│   └── layout.tsx            # Nested layout
└── api/
    └── users/
        └── route.ts          # API: /api/users

components/
├── ui/                        # shadcn/ui components (lowercase)
│   ├── button.tsx
│   └── card.tsx
└── features/                  # Feature components (PascalCase)
    ├── UserProfile.tsx
    └── PostList.tsx
```

---

## Next.js Patterns

### Data Fetching

<data_fetching>
```typescript
// ✅ Server Component - fetch directly
export default async function PostsPage() {
  const posts = await db.post.findMany();
  return <PostList posts={posts} />;
}

// ✅ With caching and revalidation
export default async function PostsPage() {
  const posts = await fetch('https://api.example.com/posts', {
    next: { revalidate: 3600 } // Cache for 1 hour
  }).then(res => res.json());
  
  return <PostList posts={posts} />;
}

// ✅ Client Component - use TanStack Query
'use client';

export function PostListClient() {
  const { data, isLoading } = useQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts
  });
  
  if (isLoading) return <PostsLoading />;
  return <PostList posts={data} />;
}

// ❌ Don't use useEffect for data fetching
'use client';

export function PostListBad() {
  const [posts, setPosts] = useState([]);
  
  useEffect(() => {
    fetch('/api/posts')
      .then(res => res.json())
      .then(setPosts);
  }, []); // BAD
  
  return <PostList posts={posts} />;
}
```
</data_fetching>

### Loading States

```typescript
// ✅ Use loading.tsx for route segments
// app/dashboard/loading.tsx
export default function DashboardLoading() {
  return <DashboardSkeleton />;
}

// ✅ Use Suspense for component-level loading
import { Suspense } from 'react';

export default function Page() {
  return (
    <Suspense fallback={<PostsSkeleton />}>
      <Posts />
    </Suspense>
  );
}

// ✅ Use streaming with multiple Suspense boundaries
export default function Page() {
  return (
    <>
      <Suspense fallback={<HeaderSkeleton />}>
        <Header />
      </Suspense>
      
      <Suspense fallback={<ContentSkeleton />}>
        <Content />
      </Suspense>
    </>
  );
}
```

### Error Handling

```typescript
// ✅ Use error.tsx for error boundaries
// app/dashboard/error.tsx
'use client';

export default function DashboardError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={reset}>Try again</button>
    </div>
  );
}

// ✅ Throw errors in Server Components
export default async function Page() {
  const user = await fetchUser();
  
  if (!user) {
    throw new Error('User not found');
  }
  
  return <UserProfile user={user} />;
}
```

### Metadata

```typescript
// ✅ Static metadata
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Dashboard',
  description: 'User dashboard',
};

// ✅ Dynamic metadata
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const user = await fetchUser(params.id);
  
  return {
    title: user.name,
    description: `Profile of ${user.name}`,
    openGraph: {
      title: user.name,
      images: [user.avatar],
    },
  };
}
```

---

## Server Actions

<server_actions>
```typescript
// ✅ Define Server Actions
// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;
  
  // Validate
  if (!title || !content) {
    return { error: 'Title and content required' };
  }
  
  // Create
  await db.post.create({
    data: { title, content }
  });
  
  // Revalidate and redirect
  revalidatePath('/posts');
  redirect('/posts');
}

// ✅ Use in Client Component
'use client';

import { createPost } from '@/app/actions';

export function CreatePostForm() {
  return (
    <form action={createPost}>
      <input name="title" required />
      <textarea name="content" required />
      <button type="submit">Create</button>
    </form>
  );
}

// ✅ Progressive enhancement with useFormStatus
'use client';

import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Creating...' : 'Create Post'}
    </button>
  );
}
```
</server_actions>

---

## Performance Optimization

<performance>
```typescript
// ✅ Use dynamic imports for heavy components
import dynamic from 'next/dynamic';

const HeavyChart = dynamic(() => import('@/components/HeavyChart'), {
  loading: () => <ChartSkeleton />,
  ssr: false // Disable SSR if not needed
});

// ✅ Optimize images
import Image from 'next/image';

export function UserAvatar({ src }: { src: string }) {
  return (
    <Image
      src={src}
      alt="User avatar"
      width={48}
      height={48}
      priority={false} // Set true for LCP images
    />
  );
}

// ✅ Use React.memo for expensive components
import { memo } from 'react';

export const ExpensiveList = memo(function ExpensiveList({ items }) {
  return (
    <ul>
      {items.map(item => <li key={item.id}>{item.name}</li>)}
    </ul>
  );
});

// ✅ Minimize client-side JavaScript
// Prefer Server Components over Client Components
// Use 'use client' only when necessary:
// - useState, useEffect, event handlers
// - Browser APIs (localStorage, etc.)
// - Third-party libraries that use client features
```
</performance>

---

## State Management

<state_management>
```typescript
// ✅ Use Zustand for global state
// lib/store.ts
import { create } from 'zustand';

interface UserStore {
  user: User | null;
  setUser: (user: User) => void;
  clearUser: () => void;
}

export const useUserStore = create<UserStore>((set) => ({
  user: null,
  setUser: (user) => set({ user }),
  clearUser: () => set({ user: null }),
}));

// ✅ Use TanStack Query for server state
'use client';

export function UserProfile({ userId }: { userId: string }) {
  const { data: user, isLoading } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
    staleTime: 5 * 60 * 1000, // 5 minutes
  });
  
  if (isLoading) return <Skeleton />;
  return <UserCard user={user} />;
}

// ❌ Don't use Context for server state
// Use TanStack Query instead
```
</state_management>

---

## Styling with TailwindCSS

```typescript
// ✅ Use Tailwind utility classes
export function Button({ children }: { children: React.ReactNode }) {
  return (
    <button className="rounded-lg bg-blue-600 px-4 py-2 text-white hover:bg-blue-700">
      {children}
    </button>
  );
}

// ✅ Use cn() helper for conditional classes
import { cn } from '@/lib/utils';

export function Button({ variant, children }: ButtonProps) {
  return (
    <button
      className={cn(
        "rounded-lg px-4 py-2",
        variant === 'primary' && "bg-blue-600 text-white",
        variant === 'secondary' && "bg-gray-200 text-gray-900"
      )}
    >
      {children}
    </button>
  );
}

// ✅ Use shadcn/ui for complex components
import { Button } from '@/components/ui/button';
import { Card } from '@/components/ui/card';

export function UserCard() {
  return (
    <Card>
      <Button>Click me</Button>
    </Card>
  );
}
```

---

## API Routes

```typescript
// ✅ Use Route Handlers
// app/api/users/route.ts
import { NextResponse } from 'next/server';

export async function GET(request: Request) {
  const users = await db.user.findMany();
  return NextResponse.json(users);
}

export async function POST(request: Request) {
  const body = await request.json();
  
  // Validate
  const result = UserSchema.safeParse(body);
  if (!result.success) {
    return NextResponse.json(
      { error: result.error },
      { status: 400 }
    );
  }
  
  // Create
  const user = await db.user.create({
    data: result.data
  });
  
  return NextResponse.json(user, { status: 201 });
}

// ✅ Use dynamic route segments
// app/api/users/[id]/route.ts
export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  const user = await db.user.findUnique({
    where: { id: params.id }
  });
  
  if (!user) {
    return NextResponse.json(
      { error: 'User not found' },
      { status: 404 }
    );
  }
  
  return NextResponse.json(user);
}
```

---

## Security

<security>
```typescript
// ✅ Validate all inputs with Zod
import { z } from 'zod';

const CreatePostSchema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().min(1).max(10000),
});

// ✅ Use Server Actions for mutations
'use server';

export async function createPost(formData: FormData) {
  const session = await getServerSession();
  if (!session) {
    throw new Error('Unauthorized');
  }
  
  // Validate and create
}

// ✅ Use middleware for authentication
// middleware.ts
import { withAuth } from 'next-auth/middleware';

export default withAuth({
  callbacks: {
    authorized: ({ token }) => !!token
  },
});

export const config = {
  matcher: ['/dashboard/:path*', '/api/protected/:path*']
};

// ❌ Never expose secrets client-side
// .env.local
DATABASE_URL="..." // Server-only
NEXT_PUBLIC_API_URL="..." // Client-safe
```
</security>

---

## Testing

```typescript
// ✅ Test Server Components
import { render } from '@testing-library/react';
import UserPage from '@/app/users/[id]/page';

jest.mock('@/lib/db', () => ({
  user: {
    findUnique: jest.fn()
  }
}));

it('renders user profile', async () => {
  const mockUser = { id: '1', name: 'John' };
  db.user.findUnique.mockResolvedValue(mockUser);
  
  const Component = await UserPage({ params: { id: '1' } });
  const { getByText } = render(Component);
  
  expect(getByText('John')).toBeInTheDocument();
});

// ✅ Test Client Components
import { render, screen, fireEvent } from '@testing-library/react';
import { Counter } from '@/components/Counter';

it('increments counter', () => {
  render(<Counter />);
  
  const button = screen.getByRole('button');
  expect(button).toHaveTextContent('0');
  
  fireEvent.click(button);
  expect(button).toHaveTextContent('1');
});
```

---

## Common Mistakes to Avoid

### ❌ Using 'use client' unnecessarily
```typescript
// BAD
'use client';
export function StaticComponent() {
  return <div>Static content</div>;
}

// GOOD - No 'use client' needed
export function StaticComponent() {
  return <div>Static content</div>;
}
```

### ❌ Fetching in useEffect
```typescript
// BAD
'use client';
function Component() {
  const [data, setData] = useState(null);
  useEffect(() => {
    fetch('/api/data').then(r => r.json()).then(setData);
  }, []);
  return <div>{data}</div>;
}

// GOOD - Server Component
async function Component() {
  const data = await fetch('/api/data').then(r => r.json());
  return <div>{data}</div>;
}
```

### ❌ Not handling loading states
```typescript
// BAD
export default async function Page() {
  const data = await fetchData();
  return <Display data={data} />; // No loading state
}

// GOOD
// page.tsx
export default async function Page() {
  const data = await fetchData();
  return <Display data={data} />;
}

// loading.tsx
export default function Loading() {
  return <Skeleton />;
}
```

---

## Deployment Checklist

- [ ] Environment variables set in Vercel
- [ ] Database connection string configured
- [ ] Analytics enabled (Vercel Analytics)
- [ ] Error tracking setup (Sentry)
- [ ] Custom domain configured
- [ ] SSL certificate active
- [ ] Build succeeded locally: `npm run build`
- [ ] All tests passing: `npm test`
- [ ] No console.log in production code
- [ ] Lighthouse score > 90

---

**This rule file helps Cascade/Windsurf understand your Next.js 14 App Router patterns and generate code that follows your standards!**
