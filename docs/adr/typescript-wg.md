# ADR #005: TypeScript Working Group

## Status

Proposed

## Submitters

- @bjohansebas

## Decision Owners

- @expressjs/express-tc

## Context

TypeScript is now widely adopted and has a large, active community. Many developers use it alongside packages maintained by the Express project. As such, there's a growing need to enhance TypeScript support — both by maintaining accurate type definitions and by addressing existing and future issues related to typings.

- **Why do we need this decision?**

As the Express project continues to evolve, we want to improve our engagement with the community. Having a dedicated group focused on TypeScript-related matters will enhance our support, especially since not all maintainers are familiar with TypeScript. An active working group will help manage and resolve type-related issues effectively.

- **What problem does it solve or avoid?**

This proposal aims to establish a dedicated team responsible for maintaining TypeScript types, whether in DefinitelyTyped or directly within the repositories. It will also help triage and resolve the numerous issues reported daily related to type definitions.

- **Are there any existing issues/discussions/pull requests related to this?**:

   - https://github.com/expressjs/discussions/issues/192
   - https://github.com/expressjs/express/issues/2818

## Decision

Similar to the Security WG and Performance WG, we will set up a new repository (expressjs/typescript-wg) along with a corresponding action plan. This working group will be empowered to build consensus and make decisions within its scope. Its core responsibilities will include:

- Ensure `express` package types are semantically correct, current with the implementation, and provide a high-quality developer experience
- Making decisions on TypeScript support within Express-maintained repositories
- Maintaining up-to-date TypeScript type definitions
- Enhancing documentation for TypeScript usage across Express packages
- Support testing type accuracy while in development and CI
- Ensure that types shared across Express Project packages are updated in sync to prevent build errors across the ecosystem

### Initial Goal

The TypeScript Working Group’s initial goal is to ensure that the `express` package has **high-quality, semantically correct, and up-to-date type definitions that accurately reflect its public API.**

As part of this effort, the working group will also evaluate whether and how to support types in other Express-maintained packages. Decisions around publishing first party types vs. DefinitelyTyped maintenance, release cycle alignment, and tooling support will be developed collaboratively and documented as part of the group's ongoing charter.
We will need to take a few actions to get this started:

1. Create a new `typescript-wg` repo
2. Create teams: `@expressjs/typescript-wg`, `@pillarjs/typescript-wg`, `@jshttp/typescript-wg`
3. Write charter/goals doc in the new repo

## Rationale

Since Express has become active again and the new version, Express v5, has been released, several issues related to typings have emerged. There have also been discussions about actively integrating type definitions within the repositories.

Most packages under the Express umbrella are not written in TypeScript, and not all maintainers are interested in taking on type maintenance responsibilities. A small number (such as `path-to-regexp`) maintain their own types, but this is the exception rather than the norm.

The proposed working group provides a focused avenue for contributors who are interested in TypeScript to collaboratively improve, maintain, and guide the strategy for type definitions across Express projects, starting with the core `express` package.

## References

- https://github.com/expressjs/discussions/issues/192
- https://github.com/expressjs/express/issues/2818

## Changelog

Track changes or updates to this ADR over time. Include the date, author, and a brief description of each change.

- **[2025-05-28]**: [@bjohansebas] - Initial draft

