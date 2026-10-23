# Challenge 5 — BadEngineer's BeginnerFriendly

**Author:** BadEngineer  
**Platform / architecture:** Windows x86  
**Focus:** Debugger detection and a character-sum password check  
**Tools:** IDA Free (static analysis)

## Description

The challenge is a small password checker with a basic anti-debugging check. When it detects a debugger, it prints `Stop using debugger`. Otherwise, it prompts for a password and reads it with `scanf("%63s", Str)`.

The input must be no longer than 16 characters. For an acceptable length, the program adds the input characters' byte values and prints `Congrats!` when the total is `1240`; otherwise it prints `Wrong Password!`. Longer input produces `Try something smaller!`.

## How I solved it

I followed the password input in IDA's pseudocode. The loop adds each character value into a running total, and the success branch checks whether that total equals `1240`. The debugger check is a separate gate, so I ran the program without a debugger attached rather than patching it.

A working 16-character input is eight `N` characters followed by eight `M` characters:

```text
NNNNNNNNMMMMMMMM
```

`N` is ASCII 78 and `M` is ASCII 77, so the sum is `8 × 78 + 8 × 77 = 1240`. This satisfies both the length limit and the sum check.

## Concepts learned

- Identify anti-debugging checks separately from the password validation logic.
- A password validator may check a derived property, such as a character-value sum, rather than compare against one literal string.
- Confirm input constraints (here, at most 16 characters) as well as the success condition.
