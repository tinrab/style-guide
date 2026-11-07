# Rust Style Guide

A consistent style for handwritten Rust in libraries, applications, tools, games, graphical software, and systems components. It focuses on what a reader sees: names, signatures, types, expressions, ownership, errors, and module boundaries.

Use direct, conventional Rust. Make a value's meaning, data ownership, and an operation's possible outcomes apparent at the point of use. Add an abstraction when it expresses a real concept or removes meaningful duplication.

The [official Rust Style Guide][style] wins any conflict with a rule here. Use the [Rust API Guidelines][api] for supporting API conventions.

## How to apply this guide

An imperative such as "use," "keep," or "do not" is a requirement. "Prefer" is the default; depart from it for a concrete reason. "May" is an allowed choice. Keep exceptions narrow and explain non-obvious ones next to the code. Correctness and soundness are always required.

API design requirements bind libraries intended for other projects. For code confined to one application they are strong recommendations: favor direct solutions to actual needs over speculative reuse. Formatting, safety, and implementation restrictions still apply. Favor code readers can understand and maintain, conventions tools can process consistently, and familiar idioms. Apply rules consistently so reviews can focus on behavior.

A rule applies when its feature is used. Async Rust, Tokio, `std`, and any particular external crate are optional. Generated or vendored code may keep its own conventions. Examples may assume declarations shown or described nearby.

## Formatting

### Let rustfmt determine layout

Format with the repository's approved formatter, normally `cargo fmt`, using the shared configuration: four spaces, no tabs, `max_width = 100`. Let it determine line breaks, trailing commas, indentation, and compact forms, and follow the [official conventions][style]. Do not manually align unrelated statements or maintain a competing layout. Prefer block indentation to alignment under an opening delimiter. Multiline comma-separated lists take a trailing comma except where syntax forbids it, such as after a struct update expression.

When the pinned formatter supports them, also set `hex_literal_case = "Lower"`, `format_macro_matchers = true`, and `imports_granularity = "Crate"`. Format macro matchers as ordinary Rust where possible. Do not require nightly solely for a formatter option. Follow source-level rules even when the formatter cannot enforce them. [rustfmt configuration][rustfmt-config]

Separate items and statements with zero or one blank line: one between logical steps, with related statements together. Remove trailing whitespace, including on blank and comment lines, using escapes to preserve literal values. Do not organize a file with decorative comment banners or large blocks of empty lines; use modules, and ordinary headings in documentation.

Use `#[rustfmt::skip]` only for meaningful manual alignment, such as a data table or matrix, or to preserve generated layout (prefer a reproducible generator). Explain a non-obvious reason. Personal preference is not sufficient.

### Make literals readable

Group long numeric literals meaningfully, such as `1_000_000` or `0xfeed_beef`, with lowercase hexadecimal digits. Name repeated literals that represent a domain rule, unit, sentinel, or limit. Keep an obvious local value inline when a constant would add no meaning.

Use escapes for invisible or ambiguous characters. Use a raw string when it makes quotes or backslashes easier to read, with the fewest delimiters that work.

## Naming

### Use conventional Rust names

| Item                                  | Convention             | Example                                 |
| ------------------------------------- | ---------------------- | --------------------------------------- |
| Modules and source files              | `snake_case`           | `record_id.rs`                          |
| Functions, methods, fields, variables | `snake_case`           | `read_header`, `byte_len`               |
| Types, traits, enum variants          | `UpperCamelCase`       | `RecordId`, `DecodeError`, `Ready`      |
| Constants and statics                 | `SCREAMING_SNAKE_CASE` | `MAX_NAME_BYTES`                        |
| Declarative macros                    | `snake_case`           | `define_flags!`                         |
| Boolean queries                       | A predicate            | `is_empty`, `has_children`, `can_retry` |

Treat acronyms as words in type names: `HttpClient`, `Utf8Error`, `Uuid`, not `HTTPClient` or `UUID`. Preserve established spellings in external interfaces when necessary. For a reserved word, use a raw identifier such as `r#type` or a trailing underscore such as `type_`; do not misspell it. See the [API naming conventions][naming].

### Name the concept and the unit

Choose names that stay meaningful outside their declaring statement. Prefer `record_count`, `byte_offset`, and `timeout` to `n`, `pos`, and `value` when those distinctions matter. Short names suit a small, obvious scope: a simple loop index, coordinates in a formula, or `T` in a generic container.

Do not shorten words into unfamiliar abbreviations or drop letters from a word's middle; established names such as `len`, `id`, `io`, and `utf8` are fine. Do not use vague suffixes such as `Manager`, `Helper`, `Processor`, or `Data` instead of identifying a responsibility. Avoid names such as `new_value2` that describe editing history. Do not encode the storage type into a name unless it distinguishes relevant values.

Put units in the type where practical, otherwise in the name: `elapsed_ms`, `size_bytes`, `angle_radians`. Do not make a reader infer whether a number counts elements, bytes, or seconds.

### Name methods for their behavior

Use `name()` for a property accessor, not `get_name()`. Keep `get` for established lookup semantics, such as a possibly missing indexed element.

| Behavior                                                      | Typical name                       |
| ------------------------------------------------------------- | ---------------------------------- |
| Borrow a property, or return a small `Copy` property by value | `name()`, `items()`                |
| Mutably borrow a property                                     | `name_mut()`, `items_mut()`        |
| Replace a property with an owned value                        | `set_name(value)`                  |
| Replace and return the previous value                         | `replace_name(value)`              |
| Return the old value and leave `None` or a valid default      | `take_name()`                      |
| Access a property through a closure                           | `with_name(f)`, `with_mut_name(f)` |

Conversions: `as_` is a cheap borrowed view (`as_bytes()`), normally taking `&self` or `&mut self`. `to_` retains the original (`to_string()`), normally taking `&self`, and may allocate or do substantial work. `into_` consumes `self` (`into_inner()`). A small `Copy` receiver may be taken by value where conventional. Use `try_` to distinguish a fallible alternative in a meaningful pair; a naturally fallible operation such as `parse` or `open` needs no redundant prefix. [Conversion naming][naming]

Use `new` for the primary constructor and `new_with_<detail>` for an additional construction path when that pattern fits. A constructor returning `Result` must use `try_new` or `try_new_with_<detail>`. For a value-to-value conversion, use `from_<source>` with the corresponding `From` implementation when that contract applies, and a fallible conversion for validation. Do not rename required standard trait methods or externally imposed APIs.

For collection-like APIs, provide `is_empty` alongside `len`. Name borrowed, mutable, and consuming iteration methods `iter`, `iter_mut`, and `into_iter` when they have those meanings.

### Name generic parameters and lifetimes proportionately

Prefer short, indicative single-letter generic names, per the official guide: `R: Read`, `E`, `K`, `V`, and `T` or `U` for broadly unconstrained types. A short descriptive name is acceptable when it materially clarifies distinct roles, but not as the default. Associated types such as `Item` and `Output` describe their roles. [Official generics style][style-items]

