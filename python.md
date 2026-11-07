# Python Style Guide

An opinionated standard for Python that is consistent to read, explicit about its contracts, and straightforward to change. It covers backend services, CLIs, libraries, data pipelines, machine-learning code, workers, automation scripts, and other Python projects, and governs handwritten source, tests, examples, notebooks promoted to source, and configuration code. Frameworks and runtimes may add conventions, but the core style stays the same.

The emphasis is code style: names, expressions, functions, types, modules, and making behavior visible in source. File organization supports these goals without prescribing an application architecture. This is the Python companion to the TypeScript style guide and shares its principles: explicit contracts, validation at trust boundaries, typed state models, and automated enforcement.

## 1 Rules and scope

**Must**, **must not**, and imperative instructions are requirements. **Prefer** sets the default; depart from it only for a concrete improvement in clarity, correctness, or compatibility. **May** marks an explicitly permitted choice.

A project must record its supported Python versions, tooling, type checker, and narrow integration conventions in a short addendum (section 26). The addendum must not silently weaken the core rules.

Examples illustrate the rule under discussion and may omit surrounding imports and declarations. They assume the package is named `app` and lives under `src/app/`.

Generated and vendored files are exempt from formatting and naming rules: protobuf and gRPC stubs, tool-produced ORM migrations, OpenAPI clients, and copied third-party code. Keep them clearly identified, regenerate them from source, and do not edit them to imitate handwritten style. Handwritten wrappers and integration code remain subject to this guide.

Where this guide differs from PEP 8 or Google's Python guide, this guide wins inside the project. Where it is silent, follow PEP 8.

## 2 Required language baseline

New projects must declare their supported Python versions in `requires-python` and should target the newest stable CPython release their dependencies support. Libraries may support a range; applications and services run one pinned interpreter version.

Every handwritten module, including tests, scripts, and tooling, must be fully annotated and pass a static type checker in strict mode, run as a separate command (section 26). Python does not enforce annotations at runtime, so a program that runs proves nothing about its types.

Choose exactly one type checker per project and record it in the addendum: Pyright in strict mode or mypy with `strict = true`. Astral's ty may be adopted once the pinned version covers the project's typing features; record its version and any gaps.

Do not write:

- `TypeVar`, `ParamSpec`, or `TypeVarTuple` declarations where PEP 695 syntax works (`def first[T](...)`, `class Box[T]:`, `type Pair = ...`).
- `typing.List`, `Dict`, `Tuple`, `Set`, `Optional`, `Union`, `Text`, `TypeAlias`, or type-annotation comments (`# type: int`).
- Custom metaclasses. Use `__init_subclass__`, class decorators, or explicit registration.
- `exec`, `eval`, or `compile` on constructed source text.
- Mutation of `sys.path`, custom import hooks, or `importlib` tricks to reach modules the packaging does not expose.
- Monkeypatching of modules, classes, or builtins outside tests.
- `__del__` for cleanup of anything observable. Use context managers (section 18).
- Frame and bytecode inspection (`sys._getframe`, `inspect.currentframe`) in program logic.
- Name-mangled `__private` attributes. Use a single leading underscore.
- `global` statements. Use explicit state owned by an object or passed as a parameter.
- Star imports (`from module import *`).

Standard-library and framework features that use these mechanisms internally (`enum`, `dataclasses`, `abc`, Pydantic) are fine; the ban targets handwritten implementations.

Code must run correctly on free-threaded builds unless the addendum records a dependency that prevents it (section 19).

Features newer than the oldest supported version may be used through `__future__` imports and, for typing constructs, `typing_extensions`. `from __future__ import annotations` stores annotations as strings, which runtime introspection must resolve. Do not branch on `sys.version_info` for language features within the supported range.

Pin the interpreter, type checker, linter, and formatter through the lockfile and a dependency group. Upgrade deliberately and resolve new diagnostics. Do not rely on globally installed tools.

## 3 Formatting and source text

### Let the formatter own layout

Every project must use Ruff's formatter, commit the `[tool.ruff]` and `[tool.ruff.format]` policy from section 26 in `pyproject.toml` (formatter defaults do not replace this), and expose commands to format and check formatting. Do not combine it with Black, autopep8, or YAPF. The linter owns correctness and semantic style; disable lint rules that conflict with the formatter. Also require UTF-8 source without an encoding declaration and one final newline.

A trailing comma after the last element keeps one element per line. Add it when a call, signature, or literal is easier to review that way, and do not fight the formatter when it collapses a short one.

Line length is a wrapping preference, not a demand to split URLs, regular expressions, or meaningful string literals. Do not manually align columns or add `# fmt: off` for ordinary code. A narrowly scoped `# fmt: skip` is acceptable for a literal whose layout conveys information, such as a matrix, with a comment explaining why.

### Write clear source text

Keep one statement per line. Do not use semicolons or put a body on the same line as `if`, `for`, `while`, `with`, or `def`, even when it fits.

Do not use backslash continuation. Wrap inside parentheses, brackets, or braces, adding parentheses around an expression when needed, and prefer breaking at the highest syntactic level.

Use a single blank line between logical steps inside a function and no decorative comment banners.

Escape invisible or control characters in strings and explain them when their purpose is not apparent. Readable Unicode is allowed in strings and comments; identifiers must use ASCII.

### Strings

- Use double quotes, switching only to avoid escapes. Docstrings always use `"""`.
- Use f-strings for interpolation, not `+`, `.format()`, or `%`, except `%`-style logging arguments (section 21) and templates stored as data.
- Use `!r` or `=` in f-strings when the reader must see the exact value: `f"Unknown region: {region!r}"`.
- Accumulate pieces in a list and `"".join(...)` them, or write to `io.StringIO`, not `+=` in a loop.
- For long literals, use adjacent string literals inside parentheses on separate lines. Do not place two adjacent literals on one line: implicit concatenation there usually hides a missing comma, and the formatter silently joins them when they fit.
- Use `textwrap.dedent` for indented multiline text and raw strings for regular expressions.
- Use template strings (`t"..."`, PEP 750) only with a consumer that processes `Template` objects, such as an escaping SQL or HTML renderer. Never convert user input into an f-string to get templating.
- Keep `str` and `bytes` separate, decoding at the boundary with an explicit encoding.

### Executables

Only files meant to be executed directly carry a shebang: `#!/usr/bin/env python3`, or `#!/usr/bin/env -S uv run --script` for single-file scripts with inline metadata (PEP 723). An importable module that can also be executed puts its behavior in `main()` behind `if __name__ == "__main__":`.

## 4 Naming

Names must describe a value's domain meaning or an operation's effect. Avoid `data`, `info`, `item`, `obj`, `manager`, `helper`, `util`, and `handler` when a more specific term exists. Descriptiveness should grow with scope: `i` is fine in a three-line loop and wrong across a 40-line function. Do not abbreviate by deleting letters inside words (`cnt`, `usr_mgr`).

### Casing

| Kind                                                  | Public                                    | Internal                          |
| ----------------------------------------------------- | ----------------------------------------- | --------------------------------- |
| Packages and modules                                  | `snake_case` (`read_config.py`)           | `_snake_case` (`_wire_format.py`) |
| Distribution name in `pyproject.toml`                 | `kebab-case` (`job-runner`)               |                                   |
| Classes, exceptions, protocols, dataclasses, models   | `PascalCase` (`JobStore`)                 | `_PascalCase`                     |
| Type aliases and type parameters                      | `PascalCase` (`JobId`, `Item`)            | `_PascalCase`                     |
| Functions, methods, variables, parameters, attributes | `snake_case`                              | `_snake_case`                     |
| Module constants and `Final` class constants          | `CONSTANT_CASE`                           | `_CONSTANT_CASE`                  |
| Enum members                                          | `CONSTANT_CASE` (`JobStatus.IN_PROGRESS`) |                                   |

Casing follows a binding's role. Fixed module-level limits, lookup tables, compiled regular expressions, and default data use `CONSTANT_CASE` and a `Final` annotation. Module-level loggers (`logger`), type adapters, routers, and other service objects use `snake_case`. Local values are always `snake_case`, even when never reassigned.

A single leading underscore is the only privacy marker, and it is required: anything not part of a module's or class's contract starts with `_`. Do not invent dunder names (`__like_this__`).

### Meaning and consistency

- Treat acronyms as words in `PascalCase`: `HttpClient`, `JsonDecoder`, `UserId`. This deliberately departs from PEP 8 for consistency with the TypeScript guide. Preserve external names such as `json.JSONDecodeError`.
- Use verbs for operations and nouns for values, types, and properties.
- Name booleans as predicates (`is_ready`, `has_access`, `can_retry`, `should_persist`). Prefer positive booleans.
- Use plural names for collections (`accounts`, `account_ids`) and `<key>_to_<value>` or `<value>_by_<key>` for mappings (`price_by_sku`).
- Include units when ambiguity is possible: `timeout_s`, `delay_ms`, `size_bytes`, `ratio`, `percentage`.
- Use `on_<event>` for a caller-supplied callback and `handle_<event>` for the implementation responding to it.
- Use an `_async` suffix only to distinguish an intentional sync/async pair.
- Use descriptive type parameter names (`Key`, `Value`, `Item`). A single `T` is fine for one obvious relationship.
- Do not encode types in names (`name_str`, `user_dict`, `IJobStore`, `TJob`).
- Do not shadow builtins: `id`, `type`, `list`, `dict`, `input`, `filter`, `format`, `hash`, `max`, `min`, `next`, `sum`. Use `job_id`, `kind`, `values`.
- Name alternative constructors `from_<source>` (`Config.from_env`) and conversions `to_<target>` (`to_json`).
- Use `_` for intentionally discarded values. A required but unused parameter keeps its name and gets a `_` prefix only when the contract is positional; remove unused parameters when the contract allows it.
- Preserve external field names in the external contract (Pydantic aliases for `camelCase` JSON), translating once at the boundary.
- Alias imports only to resolve a collision, to clarify a generic external name, or for the community aliases in section 6.

