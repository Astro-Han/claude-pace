# Changelog

## 0.9.2

- Fix the auto-compact bar briefly showing the model's full window (e.g. `0% 1M`) at the very start of a session. Claude Code sends `context_window.total_input_tokens` as `0` on a fresh session's first frame, and v0.9.1 only recomputed against the auto-compact window when that value was `> 0` — so the bar (and its `1M` label) fell back to the full window until the first API response landed, the exact `1M` flash issue #15 set out to remove. The recompute now fires whenever the field is present, including a genuine `0`, and falls back to the full window only when Claude Code omits the field entirely (older CC that predates `total_input_tokens`). jq now emits a `-1` sentinel for the missing field so a real `0` (fresh-session first frame) and an absent field no longer collapse to the same value (follow-up to https://github.com/Astro-Han/claude-pace/issues/15)
- Add regression coverage for the present-zero first frame measuring against the compact window, plus an explicit guard that a missing field never relabels to the compact window

## 0.9.1

- Track context against `CLAUDE_CODE_AUTO_COMPACT_WINDOW` when set. Since Claude Code 2.1.117 the stdin `context_window.context_window_size` is the model's full window (e.g. 1M for Opus 4.7) and `used_percentage` is measured against it — so on a context capped by `CLAUDE_CODE_AUTO_COMPACT_WINDOW` (e.g. 400K) the bar filled against 1M and looked nearly empty right as auto-compaction was about to fire. When the env var is set and `context_window.total_input_tokens` (CC 2.1.132+) is available, the bar now measures used tokens ÷ the auto-compact window and relabels the size to that window (e.g. `15% 400K`), matching the desktop app's context indicator. Clamped to the real window and capped at 100%; falls back to full-window behavior when the env var is unset or token data is missing (early session). Line 1 keeps the model's full-window label (e.g. `(1M)`) because that reflects the model's capability, not the active budget (https://github.com/Astro-Han/claude-pace/issues/15)
- Add regression coverage for the recompute, full-window fallback, missing-token fallback, over-threshold cap, and real-window clamp

## 0.9.0

- **Breaking:** Remove the last-known quota cache fallback. When stdin `rate_limits` is absent, claude-pace now shows `--` for 5h/7d quota and the session cost when available, instead of reusing a cached snapshot from a previous run. Rationale: stdin carries no provider/account identifier, so the cache could not prove the cached payload belonged to the current session — multi-provider users (e.g. Claude Max + Microsoft Foundry) saw one account's quota leak into the other. Surfacing `--` is an honest failure mode; a wrong-account snapshot is a silent wrong answer. See [docs/decisions/2026-05-20-quota-cache-removal.md](docs/decisions/2026-05-20-quota-cache-removal.md) for the full reasoning and the conditions under which a cache could be reintroduced. Cross-provider contamination reported by @kvdb in https://github.com/Astro-Han/claude-pace/pull/14, which surfaced the underlying identity gap
- Existing `~/.cache/claude-pace/claude-sl-quota*` files from v0.8.x are now orphans, ignored by claude-pace, and safe to delete manually
- Drop ~250 lines of code + tests covering quota cache write/read, snapshot validation, symlink and unreadable-file hardening, and expired-reset guards

## 0.8.6

- Fix Windows Git Bash compatibility: replace `jq --slurpfile` + process substitution with `--argjson`, since `/proc/<pid>/fd/N` is unavailable on Windows and previously caused `MODEL` and `DIR` to render blank (thanks @capraCoder in https://github.com/Astro-Han/claude-pace/pull/13)
- Validate `~/.claude/settings.json` with `jq -e .` before passing to `--argjson`, falling back to `{}` for empty, whitespace-only, or malformed JSON so settings parsing failures stay localized to effort level

## 0.8.5

- Show the effort level as a word (`low` / `medium` / `high` / `xhigh` / `max`) on line 1 instead of a glyph, for readability (https://github.com/Astro-Han/claude-pace/issues/12)
- Bump model-name truncation budget on line 1 from 22 to 28 chars to fit the longest word (`medium`) without clipping
- Update regression tests to assert effort word + pipe alignment for all five levels, and pin the new 28-char truncation budget so a `Sonnet 4.6 (200K) medium` line stays unclipped

## 0.8.4

- Read `effort.level` from stdin when available (Claude Code 2.1.119+), falling back to `~/.claude/settings.json` `effortLevel` on older versions
- Add regression coverage for stdin effort levels and stdin-over-settings precedence (thanks @lifebugz in https://github.com/Astro-Han/claude-pace/pull/11)

## 0.8.3

- Add support for the new `xhigh` effort level and handle `max` explicitly
- Redesign effort indicator glyphs to a 5-step circle family (`◌ ○ ◎ ◉ ●`) for consistent cell width across fonts
- Extend test coverage with per-level glyph and pipe alignment assertions

## 0.8.2

- Avoid rewriting the quota cache when the live stdin snapshot is unchanged
- Keep quota cache hardening intact for symlinked or unreadable cache files while preserving atomic rewrite fallback
- Simplify test helpers and add regression coverage for symlinked and unreadable quota cache files on the live path

## 0.8.1

- Reuse the last known stdin quota snapshot when `rate_limits` is absent, as long as both cached reset times are still in the future
- Ignore invalid, expired, or partial-live quota snapshots instead of overwriting a previously good cache

## 0.8.0

- Remove the Anthropic Usage API fallback, quota tracking now reads only from stdin `rate_limits`
- Quota tracking now requires Claude Code `2.1.80+`; when `rate_limits` is absent, claude-pace shows `--` for quota and may still show session cost

## 0.7.3

- API fallback can now be disabled: set `CLAUDE_PACE_API_FALLBACK=0` to turn off usage polling for CC <2.1.80
- Fix git cache key collision: paths like `/foo-bar` and `/foo/bar` no longer share a cache file (now uses SHA-1 hash)
- Old-style git cache files (path-based names like `claude-sl-git-_Users_*`) are orphaned; safe to delete from your cache directory (`$XDG_RUNTIME_DIR/claude-pace/` or `~/.cache/claude-pace/`)

## 0.7.2

- Fix OAuth token exposure in Usage API fallback: avoid leaking bearer token in curl argv / process listings
- Reject malformed tokens containing CR/LF before invoking curl

## 0.7.1

- Harden cache handling: move from shared `/tmp` to private per-user directory (`$XDG_RUNTIME_DIR/claude-pace` or `~/.cache/claude-pace`, mode 700)
- Validate all cache-read fields before arithmetic evaluation
- Switch cache delimiter from `|` to ASCII Unit Separator (branch names with `|` no longer corrupt parsing)
- Disable caching entirely when no safe cache directory is available

## 0.7.0

- Rename project from claude-lens to claude-pace
- Add `npx claude-pace` one-step installer
- Add plugin marketplace support

## 0.6.2

- Fix `((var++))` unsafe under `set -e` (exit status 1 when variable is 0)

## 0.6.1

- Remove ±5% silent zone for pace delta (any non-zero delta now visible)

## 0.6.0

- Display usage as used% instead of remaining% (lower = better)
- Use ⇡/⇣ arrows for pace delta (⇡ = overspend, ⇣ = surplus)
- Invert pace delta sign to match intuitive convention

## 0.5.0

- Symmetric single-pipe alignment redesign (~270 lines)
- Add performance metrics to comparison table
- Remove session duration display

## 0.4.1

- Merge formatting functions (`_uf`/`_pace`/`_rc`) into single `_usage`
- Restore extra usage display when actual spending exists
- Fix 7d reset countdown always showing

## 0.4.0

- Use stdin `rate_limits` for real-time usage (no network calls on CC >= 2.1.80)
- Add plugin marketplace support for one-command install
- Fix `jq null|floor` crash on jq 1.7.1

## 0.3.1

- Show reset countdown for usage windows
- Fix cache degradation: preserve good data on API failure

## 0.3.0

- Full rewrite (962 lines to 142)
- Remaining% display, pace delta, conditional cost display
