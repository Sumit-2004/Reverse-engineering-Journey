# Challenge 4 — Monsignors Crackme

**Status:** Solved  
**Source:** [Monsignors Crackme on crackmes.one](https://crackmes.one/crackme/6a2c80595f26f108ba18b8bd)  
**Focus:** String input and a hard-coded password check  
**Tools:** IDA Free (static analysis)

## Description

The program prompts for a password, reads a string with `scanf` using the `%s` format, and checks it against the expected value. A match prints **Access granted**; any other input prints **Access failed**.

## How I solved it

I followed the input from the `scanf` call to the password-checking branch and found the expected string, `secret`. The comparison result determines whether execution reaches the success or failure message.

## Solution

```text
Password: secret
-> Access granted
```

## Concepts learned

- `%s` with `scanf` reads a whitespace-delimited string, not a single character.
- Follow user input from the input call to the comparison and then to each conditional branch.
- Success and failure strings help locate the decision point in a disassembler.