Elide lifetimes when Rust's rules express the relationship, and use `'_` where an anonymous lifetime suffices. Give named lifetimes descriptive names, especially on types: `'input`, `'arena`, `'request`. Single letters suit very generic types such as `Cow`, up to two simple local input/output relationships, and local higher-ranked bounds such as `for<'a>`; use descriptive names beyond that. Do not add lifetime parameters to make a signature look more complete.

## Imports

### Keep imports explicit and grouped

Group module-level imports in this order, with a blank line between groups:

1. `std`, `core`, and `alloc`.
2. External crates.
3. Items from the current crate.

Version-sort within each group, so `item8` precedes `item16`, with `self` and `super` first where applicable. Combine imports from the same crate, in all three groups, into one `use` with nested brace groups for submodules. Keep imports separate when their visibility or conditional attributes differ. Let the formatter handle ordering and line breaks, and configure merging with `imports_granularity = "Crate"` when supported rather than assuming it happens by default. [Import formatting][style-items]

```rust
use std::{
    collections::{BTreeMap, BTreeSet},
    io,
    path::{Path, PathBuf},
};

use tokio::task::JoinHandle;

use crate::record::{Record, RecordId};
```

Do not use wildcard imports except:

- `use super::*` in inline test modules.
- A local import of enum variants inside a method on that enum or a function primarily concerned with it, when it makes patterns clearer.
- An established third-party prelude justified during review.
- A private adapter module combining generated bindings for the same C++ namespace from several crates. It may glob re-export those bindings internally but must itself remain private.

Keep other public re-exports explicit, including in non-leaf modules. Avoid defining new preludes. Prefer qualified enum expressions such as `State::Ready`, except prelude variants such as `Some` and `Ok`; a local variant import for patterns does not change this. [Enum expression formatting][style-expressions]

### Import types and qualify free functions

Import ordinary type and trait names and use them unqualified. Call free functions from other modules through the imported module name (`fs::read(path)`), so an unqualified function normally belongs to the current module.

Keep commonly ambiguous types qualified, such as `io::Error`, or give them a specific alias such as `IoError`. Qualification also suits a large related family of types, macros or generated code where imports are impractical, or a dependency-heavy module where imports add little clarity. Do not rename a type just to save characters.

Directly imported free functions are exceptions when their names already disambiguate them, as in C++ bindings, or when the module name collides across crates. Keep exceptions consistent within the module.

### Scope trait imports to their use

A trait needed only for method resolution in one function should normally be imported there, with `as _` when the name is unnecessary: `use anyhow::Context as _;`. Keep the import at module scope when many functions use it or signatures name the trait.

Use `crate::` for clear paths within the current crate and `super::` for a genuinely local parent relationship. Avoid long `super::super::...` chains, which make a module's position part of every import.

## Modules and files

### Organize modules around concepts

A module should own a coherent concept, such as a parser, an identifier, a geometry operation, or closely related types, and be named for it. If a leaf module has a primary user-defined item such as `RecordId`, name the module and file after it: `record_id.rs`. Otherwise choose a descriptive name freely.

Keep tightly related types together, without requiring one file per type. Do not let `utils`, `common`, or `helpers` collect unrelated code. Put a helper beside its callers until it has a clear independent responsibility.

An illustrative arrangement, using names from the actual domain (a small crate may keep its implementation in `lib.rs`):

| Path                                     | Responsibility                                                        |
| ---------------------------------------- | --------------------------------------------------------------------- |
| `src/lib.rs`                             | Crate documentation, public surface, and small root-level definitions |
| `src/record.rs`                          | The main record abstraction or its module facade                      |
| `src/record/decode.rs`, `encode.rs`      | Implementation owned by the record module                             |
| `src/main.rs`, when present              | Executable initialization and coordination                            |
| `tests/record_roundtrip.rs`, when useful | A test of the crate's public interface                                |

Prefer `record.rs` plus `record/child.rs` for new module trees; existing, consistent `record/mod.rs` layouts are acceptable. Do not introduce `#[path]` merely to produce an unusual hierarchy. [Rust modules][modules]

### Separate a module's public surface from its implementation

Keep implementation modules private when callers only need their exported types and functions, and re-export the deliberate public API explicitly:

```rust
pub use decode::{DecodeError, Decoder};
pub use encode::Encoder;

mod decode;
mod encode;
```

Keep non-leaf modules focused on out-of-line module declarations and re-exports, with API implementations in leaf modules. A small amount of core code tying children together is allowed. Inline test modules and cross-language binding modules such as `#[cxx::bridge]` do not stop an implementation module from being a leaf.

### Use the narrowest useful visibility

Start private. Use `pub(crate)` for a crate-internal interface and `pub` for the intended external API. Do not use `pub(in ...)`, `pub(super)`, or `pub(self)`. When several modules need a private detail, place it in a suitable shared module or expose a deliberate `pub(crate)` interface. Do not make an item public for a test's or distant module's convenience.

Keep fields private when construction or mutation must preserve an invariant; public fields suit a deliberately transparent record with no hidden validity rules. A public signature must not accidentally expose a private implementation type or a dependency callers should not need to know. [Visibility and privacy][visibility]

### Keep a predictable order within a file

Put module documentation and inner attributes first. Any needed `extern crate` declarations come before other items, sorted alphabetically. Put `use` statements, including re-exports, before out-of-line `mod name;` declarations, and both before other items. Version-sort module declarations as well as imports. Preserve ordering required by macros; do not automatically move `#[macro_use]` modules. Keep a type near its inherent implementation, related trait implementations nearby, and inline tests at the end. [Official item order][style-items]

Within an inherent implementation, group constructors, queries, transformations, and mutation operations in a consistent order, with private helpers near the operations they explain. Prefer conceptual to alphabetical order, and avoid scattering one type's methods across many files without a clear reason.

## Functions

### Give each function one coherent job

A function should express one operation at its own level of abstraction. Extract a helper when it names a meaningful step, isolates a separate rule, or removes substantial duplication, not to shorten a function by moving a one-line expression.

Keep state changes and early exits visible. A reader should not need to trace layers of callbacks to discover that a function mutates an argument, consumes a value, or reports failure.

Use an inherent method when the operation belongs to a type and its invariants, and a free function when it has no natural owner. Do not introduce an empty struct as a function namespace.

### Accept the data the operation needs

Use the narrowest parameter type that expresses the contract and serves the actual callers.

| Need                     | Prefer       | Avoid unless the wrapper matters |
| ------------------------ | ------------ | -------------------------------- |
| Read text                | `&str`       | `&String`                        |
| Read a sequence          | `&[T]`       | `&Vec<T>`                        |
| Modify existing elements | `&mut [T]`   | `&mut Vec<T>`                    |
| Read a filesystem path   | `&Path`      | `&PathBuf`                       |
| Borrow an inner value    | `&T`         | `&Box<T>`, `&Arc<T>`             |
| Read an optional value   | `Option<&T>` | `&Option<T>`                     |

