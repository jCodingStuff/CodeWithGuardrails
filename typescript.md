# TypeScript Project Setup Guide

This guide provides a comprehensive reference for setting up new TypeScript projects with best practices, consistent code quality, and automated tooling.

## Table of Contents

1. [EditorConfig](#editorconfig)
2. [VS Code Configuration](#vs-code-configuration)
3. [Prettier](#prettier)
4. [ESLint](#eslint)
5. [TypeScript Configuration](#typescript-configuration)
6. [Git Hooks with Husky](#git-hooks-with-husky)
7. [Git Attributes](#git-attributes)
8. [Package.json Scripts](#packagejson-scripts)
9. [Installation Steps](#installation-steps)

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
            'eqeqeq': ['error', 'always', { null: 'ignore' }],
            'no-var': 'error',
            'prefer-const': 'error',
            'prefer-template': 'error',
            'no-console': 'error',

            // TypeScript-specific rules
            '@typescript-eslint/no-unused-vars': 'off',
            '@typescript-eslint/naming-convention': [
                'error',
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

---

## TypeScript Configuration

TypeScript's compiler enforces type safety and catches errors at compile time.

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
- **Type Safety**: Strict TypeScript compiler options with advanced checks
- **Automated Quality Checks**: Husky pre-commit hooks
- **Cross-platform Consistency**: Git attributes for line endings and file handling
- **Security**: Proper handling of binary files and sensitive data
- **Developer Experience**: VS Code integration with recommended extensions

By following this guide, your TypeScript projects will have a solid foundation for maintainability, consistency, and quality.
