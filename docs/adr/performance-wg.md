# ADR #306: Performance Working Group

## Status

Proposed

## Submitters

- @wesleytodd

## Decision Owners

- @expressjs/express-tc

## Context

Express has traditionally taken a very lax approach to performance. Historically this has meant poor outcomes with
and lack of visibility around how changes impact performance. The goal of this working
group is to centralize discussion so the whole ecosystem of packages can benefit from targeted performance work. The
scope of this WG would be all active packages in the `expressjs`, `pillarjs` and `jshttp` orgs with an initial focus on
the ones which are direct dependencies of `express` itself. 

**Why do we need a working group?**

The current state is we have many open PRs, in flight initiatives, and planned future work. With a loose approach we
can easily have duplicated work, conflicting approaches, or focus on the wrong outcomes. By giving this WG scope over
this we can centralize the discussion and plans so that we unlock better and faster work in the individual packages.

Additionally, there will be continued long term work to monitor and maintain the tooling. The WG structure gives a clear
place for users, contributors, and other interested parties to go so it is less difficult to find across the many org's
and repositories the project owns.

## Decision

Similar to the Security WG, we will setup a new repo (expressjs/perf-wg) and corresponding plan of action. The WG will
be delegated responsibility for driving concensus on initiatives within scope. This group will have three primary goals:

1. Organize the effort. We want to make sure we are not rebuilding the same things or having the same disucssions in
   many package repos at once.
2. Provide and maintain tooling for all the packages/repos to produce reliable performance metrics and benchmarks.
   Examples include: workflows, infra and guidelines on types of benchmarks
3. Proritize and execute performance improvements across the packages. These do not need to be done *by* the members of
   the WG, but it would be good if we could get reviews by the team we will setup to ensure folks with most experience
   can help address performance issues across the project.

We will need to take a few actions to get this started:

1. Create a new `perf-wg` repo
2. Create teams: `@expressjs/perf-wg`, `@pillarjs/perf-wg`, `@jshttp/perf-wg`
3. Write charter/goals doc in the new repo
4. Schedule a recurring (monthly) meeting

### Relation to STF

Lastly, this kicko-off work is being funded by the STF program. It is a part of our larger work around security
improvements for the project. There are two parts of that program which fall under this WG's purview:

1. Milestone 6: Monitor performance across Express and direct dependencies
  - Owner: @wesleytodd
  - Delivery Date: Dec 31, 2025
  - Estimated Budget: €25,200
2. Milestone 9: Improve performance for Express and critical dependencies
  - Owner: @wesleytodd & @blakeembrey
  - Delivery Date: Jun 30, 2026
  - Estimated Budget: €46,200

This funding does not mean the work can only be done by the owners, it just means that we are responsible for organizing
and executing on the deliverables. This is our first attempt at doing a program like this, so feedback is welcome.

## References

- https://github.com/expressjs/express/pull/6129
- https://github.com/expressjs/express/issues/5998
- open-telemetry/opentelemetry-js-contrib#2437
- openjs-foundation/infrastructure#5