Use the container when the operation needs its capabilities: resizing requires `&mut Vec<T>`, replacing an optional value can require `&mut Option<T>`, and duplicating a shared handle can require `Arc<T>`. Noncontiguous text such as a rope may need its own borrowed interface. Callers can adapt owned optional data with `as_ref` or `as_deref`, which preserve ownership. [Option adapters][option]

Pass small `Copy` values by value. Take an owned `String`, `Vec<T>`, or other value when the function retains, transfers, or consumes it. Do not accept a borrow and clone it unconditionally when ownership is the real contract.

### Use generics for a useful range of inputs

Use concrete parameters by default in application and implementation code, which should almost never accept `impl Into<T>` merely to save a call-site conversion. In a reusable library, consider `impl Into<T>` when callers commonly hold several convertible types and convenience justifies the more complex signature, inference, and generated code. Do not add conversion generics routinely.

Use `AsRef` for a cheap borrowed view and `Into` for an owned conversion. Convert once near the start of the function when that simplifies the rest. Do not add multiple conversion traits to every parameter or pass a generic conversion object through layers that only need the converted value.

For highly reusable library operations that only traverse inputs, accept an iterator or `impl IntoIterator`. Use a slice when the function needs contiguous access or indexing, or in application code when it is simpler and iterator flexibility or allocation avoidance is not needed. Use `impl AsRef<Path>` or a similar borrowed conversion when actual callers benefit. See the [API flexibility guidelines][flexibility].

Require only the bounds the operation needs, on the method rather than the entire type when practical. Do not add `Clone`, `Send`, `Sync`, or `'static` as routine defaults. Choose `FnOnce`, `FnMut`, or `Fn` by how a callback is invoked.

### Make outputs express the outcome

Return a value instead of mutating an output argument unless in-place mutation, reuse of a destination, or an external interface is part of the operation.

Choose `Option` or `Result` as described under Errors, and reserve panics as described under Panics. Do not use an empty string, zero, or an arbitrary integer as an undocumented absence or failure sentinel.

Return a named struct when several outputs have distinct meanings. A small local pair is fine when its interpretation is conventional, such as a key and value or the two parts of a split.

Prefer an iterator return type, concrete or `impl Iterator`, for reusable traversals and transformations. Returning `Vec<T>` suits cases where most callers need one, the implementation naturally constructs or owns one, random indexing is likely, the operation explicitly creates an owned collection, or a direct loop is clearer at acceptable allocation cost. Application code may choose a vector for simplicity when iterators offer no clear benefit.

### Make arguments understandable at call sites

Do not expose a string of positional flags or several indistinguishable primitive arguments. Use an enum for a meaningful choice, as in `find_name(names, query, CaseSensitivity::Insensitive)`, and an options struct for related settings.

A boolean is appropriate when it directly represents a boolean property. The restriction concerns opaque calls such as `find_name(names, query, true)`. When an existing interface requires an unclear boolean, consider an inline comment such as `/* case_sensitive */ true`.

Keep required inputs explicit. Use an options struct or builder when callers benefit from named optional settings, but not a builder for every constructor.

## Structs and enums

### Represent distinct meanings with distinct types

Use a newtype such as `struct RecordId(u64);` when values have different meanings, an invariant deserves enforcement, a type needs its own API, or its representation may need to change independently of callers. A type alias does not prevent values from being mixed; use one only for a useful spelling of an existing type, such as a complicated map or module-local result type, never for validation or type safety.

### Give meaningful records named fields

Prefer named fields when a value contains several pieces of domain data; tuple indexing should stay obvious from local context. A single-field tuple struct suits a newtype, and tuples suit short-lived, conventional groupings. Do not return a tuple such as `(String, usize, bool)` from a public operation when readers need a field legend.

### Make invalid states difficult to construct

Use an enum for mutually exclusive states, with each state's data in its variant: `enum ParseState { Header, Body { remaining_bytes: usize }, Complete }`. Avoid independent `Option` fields and flags whose combinations require a hidden validity table.

Validate at construction and expose methods that preserve the invariant. Reuse standard invariant-bearing types such as `NonZeroUsize` when they fit. Do not create an invalid value and require a later `init` call to repair it.

Avoid a setter when arbitrary replacement would violate the type's rules; expose the actual operation instead. Returning `&mut` to an internal field also permits free mutation, so apply the same reasoning to mutable accessors.

### Use closure accessors for scoped access

Consider `with_property` and `with_mut_property` when a property lives behind interior mutability or changes require follow-up work. A closure can keep a lock or borrow guard inside the abstraction instead of exposing `MutexGuard` or another detail. Accept a suitable closure, often `FnOnce`, and normally return its result. When both read-only and mutable forms exist, use the distinct names; `with_property` may denote mutation if no read-only form exists and the contract is clear.

Document the scope of access and any validation or follow-up work. Invariants must stay valid if the callback panics; checking only after a successful return is insufficient for an invariant required by safe callers or unsafe internals.

### Make defaults meaningful

Implement `Default` when there is a useful, valid default, including for a type with a zero-argument `new() -> Self`, and keep the two consistent.

All-public structs that are bags of optional settings must implement `Default`, so callers can override selected fields with struct update syntax. Keep required identifiers, external resources, and validated values without a natural default as separate required inputs. Do not invent a dummy or invalid value to make `Default` available.

Derive `Default` only when the field defaults collectively express the intended default; otherwise implement it explicitly.

### Choose extensibility deliberately

Mark an enum exposed through a public API used by other teams or independently maintained projects `#[non_exhaustive]` by default. Exceptions are a deliberately permanent closed set, including a stable two-state choice, or a commitment to update every affected caller. A type with few controlled uses may omit it when migration risk is small. Keep internal enums exhaustive when a new state should trigger a review of their matches.

Apply the same default to public structs with public fields across those boundaries. Exceptions are fields expected to stay stable, a commitment to migrate callers, a small controlled set of uses, or a deliberate API permitting struct update syntax such as `Options { limit: 8, ..Default::default() }`.

An external caller cannot construct a `#[non_exhaustive]` struct with a literal, even using `..Default::default()`, so supply constructors, a builder, or other supported mutation paths. Private fields enforce representation boundaries and `#[non_exhaustive]` reserves room for API evolution; neither replaces the other. [The non_exhaustive attribute][nonexhaustive]

## Traits and conversions

### Start with inherent methods

Define a trait to support generic bounds, an `impl Trait` or `dyn Trait` interface, or an extension to a foreign type. Use inherent methods for ordinary behavior of one type. Do not create a trait for every struct or use traits to split a large implementation into categories. A trait is a contract for implementors as well as callers; keep it focused and document relationships among its methods.

In library function parameters, prefer generics or `impl Trait` over `dyn Trait` so callers retain concrete types; code-size concerns can justify dynamic dispatch. Use `dyn Trait` when runtime selection, heterogeneous storage, or an explicit erased interface requires it. Application code may use the simpler suitable interface. Check [dyn compatibility][traits] rather than assuming every trait can be a trait object.

