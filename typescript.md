# TypeScript Project Setup Guide

This guide provides a comprehensive reference for setting up new TypeScript projects with best practices, consistent code quality, and automated tooling.

## Table of Contents

1. [EditorConfig](#editorconfig)
2. [VS Code Configuration](#vs-code-configuration)
3. [Prettier](#prettier)
4. [ESLint](#eslint)
5. [TypeScript Configuration](#typescript-configuration)
6. [Zod (Runtime Validation)](#zod-runtime-validation)
7. [Error Handling with Discriminated Unions](#error-handling-with-discriminated-unions)
8. [Git Hooks with Husky](#git-hooks-with-husky)
9. [Git Attributes](#git-attributes)
10. [Package.json Scripts](#packagejson-scripts)
11. [Installation Steps](#installation-steps)
12. [Code Smart: Readability Principles](#code-smart-readability-principles)
13. [A Word on "Prototypes" and Strict Settings](#a-word-on-prototypes-and-strict-settings)
14. [References](#references)

---

## EditorConfig

EditorConfig helps maintain consistent coding styles across different editors and IDEs.

**Check if your editor needs a plugin:** https://editorconfig.org/

### `.editorconfig`

```ini
# EditorConfig is awesome: https://EditorConfig.org

# Top-most EditorConfig file
root = true

# Default settings for all files
[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 4
max_line_length = 100

# JSON, YAML, and YML files use 2 spaces
[*.{json,yaml,yml}]
indent_size = 2

# Markdown files may have trailing spaces for line breaks
[*.md]
trim_trailing_whitespace = false
```

---

## VS Code Configuration

Configure VS Code to work seamlessly with your tools.

### `.vscode/settings.json`

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  "files.eol": "\n",
  "files.insertFinalNewline": true,
  "files.trimTrailingWhitespace": true,
  "[markdown]": {
    "files.trimTrailingWhitespace": false
  }
}
```

### `.vscode/extensions.json`

```json
{
  "recommendations": [
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint",
    "editorconfig.editorconfig",
    "wayou.vscode-todo-highlight"
  ]
}
```

---

## Prettier

Prettier enforces consistent code formatting.

### `.prettierrc.json`

```json
{
  "printWidth": 100,
  "tabWidth": 4,
  "useTabs": false,
  "semi": true,
  "singleQuote": true,
  "quoteProps": "consistent",
  "trailingComma": "all",
  "bracketSpacing": true,
  "endOfLine": "lf",
  "arrowParens": "always",
  "overrides": [
    {
      "files": ["*.json", "*.yaml", "*.yml"],
      "options": {
        "tabWidth": 2,
        "bracketSpacing": true,
        "printWidth": 100
      }
    },
    {
      "files": ["*.jsx", "*.tsx"],
      "options": {
        "singleQuote": false
      }
    }
  ]
}
```

### Installation

```bash
npm install --save-dev prettier
```

---

## ESLint

ESLint catches bugs and enforces code quality standards.

> **Note**: These rules are strict by design. They may feel restrictive initially, but they prevent entire categories of bugs and enforce consistency. Resist the urge to disable rules when starting a new project - it's much harder to enable them later when you have a large codebase. See "A Word on Prototypes and Strict Settings" at the end of this document.

### `eslint.config.js` (ESLint 9+ Flat Config)

```javascript
const typescriptParser = require('@typescript-eslint/parser');
const typescriptPlugin = require('@typescript-eslint/eslint-plugin');
const importPlugin = require('eslint-plugin-import');
const unusedImportsPlugin = require('eslint-plugin-unused-imports');
const prettierConfig = require('eslint-config-prettier');

module.exports = [
    {
        ignores: ['dist/**', 'node_modules/**', 'coverage/**'],
    },
    {
        files: ['**/*.ts', '**/*.tsx'],
        languageOptions: {
            parser: typescriptParser,
            parserOptions: {
                ecmaVersion: 2020,
                sourceType: 'module',
                project: './tsconfig.json',
            },
        },
        plugins: {
            '@typescript-eslint': typescriptPlugin,
            'import': importPlugin,
            'unused-imports': unusedImportsPlugin,
        },
        rules: {
            // Blank line rules
            'no-multiple-empty-lines': ['error', { max: 2, maxEOF: 1, maxBOF: 0 }],
            'padding-line-between-statements': [
                'error',
                { blankLine: 'always', prev: 'import', next: '*' },
                { blankLine: 'any', prev: 'import', next: 'import' },
                { blankLine: 'always', prev: 'function', next: '*' },
                { blankLine: 'always', prev: '*', next: 'function' },
                { blankLine: 'always', prev: 'class', next: '*' },
                { blankLine: 'always', prev: '*', next: 'class' },
                { blankLine: 'always', prev: '*', next: 'export' },
                { blankLine: 'always', prev: 'export', next: '*' },
                { blankLine: 'any', prev: 'export', next: 'export' },
            ],

            // Line length rules
            'max-len': [
                'error',
                {
                    code: 100,
                    ignoreStrings: true,
                    ignoreTemplateLiterals: true,
                    ignoreUrls: true,
                    ignoreComments: false,
                },
            ],

            // Import sorting and unused imports
            'import/order': [
                'error',
                {
                    'groups': ['builtin', 'external', 'internal', 'parent', 'sibling', 'index'],
                    'newlines-between': 'always',
                    'alphabetize': { order: 'asc', caseInsensitive: true },
                },
            ],
            'unused-imports/no-unused-imports': 'error',
            'unused-imports/no-unused-vars': [
                'error',
                {
                    vars: 'all',
                    varsIgnorePattern: '^_',
                    args: 'after-used',
                    argsIgnorePattern: '^_',
                },
            ],

            // Core quality rules
            'curly': ['error', 'all'],
            // Require === and !== always, no exceptions
            // To allow == null and != null for concise null/undefined checks, use:
            // 'eqeqeq': ['error', 'always', { null: 'ignore' }],
            'eqeqeq': ['error', 'always'],
            'no-var': 'error',
            'prefer-const': 'error',
            'prefer-template': 'error',
            'no-console': 'error',

            // TypeScript-specific rules
            '@typescript-eslint/no-unused-vars': 'off',
            '@typescript-eslint/naming-convention': [
                'error',
                // Constants: Use UPPER_CASE for hardcoded values (primitives, arrays, objects)
                // Examples: const MAX_RETRIES = 3, const VALID_CODES = [1, 2, 3]
                //           const ERROR_MESSAGES = { notFound: 'Not found' }
                // Use camelCase for: functions, class instances, computed/dynamic values
                // Examples: const getUserData = () => {...}, const config = loadConfig()
                {
                    selector: 'variable',
                    modifiers: ['const'],
                    format: ['camelCase', 'UPPER_CASE'],
                    leadingUnderscore: 'allow',
                },
                {
                    selector: 'variable',
                    format: ['camelCase'],
                    leadingUnderscore: 'allow',
                },
                {
                    selector: 'function',
                    format: ['camelCase'],
                },
                {
                    selector: 'parameter',
                    format: ['camelCase'],
                    leadingUnderscore: 'allow',
                },
                {
                    selector: 'typeLike',
                    format: ['PascalCase'],
                },
            ],
            '@typescript-eslint/no-explicit-any': 'error',
            '@typescript-eslint/no-floating-promises': 'error',
            '@typescript-eslint/await-thenable': 'error',
            '@typescript-eslint/no-misused-promises': 'error',
            '@typescript-eslint/consistent-type-imports': 'error',
            '@typescript-eslint/prefer-nullish-coalescing': 'error',
            '@typescript-eslint/no-unnecessary-condition': 'error',
            '@typescript-eslint/no-non-null-assertion': 'error',
            '@typescript-eslint/no-unused-expressions': 'error',
            '@typescript-eslint/switch-exhaustiveness-check': 'error',
            '@typescript-eslint/array-type': ['error', { default: 'array-simple' }],
            '@typescript-eslint/consistent-generic-constructors': ['error', 'constructor'],
            '@typescript-eslint/explicit-function-return-type': [
                'error',
                {
                    allowExpressions: true,
                    allowTypedFunctionExpressions: true,
                    allowHigherOrderFunctions: true,
                },
            ],
            '@typescript-eslint/strict-boolean-expressions': [
                'error',
                {
                    allowAny: false,
                    allowNullableBoolean: false,
                    allowNullableString: false,
                    allowNumber: false,
                    allowNullableNumber: false,
                    allowNullableObject: false,
                },
            ],
        },
    },
    prettierConfig,
];
```

### Installation

```bash
npm install --save-dev eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin \
  eslint-plugin-import eslint-plugin-unused-imports eslint-config-prettier
```

### When Long Functions Are Actually Better

Sometimes a 100-line function that tells a complete story is more maintainable than splitting it into 10 smaller functions. Consider keeping a function long when:

- **The function tells a linear story**: Each step follows naturally from the previous one
- **Breaking it up would require passing many parameters**: Creating helper functions that need 5+ parameters is often worse than one longer function
- **The logic is cohesive**: All the code relates to a single, clear purpose
- **Breaking it up adds indirection**: If you'd need to constantly jump between functions to understand the flow

**Example - Good long function:**

```typescript
interface CheckoutResult {
    success: boolean;
    orderId?: string;
    total?: number;
    error?: string;
}

function processCheckout(order: Order, payment: Payment): CheckoutResult {
    // Validate order
    if (order.items.length === 0) {
        return { success: false, error: 'Empty order' };
    }

    // Calculate totals
    const subtotal = order.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
    const tax = subtotal * order.taxRate;
    const shipping = calculateShipping(order.address, order.items);
    const total = subtotal + tax + shipping;

    // Validate payment amount
    if (payment.amount < total) {
        return { success: false, error: 'Insufficient payment' };
    }

    // Process payment
    const paymentResult = paymentGateway.charge(payment, total);
    if (paymentResult.success === false) {
        return { success: false, error: paymentResult.error };
    }

    // Update inventory
    for (const item of order.items) {
        inventory.decrement(item.id, item.quantity);
    }

    // Create order record
    const orderId = database.createOrder(order, total, paymentResult.transactionId);

    // Send confirmation email
    email.sendOrderConfirmation(order.customer.email, orderId, total);

    // Return success
    return { success: true, orderId, total };
}
```

This 35-line function is clear and easy to follow. Breaking it into tiny functions like `validateOrder()`, `calculateTotals()`, `processPayment()`, etc., might actually make it harder to understand the complete checkout flow.

**The rule of thumb**: If you can easily understand what the function does by reading it top-to-bottom without scrolling too much, it's fine. Only split when complexity genuinely hurts comprehension.

---

## TypeScript Configuration

TypeScript's compiler enforces type safety and catches errors at compile time.

> **Note**: This configuration is intentionally strict. While you might be tempted to relax these settings for "prototyping", remember that prototypes almost always become production code. It's far easier to start strict than to add strictness later. See the section "A Word on Prototypes and Strict Settings" at the end of this document for more details.

### `tsconfig.json` (Strict Configuration)

```json
{
  "compilerOptions": {
    /* Language and Environment */
    "target": "ES2020",
    "lib": ["ES2020"],
    "module": "commonjs",
    "moduleResolution": "node",

    /* Type Checking - STRICT MODE */
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,

    /* Additional Checks for Safety */
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,
    "allowUnusedLabels": false,
    "allowUnreachableCode": false,
    "exactOptionalPropertyTypes": true,

    /* Interop Constraints */
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "isolatedModules": true,

    /* Emit */
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "removeComments": false,
    "noEmitOnError": true,

    /* Path Mapping */
    "baseUrl": ".",
    "paths": {
      "~/*": ["./src/*"]
    },

    /* Performance */
    "skipLibCheck": true,
    "incremental": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.spec.ts", "**/*.test.ts"]
}
```

### Key Strict Options Explained

- **`noUncheckedIndexedAccess`**: When accessing an array or object with an index, TypeScript will return `T | undefined` instead of just `T`. This forces you to handle cases where the value might not exist.

  ```typescript
  const arr = [1, 2, 3];
  const value = arr[5]; // Type: number | undefined (not just number)
  ```

- **`noImplicitReturns`**: All code paths in a function must explicitly return a value.

- **`noPropertyAccessFromIndexSignature`**: Prevents accessing properties with dot notation on objects with index signatures. Forces bracket notation for dynamic keys.

- **`exactOptionalPropertyTypes`**: Makes optional properties more precise. `{ prop?: string }` only accepts `string` or absence of property, not `undefined`.

### Installation

```bash
npm install --save-dev typescript @types/node
```

---

## Zod (Runtime Validation)

Zod provides runtime data validation and type-safe schema declaration using TypeScript. While TypeScript catches type errors at compile time, Zod validates data at runtime - especially valuable at system boundaries like API requests, configuration files, and external data sources.

> **Why Zod?** TypeScript's type system is erased at runtime, so it can't validate data that arrives from users, APIs, or config files. Zod bridges this gap by providing runtime validation with full TypeScript inference, giving you both compile-time and runtime type safety.

### The Progression: Types → Validation → Zod

Let's see how Zod improves upon basic TypeScript types and interfaces:

**Level 1 - Basic TypeScript Types (Compile-time only):**

```typescript
interface User {
    name: string;
    age: number;
    email: string;
}

function createUser(data: unknown): User {
    // No runtime validation - TypeScript can't help here!
    return data as User; // Dangerous cast
}

// This compiles but crashes at runtime:
const user = createUser({ name: 123, age: 'not a number', email: 'invalid' });
```

**Level 2 - Manual Validation (Verbose and error-prone):**

```typescript
function createUser(data: unknown): User {
    if (typeof data !== 'object' || data === null) {
        throw new Error('Invalid data');
    }

    const obj = data as Record<string, unknown>;

    if (typeof obj.name !== 'string') {
        throw new Error('Name must be a string');
    }
    if (typeof obj.age !== 'number') {
        throw new Error('Age must be a number');
    }
    if (typeof obj.email !== 'string') {
        throw new Error('Email must be a string');
    }

    return obj as User; // Still using unsafe cast
}
```

**Level 3 - Zod (Schema + Runtime Validation + Type Inference):**

```typescript
import { z } from 'zod';

// Define schema with validation
const UserSchema = z.object({
    name: z.string().min(1).max(100),
    age: z.number().int().min(0).max(150),
    email: z.string().email(),
});

// TypeScript type automatically inferred from schema
type User = z.infer<typeof UserSchema>;

function createUser(data: unknown): User {
    // Validates and returns typed data, or throws detailed error
    return UserSchema.parse(data);
}

// Now this throws a clear validation error:
try {
    const user = createUser({ name: 123, age: 'not a number', email: 'invalid' });
} catch (error) {
    if (error instanceof z.ZodError) {
        console.log(error.errors);
        // Shows exactly what's wrong:
        // - name: Expected string, received number
        // - age: Expected number, received string
        // - email: Invalid email
    }
}
```

### Real-World Use Cases

#### 1. API Request/Response Validation

```typescript
import { z } from 'zod';

// Request schema
const CreateUserRequestSchema = z.object({
    username: z.string().min(3).max(30).regex(/^[a-zA-Z0-9_]+$/),
    email: z.string().email(),
    age: z.number().int().min(13).max(120),
    website: z.string().url().optional(),
    role: z.enum(['user', 'admin', 'moderator']).default('user'),
});

type CreateUserRequest = z.infer<typeof CreateUserRequestSchema>;

// Response schema
const UserResponseSchema = z.object({
    id: z.number(),
    username: z.string(),
    email: z.string().email(),
    createdAt: z.date(),
    role: z.string(),
});

type UserResponse = z.infer<typeof UserResponseSchema>;

// Express route example
app.post('/users', (req, res) => {
    try {
        // Validate request body
        const userData = CreateUserRequestSchema.parse(req.body);

        // userData is now properly typed and validated
        const user = database.createUser(userData);

        // Validate response before sending
        const response = UserResponseSchema.parse(user);
        res.json(response);
    } catch (error) {
        if (error instanceof z.ZodError) {
            // Return 400 with validation errors
            res.status(400).json({ errors: error.errors });
            return;
        }
        throw error;
    }
});
```

#### 2. Configuration Files with Environment Variables

```typescript
import { z } from 'zod';

const DatabaseConfigSchema = z.object({
    host: z.string().default('localhost'),
    port: z.number().int().min(1).max(65535).default(5432),
    username: z.string(),
    password: z.string().min(8),
    database: z.string(),
});

type DatabaseConfig = z.infer<typeof DatabaseConfigSchema>;

const AppConfigSchema = z.object({
    appName: z.string().default('My Application'),
    debug: z.boolean().default(false),
    database: DatabaseConfigSchema,
    apiKey: z.string().min(32),
    logLevel: z.enum(['DEBUG', 'INFO', 'WARNING', 'ERROR']).default('INFO'),
    maxConnections: z.number().int().min(1).max(1000).default(100),
});

type AppConfig = z.infer<typeof AppConfigSchema>;

// Load and validate configuration
function loadConfig(): AppConfig {
    const rawConfig = {
        database: {
            host: process.env.DB_HOST,
            port: process.env.DB_PORT ? parseInt(process.env.DB_PORT) : undefined,
            username: process.env.DB_USERNAME,
            password: process.env.DB_PASSWORD,
            database: process.env.DB_DATABASE,
        },
        apiKey: process.env.API_KEY,
        debug: process.env.DEBUG === 'true',
    };

    // Validates and applies defaults
    return AppConfigSchema.parse(rawConfig);
}

// Usage - throws clear error if config is invalid
const config = loadConfig();
```

#### 3. Data Transformation and Coercion

```typescript
import { z } from 'zod';

// Zod can coerce types from strings (useful for URL params, env vars)
const EventSchema = z.object({
    name: z.string(),
    timestamp: z.coerce.date(), // Coerces string to Date
    attendeeCount: z.coerce.number().int(), // Coerces string to number
    isPublic: z.boolean(),
});

type Event = z.infer<typeof EventSchema>;

// Parse URL query parameters
const event = EventSchema.parse({
    name: 'TypeScript Meetup',
    timestamp: '2024-01-15T19:00:00Z', // String → Date
    attendeeCount: '42', // String → number
    isPublic: true,
});

console.log(event.timestamp instanceof Date); // true
console.log(typeof event.attendeeCount); // number
```

#### 4. Complex Validation with Refinements

```typescript
import { z } from 'zod';

const PasswordResetSchema = z
    .object({
        password: z
            .string()
            .min(12, 'Password must be at least 12 characters')
            .regex(/[A-Z]/, 'Password must contain at least one uppercase letter')
            .regex(/[a-z]/, 'Password must contain at least one lowercase letter')
            .regex(/[0-9]/, 'Password must contain at least one digit')
            .regex(/[!@#$%^&*]/, 'Password must contain at least one special character'),
        passwordConfirm: z.string(),
    })
    .refine((data) => data.password === data.passwordConfirm, {
        message: 'Passwords do not match',
        path: ['passwordConfirm'], // Error will point to passwordConfirm field
    });

type PasswordReset = z.infer<typeof PasswordResetSchema>;

// Usage
try {
    const reset = PasswordResetSchema.parse({
        password: 'weak',
        passwordConfirm: 'weak',
    });
} catch (error) {
    if (error instanceof z.ZodError) {
        console.log(error.errors); // Shows all validation errors
    }
}
```

#### 5. Optional and Nullable Fields

```typescript
import { z } from 'zod';

const ProfileSchema = z.object({
    // Required field
    userId: z.number(),

    // Optional field (may be undefined)
    bio: z.string().optional(),

    // Nullable field (may be null)
    avatar: z.string().url().nullable(),

    // Optional AND nullable (may be undefined or null)
    phoneNumber: z.string().nullable().optional(),

    // With default value
    theme: z.enum(['light', 'dark']).default('light'),

    // With fallback value (catches validation errors too)
    notifications: z.boolean().catch(true),
});

type Profile = z.infer<typeof ProfileSchema>;
```

### Integration with TypeScript

Zod's biggest strength is its seamless TypeScript integration:

```typescript
import { z } from 'zod';

// Define schema once
const UserSchema = z.object({
    id: z.number(),
    name: z.string(),
    email: z.string().email(),
    role: z.enum(['user', 'admin']),
    metadata: z.record(z.string()), // Record<string, string>
    tags: z.array(z.string()), // string[]
    createdAt: z.date(),
});

// TypeScript type is automatically inferred!
type User = z.infer<typeof UserSchema>;
// Equivalent to:
// interface User {
//     id: number;
//     name: string;
//     email: string;
//     role: 'user' | 'admin';
//     metadata: Record<string, string>;
//     tags: string[];
//     createdAt: Date;
// }

// The schema also serves as runtime validator
function validateUser(data: unknown): User {
    return UserSchema.parse(data); // Returns User or throws
}
```

### When to Use Zod vs. TypeScript Types

**Use Zod when:**
- ✅ Validating external data (APIs, user input, config files)
- ✅ Parsing data from JSON, environment variables, or URL params
- ✅ You need runtime type checking and transformation
- ✅ Building APIs (REST, GraphQL, tRPC)
- ✅ Complex validation logic is required

**Use TypeScript types alone when:**
- ✅ Data is already validated and internal to your system
- ✅ Working with compile-time-only types (generics, mapped types)
- ✅ Maximum performance is critical (Zod has some overhead)

**Use both:**
```typescript
import { z } from 'zod';

// Zod schema for runtime validation at boundaries
const UserCreateRequestSchema = z.object({
    username: z.string().min(3),
    email: z.string().email(),
    age: z.number().int().min(13),
});

type UserCreateRequest = z.infer<typeof UserCreateRequestSchema>;

// Internal domain model (TypeScript only)
interface User {
    id: number;
    username: string;
    email: string;
    age: number;
    createdAt: Date;
    updatedAt: Date;
}

function createUser(request: unknown): User {
    // Validate at boundary with Zod
    const validatedRequest = UserCreateRequestSchema.parse(request);

    // Work with validated data internally
    const userId = database.insert(validatedRequest);

    return {
        id: userId,
        username: validatedRequest.username,
        email: validatedRequest.email,
        age: validatedRequest.age,
        createdAt: new Date(),
        updatedAt: new Date(),
    };
}
```

### Zod Best Practices

1. **Validate at boundaries**: Use Zod where data enters your system (API endpoints, config loading, file parsing)

2. **Use specific validators**: Take advantage of Zod's built-in validators
   ```typescript
   z.string().email() // Email validation
   z.string().url() // URL validation
   z.string().uuid() // UUID validation
   z.string().regex(/pattern/) // Custom regex
   z.number().int() // Integer validation
   z.number().positive() // Positive numbers
   z.date().min(new Date('2020-01-01')) // Date constraints
   ```

3. **Infer types from schemas**: Always use `z.infer<typeof Schema>` instead of defining types separately
   ```typescript
   // ✅ Good - single source of truth
   const UserSchema = z.object({ name: z.string() });
   type User = z.infer<typeof UserSchema>;

   // ❌ Bad - duplication, can get out of sync
   const UserSchema = z.object({ name: z.string() });
   interface User { name: string }
   ```

4. **Use enums for fixed choices**: Better than string unions
   ```typescript
   const StatusSchema = z.enum(['pending', 'approved', 'rejected']);
   type Status = z.infer<typeof StatusSchema>; // 'pending' | 'approved' | 'rejected'
   ```

5. **Schema composition**: Break large schemas into smaller, reusable components
   ```typescript
   const AddressSchema = z.object({
       street: z.string(),
       city: z.string(),
       zipCode: z.string().regex(/^\d{5}$/),
   });

   const UserSchema = z.object({
       name: z.string(),
       address: AddressSchema, // Nested validation
   });
   ```

6. **Use `safeParse` when errors are expected**: Returns result object instead of throwing
   ```typescript
   const result = UserSchema.safeParse(data);

   if (result.success === false) {
       // Handle validation errors
       console.log(result.error.errors);
       return;
   }

   // Use validated data
   const user = result.data;
   ```

### Common Pitfalls to Avoid

❌ **Don't use Zod for performance-critical internal operations**
```typescript
// Bad - unnecessary overhead in tight loops
for (let i = 0; i < 1_000_000; i++) {
    const point = PointSchema.parse({ x: i, y: i * 2 }); // Validates on every iteration
}
```

✅ **Validate once at boundaries, use native types internally**
```typescript
// Good - validate once, use fast native types
interface Point {
    x: number;
    y: number;
}

function processPoints(rawData: unknown[]): Point[] {
    // Validate once with Zod
    const validatedData = z.array(PointSchema).parse(rawData);

    // Work with native types internally
    return validatedData.map((point) => ({
        x: point.x * 2,
        y: point.y * 2,
    }));
}
```

❌ **Don't use Zod for TypeScript-only type transformations**
```typescript
// Bad - Zod can't help with mapped types
const Schema = z.object({ /* ... */ }); // Can't represent Partial<T>, Pick<T>, etc.
```

✅ **Use TypeScript utilities for compile-time transformations**
```typescript
// Good - use TypeScript for type-level operations
interface User {
    id: number;
    name: string;
    email: string;
}

type PartialUser = Partial<User>; // Compile-time only
type UserUpdate = Omit<User, 'id'>; // Compile-time only
```

### Installation

```bash
npm install zod
```

### Integration with Popular Frameworks

**tRPC (End-to-end typesafe APIs):**
```typescript
import { z } from 'zod';
import { initTRPC } from '@trpc/server';

const t = initTRPC.create();

export const appRouter = t.router({
    createUser: t.procedure
        .input(z.object({ name: z.string(), email: z.string().email() }))
        .mutation(({ input }) => {
            // input is fully typed and validated
            return database.createUser(input);
        }),
});
```

**React Hook Form:**
```typescript
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';
import { useForm } from 'react-hook-form';

const FormSchema = z.object({
    username: z.string().min(3),
    email: z.string().email(),
});

type FormData = z.infer<typeof FormSchema>;

function MyForm(): JSX.Element {
    const { register, handleSubmit } = useForm<FormData>({
        resolver: zodResolver(FormSchema),
    });

    return <form onSubmit={handleSubmit((data) => console.log(data))}>{/* ... */}</form>;
}
```

### Further Reading

- [Zod Documentation](https://zod.dev/)
- [Zod GitHub Repository](https://github.com/colinhacks/zod)
- [tRPC with Zod](https://trpc.io/docs/server/validators#zod)

---

## Error Handling with Discriminated Unions

TypeScript's discriminated unions provide a powerful, type-safe way to handle errors without external libraries. By making success and failure states explicit in function return types, you get compile-time enforcement of error handling with zero dependencies.

> **Why discriminated unions?** Exceptions are invisible in function signatures and easy to forget. Discriminated unions make errors explicit and enforced by TypeScript's type system. This leads to more robust code where error handling is verified at compile time, using only native TypeScript features.

### The Problem with Exceptions

**Traditional exception-based code:**

```typescript
function divide(a: number, b: number): number {
    // Can throw Error, but signature doesn't show it
    if (b === 0) {
        throw new Error('Division by zero');
    }
    return a / b;
}

function getUser(userId: number): User {
    // Can throw Error, but not visible in signature
    const user = database.get(userId);
    if (user === undefined) {
        throw new Error(`User ${userId} not found`);
    }
    return user;
}

// Easy to forget error handling - no compile-time warning
const result = divide(10, 0); // Crashes at runtime!
const user = getUser(999); // Crashes at runtime!
```

**Issues with exceptions:**

1. **Invisible in signatures**: No way to know which functions can fail
2. **Easy to forget**: Nothing forces you to handle errors
3. **Break composition**: Can't easily chain operations that might fail
4. **Unclear control flow**: Exceptions can bubble up from anywhere

### Integration with TypeScript's Strict Mode

The strict TypeScript configuration (already in our tsconfig.json) enables full type narrowing benefits:

```json
{
    "compilerOptions": {
        "strict": true,
        "strictNullChecks": true,
        "noUncheckedIndexedAccess": true
    }
}
```

Benefits with strict mode enabled:

- **Complete type inference** for discriminated unions
- **Enforcement of error handling** - TypeScript errors if you don't handle all cases
- **Exhaustiveness checking** with if/else and switch statements
- **No runtime type errors** from unhandled branches
- **Native language features** - no external dependencies

### Reusable helper library

```typescript
/**
 * Result type for handling operations that can fail without throwing exceptions.
 * Inspired by Rust's Result<T, E> type.
 *
 * Usage:
 *   const result = ok(42);
 *   if (result.isOk) {
 *     console.log(result.value); // TypeScript knows this is safe
 *   } else {
 *     console.error(result.error);
 *   }
 *
 *   // Or use pattern matching
 *   match(result, {
 *     ok: (value) => console.log(value),
 *     err: (error) => console.error(error)
 *   });
 */

export type OkType<T> = {
    readonly isOk: true;
    readonly value: T;
};

export type ErrType<E> = {
    readonly isOk: false;
    readonly error: E;
};

export type Result<T, E> = OkType<T> | ErrType<E>;

/**
 * Async Result type - a Promise that always resolves to a Result (never rejects).
 * All errors are captured in the Result's Err variant.
 */
export type AsyncResult<T, E> = Promise<Result<T, E>>;

/**
 * Create a successful Result
 */
export function ok<T>(value: T): Result<T, never> {
    return { isOk: true, value };
}

/**
 * Create a failed Result
 */
export function err<E>(error: E): Result<never, E> {
    return { isOk: false, error };
}

/**
 * Transform the value inside a Result if it's Ok, otherwise pass through the error
 */
export function map<T, U, E>(result: Result<T, E>, fn: (value: T) => U): Result<U, E> {
    return result.isOk ? ok(fn(result.value)) : result;
}

/**
 * Transform the error inside a Result if it's Err, otherwise pass through the value
 */
export function mapErr<T, E, F>(result: Result<T, E>, fn: (error: E) => F): Result<T, F> {
    return result.isOk ? result : err(fn(result.error));
}

/**
 * Transform the value inside a Result if it's Ok, where the transform function also returns a Result.
 * Also known as flatMap or bind.
 */
export function andThen<T, U, E>(
    result: Result<T, E>,
    fn: (value: T) => Result<U, E>,
): Result<U, E> {
    return result.isOk ? fn(result.value) : result;
}

/**
 * Async version of andThen for chaining async operations
 */
export async function andThenAsync<T, U, E>(
    result: Result<T, E>,
    fn: (value: T) => Promise<Result<U, E>>,
): Promise<Result<U, E>> {
    return result.isOk ? await fn(result.value) : result;
}

/**
 * Pattern match on a Result, executing one of two functions based on success/failure
 */
export function match<T, E, U>(
    result: Result<T, E>,
    patterns: {
        ok: (value: T) => U;
        err: (error: E) => U;
    },
): U {
    return result.isOk ? patterns.ok(result.value) : patterns.err(result.error);
}

/**
 * Unwrap a Result, returning the value if Ok or throwing if Err.
 * Throws the raw error if it is an instance of Error, otherwise wraps it.
 */
export function unwrap<T, E>(result: Result<T, E>): T {
    if (result.isOk) return result.value;
    const e = result.error;
    if (e instanceof Error) throw e;
    throw new Error(`Unwrapped Err: ${JSON.stringify(e)}`);
}

/**
 * Unwrap a Result, returning the value if Ok or the provided default if Err
 */
export function unwrapOr<T, E>(result: Result<T, E>, defaultValue: T): T {
    return result.isOk ? result.value : defaultValue;
}

/**
 * Unwrap a Result, returning the value if Ok or computing a default from the error if Err
 */
export function unwrapOrElse<T, E>(result: Result<T, E>, fn: (error: E) => T): T {
    return result.isOk ? result.value : fn(result.error);
}

/**
 * Wrap a function that might throw into a Result
 */
export function tryCatch<T, E = Error>(fn: () => T): Result<T, E> {
    try {
        return ok(fn());
    } catch (error) {
        return err(error as E);
    }
}

/**
 * Async version of tryCatch
 */
export async function tryCatchAsync<T, E = Error>(fn: () => Promise<T>): Promise<Result<T, E>> {
    try {
        const value = await fn();
        return ok(value);
    } catch (error) {
        return err(error as E);
    }
}

/**
 * Combine multiple Results into a single Result containing an array of values.
 * If any Result is Err, returns the first Err encountered.
 */
export function collect<T, E>(results: Array<Result<T, E>>): Result<T[], E> {
    const values: T[] = [];
    for (const result of results) {
        if (!result.isOk) return err(result.error);
        values.push(result.value);
    }
    return ok(values);
}

/**
 * Convert a nullable value to a Result.
 */
export function fromNullable<T, E>(value: T | null | undefined, error: E): Result<T, E> {
    return value != null ? ok(value) : err(error);
}

/**
 * Convert a Result into a Promise, resolving if Ok, rejecting if Err.
 */
export function toPromise<T, E>(result: Result<T, E>): Promise<T> {
    return result.isOk ? Promise.resolve(result.value) : Promise.reject(result.error);
}

/**
 * Execute a side effect if the Result is Ok, returning the original Result.
 */
export function inspect<T, E>(result: Result<T, E>, fn: (value: T) => void): Result<T, E> {
    if (result.isOk) fn(result.value);
    return result;
}

/**
 * Execute a side effect if the Result is Err, returning the original Result.
 */
export function inspectErr<T, E>(result: Result<T, E>, fn: (error: E) => void): Result<T, E> {
    if (!result.isOk) fn(result.error);
    return result;
}

/**
 * Aliases for convenience
 */
export const flatMap = andThen;
export const mapError = mapErr;
```

### When to Use Result Types

**Use Result types (discriminated unions) when:**

- ✅ Errors are expected as part of normal operation (validation, parsing, user input)
- ✅ You want explicit error handling enforced by the type system
- ✅ Building APIs where failures are common and should be handled gracefully
- ✅ Need to chain operations that can fail
- ✅ Want to eliminate "forgot to handle error" bugs
- ✅ Working with financial, medical, or other critical domains

**Use traditional exceptions when:**

- ✅ Dealing with truly exceptional conditions (out of memory, hardware failure)
- ✅ Errors that should propagate up through many layers
- ✅ Working with third-party libraries that use exceptions
- ✅ Simple scripts where explicit error handling is overkill

**Hybrid approach** (recommended):

```typescript
import { ok, err, type Result } from './result';

// Business logic: Use Result types for expected failures
function validateTransfer(
    fromAccount: number,
    toAccount: number,
    amount: number,
): Result<void, string> {
    if (amount <= 0) {
        return err('Amount must be positive');
    }
    if (fromAccount === toAccount) {
        return err('Cannot transfer to same account');
    }
    return ok(undefined);
}

// Infrastructure: Wrap exceptions in Results with tryCatch
function executeTransfer(
    fromAccount: number,
    toAccount: number,
    amount: number,
): Result<void, Error> {
    return tryCatch(() => {
        database.execute('UPDATE accounts SET balance = balance - ? WHERE id = ?', [
            amount,
            fromAccount,
        ]);
        database.execute('UPDATE accounts SET balance = balance + ? WHERE id = ?', [
            amount,
            toAccount,
        ]);
    });
}

// Compose them together
function transferMoney(
    fromAccount: number,
    toAccount: number,
    amount: number,
): Result<string, string | Error> {
    const validationResult = validateTransfer(fromAccount, toAccount, amount);
    if (!validationResult.isOk) {
        return validationResult;
    }

    const executeResult = executeTransfer(fromAccount, toAccount, amount);
    if (!executeResult.isOk) {
        return executeResult;
    }

    return ok(`Transferred $${amount} successfully`);
}
```

#### Practical Examples Using the Library

**Example 1: Basic validation with `ok()` and `err()`**

```typescript
import { ok, err, type Result } from './result';

function validateEmail(email: string): Result<string, string> {
    if (!email.includes('@')) {
        return err('Email must contain @');
    }
    if (email.length < 5) {
        return err('Email too short');
    }
    return ok(email.toLowerCase());
}

const result = validateEmail('user@example.com');
if (result.isOk) {
    console.log('Valid email:', result.value);
} else {
    console.error('Invalid:', result.error);
}
```

**Example 2: Transforming values with `map()`**

```typescript
import { ok, map } from './result';

function parseAge(input: string): Result<number, string> {
    const age = parseInt(input);
    return isNaN(age) ? err('Not a number') : ok(age);
}

// Transform the age to birth year
const result = parseAge('25');
const birthYear = map(result, (age) => new Date().getFullYear() - age);

console.log(birthYear); // { isOk: true, value: 1999 }
```

**Example 3: Chaining operations with `andThen()`**

```typescript
import { ok, err, andThen, type Result } from './result';

function findUser(id: number): Result<User, string> {
    const user = database.users.find((u) => u.id === id);
    return user ? ok(user) : err('User not found');
}

function getUserPermissions(user: User): Result<Permissions, string> {
    const perms = database.permissions.get(user.role);
    return perms ? ok(perms) : err('Permissions not found');
}

// Chain the operations - stops at first error
const userResult = findUser(123);
const permissionsResult = andThen(userResult, getUserPermissions);

if (permissionsResult.isOk) {
    console.log('Permissions:', permissionsResult.value);
} else {
    console.error('Error:', permissionsResult.error);
}
```

**Example 4: Pattern matching with `match()`**

```typescript
import { ok, err, match } from './result';

function divide(a: number, b: number): Result<number, string> {
    return b === 0 ? err('Division by zero') : ok(a / b);
}

const result = divide(10, 2);

// Match handles both cases elegantly
const message = match(result, {
    ok: (value) => `Result is ${value}`,
    err: (error) => `Error: ${error}`,
});

console.log(message); // "Result is 5"
```

**Example 5: Wrapping exceptions with `tryCatch()`**

```typescript
import { tryCatch, unwrapOr } from './result';

function parseJSON<T>(jsonString: string): Result<T, Error> {
    return tryCatch(() => JSON.parse(jsonString));
}

const result = parseJSON<User>('{"name": "Alice", "age": 30}');
const user = unwrapOr(result, { name: 'Guest', age: 0 });

console.log(user.name); // "Alice" or "Guest" if parsing failed
```

**Example 6: Async operations with `tryCatchAsync()`**

```typescript
import { tryCatchAsync, andThenAsync, type AsyncResult } from './result';

async function fetchUser(id: number): AsyncResult<User, Error> {
    return tryCatchAsync(async () => {
        const response = await fetch(`/api/users/${id}`);
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}`);
        }
        return await response.json();
    });
}

async function fetchUserPosts(user: User): AsyncResult<Post[], Error> {
    return tryCatchAsync(async () => {
        const response = await fetch(`/api/users/${user.id}/posts`);
        return await response.json();
    });
}

// Chain async operations
const userResult = await fetchUser(123);
const postsResult = await andThenAsync(userResult, fetchUserPosts);

if (postsResult.isOk) {
    console.log('Posts:', postsResult.value);
}
```

**Example 7: Collecting multiple Results with `collect()`**

```typescript
import { ok, err, collect } from './result';

function validateField(name: string, value: string): Result<string, string> {
    return value.length > 0 ? ok(value) : err(`${name} is required`);
}

// Validate multiple fields
const results = [
    validateField('username', 'alice'),
    validateField('email', 'alice@example.com'),
    validateField('password', 'secret123'),
];

const allValid = collect(results);

if (allValid.isOk) {
    const [username, email, password] = allValid.value;
    console.log('All fields valid:', { username, email, password });
} else {
    console.error('First error:', allValid.error);
}
```

**Example 8: Handling nullable values with `fromNullable()`**

```typescript
import { fromNullable, map, unwrapOr } from './result';

interface Config {
    apiKey?: string;
    timeout?: number;
}

const config: Config = { timeout: 5000 };

// Convert nullable to Result
const apiKeyResult = fromNullable(config.apiKey, 'API key not configured');
const apiKey = unwrapOr(apiKeyResult, 'default-key');

console.log(apiKey); // "default-key"
```

**Example 9: Debugging with `inspect()` and `inspectErr()`**

```typescript
import { ok, err, inspect, inspectErr, map } from './result';

function processData(input: string): Result<number, string> {
    return input.length > 0 ? ok(input.length) : err('Empty input');
}

const result = processData('hello');

// Inspect without consuming the result
const doubled = result
    |> inspect((value) => console.log('Got value:', value))
    |> map((value) => value * 2)
    |> inspectErr((error) => console.error('Error occurred:', error));
```

### No Installation Required

This pattern uses only native TypeScript features - no dependencies needed! Simply define the types in your codebase or create a small utilities module.

### Why This Makes TypeScript Better

TypeScript is an excellent language, but exceptions and `undefined`/`null` can be weaknesses for type safety and composition. Discriminated unions bring the rigor of languages like Rust and Haskell to TypeScript while maintaining readability and using only native features.

The combination of:

- **Strict TypeScript** → catches type errors at compile time
- **Zod** → validates external data at runtime
- **Discriminated Unions** → makes error handling explicit and type-safe

...creates a development experience that rivals statically-typed functional languages while keeping TypeScript's expressiveness and productivity. For teams willing to embrace these patterns, TypeScript becomes a genuinely world-class language for building reliable systems.

### Further Reading

- [TypeScript Handbook: Discriminated Unions](https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html#discriminating-unions) - Official TypeScript documentation
- [Rust's Result Documentation](https://doc.rust-lang.org/std/result/) - The inspiration for this pattern
- [Railway Oriented Programming](https://fsharpforfunandprofit.com/rop/) - The conceptual foundation (F# article, concepts apply to all languages)
- [Type-Safe Error Handling in TypeScript](https://www.youtube.com/watch?v=u6uCNF0JKw0) - Video explanation

---

## Git Hooks with Husky

Husky runs linting and formatting checks before commits to ensure code quality.

### Installation

```bash
npm install --save-dev husky lint-staged
npx husky init
```

### `.husky/pre-commit`

```bash
npx lint-staged
```

### `lint-staged` configuration in `package.json`

```json
{
  "lint-staged": {
    "*.{ts,tsx}": [
      "prettier --write",
      "eslint --fix"
    ],
    "*.{json,yaml,yml,md}": [
      "prettier --write"
    ]
  }
}
```

---

## Git Attributes

Git attributes ensure consistent file handling across all environments and improve security.

### `.gitattributes`

```gitattributes
# Set default behavior to automatically normalize line endings to LF
* text=auto eol=lf

# Explicitly declare text files you want to always be normalized and converted to LF
*.js text eol=lf
*.ts text eol=lf
*.jsx text eol=lf
*.tsx text eol=lf
*.json text eol=lf
*.css text eol=lf
*.scss text eol=lf
*.html text eol=lf
*.md text eol=lf
*.yml text eol=lf
*.yaml text eol=lf
*.xml text eol=lf
*.txt text eol=lf
*.sh text eol=lf

# Declare files that will always have CRLF line endings on checkout
*.bat text eol=crlf
*.cmd text eol=crlf
*.ps1 text eol=crlf

# Denote binary files
*.png binary
*.jpg binary
*.jpeg binary
*.gif binary
*.ico binary
*.svg binary
*.pdf binary
*.zip binary
*.gz binary
*.tar binary
*.woff binary
*.woff2 binary
*.ttf binary
*.eot binary
*.otf binary

# Security: Mark sensitive files as binary to prevent Git from attempting text operations
*.key binary
*.pem binary
*.p12 binary
*.pfx binary
*.cer binary
*.crt binary
*.der binary

# Security: Use Git LFS for large files (uncomment if using Git LFS)
# *.zip filter=lfs diff=lfs merge=lfs -text
# *.pdf filter=lfs diff=lfs merge=lfs -text
# *.png filter=lfs diff=lfs merge=lfs -text

# Archive files generated by export-ignore
.gitattributes export-ignore
.gitignore export-ignore
.editorconfig export-ignore
.prettierrc.json export-ignore
eslint.config.js export-ignore
tsconfig.json export-ignore
*.md export-ignore
.vscode/ export-ignore
.husky/ export-ignore
tests/ export-ignore
```

### Why `.gitattributes` is Important for Security

1. **Consistent Line Endings**: Prevents CRLF/LF issues that can cause bugs or security vulnerabilities
2. **Binary File Protection**: Marks sensitive files (keys, certificates) as binary to prevent text-based operations that could corrupt them
3. **Prevents Accidental Exposure**: Using `export-ignore` prevents development files from being included in release archives
4. **Git LFS Support**: Allows tracking large files securely without bloating the repository

---

## Package.json Scripts

Essential scripts for development, building, testing, and quality checks.

### `package.json` (scripts section)

```json
{
  "scripts": {
    "start": "node dist/index.js",
    "dev": "nodemon -r dotenv/config --exec ts-node src/index.ts",
    "build": "tsc",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "format": "prettier --check .",
    "format:write": "prettier --write .",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "type-check": "tsc --noEmit",
    "prepare": "husky"
  }
}
```

---

## Installation Steps

Follow these steps to set up a new TypeScript project from scratch:

### 1. Initialize Project

```bash
mkdir my-project
cd my-project
npm init -y
git init
```

### 2. Install Core Dependencies

```bash
# TypeScript
npm install --save-dev typescript @types/node ts-node nodemon

# Linting
npm install --save-dev eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin \
  eslint-plugin-import eslint-plugin-unused-imports eslint-config-prettier

# Formatting
npm install --save-dev prettier

# Git Hooks
npm install --save-dev husky lint-staged
```

### 3. Create Configuration Files

Create the following files in your project root:
- `.editorconfig`
- `.prettierrc.json`
- `eslint.config.js`
- `tsconfig.json`
- `.gitattributes`
- `.vscode/settings.json`
- `.vscode/extensions.json`

Copy the configurations from the sections above.

### 4. Initialize Husky

```bash
npx husky init
```

Edit `.husky/pre-commit` and add:
```bash
npx lint-staged
```

### 5. Add Scripts to package.json

Copy the scripts section from the [Package.json Scripts](#packagejson-scripts) section above.

### 6. Create Project Structure

```bash
mkdir src
touch src/index.ts
mkdir dist
```

### 7. Add .gitignore

You can get a template for your project type from GitHub!

Here is an example for OS generated files:
```gitignore
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db
```

### 8. Test Your Setup

```bash
# Check TypeScript compilation
npm run build

# Run linting
npm run lint

# Run formatting check
npm run format

# Start development server
npm run dev
```

### 9. Make Your First Commit

```bash
git add .
git commit -m "Initial project setup with TypeScript, ESLint, Prettier, and Husky"
```

### 10. Keeping Dependencies Updated

Regularly update your dependencies to get bug fixes, security patches, and new features:

```bash
# Check for outdated packages
npm outdated

# Update all dependencies to latest versions (within semver constraints)
npm update

# For major version updates, use npm-check-updates
npx npm-check-updates -u
npm install
```

**Best practices:**
- Update dependencies at least monthly
- Review changelogs before updating major versions
- Run your full test suite after updates
- Use `npm audit` to check for security vulnerabilities
- Consider using Dependabot or Renovate for automated dependency updates

---

## Additional Recommendations

### Testing Setup (Jest)

```bash
npm install --save-dev jest @types/jest ts-jest
npx ts-jest config:init
```

### Environment Variables

```bash
npm install dotenv
```

Create `.env` file and add it to `.gitignore`.

### CI/CD Integration

Add a GitHub Actions workflow (`.github/workflows/ci.yml`):

```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run type-check
      - run: npm run lint
      - run: npm run format
      - run: npm test
```

---

## Summary

This setup provides:

- **Consistent Code Style**: EditorConfig + Prettier
- **Code Quality**: ESLint with strict TypeScript rules
- **Type Safety**: Strict TypeScript compiler options catching errors at compile time
- **Runtime Validation**: Zod for validating data at system boundaries
- **Automated Quality Checks**: Husky pre-commit hooks
- **Cross-platform Consistency**: Git attributes for line endings and file handling
- **Security**: Proper handling of binary files and sensitive data
- **Developer Experience**: VS Code integration with recommended extensions

By following this guide, your TypeScript projects will have a solid foundation for maintainability, consistency, and quality.

---

## Code Smart: Readability Principles

Beyond linters and type checkers, good code requires human judgment. These principles will make your code more maintainable and easier to understand.

### Be a Never-Nester

Deep nesting makes code exponentially harder to read and reason about. Each level of nesting adds cognitive load.

**Bad - Deep Nesting:**

```typescript
function processOrder(order: Order): string {
    if (order !== null && order !== undefined) {
        if (order.items.length > 0) {
            if (order.customer !== null && order.customer !== undefined) {
                if (order.customer.isActive) {
                    if (order.total > 0) {
                        return `Processing order for ${order.customer.name}`;
                    } else {
                        return 'Order total must be positive';
                    }
                } else {
                    return 'Customer is inactive';
                }
            } else {
                return 'Customer is required';
            }
        } else {
            return 'Order has no items';
        }
    } else {
        return 'Order is required';
    }
}
```

**Good - Early Returns (Never-Nester):**

```typescript
function processOrder(order: Order): string {
    if (order === null || order === undefined) {
        return 'Order is required';
    }

    if (order.items.length === 0) {
        return 'Order has no items';
    }

    if (order.customer === null || order.customer === undefined) {
        return 'Customer is required';
    }

    if (!order.customer.isActive) {
        return 'Customer is inactive';
    }

    if (order.total <= 0) {
        return 'Order total must be positive';
    }

    return `Processing order for ${order.customer.name}`;
}
```

**Key techniques for avoiding nesting:**

1. **Early returns**: Guard clauses at the top of functions
2. **Extract functions**: Break complex conditions into well-named functions
3. **Invert conditions**: Return early on error cases, continue with happy path
4. **Use optional chaining and nullish coalescing**: Flatten nested property access

```typescript
// Instead of nested checks
if (user !== null && user !== undefined) {
    if (user.settings !== null && user.settings !== undefined) {
        const theme = user.settings.theme;
    }
}

// Use optional chaining
const theme = user?.settings?.theme ?? 'default';
```

### Don't Use Abbreviations - Characters Are (Almost) Free

Abbreviations save a few keystrokes but cost hundreds of hours in confusion over a project's lifetime. Modern editors have autocomplete - take advantage of it.

**Bad - Cryptic Abbreviations:**

```typescript
function calcUsrDisc(usr: Usr, amt: number): number {
    const lvl = usr.memLvl;
    const discPct = lvl === 'prem' ? 0.2 : 0.1;
    return amt * discPct;
}

const btn = document.querySelector('.btn');
const msg = 'Hello';
const idx = arr.findIndex(el => el.id === tgtId);
```

**Good - Clear Names:**

```typescript
function calculateUserDiscount(user: User, amount: number): number {
    const membershipLevel = user.membershipLevel;
    const discountPercentage = membershipLevel === 'premium' ? 0.2 : 0.1;
    return amount * discountPercentage;
}

const button = document.querySelector('.button');
const message = 'Hello';
const index = array.findIndex(element => element.id === targetId);
```

**Common abbreviations to avoid:**

- `usr` → `user`
- `btn` → `button`
- `msg` → `message`
- `idx` → `index`
- `arr` → `array`
- `el` → `element`
- `err` → `error`
- `ctx` → `context`
- `cfg` → `config`
- `tmp` → `temporary`
- `prev` → `previous`
- `curr` → `current`

**Acceptable abbreviations** (widely understood in context):

- `id` - universally understood identifier
- `url` - standard acronym
- `api` - standard acronym
- `html`, `css`, `json` - standard formats
- `max`, `min` - mathematical conventions
- `i`, `j`, `k` - loop counters in simple loops (but prefer descriptive names in complex loops)

**Example of when to use full names even in loops:**

```typescript
// ❌ Bad - unclear what we're iterating over
for (let i = 0; i < usrs.length; i++) {
    for (let j = 0; j < usrs[i].ords.length; j++) {
        // What is i? What is j?
    }
}

// ✅ Good - crystal clear
for (const user of users) {
    for (const order of user.orders) {
        // Immediately clear what we're working with
    }
}
```

### Why This Matters

1. **Onboarding**: New team members can understand code faster
2. **Code reviews**: Reviewers spend less time deciphering abbreviations
3. **Debugging**: Clear names make stack traces and error messages meaningful
4. **Future you**: You'll forget what `calcUsrDisc` means in 6 months
5. **Searchability**: Full words are easier to search for across a codebase

### The Cost of Characters vs. The Cost of Confusion

- Typing 10 extra characters: **2 seconds**
- Figuring out what `usrMgr` means: **30 seconds**
- Debugging because you confused `msgCnt` (message count) with `msgCnt` (message content): **30 minutes**

**Characters are almost free. Confusion is expensive.**

---

## A Word on "Prototypes" and Strict Settings

**Warning**: A common mistake is to start a project with relaxed settings for "prototyping" or "moving fast", planning to add strictness later.

### Why This Approach Fails

1. **Prototypes become production**: What starts as a quick prototype often becomes the production codebase when it works well enough. Nobody wants to stop and refactor working code.

2. **Retrofitting is painful**: Adding strict TypeScript checks to existing code can generate hundreds or thousands of errors. Teams often give up or use `// @ts-ignore` everywhere, defeating the purpose.

3. **Technical debt compounds**: Every day you write code without strict checks, you're creating more technical debt that becomes harder to fix.

4. **Team resistance**: Once a team gets used to loose checks, there's psychological resistance to "making the build harder" by enabling strict mode.

### The Better Approach

**Start strict from day one**, even for prototypes:

- It's easier to write strict code from the beginning than to fix it later
- You catch bugs immediately instead of discovering them in production
- The "slowdown" from strict checks is minimal compared to the time saved debugging
- Your prototype is already production-ready if it succeeds

### If You Must Relax Settings

If you truly need to relax some rules temporarily, do it strategically:

1. **Create a `tsconfig.prototype.json`** that extends the strict config but disables specific checks
2. **Set a deadline** (e.g., "before first production deploy") to migrate to full strict mode
3. **Keep Prettier and basic ESLint** - formatting and imports should always be enforced
4. **Never disable**: `noImplicitAny`, `strictNullChecks`, `noUnusedLocals` - these catch too many real bugs

### Rules You Might Consider Relaxing (with caution)

```typescript
// In tsconfig.json - consider keeping these enabled even for prototypes:
"exactOptionalPropertyTypes": false,  // Can cause friction with libraries
"noPropertyAccessFromIndexSignature": false,  // Adds verbosity

// In eslint.config.js - consider relaxing:
"@typescript-eslint/explicit-function-return-type": "off",  // Reduces boilerplate
"no-console": "warn",  // Allow console for debugging, but warn
```

**Remember**: Every rule you disable is a category of bugs you're allowing into your codebase. The pain of strict checks is temporary; the pain of production bugs is permanent.

---

## References

> "The wise man learns from everything and everyone, the ordinary man learns from his experience, and the fool knows everything better." - Socrates

- The [CodeAesthetic](https://www.youtube.com/@CodeAesthetic) YouTube channel
- ["Clean" Code, Horrible Performance](https://www.youtube.com/watch?v=tD5NrevFtbU) by [Molly Rocket](https://www.youtube.com/@MollyRocket)
