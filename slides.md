---
theme: seriph
background: https://images.unsplash.com/photo-1555066931-4365d14bab8c?ixlib=rb-4.0.3&auto=format&fit=crop&w=2070&q=80
title: Code Review for Next.js
info: |
  ## Code Review for Next.js
  Best practices, tools, and techniques for effective code reviews in Next.js projects.

  Learn more at [Next.js Docs](https://nextjs.org/docs)
class: text-center
drawings:
  persist: false
transition: slide-left
mdc: true
seoMeta:
  ogImage: auto
---

# Code Review for Next.js

## Best Practices, Tools & Techniques

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover:bg="white op-10">
    Let's dive in! <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/vercel/next.js" target="_blank" alt="GitHub" title="Open in GitHub" class="slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:logo-github />
  </a>
</div>

<!--
Welcome to our comprehensive guide on code review for Next.js applications. We'll cover best practices, tools, and specific techniques for reviewing Next.js code effectively.
-->

---

transition: fade-out

# Why Code Review Matters

Code review is essential for Next.js projects to ensure quality, performance, and maintainability

- 🔍 **Quality Assurance** - Catch bugs and issues before they reach production
- 📚 **Knowledge Sharing** - Team members learn from each other's approaches
- 🎯 **Consistency** - Maintain coding standards and architectural patterns
- ⚡ **Performance** - Identify performance bottlenecks and optimization opportunities
- 🛡️ **Security** - Spot potential security vulnerabilities early
- 🧹 **Clean Code** - Ensure readable, maintainable code structure
- 🚀 **Best Practices** - Enforce Next.js specific patterns and conventions

<br>

> "Code review is not about finding fault, it's about building better software together."

<!--
Code review is particularly important in Next.js projects due to the framework's complexity and the various rendering strategies (SSR, SSG, CSR) that need to be properly implemented.
-->

---

transition: slide-up

# What to Look for in Next.js Code Reviews

## Core Areas of Focus

<div class="grid grid-cols-2 gap-8 mt-8">

<div>

### 🏗️ **Architecture & Structure**

- Proper use of App Router vs Pages Router
- Component organization and file structure
- Separation of concerns
- Custom hooks and utilities placement

### ⚡ **Performance**

- Image optimization with `next/image`
- Bundle size and code splitting
- Lazy loading implementation
- Web Vitals considerations

</div>

<div>

### 🔄 **Rendering Strategies**

- Appropriate use of SSR, SSG, ISR
- Client-side rendering decisions
- Data fetching patterns
- Hydration considerations

### 🛡️ **Security & Best Practices**

- Environment variables handling
- API route security
- XSS prevention
- CSRF protection

</div>

</div>

<!--
These are the key areas that reviewers should focus on when reviewing Next.js code. Each area has specific patterns and anti-patterns to watch for.
-->

---

layout: two-cols
layoutClass: gap-16

# Code Review Tools for Next.js

## Essential Tools & Integrations

### 🔧 **Static Analysis**

- **ESLint** with Next.js config
- **TypeScript** for type safety
- **Prettier** for code formatting
- **SonarQube** for code quality

### 🚀 **Performance Tools**

- **Next.js Bundle Analyzer**
- **Lighthouse CI**
- **Web Vitals monitoring**
- **React DevTools Profiler**

### 🔄 **CI/CD Integration**

- **GitHub Actions** workflows
- **Vercel** preview deployments
- **Automated testing** suites

::right::

### 📋 **Review Platforms**

- **GitHub Pull Requests**
- **GitLab Merge Requests**
- **Bitbucket Code Reviews**
- **Azure DevOps**

### 🤖 **AI-Powered Tools**

- **CodeRabbit**
- **DeepCode**
- **GitHub Copilot**
- **SonarCloud**

### 📊 **Metrics & Reporting**

- **Code coverage** reports
- **Performance budgets**
- **Security scanning**
- **Dependency audits**

---

# Code Review Examples: Good vs Bad

## Image Optimization

<div class="grid grid-cols-2 gap-4">

<div>

### ❌ **Bad Practice**

```jsx
// Unoptimized image loading
<img
  src="/hero-image.jpg"
  alt="Hero"
  style={{ width: "100%", height: "auto" }}
/>
```

### ❌ **Poor Data Fetching**

```jsx
// Client-side only data fetching
export default function Page() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch("/api/data")
      .then((res) => res.json())
      .then(setData);
  }, []);

  return <div>{data?.title}</div>;
}
```

</div>

<div>

### ✅ **Good Practice**

```jsx
// Optimized with next/image
import Image from "next/image";

<Image
  src="/hero-image.jpg"
  alt="Hero"
  width={800}
  height={600}
  priority
  placeholder="blur"
/>;
```

### ✅ **Server-Side Data Fetching**

```jsx
// Using App Router with async components
export default async function Page() {
  const data = await fetch("/api/data", {
    next: { revalidate: 3600 },
  }).then((res) => res.json());

  return <div>{data.title}</div>;
}
```

</div>

</div>

<!--
These examples show common patterns that reviewers should look for. The bad practices can lead to performance issues, while the good practices leverage Next.js optimizations.
-->

---

# API Routes & Security Review

## Common Security Issues to Watch For

````md magic-move {lines: true}
```ts
// ❌ Insecure API route - No validation
export async function POST(request: Request) {
  const body = await request.json();

  // Direct database query without validation
  const result = await db.user.create({
    data: body,
  });

  return Response.json(result);
}
```

```ts
// ✅ Secure API route - With validation
import { z } from "zod";
import { rateLimit } from "@/lib/rate-limit";

const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
});

export async function POST(request: Request) {
  // Rate limiting
  const limiter = rateLimit({
    interval: 60 * 1000, // 1 minute
    uniqueTokenPerInterval: 500,
  });

  try {
    await limiter.check(10, "CACHE_TOKEN");
  } catch {
    return new Response("Rate limit exceeded", { status: 429 });
  }

  // Input validation
  const body = await request.json();
  const validatedData = createUserSchema.parse(body);

  // Secure database operation
  const result = await db.user.create({
    data: validatedData,
  });

  return Response.json({ id: result.id });
}
```
````

<!--
API route security is crucial in Next.js applications. Always check for proper input validation, rate limiting, authentication, and sanitized responses.
-->

---

# Performance Review Checklist

<div grid="~ cols-2 gap-6">
<div>

## 🚀 **Core Web Vitals**

### Largest Contentful Paint (LCP)

- ✅ Images optimized with `next/image`
- ✅ Critical resources preloaded
- ✅ Server-side rendering for above-fold content

### First Input Delay (FID)

- ✅ JavaScript bundles optimized
- ✅ Code splitting implemented
- ✅ Heavy computations moved to Web Workers

### Cumulative Layout Shift (CLS)

- ✅ Image dimensions specified
- ✅ Font loading optimized
- ✅ Dynamic content properly handled

</div>
<div>

## ⚡ **Next.js Optimizations**

### Bundle Analysis

```bash
# Check bundle size
npm run build
npm run analyze
```

### Image Optimization

```jsx
// Always specify dimensions
<Image
  src="/image.jpg"
  width={800}
  height={600}
  alt="Description"
  priority={isAboveFold}
/>
```

### Dynamic Imports

```jsx
// Lazy load heavy components
const HeavyComponent = dynamic(() => import("./HeavyComponent"), {
  loading: () => <Spinner />,
});
```

</div>
</div>

<!--
Performance is critical for user experience and SEO. These checks ensure your Next.js app meets modern web standards.
-->

---

class: px-20

# Complete Code Review Checklist

<div grid="~ cols-3 gap-4" class="text-sm">

<div>

## 🏗️ **Architecture**

- [ ] Proper folder structure
- [ ] App Router vs Pages Router usage
- [ ] Component composition
- [ ] Custom hooks extraction
- [ ] Utility functions organization
- [ ] Type definitions location

## 🔒 **Security**

- [ ] Environment variables secure
- [ ] API routes validated
- [ ] Input sanitization
- [ ] Authentication implemented
- [ ] CORS configured properly
- [ ] No sensitive data exposed

</div>

<div>

## ⚡ **Performance**

- [ ] Images optimized with `next/image`
- [ ] Bundle size analyzed
- [ ] Code splitting implemented
- [ ] Dynamic imports used
- [ ] Fonts optimized
- [ ] Critical CSS inlined
- [ ] Web Vitals considered

## 🔄 **Data Fetching**

- [ ] Appropriate rendering strategy
- [ ] Error handling implemented
- [ ] Loading states handled
- [ ] Caching strategy defined
- [ ] Revalidation configured

</div>

<div>

## 🧪 **Testing**

- [ ] Unit tests written
- [ ] Integration tests covered
- [ ] E2E tests for critical paths
- [ ] Accessibility tested
- [ ] Performance tested

## 📝 **Code Quality**

- [ ] TypeScript types defined
- [ ] ESLint rules followed
- [ ] Prettier formatting applied
- [ ] Comments where needed
- [ ] No console.log statements
- [ ] Error boundaries implemented

## 🚀 **Deployment**

- [ ] Build process verified
- [ ] Environment configs set
- [ ] Preview deployments work
- [ ] Performance budgets met

</div>

</div>

<!--
This comprehensive checklist covers all major areas that should be reviewed in Next.js applications. Use this as a guide during your code review process.
-->

---

# Best Practices for Code Reviews

## Creating a Positive Review Culture

<div class="grid grid-cols-2 gap-8 mt-6">

<div>

### 🎯 **For Reviewers**

<v-click>

- **Be constructive**, not critical
- **Explain the 'why'** behind suggestions
- **Praise good code** when you see it
- **Focus on the code**, not the person
- **Provide examples** of better approaches
- **Ask questions** instead of making demands

</v-click>

<v-click>

### 💡 **Example Comments**

```
✅ "Consider using useMemo here to optimize
   re-renders when props.data changes"

❌ "This is wrong, use useMemo"
```

</v-click>

</div>

<div>

### 👨‍💻 **For Authors**

<v-click>

- **Provide context** in PR descriptions
- **Keep PRs small** and focused
- **Self-review** before requesting review
- **Respond promptly** to feedback
- **Ask for clarification** when needed
- **Test thoroughly** before submitting

</v-click>

<v-click>

### 📝 **PR Template Example**

```markdown
## What changed?

- Added user authentication flow
- Implemented JWT token handling

## How to test?

1. Run `npm run dev`
2. Navigate to /login
3. Test with valid/invalid credentials

## Screenshots

[Include relevant screenshots]
```

</v-click>

</div>

</div>

<!--
Creating a positive code review culture is essential for team productivity and learning. Focus on collaboration and continuous improvement.
-->

---

layout: center
class: text-center

# Key Takeaways

<div class="text-left max-w-4xl mx-auto">

<div class="grid grid-cols-2 gap-12 mt-12">

<div>

## 🎯 **Remember**

- Code review is about **collaboration**, not criticism
- Focus on **Next.js specific patterns** and optimizations
- Use **automated tools** to catch basic issues
- **Performance and security** should be top priorities
- Keep reviews **constructive and educational**

</div>

<div>

## 🚀 **Action Items**

- Implement the **code review checklist** in your team
- Set up **automated tools** (ESLint, TypeScript, etc.)
- Create **PR templates** for consistency
- Establish **review guidelines** and culture
- Regularly **update your practices** as Next.js evolves

</div>

</div>

<div class="mt-16 text-center">

### 💡 **"Great code reviews lead to great applications"**

<div class="mt-8 text-sm opacity-75">
Start implementing these practices in your Next.js projects today!
</div>

</div>

</div>

<!--
Effective code reviews are essential for building high-quality Next.js applications. Focus on creating a positive, educational environment that helps the entire team grow.
-->

---
layout: center
class: text-center

# Thank You!

## Questions & Discussion

<div class="mt-12">

### 📚 **Resources**

[Next.js Documentation](https://nextjs.org/docs) · [Code Review Best Practices](https://github.com/features/code-review/) · [Web Performance](https://web.dev/performance/)

</div>

<div class="mt-8 text-sm opacity-75">

**Happy Reviewing!** 🚀

</div>

<!--
Thank you for attending this presentation on Next.js code reviews. Remember to implement these practices gradually and adapt them to your team's specific needs.
-->
