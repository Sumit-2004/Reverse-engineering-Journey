# Reverse engineering journey

A log of the crackmes I've reversed while learning, how each one works, and how I solved it. Tools used: **IDA Free** (static) + **Local Windows debugger**.

## Challenge 1 — “Enter Your Number” (integer compares)

### What the program does

Asks for a number, then a “secret key” (also a number). If both match hardcoded values, it prints a congratulations message.

### Key disassembly

```asm
lea   rax, _Format          ; "Enter Your Number : "
call  printf
lea   rax, [rbp+var_4]
lea   rcx, aI               ; "%i" -> reads an integer
call  scanf
mov   eax, [rbp+var_4]      ; eax = my number
cmp   eax, 21h              ; compare with 0x21
jnz   short loc_140001520   ; not equal -> fail path
; second gate
cmp   eax, 66h              ; compare with 0x66
jnz   short loc_140001520
lea   rax, aCongratulation  ; "Congratulations..."
call  printf
```

### How I solved it (static)

- The expected values are constants sitting right in the `cmp` instructions.
- `cmp eax, 21h` -> `0x21` = **33**.
- `cmp eax, 66h` -> `0x66` = **102**.
- `jnz` means “jump if not equal,” so typing the matching value falls through to success.
- IDA shows `; '!'` and `; 'f'` next to the hex — that's just the ASCII of the byte. Since `scanf` uses `%i`, the program wants the **number**, not the character.

### Answer

```text
Enter Your Number : 33
Enter The Secret Key : 102
-> Congratulations, you have completed the challenge!
```

### Gotcha I hit

The console closed instantly on success. Fix: run it from an already-open `cmd` window (`cd` to the folder, then run the exe) so the output stays visible.

### Concepts learned

- `cmp` + conditional jump (`jnz`/`jz`) = a decision point. Finding these is reversing.
- Hex-to-decimal conversion (`0x21` = 33, `0x66` = 102).
- The `; 'x'` ASCII hints are a beginner trap — check the `scanf` format first.
- One shared exit block can serve both the pass and fail paths.

---

## Challenge 2 — “Secret Access Terminal” (string compares)

### What the program does

Asks for a username and a password. Both are checked against hardcoded strings with `strcmp`. Only `admin` + the right password prints success.

### Key disassembly

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

### How I solved it (static)

- A string can't fit in a single `cmp`, so the compiler calls `strcmp`.
- IDA auto-comments the string pointers, so the expected values are shown in plain text next to the `lea` before each `strcmp`.
- `strcmp` returns 0 when strings are equal, so the code uses `test eax, eax` then `jz` (jump if zero = equal = correct).

### Answer

```text
Username: admin
Password: Sup3rS3cr3t
-> Access granted! Welcome, admin.
```

### Concepts learned

- String checks use `str*cmp`, not `cmp`. Look for the `lea <string>` feeding it.
- `strcmp` returns 0 on match -> `test eax,eax` + `jz` means “jump if matched.”
- Plaintext secrets in a binary are a real weakness — a disassembler reads them instantly.
- The Strings window (Shift+F12) often hands you the answer directly.

---

## Challenge 3 — “Password Cracker” (loop + attempts + Base64 + backdoor)

The first genuinely non-trivial one. Much more than a single check.

### Structure of `main()`

- `var_4` = attempt counter, `var_8` = success flag; 3 attempts allowed.
- `system("cls")` prints a banner and “You have 3 attempts...”.
- Loop condition: keep going while `attempts <= 2` **and** `flag == 0`.
- Reads input with `fgets(buffer, 50, stdin)`, then strips the trailing `\n` (`strlen` -> look at last char -> if `0x0A`, overwrite with `0`).
- Calls `loading_animation` (a fake effect — decoy).
- Calls **`check_password(buffer)`** and branches on the return value.
- On wrong password: prints “Wrong password. Try again.” and increments `attempts`.
- After 3 failures: prints “Too many failed attempts. System locked” and exits.
- On success: prints a **random 4-digit code** (`rand() % 10000`, done via the `imul 0x68DB8BAD` + shifts trick) and sets the success flag. This is a reward, not the check — a decoy I had to avoid getting fooled by.

### The real check — inside `check_password()`

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

### How I solved it (static)

- The password is stored **Base64-encoded**, decoded at runtime, then compared with `strcmp`. Base64 is encoding, not encryption — fully reversible.
- Decoding `Y3JhY2ttZTIwMjQ=` gives **`crackme2024`**.
- Found a second acceptance path: `strstr(input, "hack")` — any input that contains the substring `hack` also passes. A deliberate backdoor.

### Three ways to beat it

1. Type the real password: `crackme2024`.
2. Type anything containing `hack` (e.g. `hack`, `hacker`, `xhackx`).
3. Patch the branch / force `check_password` to return 1.

### Concepts learned

- Real programs split logic across functions — the check was in `check_password`, not in `main`. Follow the call, or follow where the success flag gets set to 1.
- Tell scaffolding (loop, counter, fake loading, random reward) from the actual check. Don't get fooled by impressive-looking decoys like `rand() % 10000`.
- Encoded != encrypted. Base64 is trivially reversible.
- Always look for multiple accept paths / backdoors (`strstr` here).

---

## The escalation ladder (where I am, what's next)

| Rung | Secret lives as... | Beat it with | Status |
| --- | --- | --- | --- |
| 1 | Constant in a `cmp` | Read it (static) | Done |
| 2 | Plaintext string | Read it (static) | Done |
| 3 | Base64-encoded + backdoor | Decode it (static) | Done |
| 4 | XOR'd with a key | Reverse math or debugger | Next |
| 5 | Computed from input | Debugger breakpoint-and-peek | Todo |
| 6 | Hashed (one-way) | Can't read — patch/find collision | Todo |

**Next goal:** rung 4 — a real XOR-with-key check where the password is never stored readably. That's where static reading stops being enough and I set my first breakpoint to read the decrypted value live in memory.

---

## My standard workflow (works on easy crackmes)

1. Run it once — note the exact success/failure message strings.
2. In IDA, find those strings (Shift+F12), or open `main` and read top-down.
3. Find the decision point: a `cmp`/`test` followed by a conditional jump, or a `call str*cmp` / custom check function.
4. Read the branch: figure out which way = success.
5. Recover the input (keygen) **or** patch the jump (quick win).
6. Verify by running it. Then read the source/write-up to grade myself.
