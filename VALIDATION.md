# Validation record

Validated on 2026-10-14. This record distinguishes actual toolchain checks,
simulated version output, and behavior that has not been verified.

## Environment

| Component | Observed environment |
| --- | --- |
| OS | macOS 27.2, Apple Silicon (`arm64`) |
| Starship | 1.26.0 |
| Shell | Zsh 5.9 |
| Git | 2.54.0, Apple Git |
| Font | JetBrains Mono Nerd Font / Mono, Nerd Fonts 3.5.1 |
| Rendering | CLI output with `TERM=xterm-256color`, true-color escapes |

Each Starship invocation selected the candidate using `STARSHIP_CONFIG` and used
a separate cache directory. Fixtures and disposable Git repositories were
isolated from the active terminal configuration. The uploaded v1 file remains
unchanged and its full text was retained
before editing the candidate.

## Configuration and rendering checks

The initial local validation run evaluated **139 checks**, all passing after correcting
the fixture command outputs and expectations. This total includes static checks,
fixture rendering, font checks, and actual toolchain checks; it is not a count of
139 distinct projects or platforms.

After the TypeScript icon-only update, follow-up checks confirmed that the parsed
configuration changed only `custom.typescript.symbol`. The new U+E8CA glyph exists
and has a single-cell advance in both checked fonts. Real Node version rendering
and JS/TS suppression were rechecked in JS, TS, mixed JS/TS, and TS-config fixtures.
The full initial suite and unrelated timing cases were not rerun for that cosmetic change.

| Area | Result |
| --- | --- |
| TOML syntax | Parsed successfully with Python 3.11 `tomllib` |
| Native module options | Candidate keys checked against Starship 1.26.0 default module options |
| Format parsing | Candidate prompts rendered without configuration/format warnings in normal cases |
| Non-language modules | `os`, `directory`, `git_branch`, `git_status`, `cmd_duration`, `time`, and `character` are structurally equal to v1 |
| Existing language style | Swift/Rust/Java colors and version precision match v1 |
| Prompt layout | Original two-line scaffold retained; only language references added |
| Language symbols | `$symbol` is referenced by every configured language format |
| Optional versions | Native versions use conditional groups; TypeScript uses conditional `$output` |
| Zsh setup | Generated `starship init zsh` script passed `zsh -n` |
| Glyph mapping | Selected icons matched the official Nerd Fonts glyph-name mapping |
| Glyph coverage | All 15 language glyphs present in the installed regular and Mono fonts |
| Glyph advance | Each selected glyph has the same 600-unit advance as `M` in both checked fonts |

The schema URL is included for editor support. A separate remote JSON Schema
validation was not executed; configuration checks used the installed Starship
module options and actual rendering instead.

## Language detection fixtures

These checks used real Starship with temporary, minimal executable stubs on
`PATH`. The stubs emitted fixed toolchain output and exercised Starship's parsers;
they did not install or run the corresponding SDKs.

| Scenario | Observed result |
| --- | --- |
| Each of the 15 supported languages | Correct language icon/name and a parsed stub version |
| Empty directory | No language modules |
| Two languages | Both displayed |
| Three languages | All three displayed |
| Four languages | All four displayed; no claimed upper limit |
| Large mixed fixture | All matching configured languages displayed, with JS suppressed by TS |
| JS plus TS sources | TS only |
| JS plus `tsconfig.json` | TS only |
| `tsconfig.app.json` / `tsconfig.node.json` | TS only |
| `.tsx`, `.mts`, `.cts` | TS detected |
| `.jsx` | JS detected |
| `.cpp` plus `.h` | C++ only |
| `.c` plus `.cpp` plus shared `.h` | C and C++ |
| Only `.h` | Neither C nor C++ |
| Only `.hpp` | C++ |
| `build.gradle.kts` | Kotlin only |
| `build.gradle` | Java project/runtime hint |
| Java and Kotlin sources together | Both displayed |
| Sources only inside `src/` | No recursive language detection |
| Only `node_modules/` | No JS module |
| Only an F# `.fsproj` | No C# label |
| Supported manifest-only fixtures | Swift, Rust, Python, Go, PHP, Dart, Zig, Ruby, Java detected |
| Virtualenv / Ruby version variables in an empty directory | No language modules |
| TS with an unavailable Node command | TS icon/name without a fake version or `Node` version label |

## Real installed toolchains

These versions were obtained by the configured modules from installed tools in
separate source fixtures. They are the values displayed by the theme, not a list
of SDK versions promised on other machines.

