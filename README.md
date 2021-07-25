# EIP Linting Bot

This Github Actions integrated bot lints EIPs and provides feedback for authors, its goal is to catch simple problems, notify the relevant individuals to review, and merge simple changes automatically.

# Usage

```yml
on: [pull_request_target]

jobs:
  hello_world_job:
    runs-on: ubuntu-latest
    name: EIP Auto-Merge Bot
    steps:
      - name: auto-merge-bot
        uses: ./.github/bot
        id: auto-merge-bot
        env:
          GITHUB_TOKEN: ${{ secrets.TOKEN }}
```

# Contributing

## Standard Practices

### Function Naming
This library uses concepts that may appear strange,

- **require...** : if a function starts with `require`, then it will
  - throw an error if the requirement is not met
  - return the specified check (`requirePR` returns the pull request object)
- **assert...** : if a function starts with `assert`, then it will
  - return an error string if the assertion fails
  - return undefined if the assert succeeds
- **...Purifier** : if a function ends in `purifier`, then it will
  - return a mutated testResult (the input will be mutated)
  - will remove any assertion erorrs based on edge cases
  - can only remove errors but can not negate removal or add

These practices are applied here for the primary purpose of streamlining logic to maximize readability and minimize errors. It is, in effect, untangling a messy logic stream by mostly) removing race conditions and side effects.

### Testing
This bot employees two types of tests

- functional
- integration

A functional test if your standard unit test. Take a small function and test its behavior thoroughly. You don't need anything more than jest to do this, and your code should be organized such that the sub functions are abstracted and tested. It also uses dependency injection for this reason (it's typically easier to mock that way). Everything should have unit tests.

An integration test is a test that considers the behavior as a whole. In this bot, we mock a network response from the github api using `nock`. When you do this for every network request you're able to get a snapshot and test the whole's behavior. All integration tests were once bugs that were fixed, so if you implement a feature you don't need to add an integration test. It's easier to manage this way, and it serves the purpose of reducing code regression. Integration tests tend to be brittle because of the number of different facets. So the code uses several homebrewed tools to maximize reliability.

Feel free to share ideas on how to improve testing procedures.

### Requirements

1. node package manager (npm)
2. Github Token
3. Forked Repo
4. nodejs

### Quick Start

1. Download your forked `EIPS` repo
2. Create a [Github Token](/creating-a-personal-access-token)
3. Create a PR in your forked repo doing anything, I recommend just editing a couple lines in an already existing EIPs
4. Create a .env variable in the root dir with the following information defined:

```
GITHUB_TOKEN = <YOUR GITHUB TOKEN>
NODE_ENV = development

PULL_NUMBER = <pull_number>
BASE_SHA = <base sha of the PR>
HEAD_SHA = <head sha of the PR>
REPO_OWNER_NAME = <your login>
REPO_NAME = EIPs
GITHUB_REPOSITORY = <your login>/EIPs
```

5. `npm run it`

### Troubleshooting

- <i>When I run it, I'm getting unexplainable errors with my github requests.</i>
  - Github limits the number requests from a given IP, this may be avoidable if you only use the `octokit` but a VPN also works just fine

## Code Style Guidelines (in no particular order)

This repo is a living repo, and it will grow with the EIP drafting and editing process. It's important to maintain code quality.

1. Define every type (including octokit)
2. Make clean and clear error messages
3. Avoid abstraction
4. Use [enums](https://www.sohamkamani.com/javascript/enums/) as much as possible

## Explanations of Style Guidenlines

A couple things to keep in mind if you end up making changes to this

#### 1. <ins>Define every type</ins>

Define every type, no `any` types. The time it takes to define a type now will save you or someone else later a lot of time. If you make assumptions about types, protect those assumptions (throw exception if they are false).

Sometimes [Octokit types](https://www.npmjs.com/package/@octokit/types) can be difficult to index, but it's important that whenever possible the types are defined and assumptions protected.

#### 2. <ins>Make clean and clear error messages</ins>

This bot has a single goal: catch simple mistakes automatically and save the editors time. So clear error messages that allow the PR author to change it themselves is very important.

#### 3. <ins>Avoid Abstraction</ins>

Only abstract if necessary, keep things in one file where applicable; other examples of okay abstraction are types, regex, and methods used more than 3 times. Otherwise, it's often cleaner to just re-write things.

```javascript
// DON'T DO THIS
** src/lib.ts **
export const baz = () => "baz"

** src/foo.ts **
import { baz } from "./lib"
export const foo = () => baz();

** src/bar.ts **
import { baz } from "./lib"
export const bar = () => baz();

// DO THIS
** src/foo.ts **
const baz = () => "baz"
export const foo = () => baz();

** src/bar.ts **
const baz = () => "baz"
export const bar = () => baz();
```

#### 4. <ins>Always use enum when defining restricted string types</ins>

In short, enums make code easier to read, trace, and maintain.

But here's a brief info if you haven't worked with them before

```typescript
enum EnumFoo {
  bar = "BAR",
  baz = "BAZ"
}
type Foo = "BAR" | "BAZ";
```

Inline declaraion is maintained

```typescript
const foo: EnumFoo;
const bar: Foo;
// foo and bar both must be either "BAR" or "BAZ"
```

Use case is slightly different

```typescript
const foo: EnumFoo = EnumFoo.baz; // you can't directly assign "BAZ"
const bar: Foo = "BAZ";
```

But comparisons are maintained

```typescript
// taking variables from above
("BAZ" === foo) === ("BAZ" === bar) &&
  ("BAZ" === EnumFoo.baz) === ("BAZ" === "BAZ");
```

In addition to the above use case and string eradication it centralizes the strings to be matched so they can be easily changed. So, making life much easier if you wanted to change the names of statuses on an EIP.

# Contributors

- @alita-moore
