# KARR Key — research evidence

Private evidence store for the KARR Key project. This repository holds the
**binary research inputs**; the written findings derived from them live in
[`neednotapply/KARR-Key`](https://github.com/neednotapply/KARR-Key) under
`research/`.

These are research inputs, never build inputs. Nothing here is compiled,
packaged, or shipped.

## Contents

| Project | Role | APK | IDA databases | Functions |
|---|---|---:|---:|---:|
| `karr_bt_dealer_v3.131` | Dealer inventory, service, unit, CAN, and firmware workflows | 1 | 6 | 12,954 |
| `karr_bt_v2.15` | Customer account, vehicle control, alert, map, and log workflows | 1 | 6 | 13,085 |
| `karr_installer_v1.39` | Technician fitment, repair, removal, CAN, and validation workflows | 1 | 8 | 16,388 |
| **Total** |  | **3** | **20** | **42,427** |

Each project directory holds one untouched source APK, an `ida-project`
directory of annotated IDA 9.1 databases, a `KNOWN_DATA.md`, and a
`manifest.json`. The manifest hashes identify exactly which binary a finding in
the main repository refers to.

An `.i64` represents one native binary, so the databases are intentionally not
merged. Open the database for the library being investigated.

## Why this is a separate repository

Carrying ~392 MB of APKs and IDA databases alongside the application source
made every clone and every CI checkout pay for data no build ever reads.
Splitting them out keeps the application repository source-only.

Stored as ordinary Git objects rather than Git LFS: the largest file here is
about 55 MB, comfortably inside GitHub's 100 MB per-file limit, so no LFS quota
is consumed.

## Access

This repository is private and access is granted individually. Contact
[@neednotapply](https://github.com/neednotapply) to request access.

## Handling

Reconstructed output is not the publisher's literal original source and may
contain credential-like constants. Do not add secrets, BLE captures, or
account/vehicle data. No credential, device key, or challenge response is
included here, and none may be added.
