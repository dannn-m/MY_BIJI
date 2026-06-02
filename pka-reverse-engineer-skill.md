---
name: pka-reverse-engineer
description: Decrypt, analyze, modify, and re-encrypt Cisco Packet Tracer .pka activity files (PT 8.x format). Use when the user needs to: (1) decrypt a .pka file to XML, (2) read/analyze lab configuration and grading criteria, (3) modify ACL or device configurations to fix grading issues, (4) re-encrypt XML back to .pka format, or (5) extract answer networks from encrypted lab files.
metadata:
  type: user
  tags: [cisco, packet-tracer, pka, reverse-engineering, crypto, twofish, acl]
---

# PKA Reverse Engineer Skill

Decrypt, analyze, modify, and re-encrypt Cisco Packet Tracer 8.x .pka activity files.

## Prerequisites (one-time setup)

```bash
# 1. Install build tools
winget install Kitware.CMake
# Ensure Visual Studio 2022 BuildTools is installed

# 2. Setup vcpkg
cd C:\ && git clone --depth 1 https://github.com/microsoft/vcpkg.git
cd vcpkg && .\bootstrap-vcpkg.bat
.\vcpkg install cryptopp zlib --triplet x64-windows

# 3. Clone pka2xml reference implementation
cd C:\ && git clone --depth 1 https://github.com/mircodz/pka2xml.git
```

## Workflow

### Step 1: Decrypt .pka → .xml

Use the compiled C++ decryptor. The source code (`decrypt_pka.cpp`) should include:

```cpp
#include <cryptopp/eax.h>
#include <cryptopp/twofish.h>
#include <cryptopp/filters.h>
#include <zlib.h>
#include <iostream>
#include <fstream>
#include <string>
#include <array>

std::string uncompress_pka(const unsigned char *data, int nbytes) {
    unsigned long len = ((unsigned long)data[0] << 24) | (data[1] << 16) | (data[2] << 8) | data[3];
    std::string out(len, '\0');
    if (::uncompress((unsigned char*)out.data(), &len, data + 4, nbytes - 4) != Z_OK) return "";
    out.resize(len);
    return out;
}

std::string decrypt_pka_file(const std::string &input) {
    std::array<unsigned char, 16> key = {
        137, 137, 137, 137, 137, 137, 137, 137,
        137, 137, 137, 137, 137, 137, 137, 137
    };
    std::array<unsigned char, 16> iv = {
        16, 16, 16, 16, 16, 16, 16, 16,
        16, 16, 16, 16, 16, 16, 16, 16
    };

    CryptoPP::EAX<CryptoPP::Twofish>::Decryption d;
    d.SetKeyWithIV(key.data(), key.size(), iv.data(), iv.size());

    int length = input.size();
    std::string processed(length, '\0'), output;

    // Stage 1: Reverse + XOR (CRITICAL: formula is L - i*L, NOT L - i)
    for (int i = 0; i < length; i++) {
        int xor_val = length - i * length;
        processed[i] = input[length + ~i] ^ (xor_val & 0xFF);
    }

    // Stage 2: Twofish EAX decrypt
    CryptoPP::StringSource ss(processed, true,
        new CryptoPP::AuthenticatedDecryptionFilter(d,
            new CryptoPP::StringSink(output),
            CryptoPP::AuthenticatedDecryptionFilter::THROW_EXCEPTION));

    // Stage 3: XOR deobfuscation
    for (int i = 0; i < (int)output.size(); i++)
        output[i] ^= (output.size() - i) & 0xFF;

    // Stage 4: zlib decompress
    return uncompress_pka((const unsigned char*)output.data(), output.size());
}
```