Give extension traits an `Ext` suffix. They add methods to a foreign type or implementors of a foreign trait and are not contracts for generic bounds or dynamic dispatch. Prefer that the underlying type's owner also owns the extension trait; a foundational library or framework may justify an exception for valuable methods expected to become widely known. Keep methods unsurprising and unlikely to collide. A capability trait implemented for several foreign types, intended for generic inputs, is a different kind of trait.

### Implement standard traits when the semantics fit

Low-level reusable libraries should anticipate plausible users' trait needs; application code should add implementations when needed. For a type with public fields, consider the standard traits shared by all its fields, but do not add one with incorrect semantics because deriving it is possible.

Use standard contracts instead of similarly named methods: `FromStr` for parsing text, `From` or `TryFrom` for conversions, `Iterator` for iteration, and `Display` for intended textual presentation.

Prefer `#[derive(...)]` when its behavior is correct. Implement manually when field-by-field behavior is wrong or the derive adds unneeded bounds. Do not derive a trait just because another generic function requests it.

| Trait                | Required semantic decision                                                                                                                                                                                                     |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `Debug`              | Developer-facing output; redact fields that must not be exposed                                                                                                                                                                |
| `Clone`              | Most value types should implement it; external resources may not have meaningful duplication. State whether cloning duplicates data or a shared handle.                                                                        |
| `Copy`               | Implement when implicit duplication fits and the type can reasonably remain `Copy` as it evolves. Error types should implement it whenever representation and semantics permit, without removing useful context to achieve it. |
| `PartialEq` / `Eq`   | `PartialEq` provides `==`; add `Eq` only when full equivalence, including reflexivity, holds.                                                                                                                                  |
| `PartialOrd` / `Ord` | `PartialOrd` provides comparisons; add `Ord` only when a total order exists.                                                                                                                                                   |
| `Hash`               | Equal values must produce equal hashes. Implement `Hash` and `Eq` for likely map or cache keys when their contracts fit. Add `Ord` and its prerequisites for naturally totally ordered keys.                                   |

Keep equality, ordering, and hashing consistent. Do not claim a stronger contract to enable comparisons, or use an arbitrary field ordering that misrepresents the domain. Follow each trait's documented contract and the [interoperability guidelines][interoperability].

### Use From for infallible conversions and TryFrom for validation

Implement `From`, not `Into`, when converting into a type you can implement it for, and `TryFrom` when conversion can fail. An infallible conversion must preserve the intended meaning; do not hide a validation panic or surprising information loss inside `From`.

```rust
use std::num::NonZeroUsize;

#[derive(Clone, Copy, Debug, Eq, PartialEq)]
struct BatchSize(NonZeroUsize);

#[derive(Clone, Copy, Debug, Eq, PartialEq, thiserror::Error)]
#[error("batch size must be greater than zero")]
struct InvalidBatchSize;

impl TryFrom<usize> for BatchSize {
    type Error = InvalidBatchSize;

    fn try_from(value: usize) -> Result<Self, Self::Error> {
        NonZeroUsize::new(value).map(Self).ok_or(InvalidBatchSize)
    }
}

impl BatchSize {
    #[must_use]
    fn get(self) -> usize {
        self.0.get()
    }
}
```

When a `From` implementation is a primary construction path, also provide a discoverable inherent constructor such as `new`, `new_from_<source>`, or `from_<source>`, delegating one to the other. A `TryFrom` implementation may similarly have a `try_new` or `try_from_<source>` constructor, with validation in one place. Where `.into()` would leave the result unclear, use a target type or named conversion. [From][from] [TryFrom][tryfrom]

### Keep borrowing traits and operators unsurprising

Use `AsRef` to expose a cheap reference conversion. Implement `Borrow` only when the borrowed value preserves the equality, ordering, and hashing that trait requires. They are not interchangeable. [Borrow][borrow]

Use `Deref` and `DerefMut` only for a deliberate pointer-like abstraction with a natural target, not to simulate inheritance or expose every operation of an internal field. [Deref][deref]

Overload operators only for their established mathematical or logical meanings, and keep indexing faithful to indexed access. Use a named method when meaning would be hidden behind `+`, `*`, or `[]`.

## Ownership and borrowing

### Make ownership visible in the interface

Prefer one clear owner with borrows for temporary access. Pass ownership when the recipient must retain or consume a value. Choose `&self`, `&mut self`, or `self` by whether an operation observes, modifies, or consumes its receiver; small `Copy` receivers may be taken by value to observe, as in `BatchSize::get`.

Do not return references tied to an unrelated temporary. Do not extend a lifetime with a leak, a cast, or an unnecessary `'static` requirement to make an ownership error disappear; adjust the relationship or return an owned value.

Prefer an owned field when the type stands independently, and borrowed fields when it is intentionally a view. Do not give every struct a lifetime parameter to avoid owning a string.

### Clone for an intentional duplicate

Use `clone()` when the code needs a duplicate and the type's cloning semantics fit. Do not add clones mechanically until borrow errors disappear; first check whether the operation should borrow, consume the value, or run in a smaller scope. A clone can reasonably simplify ownership if that choice stays visible. Do not describe all clones as deep copies: cloning a shared handle retains the same object. [Clone][clone]

Prefer unique ownership and borrows to `Rc` or `Arc`. Use shared ownership when the alternative would be unreasonably complex to write, understand, or maintain, and make that tradeoff clear in review. Do not wrap a value merely because several functions receive it. Consider allocation and copying costs when choosing borrows, clones, or shared handles. Write `Rc::clone(&value)` or `Arc::clone(&value)` so the kind of duplication is apparent.

### Keep mutation and borrows local

Use immutable bindings by default, add `mut` where modification occurs, and keep the mutable phase as small as possible. Use a short block or helper to limit a borrow when that clarifies the next step.

Prefer ordinary borrowing to interior mutability. If interior mutability is part of a type's design, make the reason and borrowing rules clear in that type.

Use shadowing for a direct refinement, such as raw text becoming parsed data, when one conceptual name helps. Use different names when both values remain relevant or differ in units or meaning.

### Use take and replace for intentional moves

Use `Option::take`, `mem::take`, or `mem::replace` to move a value out of a field while leaving a valid replacement, not a clone to work around a move when the original is meant to be consumed. The replacement must preserve the type's invariant, and `mem::take` requires a default that is valid in that position. [Option::take][option] [mem::take][take]

## Control flow and expressions

### Bind values with patterns

Use `if let` when one variant matters, `let ... else` for a guard that exits early, and `match` when several outcomes need distinct behavior. Do not check `is_some()` or `is_ok()` and then unwrap the same value. These forms tie a value's existence to the scope that uses it, so prefer them to a separate check whose conclusion a later operation merely assumes.

Destructure a struct or tuple when it gives used fields clear names, and use `..` for intentionally irrelevant fields rather than binding a long list of unused ones.

