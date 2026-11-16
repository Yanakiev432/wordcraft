# WordCraft upstream candidate — authenticated Office Sign only

Date: 2026-11-17. Branch `qa/upstream-authenticated-sign-only-20261117`.

## Why

The isolated Windows QA implementation displayed both a legacy direct-CLI signing action and a CMS-authenticated v2 signer. These are conflicting trust paths for an official application. Users must not be offered a choice that silently bypasses module-publisher verification.

This upstream candidate removes the old native CLI signature method, command spawning and `taskkill.exe` dependency, the legacy signing button and its now unreachable progress state. The only Sign UI action is **Sign new copy**. It calls the authenticated `craft_signing_core::office_host_sign::sign_office_via_installed_signer`, never a PATH-selected worker. Native read-only OPC Verify and authenticated signed-module Verify remain available.

The pinned signing core `5b5c14c4e7668cf15cce128750b80a6113fc5f31` refuses signed module 0.2.2 for Office Sign *before* any IPC; 0.2.3 is the minimum. Sign requires a saved DOCX, explicit certificate choice and per-document user consent, a new unoccupied signed-copy location and supported additional-signature policy. A PIN is accepted only in the native Windows/B-Trust provider prompt. Successful DOCX OPC integrity does not by itself prove qualified eIDAS signature status.

## Actual Windows acceptance

- `cargo test -p wordcraft-ui-egui --locked --offline --lib office_sign_ui::tests`: **11/11 PASS**.
- Review tab UI regression: **1/1 PASS**.
- `cargo build -p wordcraft --locked --offline`: **PASS (exit 0)**.
- `cargo check -p wordcraft-cli --locked --offline`: **PASS (exit 0)**.
- Locally compiled, **unsigned** QA GUI: `target/debug/wordcraft.exe`, **91,010,048 bytes**, SHA-256 `8C95B650A78FB30CA37A3009BE03A896E34A71A0176FA89716CE572D6E8923E2`.
- Source scan of the native signing dialog: **0** remaining references to `taskkill.exe`, legacy Sign button or direct `resolve_worker()`.
- No official app installation, user publisher trust policy or B-Trust PIN has been modified.

## Explicit upstream boundaries

This is a fork-source QA branch, **not** a merged or even submitted PR on `storytold/wordcraft`. The shared upstream `craft-plugin-abi` proposal (craft-libs #432) is for sandboxed, currently unsigned WASM guests, and the draft `craft-esign` effort (craft-libs #661) targets signing-request envelopes and UI. Neither is a safe replacement for the separate publisher-signed native Windows certificate broker. Any upstream integration must be approved by the maintainer and should align with the plugin-host RFC craft-libs #414, without granting a sandboxed plugin direct native private-key access.

No real hardware PIN, installed-module end-to-end signature or production QES validation has been performed. Official host lifecycle, publisher public trust, external certificate-chain/eIDAS verification and release integration remain outstanding. **Production NO-GO.**
