# CAFFEE Editor Remote Code Execution (RCE) via Filename Placeholders

This repository contains a Proof of Concept (PoC) for a **Remote Code Execution (RCE)** vulnerability in the CAFFEE text editor (tested on version 2.10.2).

## Vulnerability Details

*   **Vulnerability Type:** OS Command Injection (CWE-78) leading to Remote Code Execution (RCE).
*   **Affected Software:** CAFFEE Editor (Tested on v2.10.2 on Linux).
*   **Impact:** An attacker can craft a filename containing shell metacharacters (e.g., backticks `` ` `` or `$()`). When a user opens the file and triggers the "Build" command (Ctrl+B), arbitrary commands are executed with the user's privileges.

## Root Cause Analysis

The vulnerability stems from a data flow flaw in `caffee.py`:

1.  **Unsanitized Substitution:**
    The `DEFAULT_BUILD_COMMANDS` dictionary contains templates like `"gcc \"{filename}\" -o \"{base}\" && \"./{base}\""`.
    In the `run_build_command()` function, the raw filename is substituted into the template using Python's `.format()` without any shell escaping.

2.  **Terminal Injection:**
    The generated command string is passed directly to the embedded terminal using `self.terminal.write_input(cmd + "\n")`. Because the command is executed by a shell, command substitution (backticks or `$()`) inside the filename is evaluated, even when enclosed in double quotes.

## Reproduction Steps

1.  **Create a Malicious File:**
    Open a terminal on Linux and create a file with a name designed to execute the GNOME Calculator:
    `touch 'test`gnome-calculator`.c'`
    *(Note the use of single quotes in the terminal to prevent immediate execution).*

2.  **Open in CAFFEE:**
    Launch CAFFEE and open the file you just created.

3.  **Trigger Execution:**
    Press **Ctrl+B** (Build).

4.  **Observe the Result:**
    The terminal executes the substituted command. The backtick expression is evaluated by the shell, launching `gnome-calculator`.

## Expected Fix

CAFFEE should sanitize or escape filenames before substituting them into shell commands. Using `shlex.quote(filename)` or avoiding the shell entirely (by splitting the command into an argument list and using `subprocess` directly) would mitigate this vulnerability.
