# TypeScript Style Guide

An opinionated standard for TypeScript that is consistent to read, explicit about its contracts, and straightforward to change. It covers backend services, browser applications, full-stack systems, CLIs, libraries, workers, build scripts, and other TypeScript projects, and governs handwritten source, tests, examples, and configuration code. Frameworks and runtimes may add conventions, but the core style stays the same.

The emphasis is code style: names, expressions, functions, types, modules, and making behavior visible in source. File organization supports these goals without prescribing an application architecture.

## 1 Rules and scope

**Must**, **must not**, and imperative instructions are requirements. **Prefer** sets the default; depart from it only for a concrete improvement in clarity, correctness, or compatibility. **May** marks an explicitly permitted choice.

A project must record its runtime, module-resolution profile, tooling, and narrow integration conventions in a short addendum (section 25). The addendum must not silently weaken the core rules.

Examples illustrate the rule under discussion and may omit surrounding declarations. Unless another profile is named, relative imports use the emitted-JavaScript convention from section 25.

Generated and vendored files are exempt from formatting and naming rules. Keep them clearly identified, regenerate them from source, and do not edit them to imitate handwritten style. Handwritten wrappers and integration code remain subject to this guide.

## 2 Required language baseline

Every TypeScript configuration that checks handwritten code must enable `strict` and `erasableSyntaxOnly`. Do not disable them for tests, scripts, or a difficult dependency.

Do not write:

- `enum` or `const enum` declarations.
- `namespace` declarations, including type-only namespaces, or legacy internal `module` declarations used as namespaces.
- Constructor parameter properties such as `constructor(private client: Client)`.
- TypeScript `import =` or `export =` assignments.
- Angle-bracket assertions such as `<User>value`.
- Decorators or a dependency on decorator metadata emit.

Use modules, ordinary objects, functions, explicit class fields, and types instead. The namespace and decorator bans are style rules beyond the compiler flag, which neither rejects every type-only namespace nor establishes universal runtime compatibility. [2]

`declare module "external-package"` that accurately describes or augments an external module is not an internal namespace. Keep necessary declarations in focused `.d.ts` files. Do not use ambient declarations to conceal missing runtime implementations, and do not add handwritten ambient enums or namespaces.

Transpilation, bundling, and type stripping do not replace a full type check. Run a separate type-check command even if the development runner executes TypeScript successfully.

Erasable TypeScript can still use syntax or APIs the runtime does not support, and JSX needs its own compilation path. Node's type stripping does not type-check, ignores `tsconfig.json`, and does not execute `.tsx`. [3]

Pin compiler and tool versions through the dependency and lockfile policy. Upgrade deliberately and resolve new diagnostics. Do not rely on an unpinned global compiler.

## 3 Formatting and source text

### Let the formatter own layout

Every project must commit a formatter configuration and expose commands to format and check formatting. Use one formatting engine for TypeScript. Configure a linter for correctness and semantic style without conflicting layout rules.

The required policy, as Prettier settings (use equivalents for another formatter):

```json
{
  "tabWidth": 2,
  "useTabs": false,
  "printWidth": 100,
  "semi": true,
  "singleQuote": false,
  "jsxSingleQuote": false,
  "trailingComma": "all",
  "arrowParens": "always",
  "bracketSpacing": true,
  "endOfLine": "lf"
}
```

Also require UTF-8 and one final newline. Trailing commas apply wherever the syntax and formatter permit. Keep import-order tooling separate where necessary. Formatter defaults and lint presets do not replace committing these settings. [4]

Use the formatter's escape-minimizing quote choice when it selects another delimiter. Print width is a wrapping preference, not a demand to split URLs, regular expressions, or meaningful string literals. [4]

Do not manually align columns, wrap code to preserve a visual shape, or add formatter ignores for ordinary code. A narrowly scoped ignore is acceptable for a literal whose layout conveys information, with a comment explaining why.

### Write clear source text

Keep one statement per line and one binding per declaration; destructuring may introduce several related bindings. Use a single blank line between logical steps, none immediately inside function or block braces, and no large decorative comment banners.

Use ordinary spaces for indentation and separation. Escape invisible or control characters in strings and explain them when their purpose is not apparent. Readable Unicode is allowed in strings and comments; identifiers must use ASCII unless an external interface requires otherwise.

Use template literals for interpolation or intentional multiline text, and ordinary strings otherwise. Do not use line-continuation escapes to shorten a long string in source.

## 4 Naming

Names must describe a value's domain meaning or an operation's effect. Avoid `data`, `item`, `thing`, `manager`, `helper`, and `handler` when a more specific term exists. A short name is acceptable in an obvious, small scope; an abbreviation should not require a glossary.

### Casing

- `kebab-case`: files and directories (`read-config.ts`, `account-menu.tsx`).
- `camelCase`: variables, parameters, properties, functions, methods, services, constructed instances (`retryCount`, `loadAccount`, `accountStore`).
- `camelCase` ending in `Schema`: Zod schemas (`accountSchema`).
- `PascalCase`: types, interfaces, classes, JSX components, enum-style value objects (`Account`, `AccountMenu`, `JobStatus`).
- Descriptive `PascalCase`: type parameters (`Item`, `Key`, `Result`).
- `PascalCase` ending in `Error`: error subclasses (`ConfigurationError`).
- `CONSTANT_CASE`: fixed module-level data constants (`MAX_RETRY_COUNT`, `DEFAULT_OPTIONS`) and members of enum-style value objects (`JobStatus.IN_PROGRESS`).

Casing follows a binding's role, not the presence of `const`. Fixed module-level limits, lookup tables, regular expressions, and default data use `CONSTANT_CASE`. Functions, schemas, services, and framework builders do not, and local constants use `camelCase`.

`CONSTANT_CASE` communicates intended stability; it does not freeze an object. Exported constant collections must also expose a readonly contract where practical.

### Meaning and consistency

- Treat acronyms as words: `userId`, `apiUrl`, `HttpClient`, `parseJson`.
- Use verbs for operations and nouns for values and types.
- Name booleans as predicates (`isReady`, `hasAccess`, `canRetry`, `shouldPersist`). Prefer positive booleans and avoid double negatives at call sites.
- Use plural names for collections: `accounts`, `accountIds`.
- Include units when ambiguity is possible: `timeoutMs`, `sizeBytes`, `ratio`, `percentage`.
- Where the distinction applies, use `onSave` for a caller-supplied callback and `handleSave` for the implementation responding to that event.
- Use the `Async` suffix only to distinguish an intentional synchronous/asynchronous pair.
- Use a short generic name such as `T` only for a simple, conventional relationship. Prefer `Input` and `Output` when several type parameters participate.
- Do not prefix interfaces with `I`, types with `T`, or variables with their primitive type.
- Do not use leading or trailing underscores for privacy. A required but unused positional parameter may use `_name`; remove unused parameters when the contract allows it.
- Preserve external field names in the external contract. Translate them once when constructing the internal model.
- Alias imported symbols only to resolve a collision or clarify an ambiguous external name, not to apply a local naming preference.

## 5 Files and directories

### File names

Use `kebab-case` for all handwritten `.ts` and `.tsx` filenames: a component named `AccountSettings` belongs in `account-settings.tsx`. Use conventional suffixes when they add information:

- Unit test: `read-config.test.ts`; TSX test: `account-settings.test.tsx`.
- Integration test: `read-config.integration.test.ts`.
- Boundary schema: `job.schema.ts`.
- Ambient declarations: `asset-types.d.ts`.
- Platform-specific implementation: `file-store.node.ts`.
- Tool configuration: `vitest.config.ts`.

Dots separate recognized suffixes. Do not invent a new suffix vocabulary in each directory.

Use `.tsx` only for files containing JSX and `.ts` otherwise. Use `.mts` only where the toolchain needs an explicit ESM marker. Do not introduce handwritten `.cts` or `.cjs` files.

A tool-required filename or route syntax is a narrow exception. Keep it within the integration that requires it.

### Group code by ownership

Group by capability or domain before technical kind. A cohesive directory can contain implementation, schemas, types, tests, and private helpers. Do not require `controllers`, `services`, `repositories`, `components`, or a particular monorepo structure. Start small and add boundaries when there are distinct responsibilities or consumers.

An illustrative arrangement, not required directories: `src/config/read-config.ts` loads and validates configuration; `src/jobs/job.schema.ts` holds data accepted at the job boundary; `src/jobs/plan-job.ts` holds pure planning behavior; `src/adapters/file-job-store.ts` integrates platform storage; `src/main.ts` composes dependencies and starts execution. Tests sit beside each source file.

Keep a helper near its first owner, and move it to a shared module only once its responsibility and consumers are clear. Do not create large catch-all `utils.ts`, `helpers.ts`, `common.ts`, or root-level `types.ts` files. A focused `types.ts` within a small, cohesive API is acceptable.

### Keep modules cohesive

A file should have one primary responsibility and a small public API. It may contain several tightly related exports; do not impose one export per file or split every type and helper into its own module.

Keep a type or schema with the behavior that owns it unless independent consumers need a separate dependency. Split when unrelated responsibilities, environment requirements, or dependency cycles make the boundary useful.

Keep entry points thin: read inputs, compose dependencies, invoke behavior, translate outputs. Substantial domain logic belongs in modules callable without starting the application.

Colocate unit tests with their source. Keep cross-cutting integration or end-to-end tests in a dedicated test area when they span several modules. Store small fixtures near the tests that own them.

### Order code for reading

1. A required shebang, then any license or short file-level explanation.
2. Required directive prologues, such as a framework directive.
3. Imports, preserving any evaluation-sensitive setup order.
4. Fixed data, schemas, and supporting types needed to understand the API.
5. Exported behavior and the private helpers that explain it.

Keep related declarations together, with a schema-inferred type beside its schema. A private helper may follow the exported function that calls it. Do not read a `const`, `let`, or class binding before it is initialized.

Do not alphabetize implementation declarations or collect unrelated types at the top for symmetry. Preserve dependency order for values evaluated during module initialization.

## 6 Modules and dependencies

### Use ESM consistently

Use `import` and `export` in all handwritten source and tooling code. Do not use `require`, `module.exports`, or `exports`.

Set `"type": "module"` in each package whose JavaScript is intended to be ESM. Align compiler output, package metadata, test runners, and production execution; ESM-looking source does not establish the runtime's module format. [5]

A dependency may internally use CommonJS if the runtime supports consuming it through ESM; verify that integration. Do not spread compatibility code or change project source to CommonJS for one dependency.

### Make dependency direction visible

Lower-level data models and domain behavior must not import entry points, route files, view composition, or platform startup code. Introduce a small explicit contract when core behavior needs an external capability.

Import another package through its declared entry point or public subpath, never its private source tree. Declare a dependency in the package that imports it, including type-only dependencies where the packaging contract requires them.

Do not create circular dependencies, including through re-exports. Move the shared contract to the appropriate owner instead of hiding a cycle behind dynamic imports.

Use a public `index.ts` only where it defines an intentional package or feature API. Do not create barrels in every directory. Code inside a package must not import its own public barrel to reach a sibling module.

### Limit module initialization

Reusable modules must not open network connections, start timers, read process-specific configuration, launch tasks, or register global behavior merely because they are imported.

Prefer exported initialization functions and factories. Construct pure schemas and fixed data at module scope. Keep runtime configuration reads and lifecycle ownership at the composition boundary.

Use explicit, reviewed side-effect imports only for integrations that require them, such as a polyfill or stylesheet, and preserve their evaluation order. Suffixes such as `.server.ts` are documentation unless tooling enforces the boundary; use real module restrictions for incompatible environments.

## 7 Imports and exports

### Import syntax and ordering

Prefer named imports. A namespace import such as `import * as path from "node:path"` is allowed for an API clearer as a coherent namespace; it is an ESM import, not a namespace declaration.

Use a whole `import type` declaration when every imported binding is type-only, and `export type` for type re-exports. When a module supplies both values and types, use separate adjacent declarations.

```ts
import { readFile } from "node:fs/promises";

import { z } from "zod";

import { createJob } from "./job.js";
import type { Job, JobId } from "./job.js";

export type { JobId } from "./job.js";
```

Under `verbatimModuleSyntax`, a whole `import type` is erased, but a declaration of only inline `type` specifiers can leave an empty runtime import, so do not use that form for an entirely type-only dependency. [6]

Group ordinary imports as runtime built-ins, third-party packages, project aliases, relative modules, with one blank line between nonempty groups. Use checked-in tooling to sort module specifiers and named bindings, not manual ordering.

Sorting must not change evaluation-sensitive order. Put required side-effect setup in a clearly identified block and configure the sorter to preserve it. Prefer explicit initialization over incidental ordering among value imports.

Use `node:` for Node built-ins. Do not append extensions to bare package names.

### Paths and extensions

Use relative imports for nearby code in the same package and package entry points across packages.

Use an alias only when the checker, runtime, bundler, and tests all resolve it consistently. TypeScript's `paths` does not rewrite emitted imports. Prefer a supported package import/export mapping to an alias only the editor understands. [24]

Choose exactly one relative-import convention per compilation unit (profiles in section 25):

- **Emitted JavaScript:** the emitted extension, usually `./read-config.js`.
- **Bundled source:** explicit source extensions, such as `./read-config.ts` or `./account-menu.tsx`.
- **Direct TypeScript execution:** the actual source extension, such as `./read-config.ts`.

Do not use implicit directory imports. Write `./jobs/index.js` or its profile equivalent; Node ESM requires explicit relative extensions and directory indexes. [5]

A framework-generated import convention may be documented as an integration exception. Do not switch resolver modes merely to make a broken runtime import type-check.

### Import JSON with attributes

```ts
import defaultsJson from "./defaults.json" with { type: "json" };
import { defaultsSchema } from "./defaults.schema.js";
import type { Defaults } from "./defaults.schema.js";

export const DEFAULTS: Readonly<Defaults> = defaultsSchema.parse(defaultsJson);

export async function loadDefaults(): Promise<unknown> {
  const { default: lazyDefaults } = await import("./defaults.json", {
    with: { type: "json" },
  });

  return lazyDefaults;
}
```

Require these attributes on static and dynamic JSON imports (the dynamic form is for genuinely lazy loading). Do not use the legacy `assert` syntax. JSON uses a default export, so its default import is allowed. [5]

Enable `resolveJsonModule` in compilation units that import JSON. It provides static resolution and shape information, not validation of application constraints or runtime loading. [23] Validate configuration and other boundary data with Zod.

Attributes describe a module's interpretation; they do not make arbitrary file types importable. Other assets require a documented convention supported by the runtime or bundler. Do not add attributes to ordinary TypeScript or JavaScript imports.

