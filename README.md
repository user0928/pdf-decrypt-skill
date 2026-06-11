# pdf-decrypt

Codex skill for decrypting known-password PDF files with `qpdf`.

This skill is intended for PDFs where the user already knows the password and is authorized to remove password protection. It does not support password cracking, brute forcing, guessing, or bypassing unknown access controls.

## What It Does

- Uses `qpdf --decrypt` to remove password protection from a PDF.
- Defaults to in-place replacement: the final PDF keeps the original filename and original folder.
- Writes to a temporary file first, verifies the decrypted result, then replaces the original PDF.
- Verifies the final file with `qpdf --show-encryption` and `qpdf --check`.
- Avoids leaving `_decrypted.pdf`, `_unlocked.pdf`, numbered copies, or other final duplicate files unless explicitly requested.

## Wake Words

The skill metadata includes English and Chinese triggers such as:

- `PDF解密`
- `pdf解密`
- `解密PDF`
- `去除PDF密码`
- `移除PDF密码`
- `已知密码的PDF`
- `原地解密`
- `覆盖原PDF`
- `qpdf --decrypt`
- `password-protected PDF`
- `encrypted PDF`
- `decrypt PDF with password`

You can also invoke it explicitly:

```text
Use $pdf-decrypt to decrypt this known-password PDF.
```

## Dependency

Install `qpdf` and make sure it is available on `PATH`.

Verify:

```powershell
qpdf --version
```

## Install

Copy this folder into your Codex skills directory:

```text
$CODEX_HOME/skills/pdf-decrypt
```

The installed folder should contain:

```text
pdf-decrypt/
  SKILL.md
  agents/
    openai.yaml
```

Restart Codex after installing or updating the skill so it can be picked up.

## Default Workflow

For an input file such as:

```text
input.pdf
```

The skill uses a temporary file:

```text
input.qpdf-decrypt.tmp.pdf
```

Then verifies it and replaces the original:

```powershell
qpdf --password=<password> --decrypt "input.pdf" "input.qpdf-decrypt.tmp.pdf"
qpdf --show-encryption "input.qpdf-decrypt.tmp.pdf"
qpdf --check "input.qpdf-decrypt.tmp.pdf"
Move-Item -LiteralPath "input.qpdf-decrypt.tmp.pdf" -Destination "input.pdf" -Force
qpdf --show-encryption "input.pdf"
qpdf --check "input.pdf"
```

Expected verification output includes:

```text
File is not encrypted
```

## Safety Notes

- The default behavior overwrites the original PDF only after the temporary decrypted file passes verification.
- If the temporary file already exists before the run, the skill should stop instead of silently overwriting it.
- If a failed run leaves the one explicit temporary file created by the workflow, remove only that exact file path or ask the user before cleanup.
- Do not use this skill for unknown-password PDFs.
