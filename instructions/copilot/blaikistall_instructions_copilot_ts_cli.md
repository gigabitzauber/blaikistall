# Blaikistall Copilot instructions for Node.js TypeScript CLI Application

## Ground rules

- You must follow all rules depicted in the file .blaikistall/blaikistall_ground_rules.md in all interactions. If you are unsure about how to apply these rules in a specific situation, ask for clarification.

## Target environment

- Target runtime: **Node.js 22**.
- The application is **portable** and must run as a **CLI**.
- Use **npm** for dependency management and scripts.
- Codebase language: **TypeScript**.
- The app must be **high-quality** and **production-ready**.
- Use **ESM only** (no CommonJS). Prefer modern platform APIs.
- Ensure excellent **DX**, **testability**, **linting/formatting**, and **clear error handling**.
- Avoid runtime ts-node; ship compiled JS.
- Ensure the CLI entrypoint has a stable command name.

## Platform support

- The app must support both: Bash 4+ and Windows cmd.exe.
- When in doubt, favor Bash support.
- When providing usage examples, use **Bash syntax**, neither PowerShell nor cmd.exe syntax.
- When spawning subprocesses:
  - Prefer `node:child_process` `spawnSync(cmd, args, { shell: false, stdio, encoding: 'utf8' })`.
  - Do **not** rely on PowerShell features or cmdlets.
  - Never build command strings
- Avoid instructions that require PowerShell (execution policy changes, PS-only piping idioms, etc.).
- CLI interface expectations:
  - Provide `--help` and a non-zero exit code on invalid usage.
  - Use `stdout` for normal output and `stderr` for errors/warnings.
  - Return meaningful exit codes (`0` success, `1` general failure, `2` usage/config errors).
  - prefer flags and stdin piping over interactive prompts

## Coding Standards

- Use **ESM** (ECMAScript modules).
- Use strict TypeScript.
- Prefer type safety:
  - Avoid `any`; prefer `unknown` + narrowing when needed.
  - Prefer explicit types at module boundaries (public APIs, IO) when inference isn’t clear.
- Use `satisfies` where it improves correctness.
- Prefer readonly data and immutability when reasonable.
- Use discriminated unions for error variants and command results.
- Write clear, predictable async code:
  - Prefer `async/await` over raw promise chains.
- Stream large files instead of loading them fully into memory.
- Treat all input (args, env vars, stdin, files) as untrusted; validate and sanitize.
- Prefer `import`/`export` syntax everywhere.
  - Use `node:` prefix for built-ins (e.g. `node:fs/promises`).
  - Avoid `require`, `__dirname`, `module.exports`.
- Prefer small, single-purpose functions.
- No deeply nested logic; use early returns.
- Avoid cleverness; optimize for readability.
- Use `assert`-style checks sparingly; prefer validation with helpful messages.
- Validate all external input: CLI args, env vars, file contents, stdin.
- Update help text / docs when flags or commands change.
- Eliminate duplication ruthlessly
- Express intent clearly through naming and structure
- Make dependencies explicit
- Minimize state and side effects

## Linting & formatting (ESLint drives Prettier)

- ESLint is the single entry point.
- Prettier is enforced via ESLint (`eslint-plugin-prettier` / `eslint-config-prettier`) so formatting issues show as lint issues.
- It is always ok to override the default prettier rules via ESLint settings.
- Rules:
  - keep imports sorted and consistent,
  - ban unused vars,
  - prefer `const`,
  - consistent type imports when helpful.
- Never fight the formatter: adjust code to satisfy Prettier.

## Architecture

### General

- Separate concerns (e.g., command parsing/dispatch vs business logic vs IO).
- Keep CLI entrypoint thin; delegate logic to modules in `src/`.
- Separate modules:
  - `src/cli/` for command parsing, help text, stdin/stdout, exit codes.
  - `src/core/` for business logic (pure functions where possible).
  - `src/io/` for filesystem, network, process interactions (injectable/mocked in tests).
- Use stable, cross-platform filesystem/path handling.
- Prefer dependency injection for side effects (fs, stdout, stderr, process exit).
- Provide stable, documented exit codes:
  - `0` success
  - `1` expected user error (validation, bad args, not found)
  - `2` unexpected/internal error
- Provide `--help` and `--version` for all commands.
- Build output goes into `<root>/dist`.
- Handle Windows/macOS/Linux path differences.
- Avoid unnecessary filesystem traversal.
- Prefer async APIs; avoid blocking I/O.
- Keep dependencies minimal; prefer built-ins.

### Errorhandling

- Handle errors intentionally; avoid swallowing errors.
- Error messages must be:
  - short,
  - actionable,
  - printed to **stderr**,
  - followed by a one-line hint to run `--help` when relevant.
- Do not print stack traces for expected user errors.
- For unexpected errors:
  - print a concise message,
  - optionally print stack trace only under `--debug` or `DEBUG=1`.
- Use typed errors:
  - Create custom error classes for domain/user errors.
  - Keep a single place that converts errors into CLI messages + exit codes.
- Avoid throwing strings.

### Output rules

- Default to human-readable output.
- If machine consumption is plausible, offer `--json` producing strict JSON.
- Never mix structured JSON with human logs in `stdout`; send logs to `stderr`.

### Logging

- No noisy logging by default.
- Support `--verbose` and/or `--debug`.
- Logging should be easily stubbed in tests.

## Versioning

- Always follow **SemVer**.
- Version is defined in `package.json` and synced to `src/core/version.ts` via `scripts/sync-version.js` during prebuild.
- Runtime code must import `VERSION` from `src/core/version.ts`, never read `package.json`.

## Documentation & Commenting

- Use **TSDoc** on **public APIs only** (exported functions/classes/types intended for external or cross-module use).
- Use inline comments **very sparingly**:
  - Only when the purpose (“what”) cannot be made obvious by naming/structure.
  - Prefer self-documenting names and small functions over comments.
- CLI usage examples are **not required**.

## Testing

- Use **Vitest**.
- If unsure which kind of tests to write, choose unit tests.
- Test files:
  - Must be named like `*.test.ts`.
  - Must be stored in `tests`.
- Tests must be runnable in CI (favor non-interactive execution via `vitest run` in CI contexts).
- Write tests for:
  - core logic (pure unit tests),
  - CLI argument parsing (light integration tests),
  - I/O modules with mocked dependencies.
- Prefer deterministic tests (no network, no time dependencies) unless explicitly required.
- Use fake timers for time-based behavior.
- If it is required to use file I/O, mock it by the help of memfs.
- Never write a test that depends on build artifacts.
- When mocking, make clear if a mock is actually a mock or a spy by choosing an appropriate name.
- An example of a proper CLI entrypoint implementation is:

  ```ts
  // cli.ts
  import { businessLogic } from "./businessLogic.js";
  import { pathToFileURL } from "node:url";

  export function run(argv: string[]): number {
    const input = args.join(" ").trim();
    console.log(businessLogic(input));
    return 0;
  }

  if (import.meta.url === pathToFileURL(process.argv[1]).href) {
    process.exit(run(process.argv));
  }
  ```