Use static imports for fixed module assets. Read user-selected, mutable, or runtime-discovered files with the platform's I/O API, then parse and validate them.

### Export deliberately

Use named exports for first-party APIs, exported where they are defined. A default export is allowed only when a framework, tool, or external format requires it; name the declaration meaningfully when possible. A dependency's required default import is also allowed.

Export only what another module needs, and do not export private helpers solely to test them.

Do not use `export *` in handwritten public APIs; list re-exports so API growth is visible in review. Do not export mutable bindings such as `export let`. Encapsulate changing state behind a function or owned object API.

## 8 Variables and expressions

### Bind values deliberately

Use `const` by default, `let` only when reassignment is part of the algorithm, and never `var`.

Declare a binding close to its first use. Do not declare a variable uninitialized and assign it far away when an expression or helper can produce its value directly. Do not reuse one variable for different concepts. Avoid shadowing an outer binding when it obscures which value is meant.

Destructure when it makes a small set of properties clearer. Keep parameter destructuring shallow, and do not destructure a long path merely to save characters.

### Prefer direct expressions

Use `===` and `!==`. Do not use coercive equality, including `value == null`; write the nullish check explicitly.

Use `??` when only `null` and `undefined` should trigger a fallback (so `options.retryCount ?? 3` keeps `0`), and `||` only when every falsy value should.

Use optional chaining only when absence is valid, not to silently skip required behavior.

Use explicit conversion such as `Number(value)` and `String(value)`, and `Boolean(value)` when truthiness conversion is the actual operation. Avoid unary-plus coercion, `~~value`, implicit string concatenation for conversion, and `!!value`. Conversion is not validation: numeric conversion needs a policy for blank strings, non-finite numbers, and accepted syntax.

Use parentheses when mixed operators would otherwise require recalling precedence. Do not use assignments inside conditions, comma operators, nested ternaries, or logical operators solely for side effects.

Do not use `eval`, the `Function` constructor, or strings as timer callbacks. Do not modify built-in constructors or prototypes, or commit `debugger` statements.

## 9 Control flow

### Make branches explicit

Always use braces for `if`, `else`, `for`, `for...of`, `while`, and `do` bodies.

Prefer guard clauses when they reduce nesting. Do not add `else` after an unconditional `return` or `throw` unless the paired structure clarifies a larger algorithm.

Use a ternary only for one small choice of value, and never nest ternaries. Use named intermediate values or `if` statements for more involved decisions.

Boolean values may be tested directly. For numbers, strings, and nullable primitives, test the intended condition: `count > 0`, `name.length > 0`, `value !== undefined`. An optional object may use a presence check when no other falsy value is possible.

Avoid large compound conditions. Name an independently meaningful condition or extract a predicate.

### Switches

Use `switch` for a finite set of alternatives when it makes cases easier to scan. Do not fall through between executable cases; multiple empty labels sharing a body are allowed. Use braces around a case that declares a lexical binding. Return from cases when the switch computes the function's result.

For a closed union, require exhaustiveness with `never` (section 14). For open external values, validate first or provide meaningful unknown-value handling. Do not add an empty `default` for style.

### Loops

Prefer `for...of` for iteration and `for await...of` for asynchronous iterables. Use an indexed loop when the index or traversal strategy is part of the algorithm.

Use `continue` and `break` when they make control flow clear; do not force nested conditionals to avoid them.

Do not use `for...in`. Use `Object.keys`, `Object.values`, or `Object.entries` for object data.

## 10 Functions and API design

### Function form

Use named function declarations for reusable module-level behavior. Use arrow functions for callbacks, closures, and values that must conform to a function type. A framework's inference-preserving function builder may also produce an exported binding.

Use a concise arrow body for a single returned expression. Use a block body for side effects, even when the callee returns `void`, so a later return-value change cannot silently alter callback behavior.

Use function expressions only when dynamic `this` or an external API requires them.

### Parameters and returns

Require explicit parameter and return types for ordinary exported functions and public methods, so an implementation change cannot silently change the public contract.

Private helpers and contextually typed callbacks may infer returns. Schema definitions, inference-oriented builders, and framework callbacks may keep inferred public types when an annotation would discard useful type information. Do not replace precise inference with a broad annotation to satisfy a lint rule.

Prefer an options object when there are more than two independent inputs, several inputs of the same primitive type, or an argument unclear at the call site. A natural pair such as a key and value may remain positional. Do not use unexplained boolean arguments such as `saveRecord(record, true, false)`; use named options or separate operations.

Put required parameters before optional ones. Use defaults at the point that owns the policy. Defaults apply to missing or `undefined` arguments, so do not use them when explicit `undefined` has a distinct meaning.

Avoid output parameters. Return a named object for several related results, and use tuples only for a small positional convention callers already understand.

Do not use sentinel values such as `-1` or `""` for a domain failure when a union or explicit absence is clearer. Preserve sentinel conventions of standard APIs at their boundary.

### Responsibility and dependencies

Keep a function focused on one operation at a consistent level of abstraction. Extract a block when it has an independently useful name, hides a distracting detail, or needs a different dependency. Do not extract mechanically, and avoid arbitrary function-length limits that reward fragmentation.

Pass dependencies that affect execution to reusable behavior: clocks, random generators, stores, network clients, output sinks. Reusable functions must not read ambient process state or reach into a global service registry.

Prefer composition and ordinary parameters to a generic framework invented for a single call site.

## 11 Type declarations and inference

### Use type aliases by default

Use `type` for application data, unions, tuples, function contracts, derived types, and schema-inferred types. Use `interface` when an API intentionally supports extension or declaration merging, or an external integration requires it. Do not use merging to invisibly add fields to ordinary domain models.

This is a consistency policy: both participate in structural typing, and an alias does not create an exact or sealed shape. [8]

Prefer a direct named shape over deep intersections. For a substantial intentional open hierarchy, interface extension is clearer than stacked intersection aliases.

### Use inference where it helps

Infer obvious local values; do not write `const retryCount: number = 3`. Annotate a local when it intentionally has a wider type than its initial value, when an empty collection needs an element type, or when the type explains an unclear contract.

Use `satisfies` for authored literals that must meet a contract while preserving specificity. Use an annotation when the binding should present the broader contract.

```ts
type OutputFormat = "json" | "text";

export const MIME_TYPES = {
  json: "application/json",
  text: "text/plain",
} as const satisfies Record<OutputFormat, string>;
```

`satisfies` is a static check, not runtime validation or an exact-object mechanism, and contextual typing can still affect inference. [9]

### Use precise basic types

Use `string`, `number`, `boolean`, `bigint`, and `symbol`, never wrapper types such as `String`, `Number`, or `Boolean`.

Use `unknown` when a value is unspecified or untrusted. Use `object` only for truly any non-primitive value, and `Record<string, unknown>` only for a true string-keyed record.

Do not use `any`, `Object`, `Function`, or `{}` as placeholder types; write the needed shape or signature. `{}` is not an empty-object contract. [8] Do not let a third-party `any` propagate: assign it to `unknown` at the adapter boundary, then validate or narrow it.

Use `void` for a return value callers must ignore, `never` for a path that cannot return, and `undefined` when absence is a value in the contract.

### Collections and generic types

Use `T[]` and `readonly T[]`. Parenthesize union elements: `(string | number)[]`. Name the element type when the expression becomes hard to read.

