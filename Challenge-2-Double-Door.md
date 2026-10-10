# Challenge 2 — String check (claude given challenge)

**Topic:** String comparisons  
**Tools:** IDA Free (static) + Local Windows debugger

## What the program does

Asks for a username and a password. Both are checked against hardcoded strings with `strcmp`. Only `admin` + the right password prints success.

## Key disassembly

```asm
lea   rax, [rbp+Str1]
lea   rdx, Str2             ; "admin"
call  strcmp                ; compare username vs "admin"
test  eax, eax
jnz   short loc_...         ; not equal -> "Unknown user"
lea   rax, [rbp+var_80]
lea   rdx, aSup3rs3cr3t      ; "Sup3rS3cr3t"
call  strcmp                ; compare password
test  eax, eax
jz    short loc_1400015F6   ; equal (0) -> "Access granted!"
```

## How I solved it (static)

- A string can't fit in a single `cmp`, so the compiler calls `strcmp`.
- IDA auto-comments the string pointers, so the expected values are shown in plain text next to the `lea` before each `strcmp`.
- `strcmp` returns 0 when strings are equal, so the code uses `test eax, eax` then `jz` (jump if zero = equal = correct).

## Answer

```text
Username: admin
Password: Sup3rS3cr3t
-> Access granted! Welcome, admin.
```

## Concepts learned

- String checks use `str*cmp`, not `cmp`. Look for the `lea <string>` feeding it.
- `strcmp` returns 0 on match -> `test eax,eax` + `jz` means “jump if matched.”
- Plaintext secrets in a binary are a real weakness — a disassembler reads them instantly.
- The Strings window (Shift+F12) often hands you the answer directly.
