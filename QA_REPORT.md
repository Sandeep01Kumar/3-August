# QA report — hello_world HTTP server

## Missing test coverage

- No test framework, test file, fixture or coverage configuration exists anywhere in the tree: the root holds only the four tracked files this record leaves untouched — `server.js` [server.js:1], `package.json` [package.json:1-11], `package-lock.json` [package-lock.json:1-13], `README.md` [README.md:1-2] — plus this record itself, with no test directory, and neither the manifest nor the lockfile declares one.
- `npm test` is the npm placeholder that prints `Error: no test specified` and exits 1 [package.json:7]; it was not run for this record, so no test result is claimed.
- `lockfileVersion` is 3 with a single root package record and no `resolved` or `integrity` entry, so no test dependency and no transitive package is locked [package-lock.json:4] [package-lock.json:6-12].
- Nothing asserts status, header or body, so the served contract can only be confirmed by the two manual requests below.

## Response check

- Request 1, `curl -sS -D - http://127.0.0.1:3000/`, returned `HTTP/1.1 200 OK` with `Content-Type: text/plain` (no charset parameter) and `Content-Length: 14`.
- The body was 14 bytes, byte-identical to the source literal `'Hello, World!\n'` [server.js:9] — comma, exclamation mark and trailing newline included — so the plain "Hello World" phrasing does not describe what the server sends.
- Request 2, `curl -sS -o /dev/null -w 'status=%{http_code} bytes=%{size_download} type=%{content_type}\n' http://127.0.0.1:3000/`, wrote `status=200 bytes=14 type=text/plain`, identical to request 1, so the reply did not vary between the two requests.
- The launch printed exactly one line, `Server running at http://127.0.0.1:3000/` [server.js:13], and the runtime under test is the installed `node` `v22.23.5` (`node --version`); no other version was installed or exercised, no tracked file changed, no dependency or lockfile entry was added, and `git status --porcelain` listed only this new untracked report when this record was written.

## Untested error paths

- Startup and lifecycle (untested): the port already bound at startup; signal termination, including the `SIGTERM` and `SIGINT` exit codes.
- Transport and parser (untested): an unparsable request head; an unsupported HTTP version.
- Connection level (untested): a connect before the listener exists; a client abort mid-request; an idle kept-alive close; a stalled partial head.
- Exposure (untested): a connection from off the loopback interface, including `[::1]`.
- Packaging and gating (untested): package-root resolution for the absent `index.js` [package.json:5]; the placeholder `test` script [package.json:7].

These paths were skipped by instruction rather than omission, and no outcome is stated for any of them because none was observed.

**Verdict** — The served contract was confirmed on the installed runtime, with status, content type and body as observed; test coverage is absent and the unchecked paths are listed above.
