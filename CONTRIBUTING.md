# Contributing to Hiver Examples

Thanks for your interest in contributing! This repository holds the runnable
examples for [Hiver](https://hiver.sh) — the agent runtime that runs agents
autonomously with controlled network access, auditable file operations, and full
execution visibility. This guide covers how to set up your environment, add or
change an example, and submit it.

For the runtime itself, see the main Hiver repository's contributing guide.

## Prerequisites

- **Docker**, which the local stack runs on.
- **Hiver CLI**:

  ```sh
  npm install -g @hiver.sh/cli
  ```

  Then bring up the gateway that every example talks to, on
  `http://localhost:10000` by default:

  ```sh
  hiver up      # hiver down to stop it
  ```

- **Node.js 20+** — only if you're working on a TypeScript example.
- **Python 3.11+** — only if you're working on a Python example.
- An **API key** for whichever model provider your example uses.

## Project layout

Every example lives in its own top-level directory, with a `typescript/` and/or
`python/` implementation inside it. Each of those has its own `README.md`, and is
standalone — you install its dependencies and run it from that directory.

Examples come in two shapes, and the shape decides the file layout:

| Shape | Layout | How it runs |
| --- | --- | --- |
| **Agent SDK server** | `<example>/<lang>/agent/` holding the agent server plus its `Dockerfile` and `.hiver.json`, alongside a `client.ts` / `client.py` that drives it | `hiver run . <key>` bundles the directory into an image and runs the agent loop **inside** the sandbox as an HTTP service |
| **CLI / browser driver** | a single `index.ts` / `main.py` | runs **on your machine**, provisioning a prebuilt image (`claude`, `codex`, `copilot`, `browser`) and driving it over the gateway with `exec` or CDP |

A TypeScript example carries `package.json`, `tsconfig.json`, and
`eslint.config.mjs`; a Python example carries `requirements.txt`.

The `.hiver.json` in an Agent SDK server is what locks egress to the model
provider and injects the API key via an `override`, so the key is applied by the
proxy after the request leaves the sandbox and never lives in the sandbox's env
or context. Keep that pattern when you add one — deny `*` and allow only the
provider host:

```json
{
  "image": "claude-agent-sdk-ts",
  "egress": [
    {
      "access": "allow",
      "host": "api.anthropic.com",
      "override": { "headers": { "x-api-key": "sk-ant-..." } }
    },
    { "access": "deny", "host": "*" }
  ]
}
```

## Running an example

Each example is self-contained. From its `typescript/` or `python/` directory:

```sh
# TypeScript
npm install
npm start

# Python
pip install -r requirements.txt
python main.py
```

Agent SDK servers are launched with `hiver run . <key>` from the example
directory instead; see that example's `README.md` for the exact command, the
key/image to use, and how to send it a prompt.

## Verifying your change

There is no test suite here — an example *is* its own test, so the bar is that it
actually runs end to end against a live gateway:

- **Run it.** Start from a clean checkout (`npm install` / `pip install -r
  requirements.txt`), bring the stack up with `hiver up`, and confirm the example
  produces the output its `README.md` promises.
- **Keep the README honest.** If you change what an example does, what it prints,
  or what it needs, update its `README.md` in the same commit. The commands in it
  should be copy-pasteable and work as written.
- **Check it from scratch, not from your shell.** Don't rely on a key, image, or
  gateway you happen to have lying around — a contributor following the README
  should get the same result.
- **List a new example in the root [`README.md`](README.md)** table, under the
  shape that matches it.

## Code style

For TypeScript examples, before committing:

```sh
npm run typecheck    # tsc --noEmit
npm run lint         # eslint .
```

Keep an example's dependencies minimal — the point is to show the Hiver API, not
a framework. Prefer the smallest script that makes the behavior obvious, and
comment the parts that are Hiver-specific rather than the parts that are ordinary
TypeScript or Python.

If you add an example in one language, adding its counterpart in the other is
welcome but not required; note in the root `README.md` table which languages
exist (use `—` for the one that doesn't).

## Submitting changes

1. Fork the repository and create a topic branch off `main`.
2. Make your change, keeping commits focused and descriptive.
3. Run the example end to end and update its `README.md` (see
   [Verifying your change](#verifying-your-change)).
4. For a TypeScript example, run `npm run typecheck` and `npm run lint` — make
   sure they pass.
5. Push your branch and open a pull request against `main`.
6. Describe what changed and why. Link any related issues.

Please open an issue first for large changes, or for a new example, so we can
discuss the approach before you invest significant effort.

## License

By contributing, you agree that your contributions will be licensed under the
project's [Apache 2.0](LICENSE) license.