Use labelled readonly tuples for small fixed positional structures, such as `readonly [width: number, height: number]`. Prefer an object when positions need repeated explanation.

Use generics to preserve meaningful type relationships. In a function signature, do not add a type parameter when a concrete type or `unknown` expresses the same contract, solely to let the caller assert a return type, or to make a fixed domain model look configurable. Generic data containers may use their parameter in a single field.

Prefer a union parameter to overloads when the return type does not depend on the input alternative. Keep required overloads adjacent with one clear implementation.

Use built-in utility types when they clarify intent. Do not expose a large nested conditional or mapped type to avoid a small explicit declaration. Name and test reusable type-level operations.

## 12 Absence and optional values

Use `undefined` for optional application values. Preserve `null` when it carries intentional meaning in a protocol, database, or domain. Do not make every type globally nullable; add absence at the boundary where it occurs.

Under `exactOptionalPropertyTypes`:

| Declaration                   | Meaning                                            |
| ----------------------------- | -------------------------------------------------- |
| `label?: string`              | May be omitted; a present value must be a string   |
| `label: string \| undefined`  | Must be present; its value may be undefined        |
| `label?: string \| undefined` | Omission and explicit undefined are both permitted |
| `label: string \| null`       | Required; null is an intentional value             |

This concerns optional properties. An optional function parameter still accepts an explicit `undefined` argument, and reading an optional property can produce `undefined`. [10]

Omit an optional property when it is absent instead of constructing `{ label: undefined }` for a `label?: string` contract: `return label === undefined ? {} : { label };`.

Treat omission, clearing a value, and supplying a value as separate operations when the domain needs them. For an update command, `undefined` may mean "leave unchanged" while `null` means "clear"; make that contract explicit.

Do not use optional fields to represent mutually exclusive states. Use a discriminated union.

## 13 Assertions and narrowing

### Narrow before use

Prefer checks the compiler understands: `typeof`, `instanceof`, discriminant comparisons, presence checks, and control flow. Validate external data with a schema; an assertion cannot establish the shape of a response, parsed JSON value, DOM lookup, or environment variable.

Do not write:

- Double assertions such as `value as unknown as Account`.
- `as any`.
- Non-null assertions such as `account!` or `accounts[0]!`.
- Definite-assignment assertions such as `client!: Client`.
- Assertions that force a mismatched object literal into a desired type.

Use a checked invariant instead:

```ts
export function requireValue<Value>(value: Value | null | undefined, message: string): Value {
  if (value === null || value === undefined) {
    throw new Error(message);
  }

  return value;
}
```

### Keep escape hatches narrow

`as const` is allowed to preserve literal values and readonly literal structure.

A single `as SomeType` is allowed only when a documented invariant or external declaration limitation establishes the relationship and TypeScript cannot express it. Keep it at the smallest adapter boundary, explain the invariant, and cover the behavior that justifies it. "The compiler complains" is not a reason.

Do not move an unsafe assertion into a helper and call it a type guard: the compiler trusts explicit predicates such as `value is Account` without verifying them. [11] A custom predicate or assertion function must establish every property it promises and have tests for rejection as well as success.

Prefer inferred predicates for simple filtering: `labels.filter((label) => label !== undefined)`. Do not use `filter(Boolean)` to remove only missing values, because it also removes valid falsy values.

Do not use `@ts-ignore` or `@ts-nocheck`. Temporary `@ts-expect-error` use is governed by section 24; negative type tests by section 22.

## 14 Finite values and state models

### Replace enums with ordinary values

For a finite vocabulary needed only by the type checker, use a literal union: `type SortDirection = "ascending" | "descending"`.

For values accepted from outside the program, use a Zod enum schema and infer the type. If callers need named runtime members:

```ts
import { z } from "zod";

export const JobStatus = {
  QUEUED: "queued",
  IN_PROGRESS: "in-progress",
  SUCCEEDED: "succeeded",
  FAILED: "failed",
} as const;

export const jobStatusSchema = z.enum(JobStatus);

export type JobStatus = z.infer<typeof jobStatusSchema>;
```

Zod 4 accepts an enum-like object in `z.enum`; do not use `z.nativeEnum` in new code. [12] The object and inferred type deliberately share a name. Serialized values use the exact spelling the contract requires.

Use `JobStatus.QUEUED` where named members exist. Do not repeat their serialized literals throughout the implementation or duplicate the vocabulary in a handwritten union.

For a small boundary vocabulary without useful named members, use `z.enum(["json", "text"])` directly. Use a `CONSTANT_CASE` readonly tuple only when other runtime code also needs that collection.

### Model valid states directly

Use a discriminated union when a value can be in one of several states, keeping each state's required data in that variant.

```ts
export type LoadState<Value> =
  | { readonly kind: "idle" }
  | { readonly kind: "loading" }
  | { readonly kind: "loaded"; readonly value: Value }
  | { readonly kind: "failed"; readonly error: Error };

export function describeLoadState(state: LoadState<unknown>): string {
  switch (state.kind) {
    case "idle":
      return "Not started";
    case "loading":
      return "Loading";
    case "loaded":
      return "Loaded";
    case "failed":
      return state.error.message;
    default:
      return assertNever(state);
  }
}

function assertNever(_value: never): never {
  throw new Error("Unhandled load state");
}
```

The `never` parameter makes a new variant fail type checking at the unhandled switch, and the helper throws if an invalid value reaches it at runtime. [13] Do not replace this with a generic fallback that quietly accepts a new state. Avoid state objects with several booleans and unrelated optional fields that permit impossible combinations.

For a fixed mapping rather than branching behavior, prefer a complete `Record<Variant, Value>` checked with `satisfies`.

### Validate discriminated unions with named kinds

Use `z.discriminatedUnion` rather than `z.union`; it selects the variant from the discriminator instead of trying each alternative. [15]

When a union has many variants or its kinds are used across modules, define the discriminator with the named-member pattern above and use its members everywhere: `typeof ValueKind.ARRAY` in types, `z.literal(ValueKind.ARRAY)` in schemas, `case ValueKind.ARRAY:` in switches.

```ts
import { z } from "zod";

export const ValueKind = {
  NONE: "NONE",
  TEXT: "TEXT",
  ARRAY: "ARRAY",
  MAP: "MAP",
} as const;

export const valueKindSchema = z.enum(ValueKind);

export type ValueKind = z.infer<typeof valueKindSchema>;

export type Value =
  | {
      kind: typeof ValueKind.NONE;
    }
  | {
      kind: typeof ValueKind.TEXT;
      text: string;
      language?: string | undefined;
    }
  | ArrayValue
  | MapValue;

export type ArrayValue = {
  kind: typeof ValueKind.ARRAY;
  elements: Value[];
};

export type MapValueEntry = {
  key: Value;
  value: Value;
};

export type MapValue = {
  kind: typeof ValueKind.MAP;
  entries: MapValueEntry[];
};

export const valueSchema: z.ZodType<Value> = z.lazy(() =>
  z.discriminatedUnion("kind", [
    z.object({
      kind: z.literal(ValueKind.NONE),
    }),
    z.object({
      kind: z.literal(ValueKind.TEXT),
      text: z.string(),
      language: z.string().optional(),
    }),
    z.object({
      kind: z.literal(ValueKind.ARRAY),
      elements: z.array(valueSchema),
    }),
    z.object({
      kind: z.literal(ValueKind.MAP),
      entries: z.array(
        z.object({
          key: valueSchema,
          value: valueSchema,
        }),
      ),
    }),
  ]),
);
```

