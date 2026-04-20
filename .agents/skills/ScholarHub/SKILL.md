```markdown
# ScholarHub Development Patterns

> Auto-generated skill from repository analysis

## Overview
This skill teaches the core development patterns and conventions used in the ScholarHub TypeScript codebase. It covers file organization, import/export styles, commit message habits, and testing practices. By following these guidelines, contributors can maintain consistency and quality across the project.

## Coding Conventions

### File Naming
- Use **kebab-case** for all file names.
  - Example:  
    ```
    user-profile.ts
    data-fetcher.test.ts
    ```

### Import Style
- Use **relative imports** for referencing modules within the project.
  - Example:
    ```typescript
    import { fetchData } from './data-fetcher';
    ```

### Export Style
- Use **named exports** for all modules.
  - Example:
    ```typescript
    // In user-profile.ts
    export function getUserProfile(id: string) { ... }

    // In another file
    import { getUserProfile } from './user-profile';
    ```

### Commit Patterns
- Commit messages are freeform, often short (average 13 characters).
- No enforced prefixes or structure.

## Workflows

### Adding a New Feature
**Trigger:** When implementing a new feature or module  
**Command:** `/add-feature`

1. Create a new file using kebab-case (e.g., `new-feature.ts`).
2. Use relative imports to include dependencies.
3. Export functions or constants using named exports.
4. Write corresponding tests in a file named `new-feature.test.ts`.
5. Commit changes with a concise message.

### Writing Tests
**Trigger:** When adding or updating functionality  
**Command:** `/write-test`

1. Create a test file with the pattern `*.test.ts` (e.g., `user-profile.test.ts`).
2. Write tests for all exported functions.
3. Use the project's preferred (unknown) test framework.
4. Run tests to ensure correctness.

### Refactoring Code
**Trigger:** When improving or restructuring existing code  
**Command:** `/refactor`

1. Update file names to kebab-case if needed.
2. Change imports to use relative paths.
3. Ensure all exports are named.
4. Update or add tests as necessary.
5. Commit with a brief, descriptive message.

## Testing Patterns

- Test files follow the pattern `*.test.ts`.
- Each test file corresponds to a module, testing its named exports.
- The test framework is not specified; use the project's existing setup.
- Example test file:
  ```typescript
  // user-profile.test.ts
  import { getUserProfile } from './user-profile';

  test('fetches user profile', () => {
    // Test implementation
  });
  ```

## Commands
| Command        | Purpose                                      |
|----------------|----------------------------------------------|
| /add-feature   | Scaffold and implement a new feature/module  |
| /write-test    | Create and run tests for a module            |
| /refactor      | Refactor code to match project conventions   |
```
