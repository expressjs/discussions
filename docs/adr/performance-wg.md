# ADR [Number]: Performance Working Group

## Status

Proposed

## Submitters

- @wesleytodd

## Decision Owners

- @expressjs/express-tc

## Context

Express has traditionally taken a very lax approach to performance. historically this has meant poor outcomes with regard to performance and lack of visibility around how changes
impact performance. The goal of this proposed working group is to centralize discussion so the whole ecosystem of packages can benefit from targeted performance work. The scope of
this WG would be all active packages in the `expressjs`, `pillarjs` and `jshttp` orgs with a special focus on the ones which are direct dependencies of `express` itself. 

**Why do we need a working group?**

The current state is we have many open PRs, in flight initiatives, and planned future work. With a loose approach we can easily have duplicated work, conflicting approaches, or
focus on the wrong outcomes. By giving this WG scope over this we can centralize the discussion and plans so that we unlock better and faster work in the individual packages.

## Decision

Similar to the Security WG, we will setup a new repo (expressjs/perf-wg) and corresponding plan of action. The WG will be delegated responsibility for driving concensus around the
current scope and future scope to be determined by the group once setup.

## References

- https://github.com/expressjs/express/pull/6129
- https://github.com/expressjs/express/issues/5998
