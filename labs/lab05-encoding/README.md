# Lab 5 — Encoding forensics

Name: <имя> · Date: 14.11.2026 · Tools: Git Bash (file, xxd, iconv), Excel, LibreOffice Calc

## What I received
- Excel: Russian text is readable, Kazakh letters are replaced with look-alikes ("Әбдірахманова" shows as "Јбдірахманова").
- LibreOffice: the import dialog chose Cyrillic (Windows-1251) and the separator ";"; same damage as in Excel.
- Terminal (cat): Cyrillic shows as blocks and garbage, because the terminal expects UTF-8 and the file has single bytes.

## Diagnosis (made before the fix)
- file said: Non-ISO extended-ASCII text, with CRLF line terminators (evidence/file-before.txt)
- First 16 bytes: 49 44 3b d4 e0 ec e8 eb e8 ff 3b c8 ec ff 3b ce (evidence/hexdump-before.txt)
- Encoding: KZ-1048 (iconv name RK1048) — because every Cyrillic letter is one byte (e.g. "ф" is d4, "а" is e0), and the Kazakh "Ә" in "Әбдірахманова" is the single byte a3, which Windows-1251 reads as "Ј" and RK1048 reads as "Ә".
- Line ending: CRLF — because the first line ends with 0d 0a.
- BOM: no — the file starts with 49 44 ("ID"), not EF BB BF. In the fixed file I added it, because Excel on Windows only recognises UTF-8 on double-click when the BOM is present.
- Separator: ";" (byte 3b).
- I kept CRLF in the fixed file because Excel expects it and cat in Linux is not affected by it.

## Fix
```sh
iconv -f RK1048 -t UTF-8 evidence/staff-broken.csv > evidence/staff-fixed.csv
printf '\357\273\277' | cat - evidence/staff-fixed.csv > tmp && mv tmp evidence/staff-fixed.csv
```
Full script: scripts/fix.sh

## Verification
- file after: UTF-8 (with BOM) text, with CRLF line terminators (evidence/file-after.txt); Cyrillic letters are now d0 xx / d1 xx pairs.
- Screenshot of LibreOffice, Git Bash and Excel: evidence/opens-correctly.png
- In Git Bash a few Kazakh letters are drawn with extra spacing because of the terminal font; the bytes are correct (file-after.txt) and both GUI applications show the names correctly.

## What did not work the first time
I first assumed Windows-1251 because the Russian names were readable. The Kazakh letters were wrong ("Ә" became "Ј"), which showed that it was the Kazakh code page, not Windows-1251. Switching to iconv -f RK1048 fixed it.

## Evidence
- [evidence/staff-broken.csv](evidence/staff-broken.csv) — the original, untouched
- [evidence/staff-fixed.csv](evidence/staff-fixed.csv) — UTF-8 with BOM, CRLF
- [evidence/file-before.txt](evidence/file-before.txt) — output of file before the fix
- [evidence/hexdump-before.txt](evidence/hexdump-before.txt) — first bytes: no BOM, one byte per letter
- [evidence/file-after.txt](evidence/file-after.txt) — file and xxd after the fix
- [evidence/opens-correctly.png](evidence/opens-correctly.png) — all three applications