Mathematically heavy code may use short names matching a cited paper or algorithm (`x`, `W`, `alpha`). Cite the source in a docstring or comment, and keep descriptive names on the public API.

## 5 Files, packages, and directories

### Use the src layout

Every project with more than one module uses the `src` layout: importable code under `src/<package>/`, with configuration, tests, and documentation at the root, so tests run against the installed package rather than the current directory's path. A single-file script may stand alone with PEP 723 inline metadata.

An illustrative arrangement, not required directories:

```text
pyproject.toml
uv.lock
src/app/__init__.py          # public API, explicit __all__
src/app/__main__.py          # `python -m app`: calls cli.main()
src/app/py.typed             # libraries only
src/app/cli.py               # parses arguments, composes, runs
src/app/config.py            # loads and validates settings
src/app/jobs/models.py       # job data and boundary schemas
src/app/jobs/planning.py     # pure planning behavior
src/app/jobs/_retry.py       # private to the jobs package
src/app/adapters/file_store.py
tests/jobs/test_planning.py
tests/conftest.py
notebooks/                   # exploration only, never imported
scripts/                     # one-off tooling, still type-checked
```

### File names

Never use dashes, spaces, or leading digits in module or package names: a file that cannot be imported cannot be tested. Use conventional names when they add information:

- Tests: `test_<module>.py`; shared fixtures: `conftest.py`.
- Entry points: `__main__.py`, `cli.py`, `main.py`.
- Private modules: `_<name>.py`.
- Type stubs for an untyped dependency: `<package>.pyi` under a project `typings/` directory.
- Platform-specific implementation: `file_store_posix.py`, `file_store_windows.py`.