**Compilation** (save as `build_decrypt.bat`):
```batch
@echo off
call "C:\Program Files (x86)\Microsoft Visual Studio\18\BuildTools\VC\Auxiliary\Build\vcvars64.bat"
cl /EHsc /O2 /MD /Fe:decrypt_pka.exe decrypt_pka.cpp ^
  /I "C:\vcpkg\installed\x64-windows\include" ^
  /link "C:\vcpkg\installed\x64-windows\lib\cryptopp.lib" ^
        "C:\vcpkg\installed\x64-windows\lib\z.lib" ws2_32.lib
```

**Usage**: `decrypt_pka.exe input.pka output.xml`

### Step 2: Analyze and Modify XML

The decrypted XML contains:
- Answer network (expected configuration for grading)
- User network (current user progress)
- Grading items with `nodeValue` attributes (exact string matching used by PT)

**Key XML structure**:
- Root: `<PACKETTRACER5_ACTIVITY>`
- Multiple `<PACKETTRACER5>` sections (answer + user + initial networks)
- Device configs in `<RUNNING_APPS>` as `<LINE>` elements
- Grading items: `<NAME nodeValue="..." overrideDBGrading="false" checkType="0">`

**Common fixes needed**:
- ACL wildcard `any` → specific network (e.g., `192.168.1.0 0.0.0.63`)
- Missing `access-group` on interfaces
- Incorrect ACL permit/deny order

**Python analysis script**:
```python
import re

with open('output.xml', 'r', encoding='utf-8') as f:
    content = f.read()

# Find all ACL entries
for m in re.finditer(r'(access-list \d+|ip access-list)[^<]*', content):
    print(m.group()[:120])

# Compare answer vs user ACLs
answer_pattern = 'access-list 111 deny ip 192.168.1.0 0.0.0.63 host 192.168.2.45'
user_pattern = 'access-list 111 deny ip any host 192.168.2.45'
print(f"Answer: {content.count(answer_pattern)} occurrences")
print(f"User: {content.count(user_pattern)} occurrences")

# Fix: replace incorrect pattern
content = content.replace(user_pattern, answer_pattern)

with open('fixed.xml', 'w', encoding='utf-8') as f:
    f.write(content)
```

### Step 3: Re-encrypt .xml → .pka

**Encryption steps** (reverse of decryption):
1. zlib compress with 4-byte BE size header
2. XOR: `byte[i] ^= (compressed_size - i) & 0xFF`
3. Twofish EAX encrypt (same key/IV as decryption)
4. Reverse + XOR: `output[L + ~i] = encrypted[i] ^ ((L - i*L) & 0xFF)`

**Source** (`encrypt_pka.cpp`):
```cpp
std::string encrypt_pka_file(const std::string &input) {
    std::array<unsigned char, 16> key = { /* 137*16 */ };
    std::array<unsigned char, 16> iv  = { /* 16*16  */ };

    // Stage 1: compress
    unsigned long clen = input.size() + input.size()/100 + 13;
    std::string comp(clen + 4, '\0');
    if (::compress2((unsigned char*)comp.data() + 4, &clen,
                    (const unsigned char*)input.data(), input.size(), -1) != Z_OK)
        throw std::runtime_error("compress failed");
    comp.resize(clen + 4);
    comp[0] = (input.size() >> 24) & 0xFF;
    comp[1] = (input.size() >> 16) & 0xFF;
    comp[2] = (input.size() >> 8)  & 0xFF;
    comp[3] = input.size() & 0xFF;

    // Stage 2: XOR
    for (int i = 0; i < (int)comp.size(); i++)
        comp[i] ^= (comp.size() - i) & 0xFF;

    // Stage 3: Twofish EAX encrypt
    CryptoPP::EAX<CryptoPP::Twofish>::Encryption e;
    e.SetKeyWithIV(key.data(), key.size(), iv.data(), iv.size());
    std::string encrypted;
    CryptoPP::StringSource ss(comp, true,
        new CryptoPP::AuthenticatedEncryptionFilter(e,
            new CryptoPP::StringSink(encrypted)));

    // Stage 4: Reverse + XOR
    int L = encrypted.size();
    std::string output(L, '\0');
    for (int i = 0; i < L; i++) {
        int xor_val = L - i * L;
        output[L + ~i] = encrypted[i] ^ (xor_val & 0xFF);
    }
    return output;
}
```

