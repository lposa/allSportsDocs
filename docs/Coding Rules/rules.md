# Project Code Standards

## Core Principles

### YAGNI (You Aren't Gonna Need It)
- Don't add functionality until it's actually needed
- Avoid over-engineering and premature optimization
- Write code for current requirements, not future possibilities

### KISS (Keep It Simple, Stupid)
- Prefer simple solutions over complex ones
- Code should be easy to understand at first glance
- Avoid unnecessary abstractions

### DRY (Don't Repeat Yourself)
- Extract common logic into reusable functions
- Avoid code duplication
- Use shared utilities and helpers

### SOLID Principles
- **S**ingle Responsibility: Each function/class does one thing
- **O**pen/Closed: Open for extension, closed for modification
- **L**iskov Substitution: Subtypes must be substitutable
- **I**nterface Segregation: Many specific interfaces over one general
- **D**ependency Inversion: Depend on abstractions, not concretions

### Separation of Concerns
- Each module/component has a single, well-defined purpose
- Business logic separate from presentation
- Data access separate from business logic

## Code Quality

### Clean Code
- Code must be clean, easy to understand, and maintainable
- AI-generated code must be reviewed and modified to fit project standards
- Never push AI-generated code without modification

### Functional Programming
- Prefer pure functions when possible
- Avoid side effects
- Use immutable data structures where appropriate
- Functions should return values rather than mutate state

## Project Structure

### Folder Organization

```
src/
├── types/           # Global TypeScript type definitions
├── constants/       # Global constants
├── enums/          # Global enums
├── components/     # React components
│   └── [ComponentName]/
│       ├── index.ts
│       └── [ComponentName].tsx
└── ...
```

### Types
- All global types go in `types/` folder
- Types should be exported from `types/index.ts` for easy imports
- Component-specific types can live with the component

### Constants
- All global constants go in `constants/` folder
- Constants should be exported from `constants/index.ts`
- Use descriptive, UPPER_SNAKE_CASE names

### Enums
- All global enums go in `enums/` folder
- Enums should be exported from `enums/index.ts`
- Use descriptive names

### Components
- Each component has its own folder
- Each component folder must have an `index.ts` file for cleaner imports
- Component folder structure:
  ```
  components/
  └── MyComponent/
      ├── index.ts          # Exports the component
      ├── MyComponent.tsx   # Main component file
      └── MyComponent.test.tsx  # Tests (if applicable)
  ```

## Function Guidelines

### Function Names
- Use descriptive, verb-based names
- Function names should clearly indicate what they do
- Examples:
    - ✅ `getUserById`
    - ✅ `calculateTotalPrice`
    - ❌ `process`
    - ❌ `handle`

### Single Responsibility
- Each function should do exactly one thing
- If a function does multiple things, split it into multiple functions
- Functions should be small and focused

### Function Arguments
- Limit function arguments (ideally 3 or fewer)
- If you need more arguments, consider using an options object
- Example:
  ```typescript
  // ❌ Too many arguments
  function createUser(name, email, age, role, status, createdAt) {}
  
  // ✅ Use options object
  function createUser(userData: CreateUserData) {}
  ```

## Code Style

### No Comments
- Code should be self-documenting
- Use descriptive variable and function names instead of comments
- If code needs explanation, refactor it to be clearer

### No Console Logs
- Remove all `console.log()`, `console.error()`, `console.warn()` statements
- Use proper logging framework if logging is needed
- In production code, use structured logging

### No Debug Code
- Remove all debug statements
- Remove commented-out code
- Remove temporary variables used for debugging

## React Rules

### Component Structure
- Use functional components
- Use hooks for state and side effects
- Keep components small and focused
- Extract complex logic into custom hooks

### Props
- Use TypeScript interfaces for props
- Destructure props at the component level
- Use default props when appropriate

### State Management
- Prefer local state with `useState` for component-specific state
- Use context for shared state across components
- Avoid prop drilling beyond 2-3 levels

## Error Handling

### Try-Catch
- Use try-catch for async operations
- Handle errors gracefully
- Provide meaningful error messages
- Don't swallow errors silently

### Error Logging
- Log errors using proper logging framework
- Include context in error logs
- Don't expose sensitive information in error messages

## Technical Debt

### TODO Comments
- When creating technical debt, add a `TODO` comment
- Format: `TODO: [Description] - [Ticket Number]`
- Example: `TODO: Refactor this function to handle edge cases - TICKET-123`

### Creating Tickets
- Immediately create a ticket for any technical debt
- Link the ticket number in the TODO comment
- Don't accumulate technical debt without tracking

## Import Organization

### Import Order
1. External libraries (React, third-party)
2. Internal utilities and helpers
3. Types and interfaces
4. Constants and enums
5. Relative imports (components, etc.)

### Import Style
- Use named imports when possible
- Group related imports together
- Use absolute imports for shared code

## TypeScript Guidelines

### Type Safety
- Avoid `any` type
- Use proper types for all variables and functions
- Leverage TypeScript's type inference where appropriate

### Interfaces vs Types
- Use `interface` for object shapes
- Use `type` for unions, intersections, and computed types

## Testing

### Test Coverage
- Write tests for business logic
- Test edge cases and error conditions
- Keep tests simple and focused

### Test Organization
- Tests should mirror source structure
- Use descriptive test names
- Follow AAA pattern: Arrange, Act, Assert

## Git Practices

### Commit Messages
- Use clear, descriptive commit messages
- Reference ticket numbers when applicable
- Keep commits focused on a single change

### Code Review
- All code must be reviewed before merging
- Review for adherence to these standards
- Provide constructive feedback

## Examples

### Good Code
```typescript
// types/user.ts
export interface User {
  id: string;
  email: string;
  name: string;
}

// helpers/userHelpers.ts
export const getUserById = async (id: string): Promise<User | null> => {
  const user = await db.collection("users").doc(id).get();
  return user.exists ? (user.data() as User) : null;
};

// components/UserProfile/index.ts
export { UserProfile } from "./UserProfile";

// components/UserProfile/UserProfile.tsx
interface UserProfileProps {
  userId: string;
}

export const UserProfile = ({ userId }: UserProfileProps) => {
  const [user, setUser] = useState<User | null>(null);
  
  useEffect(() => {
    getUserById(userId).then(setUser);
  }, [userId]);
  
  if (!user) return null;
  
  return <div>{user.name}</div>;
};
```

### Bad Code
```typescript
// ❌ Types in component file
interface User {
  id: string;
}

// ❌ Comment explaining what code does
// This function gets a user by their ID
function get(id: string) {
  console.log("Getting user", id); // ❌ Console log
  // TODO: Add error handling // ❌ TODO without ticket
  return db.collection("users").doc(id).get();
}

// ❌ Component without index.ts
// components/UserProfile.tsx
export const UserProfile = (props: any) => { // ❌ Using any
  // ❌ Too many responsibilities
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setLoading(true);
    get(props.userId).then((doc) => {
      setUser(doc.data());
      setLoading(false);
    });
  }, [props.userId]);
  
  // ... 100 more lines
};
```

## Enforcement

### Pre-commit Hooks
- Run linters before committing
- Check for console.log statements
- Verify TypeScript compilation

### Code Review Checklist
- [ ] Follows folder structure
- [ ] No comments in code
- [ ] No console.logs
- [ ] Functions do one thing
- [ ] Types in types folder
- [ ] Constants in constants folder
- [ ] Enums in enums folder
- [ ] Components have index.ts
- [ ] No technical debt without ticket

## Questions?

If you're unsure about any of these standards, ask before implementing. It's better to clarify upfront than to refactor later.

