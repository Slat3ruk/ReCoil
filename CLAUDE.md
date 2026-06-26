# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

ReCoil is a GreyScript hacking tool suite for the game **Grey Hack**. GreyScript is a fork of MiniScript (similar to Lua/JS). All `.src` files are source code that must be compiled inside the game using `shell.build()` before they can be run.

Scripts are written and compiled via the **greybel-vs** VSCode extension — source files never touch the game filesystem directly, so hardcoded credentials in source are safe.

## Building

Compiled via the **greybel-vs** VSCode extension. Two entry points, two compiles:

- **`recoil.src`** → compile to `/bin/recoil` — greybel bundles all `#include` modules automatically
- **`wifi.src`** → compile to `/bin/wifi` — standalone, no dependencies

No manual module pre-compilation needed. greybel-vs follows the `#include` chain from each entry point and produces a single binary.

## Architecture

Two entry points:

**`recoil.src`** — main hub binary. Uses `#include "rc_*.src"` to bundle all modules at compile time via greybel-vs. Defines the shared colour palette (`R G Y C W D X DIV`), `progress_bar()` function, and the main REPL loop. All modules depend on these being in scope.

**`wifi.src`** — fully standalone binary. WiFi cracker + hopper combined under one mini-menu. No hub dependency.

Hub modules (each defines one or more functions, no standalone execution):

| File | Functions | Purpose |
|---|---|---|
| `rc_scan.src` | `rc_scan()` | LAN device + port enumeration |
| `rc_secure.src` | `rc_secure()` | Local machine hardening + password change |
| `rc_server.src` | `rc_server()` `server_status()` `server_harden()` `server_procs()` | Proxy server management via SSH |
| `rc_exploit.src` | `rc_exploit()` `rc_shells()` `rc_use()` `rc_drop()` | metaxploit chain, shell buffer |
| `rc_pivot.src` | `rc_pivot()` `rc_pull()` `rc_push()` `rc_store_creds()` `rc_creds()` | Pivot through shells, SCP, cred store |
| `rc_ops.src` | `rc_clean()` `rc_persist()` `rc_backdoors()` | Log cleaning, backdoor user creation |

## Shared state

All session state lives in `get_custom_object` (Grey Hack's built-in shared map, persists for the session):

- `RC_SHELLS` — list of `{ip, shell, level, chain}` — active shell buffer
- `RC_TARGETS` — map of ip → scan data
- `RC_CREDS` — map of ip → `{user, pass}` — cracked/stored credentials
- `RC_BACKDOORS` — list of planted backdoor records

Initialised lazily in `rc_exploit.src` via `_rc_init()`.

## Colour palette

Defined once in `recoil.src`, available to all modules via import scope:

```
R = red (errors)      G = green (success/headers)   Y = yellow (data/values)
C = cyan (IPs)        W = white (labels)             D = grey (secondary)
X = closing tag       DIV = grey divider line
```

Always use these — never hardcode colour tags in modules.

## Key API facts

- `show_procs` output: header on line 0, then `"USER PID CPU MEM COMMAND"` per line — split on `char(10)` then `" "`. Process names with spaces will break the split.
- `wifi_networks` PWR field includes a `%` suffix — strip with `[:-1]` before `.to_int`
- ACK formula: `round(300000 / (pwr + 15))`
- `scan_address` vuln names are inside `<b>name</b>` tags after `"Unsafe check: "` segments
- `overflow` returns: `shell` (shell exploit), `computer` (computer exploit), `1/0` (password/settings), `file` (file exploit) — currently only `shell` is handled in `rc_exploit.src`
- `scp(sourceFile, destFolder, remoteShell, isUpload)` — `0` = pull, `1` = push
- `import_code` requires the target binary to be built with `allowImport = 1`
- Max file size: 160,000 characters

## Server credentials

Hardcoded at the top of `rc_server.src`:
```
SERVER_IP, SERVER_USER, SERVER_PASS, SERVER_PORT
```
Update before compiling. `SERVER_IP` is also used by `recoil.src` banner for the proxy ping status — it's available because `rc_server.src` is imported before the banner runs.

## API reference

Full API doc: `GreyScript API Documentation.htm` (gitignored, local only)
Stripped text version: `api_stripped.txt` (gitignored, local only)
Online: https://documentation.greyscript.org
