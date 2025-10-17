# Python Project Setup Guide

This guide provides a comprehensive reference for setting up new Python projects with best practices, consistent code quality, and automated tooling.

## Table of Contents

1. [EditorConfig](#editorconfig)
2. [VS Code Configuration](#vs-code-configuration)
3. [Ruff (Formatter and Linter)](#ruff-formatter-and-linter)
4. [mypy (Type Checking)](#mypy-type-checking)
5. [Pre-commit Hooks](#pre-commit-hooks)
6. [Git Attributes](#git-attributes)
7. [Project Configuration (pyproject.toml)](#project-configuration-pyprojecttoml)
8. [Installation Steps](#installation-steps)
9. [Code Smart: Readability Principles](#code-smart-readability-principles)
10. [A Word on "Prototypes" and Strict Settings](#a-word-on-prototypes-and-strict-settings)
11. [References](#references)

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

# YAML and JSON files use 2 spaces
[*.{yaml,yml,json}]
indent_size = 2

# Markdown files may have trailing spaces for line breaks
[*.md]
trim_trailing_whitespace = false
```

---

## VS Code Configuration

Configure VS Code to work seamlessly with your Python tools.

### `.vscode/settings.json`

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "charliermarsh.ruff",
  "editor.codeActionsOnSave": {
    "source.fixAll.ruff": "explicit",
    "source.organizeImports.ruff": "explicit"
  },
  "files.eol": "\n",
  "files.insertFinalNewline": true,
  "files.trimTrailingWhitespace": true,
  "[markdown]": {
    "files.trimTrailingWhitespace": false
  },
  "python.analysis.typeCheckingMode": "strict",
  "python.testing.pytestEnabled": true,
  "python.testing.unittestEnabled": false
}
```

### `.vscode/extensions.json`

```json
{
  "recommendations": [
    "charliermarsh.ruff",
    "ms-python.python",
    "ms-python.vscode-pylance",
    "editorconfig.editorconfig",
    "wayou.vscode-todo-highlight"
  ]
}
```

---

## Ruff (Formatter and Linter)

Ruff is an extremely fast Python linter and formatter written in Rust. It replaces multiple tools (Black, isort, Flake8, pylint, and more) with a single, fast tool.

> **Note**: This configuration is intentionally strict. While you might be tempted to relax these settings for "prototyping", remember that prototypes almost always become production code. It's far easier to start strict than to add strictness later. See the section "A Word on Prototypes and Strict Settings" at the end of this document for more details.

### Ruff Configuration in `pyproject.toml`

```toml
[tool.ruff]
# Set line length to 100 characters
line-length = 100
# Target Python 3.11+
target-version = "py311"

# Enable auto-fixing where possible
fix = true

# Exclude common directories
exclude = [
    ".git",
    ".venv",
    "venv",
    "__pycache__",
    "build",
    "dist",
    "*.egg-info",
]

[tool.ruff.lint]
# Enable strict rule sets
select = [
    "E",      # pycodestyle errors
    "W",      # pycodestyle warnings
    "F",      # pyflakes
    "I",      # isort
    "N",      # pep8-naming
    "UP",     # pyupgrade
    "B",      # flake8-bugbear
    "C4",     # flake8-comprehensions
    "DTZ",    # flake8-datetimez (timezone-aware datetimes)
    "T10",    # flake8-debugger
    "EM",     # flake8-errmsg (error message formatting)
    "ISC",    # flake8-implicit-str-concat
    "ICN",    # flake8-import-conventions
    "PIE",    # flake8-pie
    "PT",     # flake8-pytest-style
    "Q",      # flake8-quotes
    "RSE",    # flake8-raise
    "RET",    # flake8-return
    "SIM",    # flake8-simplify
    "TID",    # flake8-tidy-imports
    "PTH",    # flake8-use-pathlib
    "ERA",    # eradicate (commented-out code)
    "PL",     # pylint
    "PERF",   # perflint (performance)
    "RUF",    # ruff-specific rules
    "FURB",   # refurb (modernization)
]

# Specific rules to enforce
ignore = [
    "ISC001",  # Conflicts with formatter
    "COM812",  # Conflicts with formatter
]

# Allow unused variables when underscore-prefixed
dummy-variable-rgx = "^(_+|(_+[a-zA-Z0-9_]*[a-zA-Z0-9]+?))$"

[tool.ruff.lint.isort]
# Organize imports with specific groups
known-first-party = ["your_package_name"]
section-order = [
    "future",
    "standard-library",
    "third-party",
    "first-party",
    "local-folder",
]

[tool.ruff.format]
# Use double quotes for strings
quote-style = "double"
# Indent with spaces
indent-style = "space"
# Normalize string quotes
skip-magic-trailing-comma = false
# Respect existing line endings
line-ending = "lf"
```

### Key Strict Rules Explained

- **`B`** (flake8-bugbear): Catches common bugs and design problems
- **`RET`**: Enforces consistent return statements
- **`SIM`**: Suggests simpler alternatives to complex code
- **`PERF`**: Identifies performance anti-patterns
- **`PL`** (pylint): Comprehensive code quality checks
- **`DTZ`**: Forces timezone-aware datetime usage (prevents timezone bugs)
- **`PTH`**: Enforces pathlib over os.path (modern, safer file handling)
- **`ERA`**: Removes commented-out code (use version control instead)

**Note**: Function complexity metrics (max arguments, cyclomatic complexity, etc.) are intentionally not configured. These are better caught in code reviews where human judgment can determine if a long function improves or hurts readability.

### When Long Functions Are Actually Better

Sometimes a 100-line function that tells a complete story is more maintainable than splitting it into 10 smaller functions. Consider keeping a function long when:

- **The function tells a linear story**: Each step follows naturally from the previous one
- **Breaking it up would require passing many parameters**: Creating helper functions that need 5+ parameters is often worse than one longer function
- **The logic is cohesive**: All the code relates to a single, clear purpose
- **Breaking it up adds indirection**: If you'd need to constantly jump between functions to understand the flow

**Example - Good long function:**

```python
def process_checkout(order: Order, payment: Payment) -> CheckoutResult:
    """Process a complete checkout flow."""
    # Validate order
    if not order.items:
        return CheckoutResult(success=False, error="Empty order")

    # Calculate totals
    subtotal = sum(item.price * item.quantity for item in order.items)
    tax = subtotal * order.tax_rate
    shipping = calculate_shipping(order.address, order.items)
    total = subtotal + tax + shipping

    # Validate payment amount
    if payment.amount < total:
        return CheckoutResult(success=False, error="Insufficient payment")

    # Process payment
    payment_result = payment_gateway.charge(payment, total)
    if not payment_result.success:
        return CheckoutResult(success=False, error=payment_result.error)

    # Update inventory
    for item in order.items:
        inventory.decrement(item.id, item.quantity)

    # Create order record
    order_id = database.create_order(order, total, payment_result.transaction_id)

    # Send confirmation email
    email.send_order_confirmation(order.customer.email, order_id, total)

    # Return success
    return CheckoutResult(success=True, order_id=order_id, total=total)
```

This 30-line function is clear and easy to follow. Breaking it into tiny functions like `validate_order()`, `calculate_totals()`, `process_payment()`, etc., might actually make it harder to understand the complete checkout flow.

**The rule of thumb**: If you can easily understand what the function does by reading it top-to-bottom without scrolling too much, it's fine. Only split when complexity genuinely hurts comprehension.

### Installation

```bash
pip install ruff
```

---

## mypy (Type Checking)

mypy provides static type checking for Python, catching type-related bugs before runtime.

> **Note**: These settings are very strict and require comprehensive type annotations. This catches entire categories of bugs at development time. Resist the urge to disable strict mode when starting a new project - it's much harder to enable it later when you have a large codebase. See "A Word on Prototypes and Strict Settings" at the end of this document.

### mypy Configuration in `pyproject.toml`

```toml
[tool.mypy]
# Strict mode enables all optional checks
strict = true

# Python version
python_version = "3.11"

# Disallow dynamic typing
disallow_any_unimported = true
disallow_any_expr = false  # Too strict for most codebases
disallow_any_decorated = false  # Decorators often have complex types
disallow_any_explicit = false  # Allow explicit 'Any' when needed
disallow_any_generics = true
disallow_subclassing_any = true

# Disallow untyped definitions
disallow_untyped_calls = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
disallow_untyped_decorators = true

# None and Optional checks
no_implicit_optional = true
strict_optional = true

# Warnings
warn_redundant_casts = true
warn_unused_ignores = true
warn_no_return = true
warn_return_any = true
warn_unreachable = true

# Strictness
strict_equality = true
strict_concatenate = true

# Error messages
show_error_context = true
show_column_numbers = true
show_error_codes = true
pretty = true

# Miscellaneous
check_untyped_defs = true
warn_unused_configs = true

# Per-module options (for third-party libraries without types)
[[tool.mypy.overrides]]
module = [
    "some_untyped_library.*",
]
ignore_missing_imports = true
```

### Key Strict Options Explained

- **`strict = true`**: Enables all optional checks - the foundation of strict typing
- **`disallow_any_generics`**: Forces you to specify generic types (e.g., `list[int]` not `list`)
- **`disallow_untyped_defs`**: All functions must have type annotations
- **`warn_return_any`**: Warns when returning values of type `Any`
- **`strict_equality`**: Prevents comparing incompatible types (e.g., `str == int`)
- **`no_implicit_optional`**: `def foo(x: int = None)` is an error; must write `int | None`

### Type Annotations Example

```python
from collections.abc import Callable
from typing import Never

# ✅ Good - fully typed
def process_data(items: list[str], callback: Callable[[str], int]) -> dict[str, int]:
    """Process a list of items with a callback function."""
    return {item: callback(item) for item in items}

# ✅ Good - using modern union syntax
def get_user(user_id: int) -> dict[str, str] | None:
    """Fetch user data, returning None if not found."""
    pass

# ✅ Good - explicit Never for functions that never return
def handle_error(message: str) -> Never:
    """Raise an exception and never return."""
    raise RuntimeError(message)

# ❌ Bad - no type annotations (mypy error with strict mode)
def process_data(items, callback):
    return {item: callback(item) for item in items}

# ❌ Bad - implicit optional (mypy error with no_implicit_optional)
def get_user(user_id: int = None):
    pass
```

### Installation

```bash
pip install mypy
```

---

## Pre-commit Hooks

Pre-commit runs linting, formatting, and type checking before commits to ensure code quality.

### `.pre-commit-config.yaml`

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-toml
      - id: check-json
      - id: check-added-large-files
      - id: check-case-conflict
      - id: check-merge-conflict
      - id: detect-private-key

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.3.0
    hooks:
      # Run the linter
      - id: ruff
        args: [--fix, --exit-non-zero-on-fix]
      # Run the formatter
      - id: ruff-format

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.9.0
    hooks:
      - id: mypy
        additional_dependencies: []  # Add your type stub packages here
        args: [--strict]
```

### Installation

```bash
pip install pre-commit
pre-commit install
```

### Running Manually

```bash
# Run on all files
pre-commit run --all-files

# Run on staged files only
pre-commit run
```

---

## Git Attributes

Git attributes ensure consistent file handling across all environments and improve security.

### `.gitattributes`

```gitattributes
# Set default behavior to automatically normalize line endings to LF
* text=auto eol=lf

# Explicitly declare text files you want to always be normalized and converted to LF
*.py text eol=lf
*.pyx text eol=lf
*.pyi text eol=lf
*.txt text eol=lf
*.md text eol=lf
*.rst text eol=lf
*.yml text eol=lf
*.yaml text eol=lf
*.toml text eol=lf
*.json text eol=lf
*.ini text eol=lf
*.cfg text eol=lf
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
*.whl binary
*.egg binary
*.pyc binary
*.pyd binary
*.so binary
*.dll binary
*.dylib binary

# Security: Mark sensitive files as binary to prevent Git from attempting text operations
*.key binary
*.pem binary
*.p12 binary
*.pfx binary
*.cer binary
*.crt binary
*.der binary

# Archive files generated by export-ignore
.gitattributes export-ignore
.gitignore export-ignore
.editorconfig export-ignore
.pre-commit-config.yaml export-ignore
pyproject.toml export-ignore
*.md export-ignore
.vscode/ export-ignore
tests/ export-ignore
```

---

## Project Configuration (pyproject.toml)

The `pyproject.toml` file centralizes all project configuration.

### Complete `pyproject.toml` Example

```toml
[build-system]
requires = ["setuptools>=68.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "your-project-name"
version = "0.1.0"
description = "A description of your project"
authors = [
    {name = "Your Name", email = "your.email@example.com"}
]
readme = "README.md"
requires-python = ">=3.11"
license = {text = "MIT"}
classifiers = [
    "Development Status :: 3 - Alpha",
    "Intended Audience :: Developers",
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.11",
    "Programming Language :: Python :: 3.12",
]

dependencies = [
    # Add your runtime dependencies here
]

[project.optional-dependencies]
dev = [
    "ruff>=0.3.0",
    "mypy>=1.9.0",
    "pytest>=8.0.0",
    "pytest-cov>=4.1.0",
    "pre-commit>=3.6.0",
]

[project.urls]
Homepage = "https://github.com/yourusername/your-project-name"
Repository = "https://github.com/yourusername/your-project-name"

# Ruff configuration (from earlier section)
[tool.ruff]
line-length = 100
target-version = "py311"
fix = true
exclude = [
    ".git",
    ".venv",
    "venv",
    "__pycache__",
    "build",
    "dist",
    "*.egg-info",
]

[tool.ruff.lint]
select = [
    "E", "W", "F", "I", "N", "UP", "B", "C4", "DTZ", "T10", "EM",
    "ISC", "ICN", "PIE", "PT", "Q", "RSE", "RET", "SIM", "TID",
    "PTH", "ERA", "PL", "PERF", "RUF", "FURB",
]
ignore = ["ISC001", "COM812"]
dummy-variable-rgx = "^(_+|(_+[a-zA-Z0-9_]*[a-zA-Z0-9]+?))$"

[tool.ruff.lint.isort]
known-first-party = ["your_package_name"]
section-order = [
    "future",
    "standard-library",
    "third-party",
    "first-party",
    "local-folder",
]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
line-ending = "lf"

# mypy configuration (from earlier section)
[tool.mypy]
strict = true
python_version = "3.11"
disallow_any_unimported = true
disallow_any_generics = true
disallow_subclassing_any = true
disallow_untyped_calls = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
disallow_untyped_decorators = true
no_implicit_optional = true
strict_optional = true
warn_redundant_casts = true
warn_unused_ignores = true
warn_no_return = true
warn_return_any = true
warn_unreachable = true
strict_equality = true
strict_concatenate = true
show_error_context = true
show_column_numbers = true
show_error_codes = true
pretty = true
check_untyped_defs = true
warn_unused_configs = true

# pytest configuration
[tool.pytest.ini_options]
testpaths = ["tests"]
python_files = ["test_*.py", "*_test.py"]
python_classes = ["Test*"]
python_functions = ["test_*"]
addopts = [
    "--strict-markers",
    "--strict-config",
    "--cov=your_package_name",
    "--cov-report=term-missing",
    "--cov-report=html",
    "--cov-fail-under=80",
]

# Coverage configuration
[tool.coverage.run]
source = ["your_package_name"]
omit = [
    "tests/*",
    "*/test_*.py",
    "*/__pycache__/*",
]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise AssertionError",
    "raise NotImplementedError",
    "if __name__ == .__main__.:",
    "if TYPE_CHECKING:",
    "@abstractmethod",
]
```

---

## Installation Steps

Follow these steps to set up a new Python project from scratch:

### 1. Initialize Project

```bash
mkdir my-project
cd my-project
git init
```

### 2. Create Virtual Environment

```bash
# Using venv (built-in)
python -m venv .venv

# Activate on Windows
.venv\Scripts\activate

# Activate on Linux/macOS
source .venv/bin/activate
```

### 3. Create Project Structure

```bash
mkdir src
mkdir tests
touch src/__init__.py
touch tests/__init__.py
touch README.md
```

### 4. Create Configuration Files

Create the following files in your project root:
- `.editorconfig`
- `pyproject.toml`
- `.pre-commit-config.yaml`
- `.gitattributes`
- `.vscode/settings.json`
- `.vscode/extensions.json`

Copy the configurations from the sections above.

### 5. Install Development Dependencies

```bash
pip install ruff mypy pytest pytest-cov pre-commit
```

Or with the `pyproject.toml` optional dependencies:

```bash
pip install -e ".[dev]"
```

### 6. Initialize Pre-commit

```bash
pre-commit install
```

### 7. Add .gitignore

Find a template for your project on GitHub!

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
# Run ruff linter
ruff check .

# Run ruff formatter
ruff format .

# Run type checking
mypy src/

# Run tests
pytest

# Run pre-commit on all files
pre-commit run --all-files
```

### 9. Make Your First Commit

```bash
git add .
git commit -m "Initial project setup with Ruff, mypy, and pre-commit"
```

---

## Code Smart: Readability Principles

Beyond linters and type checkers, good code requires human judgment. These principles will make your code more maintainable and easier to understand.

### Be a Never-Nester

Deep nesting makes code exponentially harder to read and reason about. Each level of nesting adds cognitive load. Python's emphasis on readability makes this especially important.

**Bad - Deep Nesting:**

```python
def process_order(order: dict[str, Any]) -> str:
    if order is not None:
        if "items" in order and len(order["items"]) > 0:
            if "customer" in order and order["customer"] is not None:
                if order["customer"].get("is_active", False):
                    if order.get("total", 0) > 0:
                        return f"Processing order for {order['customer']['name']}"
                    else:
                        return "Order total must be positive"
                else:
                    return "Customer is inactive"
            else:
                return "Customer is required"
        else:
            return "Order has no items"
    else:
        return "Order is required"
```

**Good - Early Returns (Never-Nester):**

```python
def process_order(order: dict[str, Any]) -> str:
    if order is None:
        return "Order is required"

    if "items" not in order or len(order["items"]) == 0:
        return "Order has no items"

    if "customer" not in order or order["customer"] is None:
        return "Customer is required"

    if not order["customer"].get("is_active", False):
        return "Customer is inactive"

    if order.get("total", 0) <= 0:
        return "Order total must be positive"

    return f"Processing order for {order['customer']['name']}"
```

**Even Better - With Type Safety:**

```python
from dataclasses import dataclass

@dataclass
class Customer:
    name: str
    is_active: bool

@dataclass
class Order:
    items: list[str]
    customer: Customer | None
    total: float

def process_order(order: Order | None) -> str:
    if order is None:
        return "Order is required"

    if not order.items:
        return "Order has no items"

    if order.customer is None:
        return "Customer is required"

    if not order.customer.is_active:
        return "Customer is inactive"

    if order.total <= 0:
        return "Order total must be positive"

    return f"Processing order for {order.customer.name}"
```

**Key techniques for avoiding nesting:**

1. **Early returns**: Guard clauses at the top of functions
2. **Extract functions**: Break complex conditions into well-named functions
3. **Invert conditions**: Return early on error cases, continue with happy path
4. **Use walrus operator**: Assign and check in one line when appropriate
5. **Prefer flat structures**: Use dictionaries and dataclasses over deeply nested structures

```python
# Instead of nested checks
if user is not None:
    if hasattr(user, "settings"):
        if user.settings is not None:
            theme = user.settings.theme

# Use walrus operator and short-circuit
theme = "default"
if user is not None and (settings := getattr(user, "settings", None)) is not None:
    theme = settings.theme

# Or even better with optional chaining pattern
theme = getattr(getattr(user, "settings", None), "theme", "default")
```

### Don't Use Abbreviations - Characters Are (Almost) Free

Abbreviations save a few keystrokes but cost hundreds of hours in confusion over a project's lifetime. Modern editors have autocomplete - take advantage of it.

**Bad - Cryptic Abbreviations:**

```python
def calc_usr_disc(usr: Usr, amt: float) -> float:
    lvl = usr.mem_lvl
    disc_pct = 0.2 if lvl == "prem" else 0.1
    return amt * disc_pct

btn = driver.find_element(By.CLASS_NAME, "btn")
msg = "Hello"
idx = arr.index(next(el for el in arr if el.id == tgt_id))
```

**Good - Clear Names:**

```python
def calculate_user_discount(user: User, amount: float) -> float:
    membership_level = user.membership_level
    discount_percentage = 0.2 if membership_level == "premium" else 0.1
    return amount * discount_percentage

button = driver.find_element(By.CLASS_NAME, "button")
message = "Hello"
index = array.index(next(element for element in array if element.id == target_id))
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
- `val` → `value`
- `args` → `arguments` (except in `*args`)
- `kwargs` → `keyword_arguments` (except in `**kwargs`)
- `num` → `number` or `count`
- `str` → `string` (for variable names, not the type)

**Acceptable abbreviations** (widely understood in context):

- `id` - universally understood identifier
- `url` - standard acronym
- `api` - standard acronym
- `html`, `css`, `json` - standard formats
- `max`, `min` - mathematical conventions
- `i`, `j`, `k` - loop counters in simple loops (but prefer descriptive names in complex loops)
- `df` - pandas DataFrame (community convention)
- `np` - numpy (community convention)

**Example of when to use full names even in loops:**

```python
# ❌ Bad - unclear what we're iterating over
for i in range(len(usrs)):
    for j in range(len(usrs[i].ords)):
        # What is i? What is j?
        process(usrs[i].ords[j])

# ✅ Good - crystal clear
for user in users:
    for order in user.orders:
        # Immediately clear what we're working with
        process(order)

# ✅ Also good - when you need the index
for user_index, user in enumerate(users):
    for order_index, order in enumerate(user.orders):
        print(f"Processing order {order_index} for user {user_index}")
```

**Python-specific naming conventions:**

```python
# ✅ Good - follow PEP 8
class UserManager:  # PascalCase for classes
    def __init__(self):
        self.active_users = []  # snake_case for variables
        self.MAX_USERS = 100  # UPPER_CASE for constants

    def get_user_by_id(self, user_id: int) -> User | None:  # snake_case for functions
        """Get user by their ID."""
        pass

# ❌ Bad - inconsistent naming
class userManager:  # Wrong case
    def GetUserByID(self, userID: int) -> User | None:  # Wrong case
        pass
```

### Why This Matters

1. **Onboarding**: New team members can understand code faster
2. **Code reviews**: Reviewers spend less time deciphering abbreviations
3. **Debugging**: Clear names make stack traces and error messages meaningful
4. **Future you**: You'll forget what `calc_usr_disc` means in 6 months
5. **Searchability**: Full words are easier to search for across a codebase
6. **PEP 20 (Zen of Python)**: "Readability counts" and "Explicit is better than implicit"

### The Cost of Characters vs. The Cost of Confusion

- Typing 10 extra characters: **2 seconds**
- Figuring out what `usr_mgr` means: **30 seconds**
- Debugging because you confused `msg_cnt` (message count) with `msg_cnt` (message content): **30 minutes**

**Characters are almost free. Confusion is expensive.**

### Remember the Zen of Python

```python
import this
```

> Beautiful is better than ugly.
> Explicit is better than implicit.
> Simple is better than complex.
> Flat is better than nested.
> Readability counts.

---

## A Word on "Prototypes" and Strict Settings

**Warning**: A common mistake is to start a project with relaxed settings for "prototyping" or "moving fast", planning to add strictness later.

### Why This Approach Fails

1. **Prototypes become production**: What starts as a quick prototype often becomes the production codebase when it works well enough. Nobody wants to stop and refactor working code.

2. **Retrofitting is painful**: Adding strict type checking to existing code can generate hundreds or thousands of errors. Teams often give up or use `# type: ignore` everywhere, defeating the purpose.

3. **Technical debt compounds**: Every day you write code without strict checks, you're creating more technical debt that becomes harder to fix.

4. **Team resistance**: Once a team gets used to loose checks, there's psychological resistance to "making the build harder" by enabling strict mode.

### The Better Approach

**Start strict from day one**, even for prototypes:

- It's easier to write strict code from the beginning than to fix it later
- You catch bugs immediately instead of discovering them in production
- The "slowdown" from strict checks is minimal compared to the time saved debugging
- Your prototype is already production-ready if it succeeds
- Python's gradual typing allows you to be strict where it matters most

### If You Must Relax Settings

If you truly need to relax some rules temporarily, do it strategically:

1. **Use per-file or per-module overrides** in mypy configuration
2. **Set a deadline** (e.g., "before first production deploy") to migrate to full strict mode
3. **Keep Ruff formatting and basic linting** - code style should always be enforced
4. **Never disable**: `disallow_untyped_defs`, `strict_optional`, `warn_unreachable` - these catch too many real bugs

### Rules You Might Consider Relaxing (with caution)

```toml
# In pyproject.toml [tool.mypy] - consider keeping these enabled even for prototypes:
disallow_any_generics = false  # Allow list instead of list[str]
warn_return_any = false  # Don't warn about returning Any

# In pyproject.toml [tool.ruff.lint] - consider relaxing:
# Remove specific rule categories if they slow you down during prototyping
# For example, remove "PL" (pylint) or "PERF" (performance checks)
```

**Remember**: Every rule you disable is a category of bugs you're allowing into your codebase. The pain of strict checks is temporary; the pain of production bugs is permanent.

### Python-Specific Advantages

Python's gradual typing system is actually designed for this strict approach:

- **Start with critical functions typed**: Type your public API and core business logic first
- **Use `Any` strategically**: When you genuinely don't know a type yet, explicit `Any` is acceptable
- **Leverage type inference**: Python's type inference reduces annotation burden
- **Protocol types**: Use `typing.Protocol` for structural subtyping when inheritance is too rigid

---

## Summary

This setup provides:

- **Consistent Code Style**: EditorConfig + Ruff formatter
- **Code Quality**: Ruff linter with comprehensive rule sets
- **Type Safety**: Strict mypy configuration catching type errors
- **Automated Quality Checks**: Pre-commit hooks for all tools
- **Cross-platform Consistency**: Git attributes for line endings and file handling
- **Security**: Proper handling of binary files and sensitive data
- **Developer Experience**: VS Code integration with recommended extensions
- **Modern Python**: Encourages Python 3.11+ features and best practices

By following this guide, your Python projects will have a solid foundation for maintainability, consistency, and quality.

---

## References

> "The wise man learns from everything and everyone, the ordinary man learns from his experience, and the fool knows everything better." - Socrates

- The [CodeAesthetic](https://www.youtube.com/@CodeAesthetic) YouTube channel
- ["Clean" Code, Horrible Performance](https://www.youtube.com/watch?v=tD5NrevFtbU) by [Molly Rocket](https://www.youtube.com/@MollyRocket)
