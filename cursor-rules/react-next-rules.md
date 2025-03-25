# Cursor Rules for Frontend Development for React, Next.js

You are an expert React / Next.js developer. You will follow the following rules when writing code with these tool.s

## Core Technologies
- **TypeScript** - For all code
- **Next.js** (App Router) - For server components and routing
- **React** - For component architecture
- **Tailwind CSS** - For styling
- **shadcn/ui** - For accessible UI components

## Code Style and Structure

### Component Design
- Use functional components with hooks; avoid classes
- Limit `'use client'` usage - prefer Server Components when possible
- Structure files: exported component, subcomponents, helpers, types
- Use early returns for improved readability

### Naming Conventions
- Use **kebab-case** for directories (`components/auth-wizard/`)
- Use **descriptive** variable and function names
- Prefix event handlers with `handle` (e.g., `handleClick`, `handleKeyDown`)
- Prefix boolean variables with `is`, `has`, or `should` (e.g., `isLoading`, `hasError`)

### TypeScript Usage
- Use interfaces for component props and data models
- Avoid enums; use const objects or maps instead
- Export shared types to dedicated files

### Styling Approach
- Use Tailwind classes for all styling; avoid CSS files
- Use `className="..."` for straightforward cases
- Use `class:` syntax instead of ternaries when conditional styling is needed
- Implement responsive design with Tailwind's mobile-first approach

### Accessibility
- Implement accessibility attributes on interactive elements
- Add `tabIndex`, `aria-label`, and keyboard event handlers as appropriate
- Ensure proper semantic HTML usage

## Example

```tsx
// UserCard.tsx
interface UserCardProps {
  name: string;
  email: string;
  isActive: boolean;
  onActivate: (id: string) => void;
}

export function UserCard({ name, email, isActive, onActivate }: UserCardProps) {
  const handleActivate = () => {
    onActivate(id);
  };
  
  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'Enter' || e.key === ' ') {
      handleActivate();
    }
  };
  
  return (
    <div className="flex items-center p-4 rounded-lg border gap-3">
      <div>
        <h3 className="font-medium">{name}</h3>
        <p className="text-sm text-gray-500">{email}</p>
      </div>
      <button
        onClick={handleActivate}
        onKeyDown={handleKeyDown}
        aria-label={isActive ? "Deactivate user" : "Activate user"}
        tabIndex={0}
        className="px-3 py-1.5 ml-auto rounded text-sm font-medium bg-blue-500 text-white"
      >
        {isActive ? "Deactivate" : "Activate"}
      </button>
    </div>
  );
}
```
