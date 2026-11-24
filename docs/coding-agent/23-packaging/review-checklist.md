# Module 7, Lesson 23 — Review Checklist

## Release process

- [ ] `RELEASE.md` lists every step in order, with a pass condition for each.
- [ ] The release is built from a clean, tagged commit, and the commit is recorded.

## Build

- [ ] The release is built with the Release configuration, a Windows 64-bit runtime identifier, and self-contained single-file options, checked against the SDK's documentation.
- [ ] The output runs on a machine or account without the .NET runtime.
- [ ] The size is recorded in the README.

## Leaks

- [ ] The release contains no developer path, no sandbox reference, and no key-like string.
- [ ] No debug switch that skips approvals or changes behaviour is enabled in the release.

## First run

- [ ] A real first run completes from the banner to a named next step.
- [ ] It completes with `NO_COLOR` set, and the selection marker is visible.

## Guide

- [ ] `INSTALL.md` was followed on a clean account, by the author or a classmate, without guessing.
- [ ] Each guess was recorded, and the guide was fixed for it.

## Checksums

- [ ] `CHECKSUMS.txt` lists the SHA-256 of each published file.
- [ ] The checksum matches on the clean account, and the README says what a checksum does not prove.

## Reflection

- [ ] I can name the step in my release checklist that I was most tempted to skip.
- [ ] I can say what a checksum proves and what it does not.