Name a variant type when other code accepts or returns that variant alone; leave it inline when only the union refers to it.

`Value` is recursive, and TypeScript cannot infer a type through this schema's self-reference. A recursive union is therefore the exception to inferring types from schemas: write the type by hand, annotate the schema as `z.ZodType<Value>`, and defer the self-reference with `z.lazy`. The annotation makes the compiler check the schema's output against the type. Write optional fields as `field?: T | undefined` to match `.optional()`.

Define a non-recursive union as a schema only and infer its type. Choose `z.object` or `z.strictObject` for each variant using the section 15 policy.

## 15 Zod and runtime boundaries

### Validate where trust changes

Use Zod for runtime validation of structured external data: configuration, environment variables, command-line arguments, URLs, forms, network payloads, persisted data, messages, and third-party responses. Treat raw values as `unknown` even when a permissive API returns `any` or accepts an unchecked generic.

Define each boundary contract once and infer its TypeScript type. Do not maintain a parallel handwritten interface for the same validated shape, except for recursive schemas TypeScript cannot infer (section 14).

```ts
import { z } from "zod";

export const createJobSchema = z.strictObject({
  label: z.string().trim().min(1).max(120),
  retryCount: z.number().int().min(0).max(10).default(3),
});

export type CreateJob = z.infer<typeof createJobSchema>;

export function parseCreateJob(input: unknown): CreateJob {
  return createJobSchema.parse(input);
}
```

Continue with the value from `parse` or `result.data` from `safeParse`, never the original input: parsing may normalize values, apply defaults, or remove properties. [14]

Parse at the boundary and pass typed values inward without repeating validation in internal helpers. Validate again when crossing a new trust boundary or when external state can invalidate the guarantee. A schema establishes only the constraints it checks, not authorization, resource ownership, or every domain invariant.

### Choose object strictness intentionally

- Commands and configuration whose field set the project owns: `z.strictObject`.
- External responses where additional fields are expected to be harmless: `z.object`.
- A contract that intentionally preserves extension fields: explicit `z.looseObject` or `.catchall(...)`.

Strict objects reject unknown properties; ordinary objects strip them. [12] An API that explicitly promises additive compatibility may choose a different input policy; record that decision with the schema rather than depending on an accidentally chosen constructor.

### Keep input and output distinct

Use `z.infer` for the parsed output type, `z.input` when a trusted caller genuinely needs the pre-parse type, and `z.output` when naming the distinction clarifies a transform. An untrusted boundary still accepts `unknown`. [14]

```ts
import { z } from "zod";

export const retryCountTextSchema = z
  .string()
  .regex(/^\d+$/)
  .transform(Number)
  .pipe(z.number().int().min(0).max(10));

export type RetryCountText = z.input<typeof retryCountTextSchema>;
export type RetryCount = z.output<typeof retryCountTextSchema>;
```

Do not annotate a concrete schema with a broad base type if that loses its shape or input/output inference; the `z.ZodType<Value>` annotation a recursive schema requires is the exception (section 14).

`.optional()` permits `undefined`; it does not automatically mean an exact optional property that rejects a present `undefined`. Keep the inferred contract honest and normalize omissions when a stricter internal model needs them. [15]

### Parse and report failures deliberately

Use `parse` when failure belongs on an exception path, such as invalid startup configuration, and `safeParse` when failure is an expected branch to display or return. Use `parseAsync` or `safeParseAsync` for asynchronous refinements or transforms, and handle the promise. [14]

Keep refinements and transforms small. Report validation failures through Zod's supported mechanism, not arbitrary exceptions; `safeParse` does not catch programmer errors thrown by custom schema code. [15]

Translate validation failures into the boundary's documented error shape. Do not expose raw input values, internal paths, or entire error objects without considering the consumer.

### Coerce only at an intentional conversion boundary

Do not use coercion to accept malformed data silently. Validate the representation before converting when the allowed syntax matters.

For boolean text, use an explicit vocabulary: `z.enum(["true", "false"]).transform((value) => value === "true")`. Do not use `z.coerce.boolean()`: it follows JavaScript truthiness, so `"false"` becomes `true`. [16]

A default should handle intentional absence. Do not use `.catch(defaultValue)` to conceal invalid required configuration.

Keep environment access in an adapter. Read `process.env["PORT"]` rather than `process.env.PORT` (required by the index-signature rule), then validate and pass a typed configuration object inward.

### Serialize deliberately

Define transport representations for dates, timestamps, durations, `bigint` values, maps, sets, and other values without one obvious JSON representation.

Use explicit units and document timezone semantics. A project may choose Unix milliseconds or an ISO timestamp for an instant but must not silently mix them. A calendar date is not interchangeable with an instant.

Do not assume `undefined`, prototypes, methods, or class identity survive serialization. Construct intentional boundary objects and avoid spreading internal records directly into public responses.

## 16 Objects and collections

### Choose the appropriate structure

Use object and array literals, not `new Object`, `Object()`, `new Array`, or `Array()`. Use `Array.from` when creating a collection from a length or iterable. Never instantiate wrappers such as `new String` or `new Boolean`.

Use:

- An object for a known record shape.
- `Record<Key, Value>` for a fixed key vocabulary.
- A partial record or checked lookup for keys that may be absent.
- `Map` for a dynamic key collection, especially with non-string keys.
- `Set` for membership or uniqueness.
- An array for ordered values.

Do not add named properties to arrays, or use an object literal as an unbounded dictionary when a `Map` is clearer.

`Record<string, Value>` does not guarantee that every key exists. Check arbitrary lookups; `noUncheckedIndexedAccess` makes missing indexed values visible in many cases. [17]

### Use collection methods for their purpose

Use `map` to transform, `filter` to select, `find` to locate, and `some` or `every` for predicates. Do not use `map` solely for side effects. Use `reduce` when an accumulation is easier to understand as a reduction; prefer a clear loop over a reducer that mutates several unrelated fields or repeatedly copies a growing collection.

Use `Object.hasOwn` to check whether an object owns a property, not an input's potentially shadowed `hasOwnProperty`.

Do not sort or reverse an input array in place; use a nonmutating method or copy first, as in `[...scores].sort((left, right) => left - right)`. Supply a comparator for numeric and domain-specific sorting, and state any locale-sensitive policy.

### Spread and destructure carefully

Spread iterables into arrays and plain record-like objects into objects. Do not spread class instances, maps, sets, or arbitrary primitives into an object literal to "copy" them.

Use an explicit empty value in a conditional spread, not `false`, `null`, or `undefined`: `...(token === undefined ? {} : { authorization: `Bearer ${token}` })`.

Object spread is shallow and later properties replace earlier ones. Make override order intentional, especially when one source is external input.

## 17 Mutation and ownership

Prefer pure functions for calculations, transformations, validation, and domain rules.

Use readonly properties, `readonly T[]`, `ReadonlyMap`, and `ReadonlySet` in contracts that do not permit mutation, where they protect a meaningful boundary rather than mechanically on every local binding.

Do not mutate arguments or caller-owned collections unless that is the function's explicit purpose. Name mutating operations clearly and document what they change.

Local mutation is allowed for a newly owned accumulator, a loop variable, or a stateful object's private implementation. Do not replace a simple owned loop with repeated object spreading to avoid mutation syntax.

