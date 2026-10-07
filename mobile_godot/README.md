# Mobile Godot Bootstrap — canonical packaging architecture

Branch: **`mobile-godot-bootstrap`**  
Status: architecture canon for Android packaging.

## Goal

Android build is a **small Godot launcher/bootstrap APK**. It is not the game
archive and must not become a 10+ GB monolithic APK.

The launcher is responsible for discovering, downloading, verifying, updating
and activating external game/mod packages.

## APK contains

- Godot launcher UI;
- bootstrap/update state machine;
- manifest client;
- local install/profile metadata;
- download queue and resume/retry logic;
- package verification;
- activation/rollback metadata;
- minimal launcher-only assets.

## APK does not contain

- the full Space Rangers game payload;
- every Children of Eltan asset;
- all optional mods;
- giant prepacked archives that can be fetched as versioned packages.

## Canonical flow

`APK -> fetch manifest -> resolve package graph -> compare local versions ->
download delta/full packages -> verify -> stage -> atomically activate profile ->
launch`

## Package layers

Keep at least these logical layers independent:

1. **launcher** — APK itself;
2. **game/runtime** — files required by the mobile runtime;
3. **Children of Eltan** — project content;
4. **mods** — optional/selected mod packages;
5. **profile/config** — enabled set, versions and launch settings.

A change in layer 3 or 4 must not require rebuilding layer 1.

## Manifest requirements

The manifest design should eventually carry, per package:

- stable package id;
- version;
- compatible runtime/game version;
- download location;
- compressed and installed sizes;
- checksum/hash;
- dependencies;
- optional/required flag;
- install target;
- rollback/previous-version metadata where practical.

Do not hard-code production URLs or signature schemes until they are confirmed.

## Download/update behaviour

Required design properties:

- resumable downloads;
- retry with bounded backoff;
- enough free-space check before download/install;
- checksum verification before activation;
- staging directory;
- atomic or recoverable activation;
- interrupted update must leave the previous working profile usable;
- cache cleanup policy;
- user-visible progress and actionable errors.

## Mod model

Mods are external versioned packages. A mod update should download only the
changed package(s), not another full APK and not the whole game.

The launcher may later consume profiles compatible with SR Mods Launcher where
that is technically useful, but Android packaging must stay independent from
Windows-only launcher/native-bridge assumptions.

## Relationship to gameplay branches

This branch owns **delivery/bootstrap/runtime packaging**.

The canonical gameplay base remains `deti_eltan`. Keller / Deep Hyper and
Smart Diplomacy are gameplay concepts and should be implemented in the gameplay
codebase; this branch should consume those files as packages rather than fork
their design independently.

## Agent rule

When working in this branch, optimize for a **small APK + independently
updateable external content**. Reject changes that solve packaging by embedding
the entire game/mod set into the APK unless the owner explicitly changes this
canon.

Do not claim the downloader/package system is implemented until code and
evidence exist. The current Godot vertical slice is only the starting UI/runtime
surface.
