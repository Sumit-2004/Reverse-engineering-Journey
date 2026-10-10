# Workflow and escalation ladder

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

## My standard workflow (works on easy crackmes)

1. Run it once — note the exact success/failure message strings.
2. In IDA, find those strings (Shift+F12), or open `main` and read top-down.
3. Find the decision point: a `cmp`/`test` followed by a conditional jump, or a `call str*cmp` / custom check function.
4. Read the branch: figure out which way = success.
5. Recover the input (keygen) **or** patch the jump (quick win).
6. Verify by running it. Then read the source/write-up to grade myself.
