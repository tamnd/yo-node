# yo-node

TypeScript and JavaScript client for [yo](https://github.com/tamnd/yo), an embedded multi-model database that lives in one `.yo` file. Your types are the schema and there is no query language to learn. Node, Deno and Bun from one package.

## Status

Nothing to use yet, and the package on npm says so out loud.

`@yodb/core` is published at `0.0.1`. Importing it succeeds and does nothing, on purpose, because a placeholder that throws on import is a broken artifact in a stranger's dependency tree and that is a real cost to impose on somebody for the sake of holding a name. Calling anything is what tells you where you are:

```js
import db from "@yodb/core";
db.open("app.yo");
// Error: yo is not usable yet. This is a reserved placeholder at 0.0.1;
// see https://github.com/tamnd/yo
```

`0.0.1` and not `0.0.0` because of that sentence. This was the one placeholder published by hand, and it went out throwing its own package description instead of the sentence every other ecosystem raises. npm never lets a version number be reused, so the correction had to ship as a new version, and `@yodb/core@0.0.0` still carries the old wording and always will. Every other ecosystem moved to `0.0.1` on the same day, so one version number still means one artifact everywhere.

The engine is at `M0`. The record plane and the file format are `M1` and in progress, so there is nothing for this binding to sit on top of yet. Watch the [milestones](https://github.com/tamnd/yo/milestones).

## Install

```bash
npm i @yodb/core
```

**The npm name is scoped and every other ecosystem's is not.** The crate, the PyPI distribution, the NuGet id, the pub.dev package, the OS packages and the command are all plain `yodb`. npm refused the unscoped `yodb` in August 2026 under its name-similarity filter, so npm is the one exception, and it is written here rather than left for you to discover at install time. Asking npm for the unscoped name again waits until there is a working shipped package to argue with, because that is the argument.

## What this will be

```ts
import { open } from "@yodb/core";

type User = { id: number; name: string; score: number };

await using db = await open("app.yo");
const users = db.docs<User>("users", { id: "id", index: ["score"] });
await users.put({ id: 1, name: "Ada", score: 99.5 });

for await (const u of users.orderBy("score", "desc").limit(10)) {
  console.log(u.name, u.score);
}
```

`"score"` is checked against `keyof User`, so a typo is a compile error, and `orderBy` accepts only keys whose value type is orderable. No string is parsed. The literal type is the check, and there is no code generation on your side at all.

`await using` is `Symbol.asyncDispose`. None of this works today.

## Planned support

| Item | Version |
|---|---|
| Node floor | 24 LTS, with 22 supported best effort |
| Node current | 26.x |
| Deno | 2.9.x, via the npm specifier |
| Bun | 1.3.x |
| napi-rs | CLI 3.8.x, N-API 9 baseline |
| TypeScript | 7.0.x, with types authored to compile under 5.9 |

Every engine call runs on the libuv threadpool by default, not by option. The async surface is the only surface. There is a synchronous one for scripts and command line tools, and it lives behind a separate entry point precisely so a server codebase can lint against importing it.

`Temporal` is detected once at load and never at a call site. If the runtime has it, the TTL and stream-time APIs accept and return `Temporal.Instant` and `Temporal.Duration` alongside numbers; if it does not, those overloads are simply absent. Detecting per call would put a `typeof` on the hot path and hand you an API that changes shape mid-process.

The ten per-platform native packages are scoped as `@yodb/<triple>` rather than taking ten bare names in the global namespace.

## Design

The full TypeScript specification is `dx/06` in the project specification.

## Licence

Apache 2.0 or MIT, at your option.