`const` prevents rebinding. `readonly` and `as const` constrain TypeScript code but do not freeze runtime data, and mutable aliases may exist. [18] If runtime immutability is required, implement and document it.

Do not export a mutable collection for callers to alter directly. Return snapshots, expose a readonly view with a clear ownership contract, or provide explicit operations.

Avoid duplicated state that can be derived cheaply from authoritative values. If a cache or snapshot is necessary, define who updates it and when it becomes invalid.

## 18 Classes

Use a class when identity, lifecycle, encapsulated state, or a platform contract benefits from it, and plain objects and functions for ordinary data and stateless operations. Do not create a class of only static members for namespacing; use a module.

### Fields and construction

Declare fields explicitly and initialize them at declaration or in the constructor. Use JavaScript `#private` fields and methods for implementation privacy. Use `protected` only for an intentional subclass contract, and omit redundant `public` modifiers.

Keep constructors synchronous and focused on establishing a valid object, with no asynchronous work or hidden network I/O. Use an explicit asynchronous factory or initialization operation when needed.

### Methods and inheritance

Use ordinary method syntax. Use an arrow field only when a callback needs a stable lexical `this`. Do not pass an unbound method as a callback when it depends on `this`; wrap the call or bind it deliberately.

Prefer composition. Use inheritance only for a real substitution contract, and mark overriding members with `override`.

Use getters for cheap, unsurprising property-like reads, not I/O or substantial work. Prefer an explicit mutating method to a setter that initiates unrelated behavior. Avoid public writable fields, and expose the smallest state and behavior contract the caller needs.

## 19 Asynchronous code

### Make promise ownership explicit

Prefer `async`/`await` for sequential workflows. Promise combinators and short chains are allowed when they express concurrency or a simple transformation more clearly.

Every promise must be awaited, returned to a caller that owns it, or deliberately detached with a rejection handler. `void` alone does not handle rejection. [19]

```ts
void refreshCache().catch((error: unknown) => {
  reportBackgroundFailure(error);
});
```

Here `reportBackgroundFailure` must be synchronous and nonthrowing; if reporting can fail, handle that too.

Detach work only when the surrounding lifecycle permits it to finish and losing it is acceptable. A request handler or short-lived process must not assume a background promise completes after its owner exits; use a durable job mechanism when completion matters.

Do not pass an `async` callback to an API that ignores its returned promise, such as `forEach(async ...)`. Use an awaited loop or an explicitly awaited concurrency operation.

### Choose concurrency deliberately

Run independent bounded operations together with `Promise.all`. Await dependent operations in order, and do not start a request before its prerequisites or access checks complete.

Use bounded concurrency for potentially large collections; an unbounded `Promise.all(values.map(...))` can start every operation at once.

`Promise.all` does not cancel remaining operations when one fails; use cancellation or cleanup where required. Use `Promise.allSettled` only when every outcome is needed, and inspect every result.

### Preserve errors and lifecycle

Use `return await` inside a `try` block when the `catch` must handle the rejection or `finally` must wait before releasing resources. A bare returned promise bypasses that catch for a later rejection and lets `finally` run before it settles.

Use `finally` for cleanup, and do not return from it. Do not replace an existing result or failure with an unrelated cleanup error; define a deliberate policy when cleanup can fail.

Accept and propagate an `AbortSignal` for cancellable long-running work when the underlying APIs support it. Cancellation is a distinct outcome; do not silently turn it into success or automatically retry it.

Retries must be bounded, limited to safely retryable failures, and explicit about repeated side effects. Do not embed an invisible retry loop in a small data-access helper.

Avoid top-level `await` in reusable modules; use it only at an explicit startup or tool boundary where blocking module evaluation is intentional.

Do not construct `new Promise` around an API that already returns a promise. Never use an `async` Promise executor.

## 20 Errors and results

### Throw useful errors

Throw `Error` instances or meaningful subclasses, never strings or arbitrary objects. Use standard subclasses such as `TypeError` and `RangeError` when they fit.

Use `cause` when adding context to an existing failure. Do not concatenate an error into a string and discard its identity.

Catch values as `unknown` and narrow them before reading properties, because external code can throw non-`Error` values.

Error messages should identify the failed operation and useful safe context, without credentials, tokens, complete request payloads, or sensitive data.

### Custom error classes

Create a custom error type when callers need to distinguish a meaningful failure category, carry structured diagnostic data, or use a stable domain-specific operation. Extend `Error` when no built-in subclass fits. Use a shared abstract base only when several errors actually share behavior or a contract.

Custom errors must follow these conventions:

- Call `super(message, options)` to preserve native message and `cause` behavior. Accept `ErrorOptions` instead of inventing a second cause field.
- Give each concrete type a stable name and set `this.name` to it. Do not depend on `constructor.name` or minified class names as an external identifier.
- Prefer `message` first and `options: ErrorOptions = {}` second. Additional context should have meaningful names and readonly types.
- Initialize properties in the constructor. Do not assign a fabricated string to `stack` or repeatedly throw and catch errors to obtain a trace.
- Where `Error.captureStackTrace` is available, use it to omit constructor and internal factory frames; otherwise keep the ordinary stack. The exact stack format is not a public contract. [25]
- Have a factory return its error type explicitly. Annotate an always-throwing helper `never` and give it a verb or failure-oriented name. A `static` helper that does not use `this` should declare `this: void`.
- Expose stable readonly error codes or domain properties only when callers need them. Do not require callers to inspect message text or stack strings to distinguish variants.

This pattern uses one stack-capture helper that checks for V8's API without requiring Node-specific types and lets a factory or throw helper identify the first useful frame:

```ts
type StackFrameStart = CallableFunction | NewableFunction;

function captureErrorStack(error: Error, start: StackFrameStart): void {
  if ("captureStackTrace" in Error && typeof Error.captureStackTrace === "function") {
    Error.captureStackTrace(error, start);
  }
}

export abstract class DomainError extends Error {
  protected constructor(
    name: string,
    message: string,
    options: ErrorOptions = {},
    stackStart?: StackFrameStart,
  ) {
    super(message, options);
    this.name = name;
    captureErrorStack(this, stackStart ?? new.target);
  }
}

export class TimeoutError extends DomainError {
  constructor(
    message = "The operation timed out",
    options: ErrorOptions = {},
    stackStart?: StackFrameStart,
  ) {
    super("TimeoutError", message, options, stackStart);
  }

  static create(this: void, message?: string, options: ErrorOptions = {}): TimeoutError {
    return new TimeoutError(message, options, TimeoutError.create);
  }

  static timedOut(this: void, message?: string, options: ErrorOptions = {}): never {
    throw new TimeoutError(message, options, TimeoutError.timedOut);
  }
}
```

Where the API is supported, passing `TimeoutError.create` or `TimeoutError.timedOut` omits that helper and its internal calls, so the trace starts at the application caller. Stacks attach when an error is constructed, not when it is thrown. [25]

The `stackStart` argument is for internal factories and throw helpers; ordinary callers use the message and options. Keep this plumbing out of application-level error handling.

An isolated error class may extend a built-in directly with the same conventions. A static predicate is useful when it meaningfully narrows the error, such as checking a structured code:

```ts
type OptionErrorCode = "INVALID_CAPACITY" | "INVALID_TIMEOUT";
type OptionsErrorOptions = ErrorOptions & { readonly code: OptionErrorCode };

export class OptionsError extends RangeError {
  readonly code: OptionErrorCode;

  constructor(
    message: string,
    options: OptionsErrorOptions,
    stackStart: StackFrameStart = OptionsError,
  ) {
    super(message, options);
    this.name = "OptionsError";
    this.code = options.code;
    captureErrorStack(this, stackStart);
  }

  static invalidCapacity(this: void): never {
    throw new OptionsError(
      "Capacity must be at least 1",
      { code: "INVALID_CAPACITY" },
      OptionsError.invalidCapacity,
    );
  }

  static isCapacityError(
    this: void,
    error: unknown,
  ): error is OptionsError & { readonly code: "INVALID_CAPACITY" } {
    return error instanceof OptionsError && error.code === "INVALID_CAPACITY";
  }
}
```

Prefer a direct `instanceof` check when the class alone establishes the condition; do not add redundant `isSomethingError` methods. `instanceof` depends on object identity and may not survive separate realms, workers, or serialization, so use an explicit validated serialized representation at those boundaries.

Do not copy `cause` into the message, rewrite an existing stack string, or expose internal stacks and causes to clients. [26]

### Catch where the code can act

Catch only to recover, translate an error at a boundary, add meaningful context, perform a safe retry, or handle a lifecycle outcome.

Do not use empty catches, represent failed work as successful, or return an undocumented placeholder. A fallback is valid when it is part of the API's documented recovery contract. Do not convert every exception into `undefined` unless failure is explicitly part of that function's absence contract.

Handle expected domain alternatives with a discriminated result when callers must branch on them:

```ts
export type ReserveResult =
  | { readonly ok: true; readonly reservationId: string }
  | {
      readonly ok: false;
      readonly reason: "unavailable" | "limit-exceeded";
    };
```

Use exceptions for failures that should interrupt the operation. Keep the convention consistent within an API, not a thrown error on one path and an error result on another for the same condition. Do not wrap every function in a custom result abstraction; add one where it makes expected outcomes explicit.

### Own reporting at boundaries

Report a failure at the layer that owns the response or lifecycle. Avoid logging and rethrowing the same failure through every layer.

Use an intentional structured logging contract for applications. A CLI may write intended output to standard output and diagnostics to standard error. Reusable libraries must not print incidental diagnostics during ordinary calls.

Translate internal errors into stable external responses. Do not expose stack traces, provider error objects, or internal storage records. Treat error causes as potentially sensitive when logging or serializing.

## 21 Comments and documentation

Write code whose names and structure explain the ordinary path. Use comments for constraints, reasons, invariants, tradeoffs, and non-obvious interoperability behavior. Do not narrate what a statement already says. Remove commented-out code and stale explanations.

Use `//` for implementation comments and `/** ... */` for API documentation.

Document exported library APIs and any application contract not clear from its name and types. Explain units, mutation, absence, ordering, thrown failures, cancellation, and lifecycle obligations when relevant.

Do not repeat TypeScript annotations in JSDoc with `{Type}` syntax. Use `@param` and `@returns` when they add semantic information, and `@throws` for intentional failure conditions without implying TypeScript checks them.

Keep examples small, valid, and consistent with the API. Document surprising behavior beside the declaration that owns it.

A TODO must describe the missing work and a concrete trigger or issue reference, not an anonymous "fix later." A permanent workaround should explain the constraint and its source.

Keep source comments in a consistent project language, English by default for shared codebases. User-facing strings follow the product's localization policy.

## 22 Tests and fixtures

Tests must follow the same naming, formatting, type-safety, and promise rules as production code.

Name test cases by observable behavior and the condition that matters: "returns no jobs when the queue is empty." Avoid names that merely repeat a private helper's name.

Keep setup, action, and assertion easy to distinguish, using blank lines and labels only when they help. A test should exercise one coherent behavior, which may need several assertions.

Use small, realistic, explicitly typed fixtures. Prefer `satisfies` to fixture casts, and do not create a valid-looking object with `{} as Account`. Do not weaken fixtures with `any` or blanket casts. Create fresh mutable fixtures for each test; no test may depend on another's state or execution order.

Control clocks, randomness, environment reads, and external services at their boundaries. Avoid arbitrary sleeps when a deterministic event or controllable clock is available.

Assert important returned values, state changes, and failures. Avoid assertions about internal call sequences unless the sequence is part of the public contract.

Await asynchronous assertions and operations. A passing test must not leave an unobserved promise that can reject after it completes.

Use snapshots only when reviewing the full representation is meaningful. Prefer focused assertions for a few fields over a large opaque snapshot.

Use `@ts-expect-error` with an explanation in negative type tests, where it asserts that a program is rejected rather than marking temporary production debt:

```ts
// @ts-expect-error -- Retry count must be numeric.
calculateRetryDelayMs("first", { maxRetries: 3, delayMs: 100 });
```

Keep type tests in a dedicated file or tool-supported type-test area. Do not execute intentionally invalid examples as ordinary runtime tests.

## 23 JSX and components

Applies only where the project uses JSX, and prescribes no framework.

Name the file after its primary component or cohesive component API. Keep one primary component API per file; small private render helpers and intentionally compound components may stay together. Extract independently reusable components into their own files.

Use a named props type when props need a reusable contract or would make the signature hard to read. Use discriminated unions for mutually exclusive modes instead of many unrelated boolean props.

Keep JSX expressions small, moving substantial branching, parsing, and transformations into named values or functions. Do not put nested ternaries in markup.

Spread props only from a known, intentional contract, with explicit override order. Do not pass an arbitrary internal object through a component boundary to avoid listing its fields.

Use stable identities for keyed collections. Follow the framework's lifecycle and purity rules. Keep asynchronous operations and mutations in the framework-supported boundary, not in expressions evaluated during rendering.

Use a framework-required default export only at the file that requires it. Ordinary shared components use named exports.

## 24 Enforcement and exceptions

### Automate what can be automated

Every project must expose and run these checks in CI:

1. Formatting in check mode.
2. A type-aware lint check.
3. Full TypeScript checking for every handwritten compilation unit, including tests and scripts.
4. The project's relevant tests.
5. A build or execution check for the actual runtime profile.

A successful build does not prove type checking ran. Do not type-check a single file in isolation as a substitute for the configured check. Use review as a final pass alongside these checks.

Pin one formatter configuration; Oxfmt or Prettier can implement the policy, and the choice is a project decision. Import sorting requires a configured sorter; do not assume every formatter sorts imports.

Use ESLint with typescript-eslint, or an equivalent type-aware toolchain, to enforce the semantic rules. The behavior is required even when rule names differ.

A practical enforcement map:

- Strict checking and erasable syntax: TypeScript configuration.
- No enums, namespaces, or decorators: compiler where applicable plus syntax restrictions.
- Kebab-case files: filename rule or repository check, with tool-path exceptions.
- Braces, strict equality, no nested ternaries: lint rules.
- Type-only imports and exports: typed lint rules and `verbatimModuleSyntax`.
- No explicit or propagated `any`: no-explicit-any and unsafe-operation rules.
- No non-null or double assertions: assertion restrictions.
- Handled promises and valid async callbacks: floating-promise and misused-promise checks.
- Exhaustive unions: `never` checks and an exhaustiveness lint rule.
- No unused bindings: one configured lint owner.
- Import boundaries and cycles: import/dependency rules where feasible.
- Public contracts and justified exceptions: review plus targeted lint rules.