### Step 4: Verify Round-trip

Always verify by decrypting the fixed .pka and checking:
1. Decryption succeeds without errors
2. All ACL patterns are correct
3. XML structure matches original

```bash
decrypt_pka.exe fixed.pka verify.xml
python -c "
with open('verify.xml') as f:
    c = f.read()
print('Wrong ACL:', c.count('deny ip any host 192.168.2.45'))
print('Correct ACL:', c.count('deny ip 192.168.1.0 0.0.0.63 host 192.168.2.45'))
"
```

## Algorithm Details

### Encryption: Twofish-128 in EAX Mode

| Parameter | Value |
|-----------|-------|
| Cipher | Twofish (128-bit block) |
| Mode | EAX (authenticated encryption with associated data) |
| Key | 16 bytes, all 0x89 (decimal 137) |
| Nonce/IV | 16 bytes, all 0x10 (decimal 16) |
| Authentication | CMAC-based, 16-byte tag appended to ciphertext |
| Crypto library | Crypto++ (compiled into PacketTracer.exe) |

### 4-Stage Processing Pipeline

```
DECRYPTION:
  encrypted.pka
  → [Stage 1] Reverse bytes + XOR(L - i*L)
  → [Stage 2] Twofish EAX decrypt (strips 16-byte auth tag)
  → [Stage 3] XOR(output_size - i)
  → [Stage 4] zlib decompress (4-byte BE size header)
  → decrypted.xml

ENCRYPTION:
  modified.xml
  → [Stage 1] zlib compress + 4-byte BE size header
  → [Stage 2] XOR(compressed_size - i)
  → [Stage 3] Twofish EAX encrypt (adds 16-byte auth tag)
  → [Stage 4] Reverse bytes + XOR(L - i*L)
  → encrypted.pka
```

### ⚠️ Critical: Stage 1 XOR Formula

The decompiled code shows: `processed[i] = input[L-1-i] ^ (L - i * L)`

| i | XOR value | i | XOR value |
|---|-----------|---|-----------|
| 0 | L | 3 | -2L |
| 1 | 0 | 4 | -3L |
| 2 | -L | 5 | -4L |

This is NOT `(L - i)` — the multiplication by L is intentional and essential.

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| `HashVerificationFilter: MAC not valid` | Wrong Stage 1 XOR formula | Ensure `(L - i*L)` not `(L - i)` |
| zlib `incorrect header check` | Wrong cipher or key | Verify key={137}*16, iv={16}*16 |
| File size mismatch after re-encrypt | XML content changed length | Expected - different ACL string lengths |
| PT crashes on opening fixed file | XML structure corrupted | Verify XML well-formed, check for encoding issues |
| Grading still shows wrong answer | `nodeValue` in grading items not updated | Replace ALL 48+ occurrences in XML |

## Fallback: Memory Analysis (when compilation not available)

If the C++ toolchain is not available, use Python to read PT's process memory:

```python
import ctypes
from ctypes import wintypes
import subprocess

# Find PT PID
result = subprocess.run(['tasklist', '/fi', 'IMAGENAME eq PacketTracer.exe', 
                        '/fo', 'csv'], capture_output=True, text=True)
pid = int(result.stdout.split('\n')[1].split(',')[1].strip('"'))

# Read process memory
kernel32 = ctypes.windll.kernel32
h_process = kernel32.OpenProcess(0x0010 | 0x0400, False, pid)

# Search for ACL patterns
patterns = [b'access-list', b'ip access-group', b'permit', b'deny', 
            b'192.168', b'vty_block', b'branch_to_hq']
# ... scan memory regions for matches (see full guide for implementation)
```

This approach can extract ACL configurations and grading criteria without file decryption, but cannot modify the .pka file.
