---
name: pdf-decrypt
description: Use this skill when the user asks to decrypt, unlock, remove password from, or 去密码/解除加密/解密 a known-password PDF using qpdf. Trigger for PDF解密, pdf解密, 解密PDF, 去除PDF密码, 移除PDF密码, 已知密码的PDF, 原地解密, 覆盖原PDF, qpdf --decrypt, password-protected PDF, encrypted PDF, and decrypt PDF with password. Default behavior is in-place replacement at the original path with the original filename; do not use for cracking unknown passwords.
---

# pdf-decrypt

## Scope

Use this skill for removing password protection from a PDF when the user provides the password or confirms they are authorized to decrypt the file.

Do not attempt password cracking, guessing, brute force, or bypassing access controls. If the password is unknown, ask the user for the password.

## Preferred Tool

Use `qpdf` for decryption.

1. Prefer `qpdf` from `PATH`.
2. If `qpdf` is not available in the current shell, ask the user for the installed `qpdf.exe` path or install qpdf through an appropriate package manager or official release.
3. Verify with:

```powershell
qpdf --version
```

## Workflow

1. Resolve the exact input PDF path with `Get-ChildItem` or `Test-Path`.
2. Default to in-place decryption: the final file must stay at the original path with the original filename.
3. Because `qpdf` should not use the same path for input and output, create one explicit temporary output path in the same folder:
   - Default temporary pattern: `<original_stem>.qpdf-decrypt.tmp.pdf`
   - If that temporary file already exists, stop and ask the user before continuing. Do not silently overwrite unrelated files.
4. Run qpdf into the temporary file:

```powershell
qpdf --password=<password> --decrypt "<input.pdf>" "<input_stem>.qpdf-decrypt.tmp.pdf"
```

5. Verify the temporary file:

```powershell
qpdf --show-encryption "<input_stem>.qpdf-decrypt.tmp.pdf"
qpdf --check "<input_stem>.qpdf-decrypt.tmp.pdf"
```

The desired confirmation is `File is not encrypted` and no syntax or stream encoding errors from `qpdf --check`.

6. Only after verification succeeds, replace the original file with the temporary file:

```powershell
Move-Item -LiteralPath "<input_stem>.qpdf-decrypt.tmp.pdf" -Destination "<input.pdf>" -Force
```

7. Verify the final original path:

```powershell
qpdf --show-encryption "<input.pdf>"
qpdf --check "<input.pdf>"
```

The final state should have no extra decrypted-copy PDF. The source folder should contain the original filename only, now decrypted.

## Safety Rules

- Default behavior is to overwrite the original PDF path after successful verification of the temporary decrypted file.
- Do not leave `_decrypted.pdf`, `_unlocked.pdf`, numbered copies, or other final output files unless the user explicitly asks for a separate copy.
- Do not batch delete files or directories.
- Do not use `del /s`, `rd /s`, `rmdir /s`, `Remove-Item -Recurse`, or `rm -rf`.
- If cleanup would require deleting a folder or many files, stop and tell the user to handle cleanup manually.
- If a failed run leaves the single temporary file created by this workflow, remove only that one explicit file path or ask the user before cleanup.

## Reporting Back

Reply with:

- the original PDF path that was replaced in place,
- the qpdf verification result,
- confirmation that the filename and folder did not change,
- confirmation that no extra final decrypted-copy PDF was intentionally created,
- any limitation, such as an incorrect password or qpdf not being available.
