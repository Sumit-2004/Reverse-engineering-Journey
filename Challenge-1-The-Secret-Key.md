# Challenge 1 — The Secret Key

**Topic:** Integer comparisons  
**Tools:** IDA Free (static) + Local Windows debugger

## What the program does

Asks for a number, then a “secret key” (also a number). If both match hardcoded values, it prints a congratulations message.

## Key disassembly

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

## How I solved it (static)

- The expected values are constants sitting right in the `cmp` instructions.
- `cmp eax, 21h` -> `0x21` = **33**.
- `cmp eax, 66h` -> `0x66` = **102**.
- `jnz` means “jump if not equal,” so typing the matching value falls through to success.
- IDA shows `; '!'` and `; 'f'` next to the hex — that's just the ASCII of the byte. Since `scanf` uses `%i`, the program wants the **number**, not the character.

## Answer

```text
Enter Your Number : 33
Enter The Secret Key : 102
-> Congratulations, you have completed the challenge!
```

## Gotcha I hit

The console closed instantly on success. Fix: run it from an already-open `cmd` window (`cd` to the folder, then run the exe) so the output stays visible.

## Concepts learned

- `cmp` + conditional jump (`jnz`/`jz`) = a decision point. Finding these is reversing.
- Hex-to-decimal conversion (`0x21` = 33, `0x66` = 102).
- The `; 'x'` ASCII hints are a beginner trap — check the `scanf` format first.
- One shared exit block can serve both the pass and fail paths.
