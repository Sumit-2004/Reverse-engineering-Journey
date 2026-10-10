# Challenge 3 — Password Cracker

**Topic:** Loop + attempts + Base64 + backdoor  
**Tools:** IDA Free (static) + Local Windows debugger

The first genuinely non-trivial one. Much more than a single check.

## Structure of `main()`

- `var_4` = attempt counter, `var_8` = success flag; 3 attempts allowed.
- `system("cls")` prints a banner and “You have 3 attempts...”.
- Loop condition: keep going while `attempts <= 2` **and** `flag == 0`.
- Reads input with `fgets(buffer, 50, stdin)`, then strips the trailing `\n` (`strlen` -> look at last char -> if `0x0A`, overwrite with `0`).
- Calls `loading_animation` (a fake effect — decoy).
- Calls **`check_password(buffer)`** and branches on the return value.
- On wrong password: prints “Wrong password. Try again.” and increments `attempts`.
- After 3 failures: prints “Too many failed attempts. System locked” and exits.
- On success: prints a **random 4-digit code** (`rand() % 10000`, done via the `imul 0x68DB8BAD` + shifts trick) and sets the success flag. This is a reward, not the check — a decoy I had to avoid getting fooled by.

## The real check — inside `check_password()`

```asm
mov   [rbp+Str1], rcx          ; Str1 = my input
lea   rax, aY3jhy2ttztiwmj     ; "Y3JhY2ttZTIwMjQ=" (Base64!)
...
call  base64_decode            ; decode it into Str2
lea   rdx, [rbp+Str2]          ; Str2 = decoded password
mov   rcx, [rbp+Str1]          ; my input
call  strcmp                   ; compare
test  eax, eax
jnz   short loc_1400016D9      ; not equal -> try the backdoor check

loc_1400016D9:
lea   rdx, SubStr              ; "hack"
mov   rcx, [rbp+Str1]
call  strstr                   ; does my input contain "hack"?
test  rax, rax
jz    short loc_1400016F8      ; not found -> return 0 (fail)
; otherwise -> return 1 (success)
```

## How I solved it (static)

- The password is stored **Base64-encoded**, decoded at runtime, then compared with `strcmp`. Base64 is encoding, not encryption — fully reversible.
- Decoding `Y3JhY2ttZTIwMjQ=` gives **`crackme2024`**.
- Found a second acceptance path: `strstr(input, "hack")` — any input that contains the substring `hack` also passes. A deliberate backdoor.

## Three ways to beat it

1. Type the real password: `crackme2024`.
2. Type anything containing `hack` (e.g. `hack`, `hacker`, `xhackx`).
3. Patch the branch / force `check_password` to return 1.

## Concepts learned

- Real programs split logic across functions — the check was in `check_password`, not in `main`. Follow the call, or follow where the success flag gets set to 1.
- Tell scaffolding (loop, counter, fake loading, random reward) from the actual check. Don't get fooled by impressive-looking decoys like `rand() % 10000`.
- Encoded != encrypted. Base64 is trivially reversible.
- Always look for multiple accept paths / backdoors (`strstr` here).
