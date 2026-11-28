---
name: responsibility-driven-crates
description: Use when designing, changing, or reviewing the structure or naming of Rust crates and workspaces.
---

# Responsibility-Driven Crates

Start with the smallest workspace structure that explains the project’s current responsibilities, including during a review with no proposed crate changes. Keep responsibilities together while they evolve together. Consider a separate crate when a responsibility becomes independently meaningful.

## Core rule

Group strongly related responsibilities together first.

Consider extracting a crate when a responsibility has developed a useful independent boundary, such as its own API, dependencies, consumers, tests, platform constraints, build concerns, or ownership.

Name an extracted crate after the responsibility it owns.

Prefer concrete responsibility names such as `api`, `auth`, `storage`, `client`, `syntax`, or `renderer` when those names describe the actual boundary.

## Design from current responsibilities

When designing a new project or revising an existing workspace:

1. Identify the responsibilities required by the current system.
2. Group responsibilities that change together and depend closely on the same implementation details.
3. Identify responsibilities that can be understood, tested, or consumed independently.
4. Consider a crate boundary where separation improves ownership, dependency direction, reuse, compilation boundaries, or conceptual clarity.
5. Name each crate for the responsibility on its side of the boundary.
6. Revisit the structure as responsibilities grow or coupling changes.

Represent plausible future split points as design notes until the corresponding responsibility has a real boundary.

## Signals that a responsibility may deserve its own crate

Consider extraction when several of these are true:

- it exposes a coherent API that can be described without referring to its current parent module;
- multiple parts of the workspace consume it independently;
- it has dependencies that most of the parent crate does not need;
- it has distinct platform, feature, build, or release requirements;
- it can be tested meaningfully through its own public boundary;
- its implementation changes for reasons different from neighboring responsibilities;
- separating it simplifies dependency direction;
- it is useful to compile, reuse, replace, or reason about independently.

No single signal is mandatory.
Use the combination that best reflects the project.

## Signals that responsibilities still belong together

Keeping code in one crate is usually clearer when:

- the responsibilities change together;
- one part is primarily an implementation detail of another;
- the proposed boundary would expose many internal types only to reconnect them immediately;
- there is one consumer and no useful independent API;
- separation would add dependency and coordination overhead without clarifying ownership;
- the responsibility cannot yet be named more precisely than its architectural position.

A module boundary is often enough before a crate boundary becomes useful.

## Naming crates

Choose names that describe what the crate owns or provides.

Prefer names such as:

- `api`
- `auth`
- `storage`
- `client`
- `config`
- `renderer`
- `protocol`
- `syntax`

Use broader names such as `core`, `common`, `shared`, or `utils` only when the project has a specific, stable meaning for that name and the crate has a coherent responsibility that can be stated directly.

If the responsibility can be named more precisely, use that name.

For example, if a proposed `core` crate mainly owns authentication rules, prefer `auth`.
If a proposed `common` crate mainly owns protocol types, prefer `protocol`.

## Directory names and Cargo package names

Treat the workspace path and the Cargo package name as separate naming decisions.

Inside a repository, a concise directory name is usually sufficient when the repository already provides the surrounding context:

```text
my-project/
└── crates/
    ├── api/
    ├── auth/
    └── storage/
```

A Cargo package name may include the project prefix when that improves identification outside the repository, especially for published packages or packages consumed independently:

```toml
[package]
name = "my-project-auth"
```

The directory can remain:

```text
crates/auth/
```

Repository context therefore handles path-level disambiguation, while the package name can carry project-level identity when useful.

For private workspace members, choose the package name that gives the clearest dependency graph and diagnostics.
For public packages, account for the package registry namespace and how users will identify related packages.

## Treat `core` as a responsibility, not a default layer

A crate named `core` should have a concrete explanation of what "core" means in that project.

Before choosing the name, complete this sentence:

> This crate owns ______.

If the blank naturally produces a more specific responsibility name, use that name for the crate.

A useful `core` boundary can still exist when the project itself defines a stable central abstraction that is genuinely consumed by several surrounding components.
The name should follow that established concept rather than serve as a placeholder for code that does not yet have an owner.

In Rust, also account for the standard `core` crate when judging whether a package or dependency name will be clear in code, diagnostics, and documentation.

## Example: grow boundaries with the application

Suppose a small application initially has one crate:

```text
app/
└── src/
```

As HTTP handling and persistence become independently meaningful responsibilities, separating them may improve the structure:

```text
app/
└── crates/
    ├── api/
    └── storage/
```

Later, authentication may gain its own policy, tests, dependencies, and consumers:

```text
app/
└── crates/
    ├── api/
    ├── auth/
    └── storage/
```

The important part is the sequence.

The application did not begin with a predetermined set of architectural crates.
Each boundary appeared after a responsibility became clear enough to own independently.

The names are illustrative.
Use the responsibilities that actually emerge in the project rather than copying `api`, `auth`, or `storage` as a template.

## Example: module before crate

A responsibility can become visible before it needs an independent package.

For example:

```text
app/
└── src/
    ├── auth.rs
    ├── storage.rs
    └── main.rs
```

This structure already gives the responsibilities names and boundaries.

If `auth` later gains independent consumers or dependencies, it can become:

```text
app/
└── crates/
    └── auth/
```

The earlier module boundary makes the later extraction easier without requiring the crate to exist in advance.

## Reviewing an existing workspace

When reviewing existing crates:

1. State the responsibility of each crate in one sentence.
2. Identify crates whose responsibilities overlap heavily.
3. Identify crates whose names describe position rather than responsibility.
4. Check whether each dependency edge reflects a meaningful ownership boundary.
5. Consider merging crates that behave as one responsibility.
6. Consider extracting responsibilities that have become independently coherent.
7. Rename crates when a more precise responsibility name has emerged.

Judge the current structure by current responsibilities and coupling rather than by the number of crates.

## When proposing a workspace

For each proposed crate, provide:

- its responsibility;
- why that responsibility deserves an independent boundary now;
- its main dependencies or consumers when relevant;
- the reason for its name.

If a responsibility is likely to grow but does not yet justify extraction, keep it in the current crate or module and mention the possible future boundary separately.

This makes the proposal explain both the present structure and the conditions under which it should evolve.

## Scope

This skill applies to Rust projects that use one or more Cargo packages, including:

- application workspaces;
- libraries with supporting packages;
- CLI tools;
- services;
- desktop applications;
- reusable Rust components.

It focuses on crate boundaries and crate naming.
It does not prescribe a fixed architecture, directory tree, layering model, or minimum number of crates.
