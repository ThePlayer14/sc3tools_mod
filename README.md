# sc3tools

A CLI tool for extracting and modifying text in .scx and .msb scripts found in visual novels based on MAGES. engine. It's meant to be a replacement for the old, overly complicated tool which had the same name and was part of the now-abandoned [SciAdv.Net project](https://github.com/CommitteeOfZero/SciAdv.Net).

## Supported games

- STEINS;GATE (Steam)
- CHAOS;HEAD Love Chu☆Chu! (PS3 & Impacto)
- ROBOTICS;NOTES ELITE
- STEINS;GATE: Linear Bounded Phenogram (Steam)
- CHAOS;CHILD (Steam & GOG)
- STEINS;GATE 0 (Steam)
- CHAOS;CHILD Love Chu☆Chu!! (PS4 & Impacto)
- ROBOTICS;NOTES DaSH
- 11eyes CrossOver (Xbox 360)

## Usage

Run `./sc3tools` with no arguments to see the list of the available commands, as well as the list of the supported games and their aliases (such as `sg0` for Steins;Gate 0).

Run `./sc3tools help <command>` to see the help message for a specific command.

Here's an example of how you can extract text from the Robotics;Notes scripts:

`./sc3tools extract-text C:/src/CoZ/rne-msb/*.msb rn`

To use the  `replace-text` function, the replacement `.txt` file has to be named in the same way as the original script file. 

It is also advised to create a copy of the original before replacement (copy source file, paste, then append `.org` to the end, so you know which is the **or**i**g**inal).

If you've made sure to create a copy of the original, just use `sc3tools replace-text SC000.scr SC000.txt 11eyes` (in the case of 11eyes CrossOver)

The output files will be placed in a subfolder named `txt` (in this case, `C:/src/CoZ/rne-msb/txt`).

## Compilation
Install the Rust toolchain for Windows from [rustup.rs](https://rustup.rs).

Clone the repository using Git: `git clone https://github.com/ThePlayer14/sc3tools_mod.git`

Navigate to the cloned folder `sc3tools_mod` and open the context menu / rightclick menu in File Explorer and click on "Open in Terminal"

From this point you can run `cargo build` to build a "dev" (debug) release, or run `cargo build --release` to make a "release" build.

## Features added alongside the color bugfix
**1. External game `resources` directory support** (`gamedef.rs` + `lib.rs`):
- New `--resources-dir <path>` CLI flag (global, works on both subcommands)
- `gamedef::load_resource()` — checks external path first, falls back to embedded
- `gamedef::load_gamedefs_json()` — loads from external file or embedded
- `gamedef::build_gamedefs_from_json_with_base()` — accepts external base path
- `GameDef::new()` now takes `external_base: Option<&Path>`
- Adding a new game = drop a folder with `charset.utf8` + `compound_chars.map` into the resources dir and add an entry to `gamedefs.json` — no code changes made

**2. MagesTools-style tags in extraction**
- New `--mgs-format` CLI flag (use with `extract-text`)
- Outputs a MagesTools-compatible script from a compiled script file
 
## Known issues
- This tool originally couldn't handle color setting in dialogue correctly (such as in the case of 11eyes CrossOver), and it will leave a truncated script if that is happened.

  Example of the telltale sign:
```
  Processing "D:\\script\\SC000.scr"... 
  Error: SC000.scr, line 73: expected more input.
```
- With the release in the Releases tab, this is no longer the case. You can also build a functioning program for other platforms from the code.
- For details of the fix, see [The bugfix report.](https://github.com/ThePlayer14/sc3tools_mod/blob/main/rust-bugfix.md)