### Make match coverage deliberate

Match a closed enum exhaustively when every state requires a decision. Avoid `_` when a future variant should force a review. Use a wildcard when the fallback genuinely applies to every remaining value.

For a foreign `#[non_exhaustive]` enum, prefer naming all known variants in explicit arms, including those with identical fallback behavior, plus a separate sensible `_` arm for future variants; this avoids the redundant wildcard-or pattern Clippy discourages. It documents today's cases, but new variants still need not cause a compilation error, so do not claim exhaustive future checking.

Use `if let` or `let ... else` when only particular variants matter; a `match` naming just those variants with a wildcard is acceptable when clearer. Do not omit a meaningful case to silence the compiler.

Use `matches!` for a simple predicate and a match guard to keep a condition close to its case. Move substantial work out of patterns and guards.

### Prefer simple expressions and early exits

Use an `if` or `match` expression to choose a value instead of assigning an uninitialized binding in each branch. Use early returns for invalid inputs and completed cases, keeping the main operation shallow.

Use the tail expression for a function's final result. Reserve `return` for an early exit or clearer control flow, and do not add it to every match arm.

Avoid nested ternary-like expressions, long `&&` and `||` chains with hidden side effects, and comparisons such as `condition == true`. Name a compound condition when the name explains a domain rule. Do not shorten an expression by hiding a mutation or failure in an innocently named helper.

### Use combinators when they state the operation clearly

Use `map`, `and_then`, `ok_or`, `transpose`, and related methods for straightforward transformations, and `match` when branches need explanation, several statements, or distinct effects.

Use the lazy form (`unwrap_or_else`, `ok_or_else`, `or_else`) when constructing a fallback does meaningful work, and the eager form for an available value or trivial error constructor.

Do not turn a failure into absence with `.ok()` unless discarding the error category is intentional. Do not use `unwrap_or_default()` to silently accept malformed data or conceal a required value.

### Add types where inference hides a decision

Let the compiler infer obvious local types. Annotate when parsing, conversion, or collection construction would leave the reader guessing, as in `let value: u32 = input.parse()?;`. Do not add a binding only to repeat a function's return type.

Prefer annotations on bindings over turbofish, especially for generic returns such as `.into()` or `.parse()` and complex iterator-chain results. Use a turbofish when inference requires it or that location is materially clearer; `_` may stand in for inferable parts. Avoid long nested type expressions mid-chain; introduce a named type or intermediate binding.

## Iteration and collections

### Iterate over the values you need

Use element iteration for element operations, `enumerate` when the index is also needed, `zip` for corresponding sequences, and slices or chunks when they express the algorithm.

Use indexing when positions are essential. Use checked access such as `get` when an index can be invalid under the normal contract; direct indexing is appropriate when surrounding code establishes the bounds.

Choose ownership explicitly: `for item in &items` yields shared references, `&mut items` mutable references, and `items` owned elements, consuming a `Vec<T>`.

Do not write `0..items.len()` just to index every element. Do not use `zip` as a length check, because it stops when either input ends; validate equal lengths separately when required. [Iterator methods][iterator]

### Use chains for transformations and loops for procedural work

Use a short chain for a clear transformation, filter, or query, such as `values.iter().any(|&value| value < 0)`. Use a `for` loop when the body has several effects, meaningful branching, an early exit, or error handling easier to follow as statements. Do not use `map` for side effects or collect a temporary vector only to force an iterator to run.

Prefer a loop to `for_each` unless there is a concrete reason. Bind iterator adapter chains to a named variable before consuming them in a `for` loop, rather than building the chain in the loop header. A simple loop over `&items`, `&mut items`, or `items` needs no extra binding.

Keep closures short enough that the enclosing operation stays apparent, moving a substantial body with independent meaning into a named function. Do not impose a fixed chain length.

### Preserve failures while collecting

Collect an iterator of results into `Result<Vec<_>, _>` when any failure should fail the operation. Use `filter_map(Result::ok)` only when intentionally skipping failures is the contract and the lost information does not matter. If the operation must collect both successes and failures, write that explicitly; do not disguise partial success as complete success. [Iterator collection][iterator]

### Use collection operations that express the intent

Use `entry` for a lookup followed by conditional insertion or modification (`*counts.entry(word).or_insert(0) += 1`), and `get_mut` for updating without insertion. Avoid repeated lookups when a single operation states the intent more clearly.

Choose ordered collections or sort explicitly when ordering is part of the result; do not rely on `HashMap` or `HashSet` iteration order. Prefer standard collection types when their semantics fit. [Standard collections][collections]

## Errors with thiserror and anyhow

### Choose the error type for the caller

| Contract                                                                                                       | Return type                        |
| -------------------------------------------------------------------------------------------------------------- | ---------------------------------- |
| Normal absence, or a simple anticipated failure whose cause needs no explanation (checked arithmetic overflow) | `Option<T>`                        |
| Failure cases are part of the interface and callers need details                                               | `Result<T, SpecificError>`         |
| A standard error fully describes the operation                                                                 | For example, `io::Result<T>`       |
| An operation may fail or find no value                                                                         | `Result<Option<T>, SpecificError>` |
| Coordination or reporting needs an error with context                                                          | `anyhow::Result<T>`                |

Apply this within a binary as well as across crates; parsing, validation, and other reusable operations need meaningful contracts inside applications too.

Do not return `String`, `&str`, or `()` as the error contract of a reusable fallible API. A small named error is enough for a single failure category. Do not introduce one project-wide enum accumulating every unrelated error.

A module-local `type Result<T> = std::result::Result<T, DecodeError>` is acceptable when it clarifies repeated signatures, if readers can still identify the error type. Keep `anyhow::Result` qualified when mixing it with typed results in one file.

### Define ordinary error types with thiserror

Use `thiserror` for project-owned error types when external dependencies are appropriate. A manual implementation is acceptable for a concrete target or dependency constraint if it preserves the same `Debug`, `Display`, and `Error` contract.

Variants should describe failures the caller can understand or act on, with meaningfully named fields. Keep messages concise, normally lowercase without a final period, and include the distinguishing facts, but no secret or irrelevant input.

```rust
use std::num::{NonZeroUsize, ParseIntError};

#[derive(Debug, thiserror::Error)]
enum ParseLimitError {
    #[error("limit is not a valid integer")]
    InvalidInteger(#[from] ParseIntError),

    #[error("limit must be greater than zero")]
    Zero,
}

fn parse_limit(input: &str) -> Result<NonZeroUsize, ParseLimitError> {
    let value: usize = input.parse()?;
    NonZeroUsize::new(value).ok_or(ParseLimitError::Zero)
}
```

`#[from]` lets `?` wrap `ParseIntError` and identifies it as the source. Use it only when the conversion is unambiguous and needs no contextual fields. [thiserror][thiserror]

### Preserve sources and contextual fields

Retain an underlying error as an error, not `to_string()` or `format!`, using `#[source]`, a field named `source`, or `#[from]`. When wrapping needs an offset, identifier, or operation name, construct the error explicitly:

```rust
use std::num::ParseIntError;

#[derive(Debug, thiserror::Error)]
#[error("field {index} is not a valid integer")]
struct FieldError {
    index: usize,
    #[source]
    source: ParseIntError,
}

fn parse_field(input: &str, index: usize) -> Result<u32, FieldError> {
    input.parse().map_err(|source| FieldError { index, source })
}
```

A nontransparent wrapper's message should describe its own context, without repeating `{source}` already exposed in the chain. Use `#[error(transparent)]` when the wrapper intentionally delegates presentation and source behavior to the contained error. [Error sources][error] [thiserror attributes][thiserror]

Do not name a non-error field `source` on a derived error; use `input`, `origin`, or another accurate name. Keep public error fields and variants as deliberate API choices. Exposing a dependency's error type is an API dependency even when the derive macro is an implementation detail.

### Propagate with question mark and add useful context

Use `?` for propagation, `match` to recover or distinguish categories, and `map_err` for an intentional conversion or contextual wrapper. Avoid a verbose match that only reconstructs `Ok` and `Err`.

Add `anyhow` context where the code knows what the attempted operation means: `parse_limit(input).context("reading the configured item limit")`. Use `.context("fixed explanation")` for fixed text and `.with_context(|| format!(...))` when construction should happen only on failure. Each layer should add information the lower layer could not know; do not attach "operation failed" repeatedly.

`anyhow` and its context preserve the underlying error and the ability to downcast to it, but repeated downcasting for ordinary control flow means the function should return a typed error. [anyhow Context][context]

Use `anyhow::bail!` and `anyhow::ensure!` in `anyhow::Result` functions, not in functions whose contract requires a specific project error. Do not replace structured categories with message matching.

### Handle a failure or return it

A function that passes an error upward should not also report it. Report at the point responsible for presenting or handling it. Recovery may deserve its own message when it describes a distinct action.

Do not discard a `Result` with `let _ =`, `.ok()`, or an empty `Err` branch without an intentional policy. When ignoring a failure is correct, make the reason visible, without an elaborate wrapper just to suppress a diagnostic.

## Panics and assertions

### Keep expected failures fallible

Use `Result` for malformed input and expected operational failures needing diagnostic context. Reserve panics for defects, violated invariants, or an unmet documented precondition. A codebase may adopt a stricter no-panic policy and return errors for those too.

Do not use production `unwrap()` or `unwrap_err()`. Prefer a pattern, `?`, or a meaningful fallback. If an invariant establishes success and a panic is appropriate, use `expect` with a message stating that invariant:

```rust
#[must_use]
fn active_name(names: &[String], active_index: usize) -> &str {
    names
        .get(active_index)
        .expect("active_index is maintained within the names collection")
}
```

This internal helper's caller must maintain the invariant, because a message does not establish it. If arbitrary callers can supply an invalid index as normal input, return `Option` or `Result` instead.

Use `unreachable!` only when control flow or a maintained invariant rules the case out, not as a fallback for unfamiliar input or a new enum variant. [Panic guidance][panic]

### Make assertions explain the broken relationship

Use `assert_eq!` or `assert_ne!` for comparisons when possible. Always consider a contextual message, especially when the assertion would otherwise need a comment, users may see the failure, diagnostic values matter, or it must distinguish caller misuse from an implementation bug. The message should explain the expected relationship, adding more than a restatement of the condition:

```rust
assert_eq!(
    index.len(),
    records.len(),
    "rebuilding the index should create one entry for each record"
);
```

Use `debug_assert!` only for checks that may be omitted, never for required validation or necessary side effects.

Remove reachable `todo!`, `unimplemented!`, and temporary panic placeholders from accepted code.

Tests may use `unwrap`, `expect`, and assertions when failure should fail the test. Examples for users should normally demonstrate the error-handling style they should adopt.

Developer-facing binaries may panic for otherwise recoverable failures when terminating the tool is appropriate, but prefer ordinary errors when useful reporting or recovery is needed. The production `unwrap` restriction still applies.

### Restrict panic recovery to an optional defensive boundary

Do not call `std::panic::catch_unwind` in first-party code except as defense in depth at a critical entry point in code already making a serious, review-established effort to avoid panics. Do not rely on it for correctness, soundness, required cleanup, or any guaranteed property.

The exception requires an environment where unwinding is possible, so it does not apply to builds requiring `panic = "abort"`, which bypasses `catch_unwind`. Use `Result` for expected failures even with a defensive boundary. [catch_unwind][catch-unwind]

## Concurrency

Do not obtain or inspect the protected contents of a poisoned mutex, including through `PoisonError::into_inner`, `get_ref`, or `get_mut`, or clear poison to bypass this rule. Report the failed operation or stop at an appropriate failure boundary instead of continuing with potentially inconsistent state.

This is a recovery policy. Rust mutex poisoning is advisory and some panics may not poison a mutex, so never rely on it to enforce a safety invariant. [Mutex poisoning][mutex]

Choose synchronization primitives and scheduling facilities appropriate to the codebase.

## Async code and Tokio

### Give async functions ordinary Rust interfaces

Use `async fn` for a named asynchronous operation and an async block for short local composition. Keep ordinary transformations synchronous unless an interface requires async. The same naming, visibility, ownership, and error rules apply.

Do not append `_async` to every name; use a suffix only to clarify an intentionally paired interface. Accept borrowed inputs when ownership is unnecessary, because being async does not by itself require owned arguments. Do not quietly use blocking calls inside an `async fn`.

### Await directly when the work belongs to the current operation

Call `.await` directly when the surrounding operation needs the result, without spawning a task merely to invoke an async function. Propagate ordinary failures with `.await?` and attach context where it explains the operation.

Use `async move` when the future should own its captures. Moving a reference moves the reference, not the referent, and does not make borrowed data `'static`.

Add `Send`, `Sync`, and `'static` bounds only when the interface requires them. `tokio::spawn` requires a `Send + 'static` future and output; a directly awaited future does not. [Tokio spawn][spawn]

### Keep the two error layers of a task visible

Give a spawned task's handle a descriptive name and an intentional owner. Observe its result unless detachment is explicitly part of the contract; dropping a Tokio `JoinHandle` detaches the task. [JoinHandle][joinhandle]

A task returning `Result<T, E>` has both a join result and the operation's result. Handle both:

```rust
use std::io;

use tokio::task::JoinHandle;

async fn finish_load(load_task: JoinHandle<io::Result<String>>) -> anyhow::Result<String> {
    use anyhow::Context as _;

    let result = load_task.await.context("joining the loader task")?;
    result.context("loading the text")
}
```

This helper consumes the handle, so cancelling it before it finishes drops the handle and detaches the task; use this signature only when that is intentional. A concise `.await??` is acceptable when both conversions and their meaning are clear; separate the layers when they need different context.

