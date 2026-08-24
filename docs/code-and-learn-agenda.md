# Workshop: ExpressJS - Code & Learn

NOTE: this doc is for CityJS Medellin, but hosted here as an example for future events folks in the ecosystem might run.

Join us for a hands-on workshop where you’ll learn how to get involved and contribute to the ExpressJS repository and organization. We’ll walk through the best ways to participate,
from joining the Security Working Group to the Triage team and engaging in discussions. You’ll also gain insights into the structure of the modules and priorities for the future.
Get ready for a practical session where we’ll triage real issues and even open a pull request, giving you a direct path to becoming an ExpressJS contributor!

---
## Introduction (5 min)

Introduce the folks running it, give a general overview of how we will spend the time. Ask people how much the know about the project and their technical skill levels. 

## Introduction to Express (10-15 min)

Express is the original Node.js server framework. The [first commit was in 2009 by TJ
Holowaychuk](https://github.com/expressjs/express/commit/9998490f93d3ad3d56c00d23c0aa13fac41c3f6b) and since has gon through a few different eras of leadership. These days, Express
is an OpenJS Impact project, and is governed by an open model with a Technical Committee and many opportunities to get involved depending on your skills and available time. In
today's Code and Learn we are going to go through some of those ways to get involved and help you all make meaningful contributions to the project.

The project is composed of 43 repos in 3 github orgs:

- `expressjs`: The primary org where the `express` repo lives. The other repos in this org are all directly part of `express` or dependent on `express` api's
- `pillarjs`: This org is where generic but high level server components are hosted. Things like the `router` and `path-to-regexp` which can be used outside of `express` but are
  still high level.
- `jshttp`:  This org is where lower level `http` and protocol level packages are hosted. Things like `cookie` and `mime-db`.

Because we are so spread out, we use a repo in the `expressjs` org called `discussions` to have cross-cutting or project wide discussions. Often when deciding what to work on, this
is the best place to start. More on that to come.

Lets talk a little bit about the project governance and leadership. We have a few roles which are great ways to start onboarding to the project:

- The triage team: this team focuses on helping answer the many questions which come into the project issue tracker, as well as how to best organize answers and update docs
- The security working group: this group is in charge of helping us define and maintain our security model, docs, and reporting

Additionally we have roles for folks who are making technical contributions:

- Contributor: these are folks who have made a few contributions to code and are given commit access to land PRs and their own work
- Repo Captains: these are folks who where contributors on a given repo but have demonstrated strong judgement and reliability who are also given publish permissions and more
  responsibility within the given repo

And finally we have the leadership group, which is the Technical Committee or as we call it, the TC. This group oversees the larger project direction and is there to help resolve conflicts or make
important decisions.

## Questions

Does anyone have questions on the structure of the project?

## Project Goals (10 - 15 min)

Because the project is made up of individual volunteers with many different skills, we attempt to organize the priorities and work with a distributed consensus model. This means
the goals are yours to set. That said, currently because the project is undergoing a revitalization effort, we are maintaining a central list of goals in the `discussions`
repository.

### General Goals

https://github.com/expressjs/discussions

### Express 5 Goals

https://github.com/expressjs/discussions/issues/266

### Express 6 Goals

https://github.com/expressjs/discussions/issues/267

### Questions

Anyone have questions before we start contributing?

## Lets Start Contributing (1-1.5h)

We picked out three specific goals the project has which we think will make simple yet very helpful contributions today:

1. Documentation updates for v5: https://github.com/expressjs/expressjs.com/
1. Updating dependencies to use `^`: ranges: https://github.com/expressjs/discussions/pull/290 https://github.com/pillarjs/router/pull/133/files
1. Adding engines fields to packages https://github.com/expressjs/discussions/pull/289
1. Remove compat deps: ex. https://github.com/pillarjs/router/blob/master/package.json
1. Triage issues: https://github.com/expressjs/express/blob/master/Triager-Guide.md


### Demo one of each

1. Update a dep, show how to test it locally, then show how to push it up and open a PR.
- (router -> parseurl) https://github.com/pillarjs/router/blob/878b0e02365047f34c66b4a252e91597f8056635/package.json

2. Update an engines field. Opening the PR and testing already demo'd.
- (router) https://github.com/pillarjs/router/blob/master/package.json

3. Update a doc. Show how to run locally. Open a PR and show the preview URL.
  - https://github.com/expressjs/expressjs.com?tab=readme-ov-file#local-setup-using-docker

4. Triage an issue.
- (bad) https://github.com/jshttp/cookie/pull/202
- (spam) https://github.com/expressjs/express/pull/6083
- (self resolved) https://github.com/expressjs/express/issues/6085
- (typescript errors) https://github.com/expressjs/express/issues/6022
- (maybe related?) https://github.com/expressjs/express/issues/6022
- (java !== javascript) https://github.com/expressjs/express/issues/6022

### Working time

@TODO

## Wrapping up

@TODO?
