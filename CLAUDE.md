# Global Guidelines

## General

### File Naming
- Always use **kebab-case** for all files (`my-component.tsx`, `user-service.ts`).

### TypeScript
- Strict typing: avoid `any`.
- Use Zod for schema validation.
- Use `.enum(['value1', 'value2'])` instead of `.nativeEnum()` for Zod schemas.
- Prefer `type` over `interface` unless extension is needed.
- Use `??` instead of `||` for default values.

```typescript
const value = input.value ?? 'default'
```

### Testing

Name tests following the pattern: should [expected behavior] when [condition].
No logic in tests — no if, no loops.
Test behavior, not implementation details.

### Functions
- Use `const` with arrow functions instead of the `function` keyword.
- Apply **RORO pattern** (Receive Object, Return Object) when a function has more than 3 parameters — applies to all technologies and layers (components, services, controllers, utils).
- Use early returns to avoid deep nesting.

```typescript
// Do this:
const process = (user: User) => {
  if (!user) return null
  if (!user.isActive) return null
  return doSomething(user)
}

// Not this:
const process = (user: User) => {
  if (user) {
    if (user.isActive) {
      return doSomething(user)
    }
  }
}
```

### Code Quality
- No magic numbers — extract named constants.
- No `console.log` left in code.
- No commented-out code — delete it, Git keeps the history.

### Comments & Documentation
- Avoid useless comments — code should be self-explanatory.
- Only use TSDoc for public APIs, complex functions, or non-obvious behavior.
- No template/section comments in templates (`// User profile section`).
- **Never** create documentation files (`*.md`, READMEs) unless explicitly requested.

### Git
- Conventional commits: `feat:`, `fix:`, `chore:`, `refactor:`, `docs:`, `test:`.

---

## React

- Use existing UI components instead of raw HTML tags (`<Button>` vs `<button>`).
- Do not use useEffect for state derivation — use useMemo or derived instead.
- Avoid unnecessary re-renders: use memo() and useCallback on handlers passed as props.
- No business logic in components — extract to hooks or utils.
- Avoid props drilling beyond 2 levels — use Context, Zustand, or Jotai.
- Use `nuqs` for query params management instead of manual `useSearchParams` manipulation.
- Tailwind: use `size-*` instead of `w-* h-*` when width equals height.
- Style concatenation: never use template literals or raw string joins for conditional classes. Use `twMerge` / `clsx` if installed, otherwise `classnames`. Check `package.json` to pick the right one.

```tsx
// Do this:
cn('base-class', isActive && 'active', className)

// Not this:
`base-class ${isActive ? 'active' : ''} ${className}`
```

```jsx
<div className="size-8" />  // not <div className="w-8 h-8" />
```

---

## Next.js App Router

- Server Components by default — add `'use client'` only when strictly necessary.
- `'use client'` is needed only for: browser APIs, event listeners, `useState`, `useEffect`, and similar client-only hooks.
- Push `'use client'` as far down the component tree as possible — isolate interactive islands in small leaf components.
- Never mark a layout or a page as `'use client'` unless absolutely unavoidable.
- Use `nuqs` with `createSearchParamsCache` for type-safe, server-compatible query params.
- If a component contains more than 5 `useState`, suggest refactoring to `useReducer()` before proceeding.

```tsx
// Do this — isolate the client boundary:
// page.tsx (Server Component)
export default const Page = () => <ProductList />

// product-list.tsx (Server Component)
const ProductList = () => <AddToCartButton />  // passes down only what's needed

// add-to-cart-button.tsx
'use client'
const AddToCartButton = () => { ... }
```

---

## NestJS

- Use `const` arrow functions for service methods consistency.
- Apply RORO pattern for complex DTOs and service method parameters.
- Validate all incoming data with `class-validator` decorators on DTOs.
- Use custom exceptions extending `HttpException` instead of throwing raw errors.
- No business logic in controllers — delegate entirely to services.
- No direct database calls in controllers or resolvers — always go through a service.

### tRPC (if installed)
- A router's only responsibilities are: input validation, permission checks, and delegating to a service.
- **Never** put business logic in a router — it belongs in a service.

```typescript
// Do this:
const userRouter = router({
  update: protectedProcedure
    .input(updateUserSchema)
    .mutation(({ ctx, input }) => {
      checkPermission(ctx.user, 'user:update') // permission here
      return userService.update(input)          // logic in service
    }),
})

// Not this:
const userRouter = router({
  update: protectedProcedure
    .input(updateUserSchema)
    .mutation(async ({ ctx, input }) => {
      const user = await db.user.findUnique({ where: { id: input.id } })
      if (!user) throw new TRPCError({ code: 'NOT_FOUND' })
      return db.user.update({ ... }) // business logic leaking into router
    }),
})
```

### Shadcn/UI (if installed)
- Use Shadcn components at maximum — never reimplement a component that already exists in the library.
- Always refer to the LLM-friendly docs: https://ui.shadcn.com/llms.txt

### Better-Auth (if installed)
- Use Better-Auth for all authentication and session management — never roll custom auth logic.
- Always refer to the LLM-friendly docs: https://www.better-auth.com/llms.txt

---

## Angular

- Use `inject()` instead of constructor injection.
- Prefer `signal()` and `computed()` over `BehaviorSubject` where possible.
- Use `nuqs` or Angular Router `queryParams` via a dedicated service — never manipulate query params ad hoc in components.
- No business logic in components — extract to services.
- Avoid props drilling beyond 2 levels — use a store (NgRx, NgRx Signal Store) or a shared service.

---

## Svelte

- Use `$state`, `$derived`, and `$effect` (Svelte 5 runes) — avoid legacy `$:` reactive statements.
- No business logic in `.svelte` components — extract to `.svelte.ts` stores or utils.
- Use `nuqs` or a dedicated store for query params — never manipulate them directly in components.
- Avoid props drilling beyond 2 levels — use Svelte context (`setContext` / `getContext`) or a shared store.

---

## Workflow & Behavior

### Planning
- Enter plan mode for architectural decisions or tasks with 5+ steps.
- If an approach goes sideways, STOP and re-plan — don't keep pushing a broken path.
- Include verification steps in the plan, not just building steps.

### Subagents
- Use subagents for parallel independent research or when results would bloat the main context.
- One focused task per subagent.
- Keep simple searches (single file/class lookups) in the main thread — don't over-delegate.

### Verification
- Never mark a task complete without proving it works.
- Run tests, check logs, demonstrate correctness.
- Diff behavior between main and changes when relevant.

### Bug Fixing
- When given a bug report: investigate and fix autonomously. Don't ask for hand-holding.
- Point at logs, errors, failing tests — then resolve them.
- Go fix failing CI tests without being told how.

### Self-Correction
- After any correction from the user: save the lesson to memory so the same mistake doesn't repeat.
- Write concrete rules that prevent the pattern, not vague reminders.

### Core Principles
- **Simplicity first**: make every change as simple as possible, impact minimal code.
- **Find root causes**: no temporary fixes, no band-aids. Senior developer standards.
- **Pause before complexity**: for non-trivial changes, ask "is there a simpler way?" before implementing.