Keep runtime construction at an explicit application or embedding boundary. Reusable helpers should expose async operations; do not put `#[tokio::main]` on ordinary helpers or call `block_on` from an ordinary async function. [Tokio runtime integration][bridging]

## Text, paths, and numbers

### Distinguish text from bytes and paths

Use `str` and `String` for UTF-8 text, `[u8]` and `Vec<u8>` for bytes, `Path` and `PathBuf` for filesystem paths, and `OsStr` or `OsString` when an operating-system string is the actual value.

Do not require a path to be UTF-8 merely to pass it through the program. Join paths with path operations, not separators in strings. Make lossy conversion an explicit presentation decision, not a silent change to an identifier.

Distinguish byte offsets, Unicode scalar values, and user-perceived characters. `str::len()` counts bytes, and a byte offset is not automatically a valid string boundary. Use text operations that express the intended unit. [str][str] [Path][path]

### Use formatting for readable text construction

Use `format!` for a formatted owned string and `write!` for a destination. Prefer them over chained `+` when combining more than two strings, and over `str::replace` when substituting into a string literal. Prefer captured arguments such as `format!("item {index}")` when clear. Use `push` and `push_str` for straightforward incremental assembly.

Use `Display` for intended textual presentation and `Debug` for diagnostics. Do not treat a derived debug representation as a stable format for serialization or identifiers. [Debug formatting][debug]

### Make numeric meaning and conversion explicit

Use `usize` for in-memory lengths and indices, a fixed-width integer when a format or value range requires it, and `Duration` for a time span instead of an undocumented integer count.

Prefer `From` and `TryFrom`, as in `u32::try_from(len)?`, to an `as` cast that can truncate, change sign, or lose precision. If a cast intentionally has those effects, make the policy or range argument visible.

Choose checked, saturating, or wrapping arithmetic when the distinction matters, and do not rely on build mode to define overflow behavior. Handle floating-point NaN, infinities, and comparison according to the operation's meaning, without mechanically substituting an epsilon comparison. [Numeric conversions][casts] [Overflow][overflow]

## Documentation and comments

### Document the contract where it is declared

Use `///` for item documentation and `//!` at the top of a file for a module or crate overview. Write Markdown and place documentation before the item's attributes. Prefer directly written doc comments to macro-generated documentation. Describe what the caller needs: the operation, input interpretation, output, failure conditions, ownership implications, and relevant invariants.

Document public APIs; consider `missing_docs` to enforce coverage. Document private items when their purpose or assumptions are not obvious. Do not add boilerplate that repeats the signature or explains basic Rust. A self-explanatory enum variant may omit documentation, and a narrow lint exception is appropriate when a comment would say nothing useful.

Use `# Errors`, `# Panics`, and `# Safety` sections when applicable. Always document panic conditions caused by malformed inputs and when an error occurs. For unsafe functions and traits, explain the obligations of using or implementing them safely. Link related items with rustdoc links. [API documentation][documentation]

### Follow ordinary Rust comment formatting

Prefer line comments to block comments. Put one space after `//`, `///`, or `//!`, and normally write complete, capitalized, punctuated sentences. Put a comment on its own line when practical, with one space before a trailing comment. Inline block notes may omit sentence punctuation.

Limit comment-only lines to 80 characters excluding indentation, within the 100-character code width. Use `//` for implementation details and private TODOs even beside public documentation. [Official comments and attributes][style]

### Explain decisions in implementation comments

Use `//` to explain an algorithm, invariant, unusual ordering, constraint, or why a surprising choice is necessary. Remove comments that translate the next statement into English.

Put an assertion's failure explanation in its message, and a safety argument immediately beside the unsafe operation it justifies.

Do not leave commented-out implementations. A TODO should describe a specific remaining change and why it matters, and must not be the only specification of behavior the current code must provide.

### Keep examples representative

Show the intended way to use an API, including ordinary error propagation. Cover enough of the crate for readers to understand how to use it; every function does not need an example. Prefer small examples with meaningful names and enough context, and avoid unrelated setup before the API becomes visible.

Keep documentation examples valid and executable where practical. Use `compile_fail` for an intentional compile failure and `no_run` for a valid example needing an unavailable environment. Do not use `ignore` to conceal a broken example. [Rustdoc examples][doctests]

## Tests

Use the repository's established test framework; otherwise `#[test]` for unit tests and the runtime's test attribute for async tests.

### Write tests that state the behavior

Prefix every unit, integration, and async test function with `test_`, then name the case and expected outcome: `test_rejects_zero_batch_size`, `test_returns_none_for_unknown_name`. Ordinary helpers do not need the prefix. Keep setup local and the assertion easy to find.

Prefer a few direct assertions about one behavior. Use table-driven cases when inputs share a contract, with enough case context in failures to identify the input. Avoid a helper that hides both action and assertion behind a generic name.

Keep unit tests beside the implementation in `#[cfg(test)] mod tests` and use integration tests for behavior through the public interface. Do not expand production visibility to inspect an implementation detail. [Test organization][test-organization]

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_preserves_positive_batch_size() -> Result<(), InvalidBatchSize> {
        let size = BatchSize::try_from(8)?;

        assert_eq!(size.get(), 8);
        Ok(())
    }

    #[test]
    fn test_rejects_zero_batch_size() {
        assert_eq!(BatchSize::try_from(0), Err(InvalidBatchSize));
    }
}
```

### Assert the contract instead of incidental representation

Compare structured values and error variants when those are the contract. Do not assert an error's entire display string or a type's debug output unless that exact text is the behavior under test.

Include relevant boundary and failure cases. Test an invariant's observable consequences rather than repeating internal steps in a second function and comparing.

Keep inputs and ordering deterministic unless the test concerns randomness or ordering, and do not assume hash collection order. For async tests, use the runtime's test facilities and explicit completion signals; an arbitrary sleep does not prove completion.

## Attributes, macros, and unsafe code

### Keep attributes narrow and meaningful

Place each attribute on its own line at the item's indentation, after documentation comments. Prefer outer attributes. Combine derives into one `#[derive(...)]`, preserving order when changing it could affect macro behavior, with no automatic alphabetical reorder. Keep conditional compilation easy to see, preferring one coherent conditional module to a complicated condition repeated across unrelated items. [Official attributes][style]

Use `#[cfg(...)]` when code must be excluded from a build. `cfg!(...)` produces a boolean expression, so both branches of an ordinary `if` must still type-check. [Conditional compilation][cfg]

Keep lint exceptions on the smallest useful item or block with a reason. Prefer `#[expect(..., reason = "...")]` when a diagnostic should remain present, and use `#[allow(..., reason = "...")]` when a justified scope may legitimately stop producing it. Do not silence warnings across a whole crate for convenience. [Diagnostic attributes][diagnostics]

### Mark results that must be used

Apply `#[must_use]` to a function when ignoring its return value prevents correct use, including pure computations and functions returning errors or other critical status. It is unnecessary when the return type already carries the attribute, as `Result` does. Add a message when the reason is not apparent.