A broad recommended preset is a starting point, not a complete implementation of this guide. A rule that accepts `void promise` does not prove rejection handling. [19]

Configure unused-parameter handling consistently with the `_name` exception. Do not enable competing compiler and lint policies for the same case.

### Handle exceptions visibly

Permitted integration exceptions are specific: required filenames, required default exports, accurately scoped declaration files, contextual inference, and a necessary localized assertion.

For a lint suppression, identify the exact rule and explain the concrete reason on the smallest applicable line or block. Do not disable a rule for an entire file to avoid one diagnostic.

A temporary `@ts-expect-error` in production code must include the external limitation or tracked issue and a removal condition, and must not skip implementing a correct contract.

Keep generated-code exclusions narrow. Do not classify handwritten code as generated to evade checking.

New exceptions that change the standard must be recorded in the project addendum or an explicit revision of this guide. Individual files must not quietly introduce a different style. The ESM and erasable-syntax baseline remains mandatory.

## 25 Project configuration profiles

### Shared compiler baseline

Store this policy in a base configuration inherited by every relevant compilation unit. It is a shared fragment, not a complete configuration for an arbitrary runtime.

```json
{
  "compilerOptions": {
    "strict": true,
    "erasableSyntaxOnly": true,
    "verbatimModuleSyntax": true,
    "isolatedModules": true,
    "moduleDetection": "force",
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noPropertyAccessFromIndexSignature": true,
    "noImplicitOverride": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedSideEffectImports": true,
    "allowUnreachableCode": false,
    "allowUnusedLabels": false,
    "forceConsistentCasingInFileNames": true
  }
}
```

`strict` does not include all of these. `erasableSyntaxOnly` requires TypeScript 5.8 or later. [2][20]

Set `noEmitOnError: true` wherever TypeScript emits output, and `noEmit: true` for checking configurations where another tool owns JavaScript transformation or execution. A library may use a separate declaration build with `declaration: true` and `emitDeclarationOnly: true`; that build must not set `noEmit: true`.

Choose `target`, `lib`, `types`, `jsx`, and output settings for the actual environment. Do not add DOM globals to a backend package or Node globals to browser code to silence a missing-name error. Separate browser, server, worker, tooling, and test configurations when their globals or module behavior differ; each must inherit the shared checks.

Keep `skipLibCheck` off by default. If an upstream declaration issue requires enabling it temporarily, record the exact limitation and track its removal. It must not disable checking of handwritten application code.

Enable `resolveJsonModule` where needed. For side-effect asset imports, use accurate tool-provided or narrowly scoped declarations. Do not use `declare module "*"` to make unresolved imports pass. [21]

### Select an ESM profile

| Profile                              | Required module settings                                                                                 | Relative code imports             |
| ------------------------------------ | -------------------------------------------------------------------------------------------------------- | --------------------------------- |
| TypeScript emits JavaScript for Node | `module: "NodeNext"`, `moduleResolution: "NodeNext"`                                                     | Emitted extensions, usually `.js` |
| A bundler consumes TypeScript        | `module: "ESNext"`, `moduleResolution: "Bundler"`, `noEmit: true`, `allowImportingTsExtensions: true`    | Explicit `.ts` or `.tsx`          |
| Node executes TypeScript directly    | `module: "NodeNext"`, `moduleResolution: "NodeNext"`, `noEmit: true`, `allowImportingTsExtensions: true` | Actual `.ts` or `.mts`            |

Verify the bundler's extension handling and dependency interoperability, and use explicit ESM package metadata. [3][7]

Emitted Node code imports `"./worker.js"` even when the source is `worker.ts`; direct TypeScript execution imports `"./worker.ts"`.

A project that executes TypeScript source and also emits JavaScript may adopt `rewriteRelativeImportExtensions: true` as a documented build variation. It rewrites supported relative TypeScript extensions, not arbitrary aliases or package mappings. [22]

Do not use `moduleResolution: "Bundler"` for code that must run as unbundled Node output merely because it accepts more import forms.

For a published package, verify the emitted JavaScript and declarations against intended consumers, and define public exports explicitly. A successful development bundle does not prove consumers can resolve the distributed package.

### Project addendum

Record only project-dependent decisions:

- Supported runtimes and deployment targets.
- Pinned compiler, validator, formatter, linter, and package-manager versions.
- Compilation units and their ESM profiles.
- Public package entry points and supported aliases.
- Tool-required filename, default-export, or asset-import conventions.
- Commands for format checking, linting, type checking, testing, and building.
- Any documented compatibility limitation with a removal condition.

Framework architecture, database choice, routing strategy, and product-specific date or localization policies belong in project documentation and must not change the common code style accidentally.

## 26 References

The prescriptions in this guide are project policy. Zod examples use the Zod 4 API. The references support language and library behavior and provide further reading. Google's guide influenced the organization and use of explicit rules and examples; its environment-specific choices are not adopted wholesale. [1]

1. [Google TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html)
2. [TypeScript erasableSyntaxOnly](https://www.typescriptlang.org/tsconfig/erasableSyntaxOnly.html)
3. [Node.js TypeScript support](https://nodejs.org/api/typescript.html)
4. [Prettier options](https://prettier.io/docs/options)
5. [Node.js ECMAScript modules](https://nodejs.org/api/esm.html)
6. [TypeScript verbatimModuleSyntax](https://www.typescriptlang.org/tsconfig/verbatimModuleSyntax.html)
7. [TypeScript choosing compiler options](https://www.typescriptlang.org/docs/handbook/modules/guides/choosing-compiler-options.html)
8. [TypeScript everyday types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)
9. [TypeScript satisfies operator](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html#the-satisfies-operator)
10. [TypeScript exactOptionalPropertyTypes](https://www.typescriptlang.org/tsconfig/exactOptionalPropertyTypes.html)
11. [TypeScript inferred type predicates](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-5.html#inferred-type-predicates)
12. [Zod 4 migration guide](https://zod.dev/v4/changelog)
13. [TypeScript narrowing and exhaustiveness](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking)
14. [Zod basic usage](https://zod.dev/basics)
15. [Zod schema API](https://zod.dev/api)
16. [Zod 4 boolean text parsing](https://zod.dev/v4#stringbool)
17. [TypeScript noUncheckedIndexedAccess](https://www.typescriptlang.org/tsconfig/noUncheckedIndexedAccess.html)
18. [TypeScript object types and readonly properties](https://www.typescriptlang.org/docs/handbook/2/objects.html#readonly-properties)
19. [typescript-eslint no-floating-promises](https://typescript-eslint.io/rules/no-floating-promises/)
20. [TypeScript 5.8 release notes](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-8.html)
21. [TypeScript noUncheckedSideEffectImports](https://www.typescriptlang.org/tsconfig/noUncheckedSideEffectImports.html)
22. [TypeScript rewriteRelativeImportExtensions](https://www.typescriptlang.org/tsconfig/rewriteRelativeImportExtensions.html)
23. [TypeScript resolveJsonModule](https://www.typescriptlang.org/tsconfig/resolveJsonModule.html)
24. [TypeScript module resolution and paths](https://www.typescriptlang.org/docs/handbook/modules/reference.html#paths-does-not-affect-emit)
25. [V8 stack trace API](https://v8.dev/docs/stack-trace-api)
26. [JavaScript Error constructor and cause](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error/Error)
