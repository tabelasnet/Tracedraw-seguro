# Security

TraceDraw has no accounts, no telemetry and no server of its own; see
`PRIVACY.md`. This file says how to report a problem and records what the
last security review looked at and changed.

## Reporting a vulnerability

Open a private report under Security > Advisories in the repository (or,
if that is not enabled, an issue that does not include a working exploit).
Include the version, the system, and a file or steps that show the problem.

## Review of 2026-10-22

Scope: the whole workspace read for the places where data from outside
(files, the network, other programs, the page's address) meets code that
acts on it. The review was a reading of the source and of the build
files; the code could not be compiled or run where it was made, so the
changes below are covered by new unit tests that CI runs, not by a run
I can vouch for. Run `cargo test --workspace` before publishing.

### What was already sound

- `unsafe` is used only to implement the garbage collector's `Trace`
  trait for the script host (three places, in the script engine glue). The
  `cdr` crate forbids it.
- `.cdr` reading limits what a ZIP member may inflate to (bombs are cut
  off), and `.svg` import follows no external references.
- Scripts that an AI service writes run without file access, with loop,
  recursion and change limits, and all or nothing. A drawing's own macros
  run only when the user starts them.
- The font fetch of the browser build accepts only plain paths under one
  pinned commit of one host.
- No secrets, tokens or keys are committed. The signing steps of the
  release workflow read repository secrets and are skipped without them.

### Fixed

| # | Where | Problem | Change |
|---|-------|---------|--------|
| 1 | `mcp_link.rs` | The file that holds the token of the local agent server was created with the default mode, so on a shared computer other users could read it and drive the open drawing. | Written with mode 0600 on Unix, replacing an older file. |
| 2 | `mcp_link.rs` | The token was compared with `!=`, and was 128 bits derived from a hasher. | Constant-time comparison; four rounds (256 bits). |
| 3 | `mcp_link.rs` | Request and header lines were read without a limit, so a local client could fill memory or stall the single-threaded server. | 8 KB per line, 100 header lines; longer requests are dropped. |
| 4 | `mcp_link.rs` | No check of the `Host` header (DNS rebinding). | A `Host` other than 127.0.0.1, localhost or ::1 is refused. The `Origin` and token checks stay. |
| 5 | `export.rs` | `open_url` passed any text to `cmd /C start` (Windows) or `open` / `xdg-open`. An address with `&` could run a second command on Windows. The address can come from the update service's answer. | Only `http(s)` addresses without control characters, quotes, `^`, `\|` and similar are opened; Windows uses `rundll32 url.dll,FileProtocolHandler` (no shell). |
| 6 | `web.rs` | The browser build opened any address in a tab that kept a reference to the page. | `http(s)` only, with `noopener,noreferrer`. |
| 7 | `updates.rs` | The in-place update followed redirects to any address, over plain HTTP too, and trusted whatever address the release answer named. | HTTPS and GitHub hosts only, checked at the first request and at every redirect. The digest check stays. |
| 8 | `app.rs`, `text_import.rs`, `eps_import.rs`, `psd.rs` | Compressed SVG, DOCX, EPS and PSD data was inflated without a limit (a small file could take gigabytes, and in the browser freeze the tab). DOCX trusted the size the archive claimed. | Reads are capped (256 MB for SVG and EPS streams, 8 times the text limit for DOCX, the size of the planes for PSD). |
| 9 | `web/index.html` | No Content-Security-Policy. | A policy that allows only the page's own scripts (the inline loader by hash), WebAssembly, and connections to HTTPS and to local AI servers. Plus `referrer: no-referrer`. |
| 10 | `web/help_site.py` | A Help link could use any scheme (`javascript:`). | Only `http`, `https` and `mailto` are allowed; others stop the build. |
| 11 | `.github/workflows/build.yml` | The workflow token had the repository default (often read and write). | `contents: read` and `actions: read` for all jobs; the jobs that publish ask for more themselves. |
| 12 | `.gitattributes`, `.github/dependabot.yml` | The CSP hash breaks if the page's line endings change; dependencies and actions had no update bot. | `index.html` is kept with LF endings; Dependabot opens weekly updates for actions and crates. |

### Known limits, not changed

- AI Drawing sends your request, and a description of the page, to the
  address you set, with your API key. With "Remember the key" on, the key
  is kept in plain text in the preferences file (in the browser, in local
  storage). Leave that off on a computer you share, and use a key with a
  spending limit.
- The browser build loads fonts from `raw.githubusercontent.com`
  (google/fonts, pinned commit). GitHub can see those requests.
- A `.cdr` member may inflate to 512 MB (3 GB when the file declares it),
  a limit set to open large real files. A browser tab has less memory than
  that; the desktop app is the place for very large files.
- The update checker and the Help links of the program point to the
  original project (`igtoth/tracedraw`). In a fork, change the constants
  listed in `PUBLICAR-NO-GITHUB.md` if you want them to point at yours.
- Third-party actions are referenced by version tag. Pin them to commit
  hashes if you need supply-chain guarantees; Dependabot can then update
  the hashes.
