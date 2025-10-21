# Python Project Setup Guide

This guide provides a comprehensive reference for setting up new Python projects with best practices, consistent code quality, and automated tooling.

## Table of Contents

1. [EditorConfig](#editorconfig)
2. [VS Code Configuration](#vs-code-configuration)
3. [Ruff (Formatter and Linter)](#ruff-formatter-and-linter)
4. [mypy (Type Checking)](#mypy-type-checking)
5. [Pydantic (Runtime Validation)](#pydantic-runtime-validation)
6. [Error Handling with Dataclasses and Pattern Matching](#error-handling-with-dataclasses-and-pattern-matching)
7. [Pre-commit Hooks](#pre-commit-hooks)
8. [Git Attributes](#git-attributes)
9. [Project Configuration (pyproject.toml)](#project-configuration-pyprojecttoml)
10. [Installation Steps](#installation-steps)
11. [Code Smart: Readability Principles](#code-smart-readability-principles)
12. [A Word on "Prototypes" and Strict Settings](#a-word-on-prototypes-and-strict-settings)
13. [References](#references)

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

## Pydantic (Runtime Validation)

Pydantic provides runtime data validation and settings management using Python type annotations. While mypy catches type errors at development time, Pydantic validates data at runtime - especially valuable at system boundaries like API requests, configuration files, and external data sources.

> **Why Pydantic?** Static type checking (mypy) ensures your code is internally consistent, but it can't validate data that arrives at runtime from users, APIs, or config files. Pydantic bridges this gap by providing automatic validation, parsing, and serialization based on type hints.

### The Progression: Types → Validation → Pydantic

Let's see how Pydantic improves upon basic type hints and dataclasses:

**Level 1 - Basic Type Hints (Static checking only):**

```python
def create_user(name: str, age: int, email: str) -> dict[str, str | int]:
    """No runtime validation - type hints are just documentation."""
    return {"name": name, "age": age, "email": email}

# This will pass at runtime, even though types are wrong!
user = create_user(123, "not a number", "invalid-email")  # mypy error, but runs
```

**Level 2 - Dataclasses (Structure, but no validation):**

```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    age: int
    email: str

# Better structure, but still no runtime validation
user = User(name=123, age="not a number", email="invalid-email")  # Runs anyway!
```

**Level 3 - Pydantic (Structure + Runtime Validation):**

```python
from pydantic import BaseModel, EmailStr, Field

class User(BaseModel):
    name: str = Field(min_length=1, max_length=100)
    age: int = Field(ge=0, le=150)
    email: EmailStr

# Now this raises a ValidationError at runtime!
try:
    user = User(name=123, age="not a number", email="invalid-email")
except ValidationError as e:
    print(e)
    # Shows exactly what's wrong:
    # - name: Input should be a valid string
    # - age: Input should be a valid integer
    # - email: value is not a valid email address
```

### Pydantic Configuration in `pyproject.toml`

```toml
[tool.pydantic]
# Use V2 strict mode for maximum type safety
# This aligns with our strict mypy configuration
strict = true

# Validate assignment after model creation
validate_assignment = true

# Validate default values
validate_default = true

# Use enum values instead of enum members in serialization
use_enum_values = true
```

### Real-World Use Cases

#### 1. API Request/Response Models

```python
from datetime import datetime
from pydantic import BaseModel, Field, EmailStr, HttpUrl
from typing import Literal

class CreateUserRequest(BaseModel):
    """Validates incoming API requests."""
    username: str = Field(min_length=3, max_length=30, pattern="^[a-zA-Z0-9_]+$")
    email: EmailStr
    age: int = Field(ge=13, le=120, description="User must be 13-120 years old")
    website: HttpUrl | None = None
    role: Literal["user", "admin", "moderator"] = "user"

class UserResponse(BaseModel):
    """Validates outgoing API responses."""
    id: int
    username: str
    email: EmailStr
    created_at: datetime
    role: str

    class Config:
        # Allow ORM objects (like SQLAlchemy models) to be converted
        from_attributes = True

# FastAPI automatically validates using Pydantic models
# from fastapi import FastAPI
# app = FastAPI()
#
# @app.post("/users", response_model=UserResponse)
# def create_user(user: CreateUserRequest) -> UserResponse:
#     # user is guaranteed to be valid!
#     # FastAPI returns 422 Unprocessable Entity if validation fails
#     pass
```

#### 2. Configuration Files

```python
from pathlib import Path
from pydantic import Field, field_validator
from pydantic_settings import BaseSettings

class DatabaseConfig(BaseModel):
    """Database configuration with validation."""
    host: str = "localhost"
    port: int = Field(ge=1, le=65535, default=5432)
    username: str
    password: str = Field(min_length=8)
    database: str

    @property
    def connection_string(self) -> str:
        """Generate connection string from validated config."""
        return f"postgresql://{self.username}:{self.password}@{self.host}:{self.port}/{self.database}"

class AppSettings(BaseSettings):
    """Application settings loaded from environment variables or .env file."""
    app_name: str = "My Application"
    debug: bool = False
    database: DatabaseConfig
    api_key: str = Field(min_length=32)
    log_level: Literal["DEBUG", "INFO", "WARNING", "ERROR"] = "INFO"
    max_connections: int = Field(ge=1, le=1000, default=100)

    class Config:
        # Automatically load from .env file
        env_file = ".env"
        env_nested_delimiter = "__"  # DATABASE__HOST in .env → database.host

# Usage
settings = AppSettings()  # Raises ValidationError if config is invalid
print(settings.database.connection_string)
```

#### 3. Data Validation with Custom Validators

```python
from pydantic import BaseModel, field_validator, model_validator

class PasswordReset(BaseModel):
    """Password reset with custom validation."""
    password: str = Field(min_length=12)
    password_confirm: str

    @field_validator("password")
    @classmethod
    def validate_password_strength(cls, value: str) -> str:
        """Ensure password meets strength requirements."""
        if not any(c.isupper() for c in value):
            raise ValueError("Password must contain at least one uppercase letter")
        if not any(c.islower() for c in value):
            raise ValueError("Password must contain at least one lowercase letter")
        if not any(c.isdigit() for c in value):
            raise ValueError("Password must contain at least one digit")
        if not any(c in "!@#$%^&*" for c in value):
            raise ValueError("Password must contain at least one special character")
        return value

    @model_validator(mode="after")
    def validate_passwords_match(self) -> "PasswordReset":
        """Ensure password and confirmation match."""
        if self.password != self.password_confirm:
            raise ValueError("Passwords do not match")
        return self

# Usage
try:
    reset = PasswordReset(password="weak", password_confirm="weak")
except ValidationError as e:
    print(e)  # Shows all validation errors
```

#### 4. Parsing and Coercion

```python
from datetime import datetime
from pydantic import BaseModel

class Event(BaseModel):
    """Pydantic automatically parses and coerces types."""
    name: str
    timestamp: datetime
    attendee_count: int
    is_public: bool

# Pydantic automatically converts compatible types
event = Event(
    name="Python Meetup",
    timestamp="2024-01-15T19:00:00",  # String → datetime
    attendee_count="42",  # String → int
    is_public="yes"  # String → bool (recognizes yes/true/1)
)

print(event.timestamp)  # datetime object
print(event.attendee_count)  # int: 42
print(event.is_public)  # bool: True

# Serialize back to dict or JSON
print(event.model_dump())  # Python dict
print(event.model_dump_json())  # JSON string
```

### Integration with mypy

Pydantic works seamlessly with mypy's strict mode. Install the mypy plugin:

```toml
# In pyproject.toml
[tool.mypy]
plugins = ["pydantic.mypy"]

[tool.pydantic-mypy]
init_forbid_extra = true
init_typed = true
warn_required_dynamic_aliases = true
```

This enables:
- Full type inference for Pydantic models
- Proper typing for model fields
- Detection of invalid field assignments

### When to Use Pydantic vs. Dataclasses

**Use Pydantic when:**
- ✅ Validating external data (APIs, user input, config files)
- ✅ Parsing data from JSON, environment variables, or other formats
- ✅ You need automatic type coercion (e.g., string → int)
- ✅ Building APIs (especially with FastAPI)
- ✅ Complex validation logic is required

**Use dataclasses when:**
- ✅ Data is already validated and internal to your system
- ✅ You want zero runtime overhead
- ✅ Simple data containers without validation needs
- ✅ Working with frozen/immutable data structures

**Use both:**
```python
from dataclasses import dataclass
from pydantic import BaseModel

# Pydantic at the boundary (validates external data)
class UserCreateRequest(BaseModel):
    username: str
    email: EmailStr
    age: int = Field(ge=13)

# Dataclass internally (already validated)
@dataclass(frozen=True)
class User:
    id: int
    username: str
    email: str
    age: int

def create_user(request: UserCreateRequest) -> User:
    """Accept validated Pydantic model, return internal dataclass."""
    # request is guaranteed to be valid
    user_id = database.insert(request.model_dump())
    return User(
        id=user_id,
        username=request.username,
        email=request.email,
        age=request.age
    )
```

### Pydantic Best Practices

1. **Validate at boundaries**: Use Pydantic where data enters your system (API endpoints, config loading, file parsing)

2. **Use descriptive Field constraints**: Make validation rules explicit
   ```python
   age: int = Field(ge=0, le=150, description="Age in years")
   ```

3. **Leverage built-in validators**: Use `EmailStr`, `HttpUrl`, `FilePath`, etc. instead of string
   ```python
   from pydantic import BaseModel, EmailStr, HttpUrl, FilePath

   class Config(BaseModel):
       email: EmailStr  # Validates email format
       website: HttpUrl  # Validates URL format
       config_file: FilePath  # Validates file exists
   ```

4. **Use Literal for enums**: Better than string validation for fixed choices
   ```python
   status: Literal["pending", "approved", "rejected"]
   ```

5. **Model composition**: Break large models into smaller, reusable components
   ```python
   class Address(BaseModel):
       street: str
       city: str
       zip_code: str = Field(pattern=r"^\d{5}$")

   class User(BaseModel):
       name: str
       address: Address  # Nested validation
   ```

6. **Use model_validator for cross-field validation**: When one field depends on another
   ```python
   @model_validator(mode="after")
   def check_dates(self) -> "Booking":
       if self.end_date < self.start_date:
           raise ValueError("end_date must be after start_date")
       return self
   ```

### Common Pitfalls to Avoid

❌ **Don't use Pydantic for performance-critical internal data structures**
```python
# Bad - unnecessary overhead in tight loops
for i in range(1_000_000):
    point = Point(x=i, y=i * 2)  # Validates on every iteration
```

✅ **Use dataclasses or plain dicts for internal data**
```python
# Good - validate once at boundary, use fast structures internally
@dataclass
class Point:
    x: int
    y: int

def process_points(raw_data: list[dict]) -> list[Point]:
    # Validate once with Pydantic
    validated = [PointModel(**p) for p in raw_data]
    # Convert to fast dataclasses for processing
    return [Point(x=p.x, y=p.y) for p in validated]
```

### Installation

```bash
pip install pydantic[email]  # Includes email validation
```

For configuration management:
```bash
pip install pydantic-settings
```

### Further Reading

- [Pydantic Documentation](https://docs.pydantic.dev/)
- [Pydantic with FastAPI](https://fastapi.tiangolo.com/)
- [pydantic-settings for configuration](https://docs.pydantic.dev/latest/concepts/pydantic_settings/)

---

## Error Handling with Dataclasses and Pattern Matching

Python 3.10+ provides powerful built-in features for explicit error handling through dataclasses, union types, and pattern matching. These native features make error handling type-safe and composable without external dependencies.

> **Why dataclasses and pattern matching?** Exceptions are invisible in function signatures and easy to forget. Using dataclasses with union types makes errors explicit and enforced by mypy. Pattern matching (Python 3.10+) provides elegant syntax for handling different cases. This leads to more robust code where error handling is verified by static type checking, using only native Python features.

### The Problem with Exceptions

**Traditional exception-based code:**

```python
def divide(a: int, b: int) -> float:
    """Can raise ZeroDivisionError, but signature doesn't show it."""
    return a / b

def get_user(user_id: int) -> dict[str, str]:
    """Can raise KeyError, but not visible in signature."""
    return database[user_id]

# Easy to forget error handling - no compile-time warning
result = divide(10, 0)  # Crashes at runtime!
user = get_user(999)    # Crashes at runtime!
```

**Issues with exceptions:**

1. **Invisible in signatures**: No way to know which functions can fail
2. **Easy to forget**: Nothing forces you to handle errors
3. **Break composition**: Can't easily chain operations that might fail
4. **Unclear control flow**: Exceptions can bubble up from anywhere

### Result Types with Dataclasses

Use dataclasses and union types to make errors explicit in function signatures:

```python
from dataclasses import dataclass
from typing import Generic, TypeVar

T = TypeVar('T')
E = TypeVar('E')

@dataclass
class Success(Generic[T]):
    """Represents a successful result."""
    value: T

@dataclass
class Failure(Generic[E]):
    """Represents a failed result."""
    error: E

# Type alias for Result
Result = Success[T] | Failure[E]

def divide(a: int, b: int) -> Result[float, str]:
    """Returns Success or Failure. Visible in signature!"""
    if b == 0:
        return Failure("Division by zero")
    return Success(a / b)

def get_user(user_id: int) -> Result[dict[str, str], str]:
    """Returns Success or Failure. Impossible to ignore!"""
    user = database.get(user_id)
    if user is None:
        return Failure(f"User {user_id} not found")
    return Success(user)

# Pattern matching (Python 3.10+) forces you to handle both cases
result = divide(10, 2)
match result:
    case Success(value):
        print(f"Result: {value}")
    case Failure(error):
        print(f"Error: {error}")

# ❌ mypy error if you forget to handle Failure!
# value: float = divide(10, 0).value  # Type error - Failure has no value attribute
```

### Chaining Operations with Helper Functions

Compose operations that might fail using simple helper functions:

```python
from dataclasses import dataclass
from typing import Callable, Generic, TypeVar

T = TypeVar('T')
U = TypeVar('U')
E = TypeVar('E')

@dataclass
class Success(Generic[T]):
    value: T

@dataclass
class Failure(Generic[E]):
    error: E

Result = Success[T] | Failure[E]

# Helper to chain Result-returning operations
def and_then(result: Result[T, E], fn: Callable[[T], Result[U, E]]) -> Result[U, E]:
    """Apply fn if result is Success, otherwise propagate Failure."""
    match result:
        case Success(value):
            return fn(value)
        case Failure(error):
            return Failure(error)

# Helper to transform success values
def map_result(result: Result[T, E], fn: Callable[[T], U]) -> Result[U, E]:
    """Transform the value if result is Success."""
    match result:
        case Success(value):
            return Success(fn(value))
        case Failure(error):
            return Failure(error)

# Each function returns Result[T, str]
def parse_int(value: str) -> Result[int, str]:
    """Parse string to int."""
    try:
        return Success(int(value))
    except ValueError:
        return Failure(f"Invalid integer: {value}")

def validate_positive(value: int) -> Result[int, str]:
    """Ensure value is positive."""
    if value <= 0:
        return Failure(f"Must be positive, got {value}")
    return Success(value)

# Chain all operations - stops at first failure
result = and_then(
    and_then(parse_int("42"), validate_positive),
    lambda v: Success(v / 2.0)
)

match result:
    case Success(value):
        print(f"Final value: {value}")  # 21.0
    case Failure(error):
        print(f"Pipeline failed: {error}")

# If any step fails, the rest are skipped
result_error = and_then(
    and_then(parse_int("-5"), validate_positive),  # Fails here
    lambda v: Success(v / 2.0)  # Skipped!
)
# Result: Failure("Must be positive, got -5")
```

### Converting Exceptions to Results

Create a decorator to automatically convert exceptions into Failure:

```python
from typing import Callable, TypeVar
from functools import wraps

T = TypeVar('T')

def safe(fn: Callable[..., T]) -> Callable[..., Result[T, Exception]]:
    """Decorator that catches exceptions and returns Result."""
    @wraps(fn)
    def wrapper(*args, **kwargs) -> Result[T, Exception]:
        try:
            return Success(fn(*args, **kwargs))
        except Exception as e:
            return Failure(e)
    return wrapper

@safe
def parse_json(text: str) -> dict:
    """Automatically catches JSONDecodeError and returns Result."""
    import json
    return json.loads(text)

# Type: Result[dict, Exception]
result1 = parse_json('{"name": "John"}')  # Success({"name": "John"})
result2 = parse_json('invalid json')       # Failure(JSONDecodeError(...))

# Chain with other operations
def get_name(data: dict) -> str | None:
    return data.get("name")

user_name_result = map_result(parse_json('{"name": "John", "age": 30}'), get_name)

match user_name_result:
    case Success(name):
        print(name)  # "John"
    case Failure(error):
        print(f"Error: {error}")
```

### Option: Type-Safe Optional Values

For operations that might return nothing, use dataclasses (better than bare `None`):

```python
from dataclasses import dataclass
from typing import Generic, TypeVar

T = TypeVar('T')

@dataclass
class Some(Generic[T]):
    """Represents a value that exists."""
    value: T

@dataclass
class Nothing:
    """Represents the absence of a value."""
    pass

Option = Some[T] | Nothing

def find_user_by_email(email: str) -> Option[dict[str, str]]:
    """Returns Some(user) if found, Nothing otherwise."""
    user = database.find_one({"email": email})
    return Some(user) if user else Nothing()

def get_user_name(user: dict[str, str]) -> Option[str]:
    """Extract name from user."""
    name = user.get("name")
    return Some(name) if name else Nothing()

# Helper to chain Option operations
def and_then_option(option: Option[T], fn: Callable[[T], Option[U]]) -> Option[U]:
    """Apply fn if option is Some, otherwise return Nothing."""
    match option:
        case Some(value):
            return fn(value)
        case Nothing():
            return Nothing()

# Chain Option operations - stops at first Nothing
result = and_then_option(find_user_by_email("john@example.com"), get_user_name)

# Extract with default
def unwrap_or(option: Option[T], default: T) -> T:
    """Return value if Some, otherwise return default."""
    match option:
        case Some(value):
            return value
        case Nothing():
            return default

name = unwrap_or(result, "Anonymous User")
print(name)

# For simple cases, T | None is often sufficient
def find_user_simple(email: str) -> dict[str, str] | None:
    """Returns user or None."""
    return database.find_one({"email": email})

user = find_user_simple("john@example.com")
user_name = user.get("name") if user else "Anonymous User"
```

### Organizing Pure and Impure Code

While Python doesn't have IO types built-in, you can use naming conventions and type hints to separate pure and impure code:

```python
from typing import TypeAlias

# Type alias to mark functions that perform IO
IOResult: TypeAlias = Result[T, E]

# Pure function - no side effects, always returns same output for same input
def calculate_total(items: list[dict]) -> float:
    """Pure function - no IO."""
    return sum(item["price"] * item["quantity"] for item in items)

# Impure function - has side effects (reads file)
# Use @safe decorator and IOResult type hint to signal IO
@safe
def read_config(path: str) -> dict:
    """Reads file - marked as impure with IOResult type and naming."""
    import json
    with open(path) as f:
        return json.load(f)

# Type annotation shows this function does IO
config: IOResult[dict, Exception] = read_config("/etc/config.json")

# Use Result for operations with side effects
def process_order_io(order_data: dict) -> IOResult[str, str]:
    """Process order with database side effects."""
    total = calculate_total(order_data["items"])  # Pure - no IO

    # Impure - database operation
    try:
        order_id = database.insert_order(order_data, total)
        return Success(f"Order {order_id} created")
    except Exception as e:
        return Failure(str(e))

# Benefits:
# 1. Naming convention makes side effects visible
# 2. Can test pure functions without mocking
# 3. Clear separation of business logic (pure) from infrastructure (impure)
# 4. Type hints document which functions perform IO
```

### Practical Example: User Registration Pipeline

Putting it all together:

```python
from dataclasses import dataclass
from typing import TypeVar

T = TypeVar('T')
E = TypeVar('E')

@dataclass
class Success(Generic[T]):
    value: T

@dataclass
class Failure(Generic[E]):
    error: E

Result = Success[T] | Failure[E]

@dataclass(frozen=True)
class User:
    """Domain model."""
    username: str
    email: str
    age: int

# Validation functions (pure, no side effects)
def validate_username(username: str) -> Result[str, str]:
    """Validate username format."""
    if len(username) < 3:
        return Failure("Username must be at least 3 characters")
    if not username.isalnum():
        return Failure("Username must be alphanumeric")
    return Success(username)

def validate_email(email: str) -> Result[str, str]:
    """Validate email format."""
    if "@" not in email or "." not in email.split("@")[1]:
        return Failure("Invalid email format")
    return Success(email)

def validate_age(age: int) -> Result[int, str]:
    """Validate age range."""
    if age < 13:
        return Failure("Must be at least 13 years old")
    if age > 120:
        return Failure("Invalid age")
    return Success(age)

def create_user_object(username: str, email: str, age: int) -> User:
    """Create user object."""
    return User(username=username, email=email, age=age)

# Impure function (database side effect)
def save_to_database(user: User) -> Result[User, Exception]:
    """Save user to database (might raise exception)."""
    try:
        database.insert({"username": user.username, "email": user.email, "age": user.age})
        return Success(user)
    except Exception as e:
        return Failure(e)

# Compose the entire pipeline
def register_user(
    username: str,
    email: str,
    age: int,
) -> Result[User, str | Exception]:
    """Register user with full validation and error handling."""
    # Validate all fields
    username_result = validate_username(username)
    match username_result:
        case Failure(error):
            return username_result

    email_result = validate_email(email)
    match email_result:
        case Failure(error):
            return email_result

    age_result = validate_age(age)
    match age_result:
        case Failure(error):
            return age_result

    # All validations passed - create and save user
    match username_result, email_result, age_result:
        case Success(uname), Success(em), Success(a):
            user = create_user_object(uname, em, a)
            return save_to_database(user)

# Usage (Python 3.10+ pattern matching)
result = register_user("john_doe", "john@example.com", 25)
match result:
    case Success(user):
        print(f"User {user.username} registered successfully!")
    case Failure(error):
        error_msg = str(error) if isinstance(error, Exception) else error
        print(f"Registration failed: {error_msg}")

# All possible errors are handled by pattern matching and checked by mypy!
```

### Integration with mypy

The strict mypy configuration (already in our pyproject.toml) enables full type checking:

```toml
# In pyproject.toml
[tool.mypy]
strict = true
python_version = "3.11"
```

Benefits with strict mode and Python 3.10+:

- **Complete type inference** for dataclass-based Result and Option types
- **Enforcement of error handling** - mypy warns about unhandled cases
- **Exhaustiveness checking** with pattern matching (Python 3.10+)
- **No external dependencies** - uses only native Python features
- **Better IDE support** - editors understand dataclasses natively

### When to Use Result Types

**Use Result types (dataclasses with pattern matching) when:**

- ✅ Errors are expected as part of normal operation (validation, parsing, user input)
- ✅ You want explicit error handling enforced by the type system
- ✅ Building APIs where failures are common and should be handled gracefully
- ✅ Need to chain operations that can fail
- ✅ Want to eliminate "forgot to handle error" bugs
- ✅ Working with financial, medical, or other critical domains
- ✅ Using Python 3.10+ (for pattern matching)

**Use traditional exceptions when:**

- ✅ Dealing with truly exceptional conditions (out of memory, hardware failure)
- ✅ Errors that should propagate up through many layers
- ✅ Working with third-party libraries that use exceptions
- ✅ Simple scripts where explicit error handling is overkill
- ✅ Python is idiomatic with exceptions for many use cases

**Hybrid approach** (recommended):

```python
# Business logic: Use Result types for expected failures
def validate_transfer(from_account: int, to_account: int, amount: float) -> Result[None, str]:
    """Business logic with explicit error handling."""
    if amount <= 0:
        return Failure("Amount must be positive")
    if from_account == to_account:
        return Failure("Cannot transfer to same account")
    return Success(None)

# Infrastructure: Wrap exceptions in Results
def execute_transfer(from_account: int, to_account: int, amount: float) -> Result[None, Exception]:
    """Database operation - wrap exceptions in Result."""
    try:
        database.execute(
            "UPDATE accounts SET balance = balance - ? WHERE id = ?",
            (amount, from_account)
        )
        database.execute(
            "UPDATE accounts SET balance = balance + ? WHERE id = ?",
            (amount, to_account)
        )
        return Success(None)
    except Exception as e:
        return Failure(e)

# Compose them together
def transfer_money(
    from_account: int,
    to_account: int,
    amount: float
) -> Result[str, str | Exception]:
    """Full transfer with validation and error handling."""
    validation_result = validate_transfer(from_account, to_account, amount)
    match validation_result:
        case Failure(error):
            return validation_result
        case Success(_):
            execute_result = execute_transfer(from_account, to_account, amount)
            match execute_result:
                case Failure(error):
                    return execute_result
                case Success(_):
                    return Success(f"Transferred ${amount} successfully")
```

### Reusable Helper Library

Create a utilities module for common operations:

```python
# result_utils.py
from dataclasses import dataclass
from typing import Callable, Generic, TypeVar

T = TypeVar('T')
U = TypeVar('U')
E = TypeVar('E')
F = TypeVar('F')

@dataclass
class Success(Generic[T]):
    """Successful result with a value."""
    value: T

@dataclass
class Failure(Generic[E]):
    """Failed result with an error."""
    error: E

Result = Success[T] | Failure[E]

# Common helper functions
def map_result(result: Result[T, E], fn: Callable[[T], U]) -> Result[U, E]:
    """Transform success value, leave failure unchanged."""
    match result:
        case Success(value):
            return Success(fn(value))
        case Failure(error):
            return Failure(error)

def map_error(result: Result[T, E], fn: Callable[[E], F]) -> Result[T, F]:
    """Transform error value, leave success unchanged."""
    match result:
        case Success(value):
            return Success(value)
        case Failure(error):
            return Failure(fn(error))

def and_then(result: Result[T, E], fn: Callable[[T], Result[U, E]]) -> Result[U, E]:
    """Chain operations that return Result (flatMap)."""
    match result:
        case Success(value):
            return fn(value)
        case Failure(error):
            return Failure(error)

def unwrap_or(result: Result[T, E], default: T) -> T:
    """Extract value or use default."""
    match result:
        case Success(value):
            return value
        case Failure(_):
            return default

def unwrap(result: Result[T, E]) -> T:
    """Extract value or raise exception."""
    match result:
        case Success(value):
            return value
        case Failure(error):
            raise error if isinstance(error, Exception) else ValueError(error)

# Usage
from result_utils import Result, Success, Failure, map_result, unwrap_or

result: Result[int, str] = Success(5)
doubled = map_result(result, lambda x: x * 2)  # Success(10)
value = unwrap_or(doubled, 0)  # 10
```

### No Installation Required

This pattern uses only native Python features - no dependencies needed! Simply define the dataclasses in your codebase or create a small utilities module.

**Requirements:**
- Python 3.10+ (for pattern matching syntax)
- Python 3.9+ can use this pattern but requires if/else instead of match/case

### Why This Makes Python Better

Python is an excellent language, but exceptions can be a weakness for type safety and composition. Dataclasses with pattern matching bring the rigor of languages like Rust and Haskell to Python while maintaining Pythonic readability and using only native features.

The combination of:
- **Strict mypy** → catches type errors at development time
- **Pydantic** → validates external data at runtime
- **Dataclasses + Pattern Matching** → makes error handling explicit and type-safe

...creates a development experience that rivals statically-typed languages while keeping Python's expressiveness and productivity. For teams willing to embrace these patterns, Python becomes a genuinely world-class language for building reliable systems - all without external dependencies.

### Further Reading

- [Python Dataclasses](https://docs.python.org/3/library/dataclasses.html) - Official dataclasses documentation
- [Pattern Matching](https://peps.python.org/pep-0636/) - PEP 636: Structural Pattern Matching Tutorial
- [Railway Oriented Programming](https://fsharpforfunandprofit.com/rop/) - The conceptual foundation (F# article, concepts apply to all languages)
- [Rust's Result Documentation](https://doc.rust-lang.org/std/result/) - The inspiration for this pattern

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
        additional_dependencies: [pydantic]  # Add your type stub packages here
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
    "pydantic[email]>=2.0.0",
    "pydantic-settings>=2.0.0",
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
plugins = ["pydantic.mypy"]
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

[tool.pydantic-mypy]
init_forbid_extra = true
init_typed = true
warn_required_dynamic_aliases = true

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

### 10. Keeping Dependencies Updated

Regularly update your dependencies to get bug fixes, security patches, and new features:

```bash
# Check for outdated packages
pip list --outdated

# Update a specific package
pip install --upgrade package_name

# Update all packages (be cautious with this)
pip install --upgrade -r requirements.txt

# Or if using pyproject.toml with dev dependencies
pip install --upgrade -e ".[dev]"

# Use pip-audit to check for security vulnerabilities
pip install pip-audit
pip-audit
```

**Best practices:**
- Update dependencies at least monthly
- Review changelogs before updating major versions
- Run your full test suite after updates
- Pin dependencies in production deployments
- Use tools like `pip-audit` or `safety` to scan for vulnerabilities
- Consider using Dependabot or Renovate for automated dependency updates
- Keep your Python version updated to get security fixes

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
- **Type Safety**: Strict mypy configuration catching type errors at development time
- **Runtime Validation**: Pydantic for validating data at system boundaries
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
