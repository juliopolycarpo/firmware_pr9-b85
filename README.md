# Machinist PR9 (B85) firmware

A [Biosmod Engine](https://github.com/juliopolycarpo/biosmod-engine) catalog: the board profile, findings, patches and evidence for one board or one board family. `bme guide catalogs` describes every document.

## This board

| | |
|---|---|
| Board | Machinist PR9, Intel B85; physical revision not recorded. The firmware programs a Nuvoton NCT5532D Super I/O; the chip marking was not read |
| Board profile | `boards/machinist-pr9-b85.json` |
| Firmware build | `x99ha00m-20221202`: X99HA00M 2022-12-02, version string `X99H BIOS Date: 12/02/2022 Ver: X99HA00M` |
| Reference dump | full-chip read of the reviewed board, 16,777,216 bytes, SHA-256 `095a6f5a15c56b245f38044c53298c03f3fb5cb13294ce5f0ebaa84cd57375ec` |

The reference dump holds that board's NVRAM, so a read of another PR9 has another SHA-256. `bme` identifies the build by its modules and patches a module only when its hash matches the recorded source.

```sh
bme scan dump.bin --catalog . --allow-unsigned-local
bme patch dump.bin --recommended --catalog . --allow-unsigned-local
```

No hardware observation is recorded for this board. Read `bme guide flashing` before you write an image to the chip.

## Shared with the MR9A PRO

The Machinist E5-MR9A PRO (Q87) ships a build of the same X99-derived AMI code base, so most defects exist on both boards. A shared defect keeps one id in both repositories: `firmware_mr9a-pro-rev1.2` records it with the same finding text and its own patch variant, and holds the hardware tests. A finding whose `affects.label` is `MR9A PRO and PR9` is shared. The catalog bundled with `bme` holds the records of both boards.

`bme` checks that the two repositories agree on every shared record, and fails with the record, the field and both values when they differ:

```sh
bme catalog merge . ../firmware_mr9a-pro-rev1.2 --check
```

The catalog part of this repository equals what `bme catalog split official --board machinist-pr9-b85` writes from the bundled catalog. A record added here reaches the bundled catalog through `bme catalog merge`.

## Layout

| Path | Holds | In git |
|---|---|---|
| `catalog.json` | the index: id, version, and the length and SHA-256 of every file in `boards/`, `findings/`, `patches/`, `evidence/` and `provenance/` | yes |
| `boards/` | board and revision profiles with the firmware builds they ship | yes |
| `findings/` | one defect or observation per file, patched or not | yes |
| `patches/` | one patch record per file, with one variant per firmware build | yes |
| `evidence/` | one hardware observation per file, bound to the image that was flashed | yes |
| `provenance/` | sources you wrote: assembly, harness scripts | yes |
| `notes/` | investigation notes in prose | yes |
| `work/` | dumps, cases, extracted modules, tool output | no |

Git records no empty directory: each directory reaches a clone with its first file.

## What stays out of git

`.gitignore` admits the paths marked "yes", `README*`, `LICENSE*`, `.github/`, `catalog.json.minisig`, `.gitignore` and `.gitattributes`. Anything else at the top level is ignored: a new top-level file does not appear in `git status` until you add a `!/name` line for it.

In every directory it also ignores firmware images and extracted modules (`*.bin`, `*.rom`, `*.pe`), ACPI tables, analysis projects, executables, archives, videos, PDFs and key files.

Patterns cannot read text. Before you commit a file under `notes/` or `provenance/`, check that it has no:

- serial number, UUID or MAC address (kernel logs, `dmidecode` output and network listings carry them);
- disassembly or decompiled code of a vendor module beyond the lines a finding quotes;
- path of your machine.

Name a dump or a flashed image by file name, SHA-256 and size. Never commit it.

## Work on it

```sh
cp /path/to/dump.bin work/
bme case init work/dump.bin -o work/case      # scan, draft a board profile and firmware build
bme case export work/case -o work/submission  # draft records to review and copy into boards/, findings/, evidence/, provenance/
bme catalog index .                           # after every change to a record
bme catalog validate .
bme scan work/dump.bin --catalog . --allow-unsigned-local
```

`bme guide cases` and `bme guide porting` cover the investigation itself.

## Commits

One record per commit, with the id in the scope and the defect in the subject, so `git log` and code search find it by id, module or symptom:

```text
feat(board): add ACME X99-A rev 1.0 with firmware X99A 2023-01-10
feat(H11): record unbounded PCU mailbox and UBOX semaphore waits
fix(B09): widen every inventoried SMM configuration-table counter
test(evidence): boot ACME X99-A with B01-B06 and B02-B03 flashed
docs(notes): describe the S3 resume investigation
```

`feat` adds a board or a finding, `fix` adds or changes a patch record together with the findings it fixes, `test` adds a hardware observation. A commit that changes a file in an indexed directory carries the re-indexed `catalog.json` and passes `bme catalog validate .`.

## Releases

`bme catalog update` refuses a version it already accepted with other content, and a version older than the newest one it accepted. So `version` in `catalog.json` changes only in a release commit: raise it, run `bme catalog index .`, sign `catalog.json` with minisign, commit `catalog.json.minisig` and tag the commit. Users register the tag's raw `catalog.json` URL:

```sh
bme catalog add pr9-b85 https://example.org/path/to/catalog.json --trust-key <public key>
bme catalog update pr9-b85
```

Give them the public key outside this repository. A key committed here proves nothing.