Apply it to types that carry this obligation across their producers, including result-like error carriers and lazy computations whose work happens only when consumed or polled, rather than repeating it on every producer. Do not use it for values callers may correctly discard.

### Use macros for syntax that earns its complexity

When designing an API or generating code, prefer these mechanisms in order:

1. Ordinary Rust functions, types, traits, objects, and modules.
2. Declarative `macro_rules!` macros.
3. Other reproducible code generation, such as a build-system generator.
4. Procedural macros.

Use metaprogramming only when its benefit justifies harder IDE navigation, less obvious expansion, reduced cross-project familiarity, and harder automated refactoring. A declarative macro suits necessary syntactic repetition or an interface functions cannot express. Established standard macros and derives, including `thiserror`, remain useful; this order concerns choosing a mechanism, not replacing existing derives with handwritten code.

Keep a custom macro's expansion predictable. Do not hide a surprising `return`, `.await`, panic, or repeated argument evaluation, and document behavior that differs from a function call. Use `$crate` for references into the defining crate in an exported declarative macro. [Declarative macros][macros]

### Isolate and explain unsafe operations

Do not write unsafe code when equivalent required functionality can be implemented safely. Encapsulate necessary unsafe operations behind a sound safe abstraction, isolated from business logic. Prefer a small separate crate with few dependencies beyond the standard library when the boundary warrants one. Keep each unsafe block as small as practical, with a `// SAFETY:` comment explaining why the required conditions hold at that operation.

Also justify using unsafe code technically, at the block, file, or target level, explaining why an equivalent safe implementation is insufficient. A performance justification requires benchmarks demonstrating the constraint and benefit.

An unsafe function still needs explicit unsafe blocks around unsafe operations. Its `# Safety` documentation explains the caller's obligations; local comments explain how the implementation satisfies its operations.

Treat the safe code maintaining an unsafe abstraction's invariants as part of its safety argument, including safe functions in the same module. Keep unsafe trait implementations, attributes, and foreign declarations equally explicit about their obligations. [Unsafe abstractions][nomicon] [Unsafe operations in unsafe functions][unsafe-op]

## Language and API boundaries

Do not use unstable language or library features. Use the repository's supported stable toolchain and avoid feature gates added for convenience. A formatter option's availability is a separate tooling question.

Treat another owner's `#[doc(hidden)]` items as private even if they are callable, and do not depend on them without owning that code.

Make a public function `const` only when it is expected to remain callable in constant evaluation or its owner commits to migrating affected clients, because removing `const` can break callers even when runtime behavior is unchanged.

Do not execute project code before `main`, for example through the `ctor` crate or equivalent startup hooks. Initialize resources through an explicit entry point or embedding interface. Compile-time constant evaluation is separate from this.

## References

These explain the language and crate behavior used above. The coding rules are project policy; a style requirement is not necessarily a language requirement.

- [Rust Style Guide][style] and [Rust API Guidelines][api] for formatting, naming, and API conventions.
- [Rust Reference][reference] and [standard library documentation][std] for language rules and type contracts.
- [thiserror][thiserror] for derived error implementations and source handling.
- [anyhow][anyhow] and [Context][context] for contextual errors at reporting and coordination boundaries.
- [Tokio][tokio] for async operations, task handles, and runtime integration.

[style]: https://doc.rust-lang.org/style-guide/
[api]: https://rust-lang.github.io/api-guidelines/
[naming]: https://rust-lang.github.io/api-guidelines/naming.html
[flexibility]: https://rust-lang.github.io/api-guidelines/flexibility.html
[interoperability]: https://rust-lang.github.io/api-guidelines/interoperability.html
[documentation]: https://rust-lang.github.io/api-guidelines/documentation.html
[reference]: https://doc.rust-lang.org/reference/
[std]: https://doc.rust-lang.org/std/
[modules]: https://doc.rust-lang.org/reference/items/modules.html
[visibility]: https://doc.rust-lang.org/reference/visibility-and-privacy.html
[nonexhaustive]: https://doc.rust-lang.org/reference/attributes/type_system.html#the-non_exhaustive-attribute
[traits]: https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility
[from]: https://doc.rust-lang.org/std/convert/trait.From.html
[tryfrom]: https://doc.rust-lang.org/std/convert/trait.TryFrom.html
[borrow]: https://doc.rust-lang.org/std/borrow/trait.Borrow.html
[deref]: https://doc.rust-lang.org/std/ops/trait.Deref.html
[clone]: https://doc.rust-lang.org/std/clone/trait.Clone.html
[option]: https://doc.rust-lang.org/std/option/enum.Option.html
[take]: https://doc.rust-lang.org/std/mem/fn.take.html
[iterator]: https://doc.rust-lang.org/std/iter/trait.Iterator.html
[collections]: https://doc.rust-lang.org/std/collections/index.html
[thiserror]: https://docs.rs/thiserror/latest/thiserror/
[anyhow]: https://docs.rs/anyhow/latest/anyhow/
[context]: https://docs.rs/anyhow/latest/anyhow/trait.Context.html
[error]: https://doc.rust-lang.org/std/error/trait.Error.html
[panic]: https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html
[tokio]: https://docs.rs/tokio/latest/tokio/
[spawn]: https://docs.rs/tokio/latest/tokio/task/fn.spawn.html
[joinhandle]: https://docs.rs/tokio/latest/tokio/task/struct.JoinHandle.html
[bridging]: https://tokio.rs/tokio/topics/bridging
[str]: https://doc.rust-lang.org/std/primitive.str.html
[path]: https://doc.rust-lang.org/std/path/struct.Path.html
[debug]: https://doc.rust-lang.org/std/fmt/trait.Debug.html
[casts]: https://doc.rust-lang.org/reference/expressions/operator-expr.html#type-cast-expressions
[overflow]: https://doc.rust-lang.org/reference/expressions/operator-expr.html#overflow
[doctests]: https://doc.rust-lang.org/rustdoc/write-documentation/documentation-tests.html
[test-organization]: https://doc.rust-lang.org/book/ch11-03-test-organization.html
[cfg]: https://doc.rust-lang.org/reference/conditional-compilation.html
[diagnostics]: https://doc.rust-lang.org/reference/attributes/diagnostics.html
[macros]: https://doc.rust-lang.org/reference/macros-by-example.html
[nomicon]: https://doc.rust-lang.org/nomicon/working-with-unsafe.html
[unsafe-op]: https://doc.rust-lang.org/edition-guide/rust-2024/unsafe-op-in-unsafe-fn.html
[style-items]: https://doc.rust-lang.org/style-guide/items.html
[style-expressions]: https://doc.rust-lang.org/style-guide/expressions.html#enum-literals
[rustfmt-config]: https://rust-lang.github.io/rustfmt/
[catch-unwind]: https://doc.rust-lang.org/std/panic/fn.catch_unwind.html
[mutex]: https://doc.rust-lang.org/std/sync/struct.Mutex.html#poisoning