A tool-required filename (`manage.py`, `noxfile.py`, Alembic's numbered migrations) is a narrow exception kept within that integration.

### Group code by ownership

Group by capability or domain before technical kind. A cohesive package can contain models, behavior, adapters, and private helpers. Do not require `services/`, `repositories/`, `controllers/`, or `models/` top-level packages. Start with flat modules and add subpackages when there are distinct responsibilities or consumers.

Keep a helper near its first owner, and move it to a shared module only once its responsibility and consumers are clear. Do not create catch-all `utils.py`, `helpers.py`, `common.py`, `misc.py`, or project-wide `types.py` or `constants.py` modules. A focused `types.py` within a small, cohesive package is acceptable.

### Keep modules cohesive

A module should have one primary responsibility and a small public API. Several related classes and functions belong together; do not impose one class per file. Keep a type or model with the behavior that owns it unless independent consumers need a separate dependency. Split when unrelated responsibilities, environment requirements, or import cycles make the boundary useful.

Keep `__init__.py` files small: a docstring, explicit re-exports, and `__all__`, or empty. Never put logic, I/O, or configuration in them.

Keep entry points thin: parse arguments, load configuration, compose dependencies, invoke behavior, translate the result into output and an exit code. Substantial logic belongs in modules callable without starting the application. Declare console commands in `[project.scripts]` rather than asking users to run file paths.

Tests live in a top-level `tests/` tree mirroring the package, with fixture data beside the tests that own it.

### Order code for reading

1. A shebang, only for executables.
2. The module docstring.
3. Imports.
4. `__all__`, when the module defines a public API.
5. Constants, type aliases, and models needed to understand the API.
6. Public functions and classes, with the private helpers that explain them nearby; a helper may follow the public function that calls it.
7. `if __name__ == "__main__":` calling `main()`, only in executable modules.

Keep related declarations together. Do not alphabetize definitions or collect unrelated types at the top for symmetry.

## 6 Imports and module boundaries

### Use absolute imports

Import first-party code by its full package path, never relative imports (`from . import planning`, `from ..config import load`). Absolute paths make every dependency greppable and behave the same in tests, scripts, and the installed package. Do not assume the current or script directory is on `sys.path`: `import planning` always means a top-level package, never a sibling file.

### Choose what to import

- Import a class, type, exception, dataclass decorator, or constant directly when its name is unambiguous: `from pathlib import Path`, `from app.jobs.models import Job`.
- Import the module when its functions only read clearly when qualified: `json.loads`, `subprocess.run`, `time.monotonic`, not a bare `loads` or `run`.
- Always import `typing` and `collections.abc` symbols directly: `from collections.abc import Iterator, Mapping`.
- Use `from package import module` for first-party modules whose functions you call often: `from app.jobs import planning`, then `planning.plan_job(...)`.
- Use these aliases and no others without an addendum entry: `numpy as np`, `numpy.typing as npt`, `pandas as pd`, `polars as pl`, `matplotlib.pyplot as plt`, `torch.nn as nn`, `torch.nn.functional as F`, `jax.numpy as jnp`.

### Order and placement

Put imports at the top of the module, after the docstring. Ruff's import sorter owns ordering: standard library, third-party, first-party, separated by one blank line and sorted within each group. Do not hand-sort or add comments between groups.

```python
import json
import logging
from collections.abc import Iterator
from pathlib import Path

import httpx
from pydantic import BaseModel, ConfigDict

from app.config import Settings
from app.jobs import planning
from app.jobs.models import Job, JobId
```

Function-local imports are allowed only to defer a heavy optional dependency in a rarely used path, to keep CLI startup fast, or inside a plugin hook that must not import at module load. Where available, prefer `lazy import heavy_module` at module scope (PEP 810).

Use `if TYPE_CHECKING:` imports only for names needed purely in annotations whose runtime import would create a cycle or load a heavy dependency. Such names need no quotes where annotations are evaluated lazily, natively or through `from __future__ import annotations`; quote them otherwise. Do not use the block to hide a real runtime cycle.

### Make dependency direction visible

Domain models and core behavior must not import entry points, CLI modules, web routes, or framework startup code. When core behavior needs an external capability, define a small `Protocol` beside the behavior and let the composition boundary supply an implementation.

Import another package only through its public API, never its `_private` modules or attributes. Declare every directly imported distribution in `pyproject.toml` rather than relying on a transitive dependency. Put optional integrations behind extras (`[project.optional-dependencies]`) and raise a clear `ImportError` naming the missing extra.

Do not create circular imports. Move the shared contract to its proper owner instead of hiding the cycle with function-local imports.

Declare `__all__` in every package `__init__.py` and any module consumed as a public API, listing names explicitly so API growth is visible in review.

### Limit import-time work

Importing a module must not open connections, read files or environment variables, start threads or tasks, configure logging, parse command-line arguments, register signal handlers, or call `sys.exit`. Import-time decorators must be cheap and cannot fail on valid input.

Construct pure constants, compiled regular expressions, enums, and models at module scope. Keep configuration reads, client construction, and lifecycle ownership at the composition boundary, usually `main()` or an application factory.

## 7 Variables and expressions

### Bind values deliberately

Rebind a local only when reassignment is part of the algorithm, and never to a value of a different type or concept.

Declare a binding close to its first use. Do not initialize a placeholder and assign it far away when an expression or helper can produce the value directly. A variable assigned inside a loop or `with` block and used after it must be bound on every path.

Use tuple unpacking when it makes a small set of values clearer (`host, port = address`), and starred unpacking (`first, *rest = items`) instead of manual slicing when the shape is the point.

### Compare precisely

- Use `is` and `is not` only for `None`, sentinel objects, and enum members, and `==` for values.
- Never compare to `True`, `False`, or `None` with `==`.
- Test booleans directly: `if is_ready:`.
- Test emptiness of strings and built-in collections with truthiness: `if not accounts:`, not `len(accounts) == 0`.
- Test an optional value explicitly: `if limit is None:`. Truthiness would also reject `0`, `""`, and empty collections.
- Compare numbers explicitly: `if count > 0:`, `if remainder == 0:`, not `if not count:`.
- Use chained comparisons for ranges: `0 <= index < size`. Every comparison with `nan` is false; check `math.isnan` when NaN is possible.
- Compare floats with `math.isclose` and an explicit tolerance. Use `Decimal` or integer minor units for money, never `float`.

### Prefer direct expressions

Use `x if x is not None else default` or an `if` statement when only `None` should trigger a fallback, and `value or default` only when every falsy value should be replaced.

Use explicit conversion: `int(text)`, `str(value)`, and `bool(flag)` only when truthiness conversion is the actual operation. Conversion is not validation: `int(" 42 ")` succeeds and `float("nan")` returns NaN. Parse external text through a model or explicit parser (section 14).

Use a conditional expression only for one small choice of value, with short parts, and never nest them.

Use the walrus operator `:=` only to avoid recomputing a value in a `while` condition, a comprehension filter, or an `if` testing a regular-expression match, never embedded in larger expressions.

Use parentheses when mixing `and` and `or`, or bitwise and comparison operators. Do not use `and`/`or` for side effects (`is_ready and start()`).

### Use modern standard-library types

- Use `pathlib.Path` for filesystem paths, not `os.path` string manipulation.
- Use timezone-aware datetimes: `datetime.now(UTC)`. Never create naive datetimes for instants; keep `date` for calendar dates.
- Use `time.monotonic()` or `time.perf_counter()` for durations, never wall-clock differences.

Do not use `eval`, modify builtins, or commit `breakpoint()` or `pdb` calls.

## 8 Control flow

### Make branches explicit

Prefer guard clauses when they reduce nesting. Do not add `else` or `elif` after an unconditional `return`, `raise`, `continue`, or `break` unless the paired structure clarifies a larger algorithm.

Avoid large compound conditions. Name an independently meaningful condition or extract a predicate function.

Do not use `for ... else` or `while ... else`, which most readers misread; use a flag, a helper that returns, or `next(..., None)`. An existing use may stay only with a comment stating when `else` runs.

### Pattern matching

Use `match` when dispatching on the variant of a closed union, an enum, or a value's structure, such as a parsed command or message, not as a replacement for a short `if`/`elif` chain of plain comparisons.

- Match classes with keyword patterns: `case Loaded(value=value):`. Positional patterns depend on `__match_args__` order and break silently when fields are reordered.
- Keep guards (`case Retry(attempt=n) if n > 3:`) short.
- For a closed union or enum, end with `case _: assert_never(value)` so a new variant fails type checking (section 13).
- For open external values, validate first or handle unknown values meaningfully. Do not add an empty `case _: pass`.
- A bare name in a pattern captures instead of comparing. Match constants through a dotted name (`case JobStatus.FAILED:`) or a literal.

### Loops

Iterate over values directly. Use `enumerate` when you need the index, `dict.items()` for keys with values, and `zip(..., strict=True)` whenever parallel inputs must have equal length, which is almost always. Never write `for i in range(len(items))` to index the same sequence.

Do not mutate a collection while iterating over it; iterate over a copy or build a new collection.

Use `continue` and `break` when they make the flow clearer, and `while True:` with an explicit `break` when the exit test sits in the middle.

Use `itertools` (`pairwise`, `batched`, `chain`, `groupby`, `islice`) instead of hand-written index arithmetic, and `itertools.batched(items, n, strict=True)` when a short final batch is a bug.

### Exceptions as control flow

Keep `try` bodies to the statements that can raise the handled exception, with the success path in `else`. Prefer EAFP when checking first would race, as with files and dictionary keys under concurrency, and an explicit check when the failure is common and cheap to detect.

## 9 Functions and API design

### Function form

Use `def` for every named function. Use `lambda` only inline for a trivial key or callback (`key=lambda job: job.priority`), never assigned to a name. Prefer `operator.attrgetter` and `operator.itemgetter` when they read as clearly.

Use module-level functions for stateless behavior, not `@staticmethod`. Nest a function only to close over local values, as in a decorator or callback, not to hide it, since nested functions cannot be tested directly.

### Parameters

Annotate every parameter and return type, including private helpers and `-> None`.

Make parameters keyword-only once a call would be unclear positionally: in practice, at most two positional parameters, then `*`. Boolean and optional configuration parameters must always be keyword-only, so `save(record, True, False)` cannot be written.

```python
def fetch_page(
    client: HttpClient,
    url: str,
    *,
    timeout_s: float = 10.0,
    follow_redirects: bool = True,
) -> Page: ...
```

Use positional-only parameters (`/`) when names are not part of the contract and may change, as in small generic utilities and dunder methods.

Never use a mutable default (`[]`, `{}`, `set()`, a dataclass instance) or a default computed by a call (`datetime.now(UTC)`, `uuid4()`), because defaults are evaluated once at definition. Use `None` and resolve inside, or an immutable value such as `()` or `frozenset()`.

Put defaults at the point that owns the policy, not threaded through layers. Do not use `None` to mean both "omitted" and "explicitly cleared" (section 11).

Accept `*args` and `**kwargs` only for genuine variadic APIs or transparent forwarding, typed precisely: `*args: P.args, **kwargs: P.kwargs` for forwarding, or `**options: Unpack[RenderOptions]` with a `TypedDict` for fixed keyword options. Do not accept `**kwargs: Any` to avoid listing parameters.

### Returns

Return one type of thing. A function that sometimes returns a list and sometimes a single item, or a value on one path and `None` by accident on another, is a bug.

Return a named object (a frozen dataclass or `NamedTuple`) for several related results. Use a plain tuple only for two values whose order is a convention callers already know, such as `divmod`.

Do not use sentinels such as `-1` or `""` for failure, or `None` to signal failure. `None` means legitimate absence (a lookup that found nothing); failure raises or returns a discriminated result (section 20).

Functions that mutate their argument or receiver return `None`, following the standard library (`list.sort`), not the mutated object for chaining.

### Generators and decorators

Use generators to stream large or unbounded sequences. Annotate them as `Iterator[T]` (or `Generator[Y, S, R]` when sending values), document what they yield, and do not hold resources across `yield` without a context manager that guarantees cleanup.

Use decorators when they remove real repetition or enforce an invariant. A decorator must use `functools.wraps`, preserve the signature with PEP 695 `**P` parameters, do no I/O when applied, and state in its docstring that it is a decorator. Test decorators directly.

### Responsibility and dependencies

Keep a function focused on one operation at a consistent level of abstraction. Extract a block when it has an independently useful name, hides a distracting detail, or needs a different dependency. There is no hard length limit; past roughly 40 lines, check whether a split would help.

Pass in what affects execution: clocks, random generators, HTTP clients, stores, and output streams. Reusable functions must not read `os.environ`, call `datetime.now()`, construct clients, or reach into module-level singletons.

Use `@overload` only when the return type depends on the argument types, with overloads adjacent and followed by one implementation. Prefer a union parameter when the return type does not change.

## 10 Type annotations

### Annotate contracts, infer locals

Annotate every class and dataclass field and every module or class constant, as well as signatures. Do not annotate locals whose type is obvious (`retry_count = 3`). Annotate a local when it starts empty (`seen: set[JobId] = set()`), should be wider than its first value, or explains an unclear contract.

### Write modern syntax

- Built-in generics: `list[int]`, `dict[str, Job]`, `tuple[int, ...]`, `type[Job]`.
- Unions with `|`: `int | str`, `Job | None`.
- Type aliases with the `type` statement: `type JobsById = Mapping[JobId, Job]`.
- Generic functions and classes with PEP 695 parameters and bounds:

```python
def first[Item](items: Sequence[Item]) -> Item | None:
    return items[0] if items else None


def retry[**Params, Result](
    operation: Callable[Params, Result],
) -> Callable[Params, Result]: ...


class Registry[Key: Hashable, Value]:
    def __init__(self) -> None:
        self._entries: dict[Key, Value] = {}
```

Variance is inferred for PEP 695 parameters; do not declare it by hand.

### Choose parameter and return types

Accept the most general abstract type the function needs, from `collections.abc`: `Iterable`, `Sequence`, `Mapping`, `Collection`, `Callable`. Read-only types such as `Sequence` and `Mapping` also document and enforce non-mutation. Return a concrete type (`list[Job]`) or a deliberately abstract one when callers must not depend on the implementation.

Use `Self` for methods returning an instance of the receiver's class, including alternative constructors, and `ClassVar` for class-level attributes.

### Use precise types

- Use `object` for a value of any type that will be narrowed before use, the role TypeScript's `unknown` plays.
- Do not use `Any`, which disables checking for everything it touches. Convert a third-party `Any` to `object` or validate it at the adapter boundary (section 14). The only standing exception is a precise generic signature whose relationship the type system cannot express, with a comment.
- Use `Literal["json", "text"]` for a small type-only vocabulary (section 13).
- Use `NewType` for identifiers that are easy to confuse: `JobId = NewType("JobId", str)`.
- Use `TypedDict` (with `NotRequired` and `ReadOnly` as needed) only for dict-shaped data you do not own, such as `**kwargs` options or a passthrough JSON object.
- Use `Protocol` for structural dependencies, such as "anything with `save(job) -> None`", and an `ABC` with `@abstractmethod` only for a nominal hierarchy sharing real implementation.
- Use `Callable[[Job], bool]` for simple callbacks and a `Protocol` with `__call__` when parameter names, keywords, or overloads matter.
- Use `Never` for a function that cannot return and `None` for one whose result callers must ignore.
- Do not use `AnyStr` unless the function genuinely preserves either `str` or `bytes`.

### Keep the type system honest

Decorate every overriding method with `@override` (from `typing`), so the checker fails when the base method is renamed or removed.

Use generics to preserve real input-output relationships. Do not add a type parameter used only once, solely to let callers assert a return type, or to make a fixed domain model look configurable.

Do not expose large nested `Callable`, `Union`, or generic expressions. Name them with a `type` alias, and test reusable type-level constructs with `assert_type` (section 23).

Libraries ship a `py.typed` marker. Untyped dependencies get focused `.pyi` stubs in `typings/` covering only what the project uses, or a typed adapter module. Do not silence a missing-stubs diagnostic for the whole project.

## 11 Absence and optional values

`None` is the single absence value, declared explicitly as `T | None`; a parameter typed `str` with a `None` default is a type error. Add `None` only at the boundary where absence occurs, not by default.

Use `mapping[key]` when the key must exist, so a missing key fails loudly, and `mapping.get(key)` only when absence is expected, handling the `None`. Do not use `getattr(obj, "name", None)` to step around the type checker.

When `None` is itself a valid value, distinguish "not supplied" with a typed sentinel, not `object()`:

```python
from enum import Enum
from typing import Final, Literal


class _Unset(Enum):
    UNSET = "UNSET"


UNSET: Final = _Unset.UNSET


def update_job(
    job_id: JobId,
    *,
    label: str | Literal[_Unset.UNSET] = UNSET,
    owner: UserId | Literal[_Unset.UNSET] | None = UNSET,
) -> None:
    """Updates a job. UNSET leaves a field unchanged; owner=None clears it."""
```

Treat omission, clearing, and supplying a value as separate operations when the domain needs them, and document which is which. In Pydantic models, use `model_fields_set` or `exclude_unset=True` to tell an omitted field from an explicit `null`.

Prefer an empty collection to `None` when "nothing" and "not provided" mean the same thing. Do not use several optional fields for mutually exclusive states; use a discriminated union (section 13).

## 12 Narrowing and escape hatches

### Narrow before use

Prefer checks the type checker understands: `isinstance`, `is None`, comparisons with `Literal` or enum discriminants, `match` patterns, and control flow. Validate external data with a model; no annotation or `cast` can establish the shape of parsed JSON, an environment variable, or a third-party response.

Do not write:

- `cast(...)` to make a mismatched value fit.
- `Any` as a laundering step (`value: Any = response; job: Job = value`).
- `assert` to narrow types or validate input in production code, or to report errors. `python -O` removes asserts, so code that depends on them changes behavior.
- A bare `# type: ignore` or `# pyright: ignore`, or one without a reason.
- `hasattr` checks in place of a `Protocol` or real validation.

Use a checked invariant instead of an assert:

```python
def require[Value](value: Value | None, message: str) -> Value:
    if value is None:
        raise ValueError(message)
    return value
```

`assert` remains correct in tests, and for internal invariants whose removal would not change behavior.

### Keep escape hatches narrow

A single `cast` is allowed only when a documented invariant or external declaration limitation establishes the relationship and the checker cannot express it. Keep it at the smallest adapter boundary, state the invariant in a comment, and cover the behavior with a test. "The checker complains" is not a reason.

A type-ignore comment must name the rule and give a reason: `# pyright: ignore[reportPrivateUsage]  # fixture inspects cache state` or `# type: ignore[attr-defined]  # stub lacks Client.timeout, fixed upstream in 2.4`. Configure the checker to report unnecessary ignores so stale ones fail.

### Write honest type guards

The checker trusts a `TypeIs` or `TypeGuard` function without verifying it. A custom guard must check every property it promises and have tests for rejection as well as acceptance. Prefer `TypeIs`, which narrows both branches, over `TypeGuard`.

```python
def is_terminal(status: JobStatus) -> TypeIs[Literal[JobStatus.SUCCEEDED, JobStatus.FAILED]]:
    return status in (JobStatus.SUCCEEDED, JobStatus.FAILED)
```

Use `@runtime_checkable` protocols with `isinstance` only at a boundary and as a coarse check: it verifies that attributes exist, not their signatures or types.

Filter out `None` with an explicit comprehension (`[label for label in labels if label is not None]`), not `filter(None, ...)`, which also drops valid falsy values.

## 13 Finite values and state models

### Use enums for runtime vocabularies

Unlike TypeScript's, Python's `enum` is a sound, ordinary runtime construct, and this guide requires it for named vocabularies that exist at runtime.

- Use `StrEnum` for string values that cross a boundary (JSON, databases, CLI flags, metrics labels), with values spelled exactly as the contract requires.
- Use `Enum` with `auto()` for internal vocabularies whose values never leave the process.
- Use `IntEnum` or `IntFlag` only when an external protocol defines integers or bit flags.
- Decorate enums with `@unique`. Do not subclass enums with members or add mutable state to them.
- Use `Literal[...]` for a small vocabulary that exists only in signatures, such as `mode: Literal["read", "write"]`.

```python
from enum import StrEnum, unique


@unique
class JobStatus(StrEnum):
    QUEUED = "queued"
    IN_PROGRESS = "in-progress"
    SUCCEEDED = "succeeded"
    FAILED = "failed"
```

Refer to `JobStatus.QUEUED` everywhere. Do not repeat `"queued"` through the implementation or keep a parallel `Literal` of the same values. Convert external text once with `JobStatus(raw)` or a model field typed `JobStatus`.

### Model valid states directly

When a value can be in one of several states, give each state its own frozen dataclass holding exactly the data that state needs, and join them with a `type` alias.

```python
from dataclasses import dataclass
from typing import assert_never


@dataclass(frozen=True, slots=True, kw_only=True)
class Idle:
    pass


@dataclass(frozen=True, slots=True, kw_only=True)
class Loading:
    started_at: datetime


@dataclass(frozen=True, slots=True, kw_only=True)
class Loaded[Value]:
    value: Value


@dataclass(frozen=True, slots=True, kw_only=True)
class Failed:
    error: Exception


type LoadState[Value] = Idle | Loading | Loaded[Value] | Failed


def describe_load_state(state: LoadState[object]) -> str:
    match state:
        case Idle():
            return "Not started"
        case Loading():
            return "Loading"
        case Loaded():
            return "Loaded"
        case Failed(error=error):
            return str(error)
        case _:
            assert_never(state)
```

`assert_never` makes a new variant fail type checking at every unhandled match, and raises `AssertionError` at runtime for an invalid value, independent of `python -O`. Do not replace it with a generic fallback that silently accepts a new state. Avoid classes with several booleans and unrelated optional fields that permit impossible combinations (`is_loading`, `is_failed`, `error: str | None`, `value: X | None`).

### Validate discriminated unions with a tag

When a union crosses a boundary, give every variant a `kind` field typed with a single enum member and let Pydantic select the variant from it instead of trying each one.

```python
from enum import StrEnum, unique
from typing import Annotated, Literal

from pydantic import BaseModel, ConfigDict, Field


@unique
class EventKind(StrEnum):
    CREATED = "created"
    CANCELLED = "cancelled"


class _JobEventModel(BaseModel):
    model_config = ConfigDict(frozen=True, extra="forbid")


class JobCreated(_JobEventModel):
    kind: Literal[EventKind.CREATED]
    job_id: JobId
    label: str


class JobCancelled(_JobEventModel):
    kind: Literal[EventKind.CANCELLED]
    job_id: JobId
    reason: str | None = None


type JobEvent = Annotated[JobCreated | JobCancelled, Field(discriminator="kind")]
```

Recursive unions work without special handling: refer to the alias inside a variant, and Pydantic resolves it lazily.

### Keep lookup tables complete

Python has no `satisfies`, so a mapping keyed by an enum cannot be checked for completeness statically. For branching behavior, prefer a function with an exhaustive `match`. For a fixed data table, annotate it `Final[Mapping[JobStatus, str]]`, wrap it in `MappingProxyType` (or `frozendict` where available), and add a test asserting that its keys equal `set(JobStatus)`.

## 14 Data models and runtime boundaries

### Choose one tool per job

| Use                                                                                                                 | Tool                                                                |
| ------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| Internal value objects, domain records, state variants, results                                                     | `@dataclass(frozen=True, slots=True, kw_only=True)`                 |
| Data crossing a trust boundary: config, env, CLI input, HTTP bodies, files, messages, third-party and LLM responses | Pydantic `BaseModel` (v2 API), or `TypeAdapter` for non-model types |
| Settings from environment and files                                                                                 | `pydantic-settings` `BaseSettings`, read once at startup            |
| A small tuple-shaped return that callers unpack                                                                     | `NamedTuple`                                                        |
| Dict-shaped data you do not own and pass through                                                                    | `TypedDict`                                                         |

Do not use plain `dict[str, Any]` for structured data. Do not mix attrs, dataclasses, and Pydantic for the same role without an addendum entry. Do not use Pydantic models as mutable internal state containers; when the domain shape differs from the wire shape, convert the parsed model to a domain dataclass once.

Enforce dataclass invariants in `__post_init__`, raising `ValueError` with a precise message. Use `dataclasses.replace` to derive a changed copy.

### Validate where trust changes

Treat raw external input as `object` (or `bytes`/`str` before parsing) even when a library hands you `Any`. Parse it once at the boundary and pass typed objects inward without re-validating in internal helpers. Validate again only when crossing a new trust boundary or when external state can invalidate the guarantee. A model establishes only the constraints it checks, not authorization, ownership, or every domain invariant.

```python
from typing import Annotated

from pydantic import BaseModel, ConfigDict, Field, StringConstraints


class CreateJob(BaseModel):
    model_config = ConfigDict(frozen=True, extra="forbid", strict=True)

    label: Annotated[str, StringConstraints(strip_whitespace=True, min_length=1, max_length=120)]
    retry_count: Annotated[int, Field(ge=0, le=10)] = 3


def parse_create_job(body: bytes) -> CreateJob:
    return CreateJob.model_validate_json(body)
```

Prefer `model_validate_json` for raw JSON text: it parses and validates in one pass and, under `strict=True`, accepts JSON representations such as ISO datetime strings, which `json.loads` followed by `model_validate` rejects. Continue with the returned model, never the original input: validation applies defaults, strips whitespace, and converts types.

### Choose strictness intentionally

- Commands, requests, and configuration whose field set the project owns: `extra="forbid"`.
- External responses where additional fields are expected and harmless: `extra="ignore"` (Pydantic's default), stated explicitly in `model_config`.
- Contracts that intentionally preserve extension fields: `extra="allow"`, documented on the model.
- Use `strict=True` where silent coercion would hide malformed input, such as `"3"` becoming `3` in a JSON API. Lax mode is acceptable for text-only sources such as environment variables and query strings.

Define a base model per boundary with these settings rather than repeating them, but keep each model's strictness visible in review.

### Keep validators small

Use `Annotated` constraints first, then `field_validator` and `model_validator` for rules constraints cannot express. Validators raise `ValueError` with a precise message, and must not perform I/O, logging, or database lookups.

A default handles intentional absence only; do not use defaults to conceal invalid required configuration.

### Read the environment in one place

Keep environment access in the configuration module. Read it once at startup into a frozen settings object and pass that object, or its values, inward. Do not call `os.environ` or `os.getenv` anywhere else.

### Serialize deliberately

Define transport representations for values without one obvious JSON form:

- Instants as timezone-aware ISO 8601 strings in UTC, or integer Unix milliseconds; pick one per contract and never mix them. A calendar `date` is not an instant.
- Durations with units in the field name (`timeout_ms`) or as ISO 8601 durations.
- `Decimal` as a string. Enums by value.

Use `model_dump(mode="json")` or `model_dump_json()`, not `json.dumps(..., default=str)`, which hides unserializable values. Construct intentional response models rather than dumping internal records or ORM objects into public responses.

Never `pickle`, `marshal`, or `shelve` data that crosses a trust boundary or must survive a version upgrade; unpickling untrusted bytes executes code. Use `yaml.safe_load`, never `yaml.load`.

## 15 Collections, comprehensions, and iteration

### Choose the appropriate structure

- `list` for ordered, homogeneous values that grow or change.
- `tuple` for fixed-size records and immutable sequences (`tuple[str, ...]`).
- `set` and `frozenset` for membership and uniqueness. Do not test membership in a large list in a loop.
- `dict` for mappings; `Counter`, `defaultdict`, and `deque` when their behavior is the point.
- A dataclass or model, not a dict, for a known record shape.

Use literals `[]`, `{}`, `()`, not `list()`, `dict()`, or `tuple()` without arguments. Use `dict(...)` with keyword arguments only when it reads better. Use `set()` for an empty set, since `{}` is a dict.

Iterate dicts directly (`for key in prices:`, `if key in prices:`), not through `.keys()`. Merge with `|` and update with `|=` when precedence is clear.

### Comprehensions

Use a comprehension to build a collection from one iterable with at most one `for` clause and one `if` filter: `[job.id for job in jobs if job.status is JobStatus.IN_PROGRESS]`. Nested loops, multiple filters, or a conditional expression in the element become a loop or named helper. Never use a comprehension for side effects (`[log(job) for job in jobs]`).

Use a generator expression when the result is consumed once, especially by `sum`, `any`, `all`, `min`, `max`, or `"".join`, without wrapping it in `[...]` first.

Do not use `map` or `filter` with a `lambda`; write the comprehension. `map` with a named function (`map(str.strip, lines)`) is acceptable.

### Sort, copy, and stream carefully

Use `sorted(values)` to get a new list and `values.sort()` only on a list you own. Supply `key=` rather than a comparison function. State locale and case rules when sorting human text.

Copying is shallow: `list(values)`, `values.copy()`, `dict(mapping)`, and `{**mapping}` copy only the outer container. Use `copy.deepcopy` only when the nested data is owned and small.

Use `itertools` and `functools` for well-known operations instead of reimplementing them. Prefer a clear loop over `functools.reduce` unless the reduction is a standard operation like `operator.or_`.

For large or unbounded data, stream with iterators and generators rather than materializing lists, and document whether a function consumes its iterable.

## 16 Mutation and ownership

Prefer pure functions for calculations, transformations, validation, and domain rules: inputs in, a new value out.

Do not mutate arguments or caller-owned collections unless that is the function's explicit, named purpose (`append_audit_entry`, `normalize_in_place`), and document what it changes.

Local mutation is fine for a newly created accumulator, a loop variable, or an object's private state. Do not replace a simple owned loop with repeated copying to avoid mutation syntax.

### Make immutability visible

- Use frozen dataclasses and frozen Pydantic models for values that should not change after construction. They are shallowly frozen, so use `tuple` and `frozenset` fields, not `list`.
- Declare read-only parameters and returns as `Sequence`, `Mapping`, `AbstractSet`, or `Collection`.
- Expose fixed collections as `tuple`, `frozenset`, `MappingProxyType`, or `frozendict` where available.
- Annotate constants `Final`, which prevents rebinding in checked code but does not freeze the object.

### Avoid shared mutable state

Do not keep mutable state at module level. When a process-wide registry or cache is genuinely needed, make it private (`_registry`), expose it only through functions, explain the design in a comment, and make it safe under concurrent access.

Do not define mutable class attributes as defaults (`items: list[str] = []` on a plain class), which every instance would share. Initialize per-instance state in `__init__` or with `field(default_factory=...)`, also for dataclass collection fields.

Do not export a mutable collection for callers to change directly. Return a copy, expose a read-only view with a clear ownership contract, or provide explicit operations.

### Cache deliberately

Use `functools.cache` and `lru_cache` only on pure module-level functions with hashable, immutable arguments, not on methods, where they keep every `self` alive. Use `functools.cached_property` for per-instance lazy values, not on objects meant to be immutable or shared across threads without a lock.

Avoid duplicated state that can be derived cheaply from authoritative values. If a cache or snapshot is necessary, define who updates it and when it becomes invalid.

## 17 Classes

Use a class when identity, lifecycle, encapsulated state, or a framework contract benefits from it, dataclasses for data, and module functions for stateless operations. Do not create classes holding only static methods or constants; use a module.

### Fields and construction

Declare every instance attribute, with its type, in `__init__` or as a dataclass field, never later in other methods or from outside the class. Use `slots=True` on dataclasses unless a dependency needs `__dict__`.

Keep `__init__` synchronous, cheap, and focused on establishing a valid object: no network calls, file reads, thread starts, or event-loop work. Use an explicit factory, such as an `async` classmethod named `create` or `connect`, when construction needs I/O.

Use `@classmethod` for alternative constructors (`from_env`, `from_path`) returning `Self`, not for general behavior that does not need the class.

### Visibility

Do not access another object's `_private` members, except in tests that deliberately inspect internal state, with a comment.

Do not write trivial getters and setters. Expose a public attribute, a read-only `@property`, or a meaningful method.

Use `@property` only for cheap, side-effect-free, unsurprising reads, named as nouns: no I/O, locking, or substantial computation. Prefer an explicit method to a property setter that triggers other behavior. Do not use properties for computations a subclass may need to override and extend.

### Inheritance

Prefer composition, and inherit only for a genuine substitution relationship. Call `super().__init__(...)` in subclasses that define `__init__`. Do not use multiple inheritance of stateful classes; limit it to protocols, ABCs without state, and documented framework mixins.

### Special methods

Implement `__repr__` on non-dataclass types that appear in logs or test failures. Define `__eq__` and `__hash__` together, or neither. Overload operators only for types with an established algebra, such as vectors, money, or units, never as shorthand for unrelated behavior.

## 18 Resources and context managers

Acquire every closable resource with `with` or `async with`: files, sockets, HTTP clients, database connections and sessions, locks, temporary directories, subprocesses, thread and process pools, GPU or device contexts. Do not rely on garbage collection or `__del__`; CPython's reference counting is an implementation detail, and free-threaded and alternative runtimes collect later.

- Use `contextlib.ExitStack` or `AsyncExitStack` for a variable number of resources, and `contextlib.closing` for objects with `close()` but no context-manager support.
- Write simple context managers with `@contextmanager` and `try`/`finally` around the `yield`. Write a class with `__enter__`/`__exit__` (or `__aenter__`/`__aexit__`) when the object owns a resource and has its own API.
- Use `contextlib.aclosing` for async generators that hold resources, so they close even when iteration stops early.
- Group several resources in one parenthesized `with` statement instead of nesting or backslashes.
- Where a resource's lifetime cannot be scoped by `with`, document who closes it and when.

Long-lived clients (an `httpx.Client`, a database engine, a model loaded onto a GPU) are created once at the composition boundary, passed to the code that uses them, and closed at shutdown, not created per call.

### Files

Open text files with an explicit `encoding="utf-8"` (or the format's documented encoding), and binary files with `"rb"`/`"wb"`. Prefer `Path.read_text`, `Path.write_text`, and `Path.open`. Write files others read concurrently atomically: write to a temporary file in the same directory, then `os.replace` it into place. Use `tempfile` for scratch files and directories.

### Subprocesses

Use `subprocess.run([...], check=True, timeout=...)` with an argument list. Never use `shell=True` with any interpolated value or build a command line by string concatenation. Capture output with `capture_output=True, text=True` when needed. Resolve executables with `shutil.which` when the path is not fixed.

## 19 Concurrency and async

### Choose the model deliberately

- `asyncio` for I/O-bound concurrency with async-native libraries.
- Threads (`concurrent.futures.ThreadPoolExecutor`) for blocking I/O libraries, or CPU-bound work on a free-threaded build.
- Processes (`ProcessPoolExecutor`, or `InterpreterPoolExecutor` for isolated subinterpreters) for CPU-bound work on GIL builds.
- A durable job queue when work must survive the process.

Record the choice in the addendum. Do not mix models within one component without a reason, and never hide an event loop inside a library function.

### Own every task

Every coroutine must be awaited or handed to an owner. Use `asyncio.TaskGroup` for concurrent work: it waits for all tasks, cancels the rest when one fails, and raises the failures as an `ExceptionGroup`. Bound concurrency for collections of unknown size with a `Semaphore`.

```python
async def fetch_all(
    client: HttpClient,
    urls: Sequence[str],
    *,
    max_concurrency: int = 10,
) -> list[Page]:
    limiter = asyncio.Semaphore(max_concurrency)

    async def fetch_one(url: str) -> Page:
        async with limiter:
            return await client.get_page(url)

    async with asyncio.TaskGroup() as group:
        tasks = [group.create_task(fetch_one(url)) for url in urls]

    return [task.result() for task in tasks]
```

Do not call `asyncio.create_task` and drop the result; the event loop keeps only a weak reference, so the task can be garbage-collected mid-flight and its exception lost. Background work belongs to a long-lived `TaskGroup` owned by the application's lifecycle, or to a durable job system when completion matters.

Prefer `TaskGroup` to `asyncio.gather`. Use `gather(..., return_exceptions=True)` only when every outcome is needed, and inspect every result.

### Respect cancellation and time

Wrap operations that wait on the outside world in `asyncio.timeout(...)` or pass an explicit timeout to the client. Never swallow `asyncio.CancelledError`: if you catch it to clean up, re-raise it. Use `finally` or `async with` for cleanup, and do not `await` long operations inside `finally` without a timeout.

Handle exception groups with `except*`, one clause per exception type you can act on.

### Do not block the loop

Inside `async def`, never call blocking functions: `time.sleep`, `requests`, synchronous database drivers, `subprocess.run`, or large synchronous file reads. Use async equivalents or `await asyncio.to_thread(...)`. Do not pass an `async` function to an API that ignores the returned coroutine, such as a synchronous callback registry.

Call `asyncio.run` only at the program's entry point. Libraries expose `async` APIs and let the caller own the loop.

### Threads and shared state

Do not rely on the global interpreter lock or built-in operations for atomicity; it does not hold on free-threaded builds and never held for compound operations like check-then-set. Communicate between threads with `queue.Queue`, or protect shared state with a `threading.Lock` held through `with`. Keep critical sections small and never call unknown code while holding a lock.

### Retries

Retries must be bounded, limited to failures known to be transient, delayed with backoff and jitter, and explicit about repeated side effects. Do not retry non-idempotent operations without an idempotency key, or automatically retry cancellation. Put the retry loop at the layer that owns the policy, not hidden in a small data-access helper.

## 20 Errors and exceptions

### Raise specific errors

Raise instances of the most specific fitting type, using built-ins where they fit: `ValueError` for a bad value of the right type, `TypeError` for a wrong type, `KeyError` and `LookupError` for missing keys, `TimeoutError`, `PermissionError`, `FileNotFoundError`. Never raise `Exception` or `BaseException` directly.

Error messages identify the failed operation and the relevant values, clearly delimited and easy to grep: `f"Unknown job status: {raw!r}"`. A message must match the actual condition, not guess at a cause the code has not established. Never include credentials, tokens, personal data, or full request payloads.

Do not raise in `__init__` of a value type for anything other than an invalid argument.

### Custom exceptions

Create a custom exception when callers need to distinguish a failure category or need structured data about it. Each package that raises its own errors defines one base class from which every custom error derives, so callers can catch the package's failures in one clause.

- Names end in `Error` and do not repeat the module (`jobs.QuotaExceededError`, not `jobs.JobsQuotaError`).
- The class docstring says what the error represents, not when it is raised.
- Structured context is stored as typed, read-only attributes passed by keyword. Callers must never parse the message.
- Also inherit from a built-in when the error is a kind of it (`class ConfigError(AppError, ValueError)`), so generic handlers still work.

```python
class AppError(Exception):
    """A failure raised by the app package."""


class QuotaExceededError(AppError):
    """An account has used its job quota for the period."""

    def __init__(self, *, account_id: AccountId, limit: int) -> None:
        super().__init__(f"Job quota exceeded for account {account_id!r} (limit {limit})")
        self.account_id: Final = account_id
        self.limit: Final = limit
```

### Preserve causes

When translating an exception, always chain it: `raise StoreUnavailableError(...) from error`. Use `from None` only when the original is irrelevant implementation detail and hiding it helps the reader, saying why in a comment. Use `error.add_note(...)` to add context while re-raising the same exception.

### Catch where the code can act

Catch the narrowest type, and only to recover, translate at a module or service boundary, add context, retry safely, or clean up.

- Never write a bare `except:`.
- Catch `Exception` only at isolation points: the top-level handler of a CLI or request, a worker loop, a thread's outermost function. Log it with `logger.exception` and translate it into a safe outcome.
- Catch `BaseException` only to clean up and re-raise.
- Never write `except ...: pass`. When ignoring an error is intended, use `contextlib.suppress(SpecificError)` with a comment saying why.
- Do not represent failed work as success or return an undocumented placeholder.

### Expected outcomes

When callers must branch on an expected alternative, return a discriminated result instead of raising:

```python
@dataclass(frozen=True, slots=True, kw_only=True)
class Reserved:
    reservation_id: ReservationId


@dataclass(frozen=True, slots=True, kw_only=True)
class Rejected:
    reason: Literal["unavailable", "limit-exceeded"]


type ReserveResult = Reserved | Rejected
```

Use exceptions for failures that should interrupt the operation. Keep one convention per API for a given condition. Do not introduce a general-purpose `Result` monad library.

### Report at boundaries

Report a failure at the layer that owns the response or lifecycle; do not log and re-raise the same error at every layer. Translate internal errors into stable external responses: HTTP status and error code, CLI message and exit code. Do not expose tracebacks, internal exception messages, or provider error objects to clients.

CLI entry points convert failures into a message on standard error and a non-zero exit code by returning it from `main()` or raising `SystemExit`. Only the entry point calls `sys.exit`.

Use `warnings.warn(..., DeprecationWarning, stacklevel=2)` for deprecations in libraries, not log messages.

## 21 Logging and output

Each module that logs creates one module-level logger: `logger = logging.getLogger(__name__)`. Do not log through the root logger (`logging.info(...)`) or pass loggers as parameters.

Pass a constant message with `%`-style placeholders and the values as arguments, as in `logger.info("Job %s finished in %.1f s", job.id, elapsed_s)`, not f-strings or `.format()`. The constant pattern lets log systems group identical events and skips formatting when the level is disabled.

Use `logger.exception(...)` inside an `except` block to include the traceback. Use levels consistently:

| Level      | Meaning                                                           |
| ---------- | ----------------------------------------------------------------- |
| `DEBUG`    | Detail useful only while diagnosing a specific problem            |
| `INFO`     | Normal lifecycle events an operator may want to see               |
| `WARNING`  | Something unexpected that the program handled; may need attention |
| `ERROR`    | An operation failed and its result is lost or degraded            |
| `CRITICAL` | The process cannot continue                                       |

Only the application entry point configures logging (`logging.config.dictConfig`, handlers, formatters, levels). Libraries and reusable modules never call `basicConfig`, add handlers, or change levels. Services emit structured logs (JSON with stable field names), through `extra=` or a structured-logging library recorded in the addendum.

Never log secrets, tokens, passwords, full request or response bodies, or personal data. Redact at the source, not with a downstream filter.

### Program output

Only CLI and script entry points write to standard output and standard error: intended output (results, data for piping) to stdout, and diagnostics, progress, and errors to stderr. Reusable modules never call `print`; they return values or log. Do not leave debug `print` calls in committed code.

Keep machine-readable output stable: a `--json` schema is part of the contract.

## 22 Comments and docstrings

### Comments

Write code whose names and structure explain the ordinary path. Use comments for constraints, reasons, invariants, trade-offs, and non-obvious interoperability behavior. Do not narrate what a statement already says. Assume the reader knows Python but not why you made this choice.

Write comments as sentences with normal capitalization and punctuation, in English unless the addendum says otherwise. Place inline comments two spaces after the code. Remove commented-out code and stale explanations.

A TODO names the tracked issue and the missing work: `# TODO(https://tracker.example/JOB-412): Batch writes once the store supports transactions.` Do not write anonymous `# TODO: fix later` or attribute TODOs to a person. A permanent workaround explains the constraint and links its source.

### Docstrings

Use Google-style docstrings (PEP 257). Every public module, class, function, and method has one; private code gets one when its behavior is not obvious from its name and signature. Test modules and functions need them only for unusual setup.

- The first line is a one-line summary ending in a period, with more detail after a blank line.
- Use descriptive mood consistently within a project ("Returns the active jobs."). Describe properties and attributes as nouns ("The job's queue name.").
- Describe the contract: what callers pass, what they get, what can fail, side effects, mutation, units, ordering, thread-safety, cancellation, and lifecycle obligations. Leave implementation details to comments.
- Do not repeat types; the annotations carry them.
- Use `Args:`, `Returns:` (or `Yields:`), and `Raises:` sections when they add information beyond the summary. List in `Raises:` only exceptions that are part of the contract, not those caused by misuse.
- A class docstring describes what an instance represents and lists public attributes under `Attributes:`.
- A method decorated with `@override` needs no docstring unless it changes or extends the base contract.
- A decorated function documents its behavior after decoration.

```python
def plan_job(job: Job, *, now: datetime, capacity: Capacity) -> JobPlan:
    """Plans when and where a queued job should run.

    The plan respects the job's deadline and never overcommits capacity.

    Args:
        job: A job in the QUEUED state.
        now: The current instant; must be timezone-aware.
        capacity: Free worker slots per region at `now`.

    Returns:
        A plan with a start time no earlier than `now`.

    Raises:
        QuotaExceededError: The job's account has no quota left for the period.
    """
```

Keep docstring examples small and valid. Either run them as doctests in CI or move them into tests.

## 23 Tests and fixtures

Use pytest for all new tests, written as plain functions with plain `assert`, not new `unittest.TestCase` classes. Tests follow the same naming, formatting, typing, and async rules as production code.

### Structure

Name test functions by observable behavior and the condition that matters: `test_returns_no_jobs_when_queue_is_empty`, not `test_plan_job_2` or `test_helper`. Group related tests in a module, or a plain class when they share fixtures.

Keep setup, action, and assertion easy to distinguish with blank lines. A test exercises one coherent behavior, which may need several assertions.

Use `@pytest.mark.parametrize` for the same behavior over several inputs, with readable `ids`, rather than looping inside one test.

Assert failures precisely: `with pytest.raises(QuotaExceededError, match="quota exceeded"):`, and check structured attributes when they matter.

### Fixtures and isolation

- Build small, realistic, typed fixtures. Construct real dataclasses and models; never fake them with `MagicMock()` or `cast`.
- Use function-scoped fixtures by default, widening scope only for expensive, immutable resources.
- Keep fixtures in the test module or nearest `conftest.py`, not one global `conftest.py`.
- No test may depend on another's state or on execution order. Tests must pass in random order and in parallel.
- Use `tmp_path` for files and `monkeypatch` for environment variables and process-level settings.

### Control the boundaries

Pass fakes for injected dependencies, preferring a small in-memory fake implementing the `Protocol` over patching. When patching is unavoidable, patch where the name is looked up, use `autospec=True` or `create_autospec`, and patch only boundaries, never the unit's own helpers.

Seed randomness explicitly. Do not use `time.sleep` to wait; wait on an event or advance a controllable clock. Block real network access in unit tests.

Assert returned values, state changes, and raised errors. Avoid asserting internal call sequences unless the sequence is the public contract.

### Async, types, and properties

Run async tests through the project's single async plugin (pytest-asyncio or AnyIO's pytest plugin), recorded in the addendum. A passing test must not leave tasks running.

Write type-level tests with `typing.assert_type` in checked files. A negative type test carries an explanation and the specific rule:

```python
assert_type(first([1, 2]), int | None)

first(42)  # pyright: ignore[reportArgumentType]  # first() rejects non-sequences
```

Use property-based tests (Hypothesis) for parsers, serializers, and invariants with large input spaces.

### Configuration

Run pytest with `--strict-markers`, `--strict-config`, `xfail_strict = true`, and warnings as errors (`filterwarnings = ["error"]`), with narrow, commented exceptions for third-party warnings. Register every custom marker. Mark slow or integration tests and keep the default run fast.

Use snapshots only when reviewing the full representation is meaningful. Treat coverage as a signal of untested code, not a target.

## 24 Numerical, data, and notebook code

Applies wherever a project uses NumPy, pandas, Polars, PyTorch, JAX, or notebooks, and prescribes no framework.

### Notebooks

Notebooks are for exploration, reporting, and demonstration. Reusable logic lives in package modules that notebooks import; package code and tests never import notebooks. Before committing, a notebook must run top to bottom in a fresh kernel, with outputs stripped unless they are the deliverable (nbstripout or Jupytext pairing, recorded in the addendum). Lint and format notebooks with Ruff. Code promoted out of a notebook gains annotations, tests, and docstrings.

### Configuration and reproducibility

Describe an experiment or pipeline run with a typed, frozen config object, loaded and validated once at the entry point. Config frameworks such as Hydra are allowed if their output is converted into the typed object at the boundary. Do not pass raw dicts or `argparse.Namespace` objects deep into the code.

Create random generators from a seed and pass them in: `rng = np.random.default_rng(seed)`, `torch.Generator().manual_seed(seed)`, explicit `jax.random` keys. Do not call `np.random.seed` or `random.seed`, or rely on global RNG state. Record the seed, config, code version, and data version with every result.

Reference data and artifact locations through config, never hard-coded absolute paths. Keep large data and model weights out of Git.

### Arrays and tensors

- Annotate arrays (`npt.NDArray[np.float32]`, `torch.Tensor`) and document shapes, dtypes, and units in the docstring or with a shape-annotation library recorded in the addendum. A name may carry the layout: `images_nchw`, `logits` with shape `(batch, classes)`.
- Specify dtypes explicitly where precision matters, not platform-default integer widths.
- Never use truthiness on an array, data frame, or tensor. Use `.size == 0`, `.empty`, `.any()`, or `.all()`.
- Compare floating results with `np.isclose`, `np.allclose`, or `torch.testing.assert_close` and explicit tolerances.
- Vectorize. Do not loop over rows in Python when a vectorized operation exists; when a loop is necessary, say why.
- Make device placement explicit: accept a `device` parameter or derive it from config, without scattered `.cuda()` calls. Use `torch.inference_mode()` for inference.

### Data frames

- Do not use `inplace=True`. Assign the result.
- Do not use chained indexing for assignment (`df["a"][mask] = 0`). Use `.loc` or the library's expression API.
- Specify `dtype`s and parse dates explicitly when reading external files, and validate the schema at the boundary.
- Prefer method chains with named intermediate steps over long sequences of reassignment to `df`.

### Model artifacts

Load model weights only from trusted sources, with `torch.load(..., weights_only=True)` or the safetensors format. Store weights and data in formats that do not execute code on load. Version artifacts and record what produced them.

## 25 Enforcement and exceptions

### Automate what can be automated

Every project must expose these checks as commands and run them in CI:

1. `uv lock --check`: the lockfile matches `pyproject.toml`.
2. `ruff format --check`.
3. `ruff check` with the rule set in section 26.
4. The type checker in strict mode over `src`, `tests`, and `scripts`.
5. The test suite.
6. For packages, `uv build` and an install-and-import smoke test of the built wheel.

A passing test run does not prove type checking ran, and a successful import does not prove the code is typed. Do not type-check single files in isolation as a substitute for the configured check. Review is the final pass alongside these checks, not a replacement.

| Rule                                                           | Mechanism                                                                         |
| -------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Formatting, quotes, line length                                | `ruff format`                                                                     |
| Import order, absolute imports, banned modules                 | Ruff `I`, `TID` (with `ban-relative-imports = "all"` and `banned-api`)            |
| Naming, shadowed builtins                                      | Ruff `N`, `A`                                                                     |
| Modern syntax: PEP 695, `X \| None`, no `typing.List`          | Ruff `UP` with the `requires-python` target                                       |
| Full annotations, no `Any`                                     | Ruff `ANN`; checker strict mode                                                   |
| Override decorators                                            | Pyright `reportImplicitOverride` or mypy `explicit-override`                      |
| Bare and broad except, `raise ... from`                        | Ruff `E722`, `BLE`, `B904`, `TRY`                                                 |
| Mutable defaults, `zip` without `strict`, cached methods       | Ruff `B006`, `B008`, `B905`, `B019`                                               |
| Dropped tasks, blocking calls in async                         | Ruff `RUF006`, `ASYNC`                                                            |
| Logging style, no `print` in libraries                         | Ruff `G`, `LOG`, `T20`                                                            |
| Naive datetimes, `os.path`                                     | Ruff `DTZ`, `PTH`                                                                 |
| Docstring format                                               | Ruff `D` with the Google convention                                               |
| Pytest style                                                   | Ruff `PT`; pytest strict options                                                  |
| Commented-out code                                             | Ruff `ERA`                                                                        |
| Bare type-ignore and `noqa` comments                           | Ruff `PGH`                                                                        |
| Security footguns: `eval`, `shell=True`, `pickle`, `yaml.load` | Ruff `S`                                                                          |
| Exhaustive matches                                             | `assert_never` plus the type checker                                              |
| Dependency direction and cycles                                | Pyright `reportImportCycles`, or an import-boundary tool recorded in the addendum |

A broad preset is a starting point, not a complete implementation of this guide. Rules a tool cannot check (cohesion, naming quality, boundary placement) are enforced in review.

### Handle exceptions visibly

Permitted exceptions are specific: tool-required filenames, generated code, focused stubs for untyped dependencies, a localized `cast`, and documented framework conventions.

A lint suppression names the exact rule and gives a reason on the smallest applicable line: `# noqa: S603  # fixed argument list, no user input`. Do not use bare `# noqa`, file-level `# ruff: noqa`, or `per-file-ignores` to silence a rule for one diagnostic. Enable the checks that report unused suppressions.

Keep generated-code exclusions narrow, by path. Do not classify handwritten code as generated to evade checking.

New exceptions that change the standard are recorded in the project addendum or an explicit revision of this guide. Individual files must not quietly introduce a different style. The section 2 baseline remains mandatory.

## 26 Project configuration

### Use pyproject.toml and uv

All project metadata and tool configuration lives in `pyproject.toml`. Do not add `setup.py`, `setup.cfg`, `requirements.txt` as the source of truth, `.flake8`, `mypy.ini`, or `pytest.ini`. Manage environments, dependencies, and the lockfile with uv, and commit `uv.lock`. Export a `requirements.txt` or `pylock.toml` only as a generated artifact for a deployment target that needs it.

Declare development tools in a dependency group (PEP 735), pinned to exact versions, so every contributor and CI job runs the same tools.

### Shared baseline

A shared fragment, not a complete configuration for every project; exact tool versions are pinned in the project. Add `requires-python` with the project's supported range: Ruff takes its target version from it, and Pyright and mypy use the project environment's interpreter.

```toml
[project]
name = "app"
dependencies = ["pydantic"]

[project.scripts]
app = "app.cli:main"

[dependency-groups]
dev = ["ruff", "pyright", "pytest", "hypothesis"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/app"]

[tool.ruff]
line-length = 100
indent-width = 4
src = ["src", "tests"]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"
skip-magic-trailing-comma = false
line-ending = "lf"
docstring-code-format = true

[tool.ruff.lint]
select = [
  "A",
  "ANN",
  "ARG",
  "ASYNC",
  "B",
  "BLE",
  "C4",
  "D",
  "DTZ",
  "E",
  "ERA",
  "F",
  "FBT",
  "FURB",
  "G",
  "I",
  "ISC",
  "LOG",
  "N",
  "PERF",
  "PGH",
  "PIE",
  "PL",
  "PT",
  "PTH",
  "PYI",
  "RET",
  "RSE",
  "RUF",
  "S",
  "SIM",
  "SLF",
  "T10",
  "T20",
  "TC",
  "TID",
  "TRY",
  "UP",
  "W",
]
ignore = [
  "D105", # magic methods are documented by their protocol
  "D107", # __init__ arguments are documented on the class
  "E501", # the formatter owns line length
  "TRY003", # precise f-string messages on built-in errors are required
]

[tool.ruff.lint.per-file-ignores]
"tests/**" = ["S101", "D", "PLR2004", "SLF001", "FBT001"]

[tool.ruff.lint.pydocstyle]
convention = "google"

[tool.ruff.lint.isort]
known-first-party = ["app"]

[tool.ruff.lint.flake8-tidy-imports]
ban-relative-imports = "all"

[tool.ruff.lint.flake8-tidy-imports.banned-api]
"typing.TypeAlias".msg = "Use the type statement."
"pickle".msg = "Do not deserialize with pickle; see section 14."

[tool.pyright]
include = ["src", "tests", "scripts"]
typeCheckingMode = "strict"
enableTypeIgnoreComments = false
deprecateTypingAliases = true
reportDeprecated = "error"
reportImplicitOverride = "error"
reportImportCycles = "error"
reportPropertyTypeMismatch = "error"
reportUninitializedInstanceVariable = "error"
reportUnnecessaryTypeIgnoreComment = "error"

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = ["--strict-markers", "--strict-config", "--import-mode=importlib", "-ra"]
xfail_strict = true
filterwarnings = ["error"]
markers = ["slow: takes more than a second", "integration: needs external services"]
```

A project that chooses mypy replaces the Pyright table with:

```toml
[tool.mypy]
files = ["src", "tests", "scripts"]
strict = true
warn_unreachable = true
enable_error_code = [
  "deprecated",
  "explicit-override",
  "ignore-without-code",
  "possibly-undefined",
  "redundant-expr",
  "truthy-bool",
  "truthy-iterable",
  "unused-awaitable",
]
plugins = ["pydantic.mypy"]
```

Do not lower `typeCheckingMode` or remove `strict` to silence a dependency. Fix the boundary with stubs or a typed adapter.

### Select a project profile

| Profile                   | Python versions                                                       | Dependencies                                                    | Packaging                                                                      |
| ------------------------- | --------------------------------------------------------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Library                   | A supported range starting at the floor; CI tests every version in it | Lower bounds in `dependencies`; lockfile for development only   | Wheel with `py.typed`; public API declared in `__all__`                        |
| Application or service    | One pinned version in `.python-version`                               | Locked with `uv.lock`, deployed from the lock                   | Container image or wheel; entry points in `[project.scripts]`                  |
| CLI tool                  | One pinned version for development, a stated range for users          | Locked; minimal runtime dependencies for fast startup           | `[project.scripts]` entry point; installable with `uv tool install`            |
| Single-file script        | Declared in PEP 723 inline metadata                                   | Declared inline, locked with `uv lock --script`                 | None; run with `uv run`                                                        |
| Research and ML workspace | One pinned version matching the accelerator stack                     | Locked, with hardware-specific indexes recorded in the addendum | Package under `src/` plus `notebooks/`; experiments configured by typed config |

### Project addendum

Record only project-dependent decisions, in `STYLE.md` or a section of the README:

- Supported Python versions and runtime targets, including free-threaded support.
- The type checker, async test plugin, structured-logging library, and shape-annotation library, with pinned versions.
- Additional community import aliases, if any.
- The concurrency model (section 19).
- Public package entry points and documented framework conventions (tool-required filenames, Django or FastAPI patterns).
- Commands for format checking, linting, type checking, testing, and building.
- Any documented exception or compatibility limitation, with a removal condition.

Framework architecture, database choice, and product-specific policies belong in project documentation and must not change the common code style by accident.

## 27 References

The prescriptions in this guide are project policy. Data-model examples use the Pydantic v2 API. Google's Python guide influenced the organization, the comprehension and docstring rules, and the treatment of power features; its environment-specific choices (pylint, 80-column lines, module-only imports, relaxed annotation coverage) are not adopted.

1. [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html)
2. [PEP 8: Style Guide for Python Code](https://peps.python.org/pep-0008/)
3. [PEP 257: Docstring Conventions](https://peps.python.org/pep-0257/)
4. [PEP 649: Deferred Evaluation of Annotations](https://peps.python.org/pep-0649/)
5. [PEP 695: Type Parameter Syntax](https://peps.python.org/pep-0695/)
6. [PEP 698: Override Decorator](https://peps.python.org/pep-0698/)
7. [PEP 742: Narrowing types with TypeIs](https://peps.python.org/pep-0742/)
8. [PEP 723: Inline script metadata](https://peps.python.org/pep-0723/)
9. [PEP 735: Dependency Groups](https://peps.python.org/pep-0735/)
10. [PEP 750: Template Strings](https://peps.python.org/pep-0750/)
11. [PEP 810: Explicit lazy imports](https://peps.python.org/pep-0810/)
12. [PEP 814: frozendict built-in type](https://peps.python.org/pep-0814/)
13. [What's New in Python](https://docs.python.org/3/whatsnew/index.html)
14. [Python typing specification](https://typing.python.org/en/latest/spec/)
15. [Ruff documentation](https://docs.astral.sh/ruff/)
16. [uv documentation](https://docs.astral.sh/uv/)
17. [ty repository](https://github.com/astral-sh/ty/)
18. [Pyright configuration](https://microsoft.github.io/pyright/#/configuration)
19. [mypy configuration](https://mypy.readthedocs.io/en/stable/config_file.html)
20. [Pydantic documentation](https://docs.pydantic.dev/latest/)
21. [pytest good practices](https://docs.pytest.org/en/stable/explanation/goodpractices.html)
22. [src layout vs flat layout](https://packaging.python.org/en/latest/discussions/src-layout-vs-flat-layout/)
23. [asyncio tasks and task groups](https://docs.python.org/3/library/asyncio-task.html)
24. [Logging HOWTO](https://docs.python.org/3/howto/logging.html)
25. [NumPy random Generator](https://numpy.org/doc/stable/reference/random/generator.html)
