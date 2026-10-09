# RFC-06: Example Code Placement

**Status:** Proposal

## Summary

This RFC defines where sample code for CDKTN lives and how to decide where a new example belongs. It establishes two homes — the `cdk-terrain` monorepo for CI-gated, single-feature examples, and a dedicated `cdktn-examples` repo for standalone, real-world projects — and a clear line between them based on whether an example must stay green against the current codebase.

Every entry in the published [Examples and Guides](https://cdktn.io/docs/examples-and-guides/examples) catalog must map to source in one of these two locations.

## Motivation

The example list grows over time, and the team has an open question about what belongs in a separate repo versus the main repo. Without an explicit line, examples accumulate ad hoc: large end-to-end projects bloat the monorepo and slow CI, while small feature demos drift out of sync with the code they are meant to validate. A documented placement rule keeps `cdk-terrain` PRs small and code-focused while giving real-world samples room to evolve on their own cadence.

## Problem Statement

The `cdk-terrain` monorepo already carries an `examples/` tree organized by language, but it is partial relative to the published catalog, and there is no stated rule for where new examples go. The high-complexity end-to-end projects in the docs are not present locally, and nothing distinguishes examples that CI must run from those that are purely instructional.

## Goals

Define a cloud-neutral placement rule, keep CI regression coverage co-located with the code it validates, give standalone learning projects an independent home and release cadence, and provide a repeatable procedure for adding new examples.

## Non-Goals

This RFC does not rewrite existing examples, change the docs catalog structure, or mandate a specific CI runner. It does not define the `cdktn-examples` repo's internal tooling beyond its placement role.

## The Line: Main Repo vs. Examples Repo

| Question | Main repo (`cdk-terrain/examples`) | Examples repo (`cdktn-examples`) |
| --- | --- | --- |
| Is it exercised by CI on every CDKTN change? | Yes | No |
| Does it pin a released CDKTN version or track `main`? | Tracks `main` | Pins a released version |
| Primary purpose | Validate the toolchain, demonstrate a single feature | Teach a real-world, multi-service architecture |
| Typical complexity (per docs) | Low | High |
| Owned by | Core maintainers | Community + maintainers |

Rule of thumb: if the example must stay green against the current codebase to catch regressions, it belongs in the **main repo**. If it is a standalone learning project that evolves on its own cadence, it belongs in the **examples repo**.

### Where the docs point

Following AWS CDK and cdk8s, the public [cdktn.io](https://cdktn.io) catalog links users to the standalone `cdktn-examples` repo as the single user-facing home for sample projects — not into the `cdk-terrain` monorepo's `examples/` tree. The monorepo examples exist to gate CI; they are not promoted as browsable samples. This keeps one public entry point for learners and avoids sending them into a tree whose layout serves CI rather than readability.

Concretely: every example link on the docs catalog resolves to `github.com/.../cdktn-examples/...`. Main-repo examples are referenced by the docs only as CI fixtures, if at all, never as the primary "go look at this" link.

## Main Repo Layout

Examples live under `examples/<language>/<example-name>/`. Backends are grouped under a `backends` folder for the language that documents them.

``` text
examples/
  typescript/<example-name>/
  python/<example-name>/
  java/<example-name>/         # Maven layout, no suffix
  java/<example-name>-gradle/  # Gradle layout, -gradle suffix
  csharp/<example-name>/
  go/<example-name>/
  typescript/backends/<backend-name>/
```

Conventions:

- One directory per example, named exactly as it appears in the docs table (`aws-prebuilt`, `azure-app-service`, `google-cloudrun`).
- Java carries two variants: Maven (no suffix) and Gradle (`-gradle` suffix). Keep both in sync when an example exists in both.
- Backends (`s3`, `gcs`, `azurerm`, `remote`) go under the language's `backends/` folder, not at the top level.
- Every example gets a `README.md` describing what it provisions and how to run it.

## Example-to-Location Map

Mapped from the docs catalog.

### Low complexity → main repo

| Example | Languages |
| --- | --- |
| aws-prebuilt | TS, Java, C#, Maven, Gradle |
| aws-multiple-stacks | TS |
| aws-cloudfront-proxy | TS |
| aws (VPC / DynamoDB / EKS) | Python, Java, C#, Go |
| aws-eks | Python |
| azure | TS, Python, Java, C#, Gradle |
| azure-app-service | TS |
| google | TS, Java, C#, Gradle |
| google-cloudrun | TS |
| docker | TS, Python, Go |
| kubernetes | TS, Python, Java, Gradle |
| ucloud | all languages |
| vault | TS |
| gradle-shared-module | Java (Gradle) |
| Backends: s3, gcs, azurerm, remote | TS (`backends/`) |

### High complexity → examples repo

| Example | Languages |
| --- | --- |
| aws-ecs-docker-and-static-frontend | TS |
| aws-lambda-end-to-end | TS, Python, Maven |
| ecs-microservices-cdktn | TS |
| google-end-to-end | Python, Maven |

### External references (no repo move)

| Resource | Where it lives |
| --- | --- |
| Pocket `recommendation-api` + `terraform-modules` | Mozilla Pocket's public repos (linked from docs) |
| YouTube playlist, release demos | External video links |

## Prior Art

Comparable CDK-style and IaC projects consistently keep a dedicated examples repo separate from the core repo, with CI regression coverage handled inside the core repo by a different mechanism. None wires the examples repo back in as a git submodule.

- **AWS CDK**: The [Developer Guide examples section](https://docs.aws.amazon.com/cdk/v2/guide/how-tos.html) points users only to the standalone [`aws-samples/aws-cdk-examples`](https://github.com/aws-samples/aws-cdk-examples) repo, organized by language for parity; large apps are linked out, not copied. CI regression coverage is a separate concern inside [`aws/aws-cdk`](https://github.com/aws/aws-cdk) as [integration tests](https://github.com/aws/aws-cdk/blob/main/INTEGRATION_TESTS.md) — small CDK apps with a 1:1 app-to-test relationship, co-located with the code and not surfaced as user-facing examples.
- **cdk8s**: The [docs examples page](https://cdk8s.io/docs/latest/examples/) links exclusively to the standalone [`cdk8s-team/cdk8s-examples`](https://github.com/cdk8s-team/cdk8s-examples) repo, split by language; every example link targets that repo, not the `cdk8s-team/cdk8s` umbrella monorepo. The project spans several repos yet keeps the user-facing example catalog in a dedicated repo, not a submodule.
- **Winglang**: Core in [`winglang/wing`](https://github.com/winglang/wing); examples in a separate [`winglang/examples`](https://github.com/winglang/examples) repo, tested against the most recent releases. Larger examples live in their own dedicated repos and are linked.
- **Pulumi**: Core in [`pulumi/pulumi`](https://github.com/pulumi/pulumi); examples in [`pulumi/examples`](https://github.com/pulumi/examples) with a flat `<cloud>-<language>` naming scheme. Documents a sparse-checkout recipe so users pull a single example without cloning the whole repo. Validation runs inside the examples repo via `make pr_preview`.

Takeaways for CDKTN: a dedicated examples repo is the norm; the public docs point users to that repo rather than the core monorepo; none uses a submodule; selective checkout is better served by sparse-checkout; oversized examples get linked rather than copied.

## Alternatives Considered

### Examples via git submodule

One option raised was to put all samples in a single `cdktn-examples` repo, embed it in `cdk-terrain` as a git submodule, and run only selected examples in CI. This RFC does not adopt it.

- **A submodule pins a commit, CI needs lockstep.** A submodule tracks one fixed commit of the examples repo, not `main`. An API change in `cdk-terrain` would not break the pinned examples until someone bumps the pointer, so CI stops catching regressions at the moment the examples drift — the opposite of what CI examples are for.
- **The dependency direction is backwards.** CI examples exist to validate CDKTN as it changes, so they must move with the code in one PR. A pinned submodule splits that into two steps and lets breaking changes pass green.
- **No strong precedent.** AWS CDK, cdk8s, Winglang, and Pulumi all keep examples separate without a submodule. Real-world submodule use is for shared libraries, SDKs, and build tooling — not example catalogs.
- **Selective checkout does not need a submodule.** Pulumi's sparse-checkout gives the "pick only the relevant ones" benefit without the staleness.

If a submodule were adopted anyway, it would require scheduled auto-bump automation and `submodule.recurse` enabled for contributors — overhead the split model avoids.

## Adding a New Example

1. Decide the location using the line defined above.
2. Create `examples/<language>/<name>/` (main repo) or a top-level project (examples repo).
3. Name the directory to match the docs table entry.
4. Add a `README.md` with purpose, prerequisites, and run steps.
5. Add the row to the [Examples and Guides](https://cdktn.io/docs/examples-and-guides/examples) catalog with its complexity rating, linking to the `cdktn-examples` repo as the user-facing source.
6. For main-repo examples, confirm it is picked up by the examples CI workflow; keep the catalog's public link pointing at `cdktn-examples`, not the monorepo tree.

## Success Criteria

Every catalog entry maps to exactly one documented location, the public cdktn.io catalog links users to the `cdktn-examples` repo rather than the monorepo tree, CI-gated examples stay green against `main`, and high-complexity projects live in `cdktn-examples` without bloating the monorepo or its CI.
