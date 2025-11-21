## Repository snapshot

- Purpose: Boilerplate for FreeCodeCamp "Quality Assurance and Testing with Chai" lessons.
- Entry points: `server.js` (main app), `test-runner.js` (collects and runs tests), `tests/` (test suites written with Mocha + Chai).

## High-level architecture (quick)

- server.js: Express app that serves `views/index.html` and `public/` static assets. On startup it calls `runner.run()` (with a 1.5s delay) which executes the test runner and populates an in-memory `runner.report`.
- test-runner.js: wraps Mocha, auto-loads every `*.js` inside `tests/`, and converts each test into a structured object (title, context, state, assertions) by parsing the test body and delegating to `assertion-analyser.js`.
- assertion-analyser.js: parses assertion calls of the form `assert.<method>(...)` (and `browser.assert.*` for functional tests). It extracts method names and argument lists; this parser must be preserved when changing test shapes.
- Public/Views: front-end client (`public/client.js`) communicates with server endpoints such as `/_api/get-tests` to show test status in the browser.

## Important developer workflows (how tests run)

- Start the app and run tests: `npm start` or `node server.js` — the server will start on port 3000 and the runner will invoke Mocha after a 1.5s delay. The tests are then available via `/_api/get-tests`.
- The test runner loads `tests/*.js` via Mocha programmatically. Editing tests will affect the runner output immediately when the server is started.
- For direct Mocha runs during development you can run Mocha locally (e.g., `npx mocha`), but the canonical runtime used by learners is the `server.js` + `test-runner` combination.

## Code patterns and project-specific conventions

- Tests directory: all tests live in `tests/` and are written with Mocha's TDD-style (`suite`, `test`) and Chai's `assert` API (see `tests/1_unit-tests.js`).
- Assertion parsing expectations:
  - `assertion-analyser` looks for `assert.<method>(...)` or `browser.assert.<method>(...)` in the test body. Keep assertion calls in that form so the analyser can extract method names and args.
  - The analyser supports nested objects/arrays/strings and restores them when splitting args. Avoid obfuscating assertions behind helper functions unless you also update the analyser.
- Tests used by learners often contain `assert.fail(...)` placeholders that must be replaced with real assertions; do not remove or drastically alter the test `context` names (e.g., `Unit Tests`, `Functional Tests`) because `server.js` filters by those strings when returning test subsets.
- test-runner.js uses Mocha's `tdd` UI and listens to runner events (`test end`, `suite`, `end`) to build the report. If changing runner behavior, preserve the emitted report shape: array of objects { title, context, state, assertions }.

## Integration points & dependencies

- package.json lists runtime dependencies used in this project: express, body-parser, cors, mocha, chai, chai-http, zombie. Functional tests may rely on `zombie` or `browser`-style calls.
- Public client fetches `/ _api/get-tests` to display tests; test filtering occurs using `type=unit` or `type=functional` query params (see `server.js -> testFilter`).

## Editing guidance for an AI agent

- When modifying or creating tests, keep to the `assert.<method>(args...)` pattern so `assertion-analyser.js` parses the assertions correctly.
- If you change how assertion objects are represented (for example, adding new assertion factories or wrappers), update `assertion-analyser.js` accordingly so the test-runner continues to produce consistent reports.
- Avoid changing the `context` strings used in `suite()` declarations; they are used at runtime to categorize tests (e.g., `Unit Tests`, `Functional Tests`).
- Keep changes minimal and self-contained for learners. The public demo expects the server to start and run tests automatically; ensure `server.js` keeps calling `runner.run()` on startup or update the running flow accordingly.

## Files to inspect when making changes

- `server.js` — app entry, test execution trigger, routes, and `/_api/get-tests` response shape.
- `test-runner.js` — how tests are collected, how Mocha is invoked, and the format of the emitted `report`.
- `assertion-analyser.js` — how assertion method and args are parsed; update when changing assertion shapes.
- `tests/*.js` — canonical test suites; follow existing TDD/Chai patterns.
- `public/client.js` and `views/index.html` — front-end test UI and how it queries the server for test status.

## Quick examples (from repo)

- Test runner auto-load: `test-runner.js` calls `fs.readdirSync('./tests').filter(file => file.substr(-3) === '.js')` and `mocha.addFile(path.join(testDir, file))`.
- Assertion parsing pattern: `assertion-analyser.js` expects `assert.someMethod(args...)` or `browser.assert.someMethod(args...)` and returns { method: 'someMethod', args: [...] }.

## QA tips for agents

- Run `npm install` before executing `npm start` locally to ensure dependencies are present.
- Use `npm start` / `node server.js` to emulate the learning environment because the runner is wired into the server lifecycle.
- If you modify tests or the analyser, validate by starting the server and inspecting `/_api/get-tests` JSON for shape and expected entries.

If anything here is unclear or you'd like specific expansions (CI integration, adding `npm test` scripts, or sample edit-to-pass examples), tell me which section to expand and I'll iterate.