| Language | Actual displayed version |
| --- | --- |
| Swift | 6.4 |
| Rust | 1.99 |
| Java | 25 |
| C / C++ | 21.0 / 21.0 (Clang) |
| Python | 3.11 |
| JavaScript | Node 24.14 |
| TypeScript presentation | Node v24.14.0; not a TS compiler version |
| Ruby | 2.6 |

Kotlin, Go, Zig, .NET, PHP, and Dart were **not tested against real installed
toolchains**. Their detection and format/parser checks used simulated output.

## Existing prompt behavior

A user-supplied Terminal.app screenshot also confirmed basic interactive rendering
before the TS icon update: C, C++, Java, Rust, Python, and JS/TS icons and version
labels were visible; Go displayed its icon/name without a version. Removing the
TS test file changed the displayed module back to JS. This is evidence of basic
Terminal.app operation, not a comprehensive Oh My Zsh integration test. The screenshot
contains personal terminal information and is not included in the public assets.

| Scenario | Verified behavior |
| --- | --- |
| Git clean repository | Branch shown; no dirty status markers |
| Modified and untracked files | `!1` and `?1` shown |
| Staged modification | `+1` shown |
| Local branch ahead of an upstream ref | `⇡1` shown |
| Deep directory | Three path components retained with `…/` truncation |
| Successful command | Arrow uses v1's `#9ECE6A` |
| Failed command | Arrow uses v1's `#F7768E` |
| 1000 ms / 2400 ms duration input | Duration hidden / `2s` displayed |
| Local clock and layout | Clock present; both prompt lines present |

Other Git state combinations, interactive cursor editing, terminal resize/wrap
behavior, and Oh My Zsh plugin interactions were not exercised. Git status
configuration itself is unchanged from v1.

## Swift timeout and performance

A fixture `swift` executable deliberately exceeded the global 3000 ms timeout.
The Starship invocation returned after **3010.78 ms**. Its language icon/name
remained, the unavailable version was omitted, and the second prompt line still
rendered. Timeout messages can be handled by Starship's logging rather than
appearing on stderr; the check used elapsed time and rendered behavior.

Performance was measured using actual installed tools, not stubs. Each case had
one warm-up followed by 12 interleaved v1/v2 subprocess invocations. Measurements
include Starship process startup and rendering; they do not measure Terminal.app
startup or Oh My Zsh initialization.

| Directory | v1 median | v2 median | v1 sample p95 | v2 sample p95 |
| --- | ---: | ---: | ---: | ---: |
| Plain | 5.43 ms | 5.68 ms | 6.91 ms | 6.65 ms |
| Swift | 93.49 ms | 94.12 ms | 95.23 ms | 95.13 ms |
| Rust | 22.24 ms | 22.29 ms | 22.92 ms | 22.53 ms |
| Swift + Rust + Java | 98.42 ms | 98.88 ms | 104.48 ms | 141.34 ms |

With 12 samples, nearest-rank p95 is the largest sample. The mixed-directory
outlier is retained in the table. The small median differences suggest little
added overhead in these local cases, but this does not establish a universal
latency bound. Cold toolchain startup, missing tools, version-manager downloads,
large directories, and network filesystems were not benchmarked.

## Preview provenance and remaining limits

`assets/preview.png`, `swift.png`, `rust.png`, and `mixed.png` were rendered from
real candidate Starship output with a Nerd Font. Paths are anonymous demo paths;
Git status comes from disposable repositories. Exit status and duration were
supplied as illustrative CLI inputs. The images were visually inspected for
clipping, icon rendering, and preservation of the original color/layout choices.

`assets/web.png` shows actual JS and TS fixture prompts after both icons were
aligned to the Devicons family. The displayed numbers are real Node.js runtime
versions. The new TS glyph was verified in the font and the rendered preview.

A user reported slowness in OneDrive project directories. Local small JS/TS
fixtures had warm medians of about 16/18 ms over six samples. No identical-project
OneDrive-versus-local comparison was performed, so the reported cause remains
unconfirmed and no synchronization-related fix is claimed.

The following remain explicitly unverified or unimplemented:

- Comprehensive Oh My Zsh end-to-end operation, including plugin interactions.
- Windows, Linux, other shells, and older Starship/font versions.
- Real SDK behavior for Kotlin, Go, Zig, .NET, PHP, and Dart.
- A strict three-language cap, aggregate relevance ranking, and a `+N` counter.
- Automatic identification of an Xcode-only project root or arbitrary TS config filenames.
- TypeScript compiler version discovery and Bun/Deno-aware TS runtime selection.

For installation, backup, restore, and customization, see [README.md](README.md).
