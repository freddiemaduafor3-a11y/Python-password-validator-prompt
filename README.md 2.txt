# SecurePass Architect Prompt


> A precision-engineered master prompt that generates a production-ready, zero-external-dependency Python 3.11+ password validation platform — built to principal/staff engineer standards.


---


## Table of Contents


- [Overview](#overview)
- [What This Prompt Generates](#what-this-prompt-generates)
- [Generated Module Architecture](#generated-module-architecture)
- [Validation Pipeline](#validation-pipeline)
- [Pattern Detection](#pattern-detection)
- [Strength Scoring](#strength-scoring)
- [Test Results](#test-results)
- [Sample Output](#sample-output)
- [How to Use This Prompt](#how-to-use-this-prompt)
- [Fixes Applied in v2.0](#fixes-applied-in-v20)
- [Author](#author)


---


## Overview


**SecurePass Architect Prompt** is a master AI prompt designed to generate a fully typed, enterprise-grade Python password validation module in a single pass. The prompt enforces principal-engineer-level architecture decisions — immutable data models, a typed validation pipeline, entropy-aware scoring, and structured issue reporting — without requiring any external libraries.


This is not a basic prompt. It specifies every class, every field, every pattern table, every scoring rule, and every expected test outcome. The AI has no room to cut corners.


**Tested on:** Claude Sonnet · ChatGPT GPT-4o  
**Language:** Python 3.11+  
**Dependencies:** Zero — stdlib only  
**Prompt version:** v2.0  


---


## What This Prompt Generates


Running this prompt produces a single Python file (`hypersecure_password_platform.py`) containing:


| Component | Description |
|---|---|
| `Severity` | Enum — LOW, MEDIUM, HIGH, CRITICAL |
| `StrengthLevel` | Enum — VERY_WEAK through VERY_STRONG |
| `ValidationIssue` | Frozen dataclass — structured issue with code, message, severity |
| `PasswordPolicy` | Frozen dataclass — 16 configurable policy fields with validation |
| `PasswordProfile` | Frozen dataclass — cached per-password character analysis |
| `PasswordAnalysis` | Frozen dataclass — final report returned to caller |
| `HyperSecurePasswordValidator` | Stateless, thread-safe validator class |
| `validate_password()` | Module-level convenience function → `bool` |
| `analyze_password()` | Module-level convenience function → `PasswordAnalysis` |


---


## Generated Module Architecture


```
hypersecure_password_platform.py
│
├── Enums
│   ├── Severity          (LOW | MEDIUM | HIGH | CRITICAL)
│   └── StrengthLevel     (VERY_WEAK | WEAK | FAIR | GOOD | STRONG | VERY_STRONG)
│
├── Models (all frozen=True, slots=True)
│   ├── ValidationIssue   — code, message, severity
│   ├── PasswordPolicy    — 16 policy fields + __post_init__ guards
│   ├── PasswordProfile   — 11 character analysis fields
│   └── PasswordAnalysis  — valid, score, strength, entropy, issues, profile
│
├── HyperSecurePasswordValidator
│   ├── Constants         — COMMON_PASSWORDS, KEYBOARD_PATTERNS,
│   │                       SEQUENTIAL_PATTERNS, YEAR_PATTERN
│   ├── Public API        — validate(password) → PasswordAnalysis
│   ├── Profile Engine    — _build_profile()
│   ├── Validation Rules  — 9 validator methods
│   └── Strength Engine   — _calculate_entropy(), _calculate_strength_score(),
│                           _map_strength()
│
└── Convenience API
    ├── validate_password()  → bool
    └── analyze_password()   → PasswordAnalysis
```


---


## Validation Pipeline


The prompt enforces this exact 9-step validation order:


| Step | Method | Issue Code | Severity |
|---|---|---|---|
| 1 | Length check | `PASSWORD_TOO_SHORT` / `PASSWORD_TOO_LONG` | HIGH / MEDIUM |
| 2 | Character classes | `MISSING_UPPERCASE` / `MISSING_LOWERCASE` / `MISSING_NUMBER` / `MISSING_SPECIAL` | MEDIUM |
| 3 | Unicode policy | `UNICODE_NOT_ALLOWED` | MEDIUM |
| 4 | Whitespace policy | `WHITESPACE_NOT_ALLOWED` | MEDIUM |
| 5 | Common passwords | `COMMON_PASSWORD` | CRITICAL |
| 6 | Keyboard patterns | `KEYBOARD_PATTERN` | HIGH |
| 7 | Sequential patterns | `SEQUENTIAL_PATTERN` | HIGH |
| 8 | Repeated characters | `REPEATED_PATTERN` | MEDIUM |
| 9 | Year patterns *(v2.0)* | `YEAR_PATTERN` | MEDIUM |


Before step 1, the password is normalised with `unicodedata.normalize("NFKC", password)`. `None` and non-string inputs are rejected immediately with a CRITICAL issue before any pipeline step runs.


---


## Pattern Detection


### Common Passwords — 50+ entries
Includes plain passwords, leet-speak variants, common names, sports terms, and standalone years.


```
password, password123, p@ssw0rd, pa$$word, admin, admin123,
qwerty, qwerty123, 123456, 12345678, welcome, welcome123,
letmein, abc123, changeme, root, guest, dragon, master,
monkey, shadow, sunshine, batman, superman, trustno1,
iloveyou, michael, jessica, 2024, 2025, 2026 …and more
```


### Keyboard Patterns — 30+ sequences
Top row, home row, bottom row, diagonals, numpad walks, gaming keys, and all reversed variants.


```
qwerty, asdfgh, zxcvbn, 1qaz, 2wsx, wasd,
ytrewq, fdsa, nbvcxz, 789456123 …and more
```


### Sequential Patterns — 50+ sequences
Full a–z forward and reversed, full 0–9 forward and reversed.


```
abc, bcd … xyz    (forward alpha)
cba, dcb … zyx    (reversed alpha)
123, 234 … 890    (forward digits)
321, 432 … 098    (reversed digits)
```


### Year Pattern *(v2.0 fix)*
Regex `(19|20)\d{2}` — catches any year from 1900–2099 embedded anywhere in the password.


---


## Strength Scoring


The generated module scores passwords 0–100 using this engine:


**Bonuses**


| Condition | Points |
|---|---|
| Length ≥ 24 | +35 |
| Length ≥ 18 | +30 |
| Length ≥ 14 | +25 |
| Length ≥ 12 | +20 |
| Each character class present (×4 max) | +10 each |
| Entropy ≥ 120 bits | +25 |
| Entropy ≥ 90 bits | +20 |
| Entropy ≥ 70 bits | +15 |
| Entropy ≥ 50 bits | +10 |
| Unique character ratio ≥ 80% | +10 |


**Penalties**


| Issue Code | Deduction |
|---|---|
| COMMON_PASSWORD | −50 |
| PASSWORD_TOO_SHORT | −25 |
| KEYBOARD_PATTERN | −20 |
| SEQUENTIAL_PATTERN | −15 |
| REPEATED_PATTERN | −10 |
| YEAR_PATTERN | −10 |
| Any other issue | −5 |


**Strength mapping**


| Score | Level |
|---|---|
| 90 – 100 | VERY_STRONG |
| 75 – 89 | STRONG |
| 60 – 74 | GOOD |
| 40 – 59 | FAIR |
| 20 – 39 | WEAK |
| 0 – 19 | VERY_WEAK |


---


## Test Results


The prompt specifies 9 mandatory test outcomes the generated code must pass:


| Password | Expected | Reason |
|---|---|---|
| `"weak"` | ❌ Invalid | Too short, missing 3 character classes |
| `"password123"` | ❌ Invalid | Common password + sequential + too short |
| `"Qwerty123!"` | ❌ Invalid | Keyboard pattern + sequential + too short |
| `"StrongPass#2026"` | ❌ Invalid | Year pattern detected *(v1 bug — now fixed)* |
| `"UltraSecure#Enterprise2026!"` | ❌ Invalid | Year pattern detected |
| `"MyUltra$Secure#BankingPass!"` | ✅ Valid | Long, diverse, no patterns |
| `None` | ❌ Invalid | NULL_PASSWORD — CRITICAL |
| `12345` (int) | ❌ Invalid | INVALID_TYPE — CRITICAL |
| `"Ünïcödé#Páss!" + extras` | ✅ Valid | Unicode allowed by default |


**v1 pass rate:** 16/17 (94%)  
**v2 pass rate:** 17/17 (100%)  


---


## Sample Output


```
Password : StrongPass#2026
Valid    : False  |  Score: 85/100  |  STRONG
Entropy  : 97.14 bits
  [MEDIUM] YEAR_PATTERN: Password contains a predictable year (e.g. 2026)


Password : MyUltra$Secure#BankingPassword!
Valid    : True  |  Score: 100/100  |  VERY_STRONG
Entropy  : 195.42 bits
  ✓ All checks passed


Password : weak
Valid    : False  |  Score: 0/100  |  VERY_WEAK
Entropy  : 18.8 bits
  [HIGH]   PASSWORD_TOO_SHORT: Length 4 < minimum 12
  [MEDIUM] MISSING_UPPERCASE: Missing uppercase letter
  [MEDIUM] MISSING_NUMBER: Missing numeric digit
  [MEDIUM] MISSING_SPECIAL: Missing special character
  [HIGH]   STRENGTH_TOO_LOW: Password strength below required threshold
```


---


## How to Use This Prompt


**Step 1 — Copy the prompt** from `prompt.md` in this repository.


**Step 2 — Paste it** into Claude (claude.ai) or ChatGPT (GPT-4o recommended).


**Step 3 — Run the output** directly. No installation needed beyond Python 3.11+.


```bash
python hypersecure_password_platform.py
```


**Step 4 — Import into your project:**


```python
from hypersecure_password_platform import analyze_password, validate_password


# Quick check
is_valid = validate_password("MySecure#Pass2077!")


# Full analysis
report = analyze_password("MySecure#Pass2077!")
print(report.score)       # 0–100
print(report.strength)    # StrengthLevel enum
print(report.entropy_bits)
for issue in report.issues:
    print(f"[{issue.severity}] {issue.code}: {issue.message}")
```


**Custom policy:**


```python
from hypersecure_password_platform import (
    HyperSecurePasswordValidator, PasswordPolicy
)


policy = PasswordPolicy(
    min_length=16,
    allow_unicode=False,
    check_year_patterns=True,
    minimum_strength_score=75,
)
validator = HyperSecurePasswordValidator(policy=policy)
report = validator.validate("MyPassword#99")
```


---


## Fixes Applied in v2.0


Six bugs were identified in the original v1 implementation and corrected in this prompt:


| Fix | Issue | Impact |
|---|---|---|
| #1 | Sequential pattern list was incomplete — no reversed sequences (`cba`, `321`) | MEDIUM |
| #2 | Keyboard patterns missing reversed walks, numpad, and gaming keys | MEDIUM |
| #4 | Common password list had only 14 entries | HIGH |
| #5 | Unicode characters not counted in entropy calculation | LOW |
| #7 | No score penalty for year patterns | MEDIUM |
| #8 | Year patterns not detected at all — `StrongPass#2026` passed as valid | HIGH |


The critical failure was Fix #8 — `StrongPass#2026` received a score of 95/100 and was marked valid in v1. It is now correctly rejected in v2.0.


---


## Author


**Created by:** [freddie]  
**Country:** Nigeria  
**Skill:** AI Prompt Engineering — Python Code Generation  
**Contact:** [Your Fiverr / freddiemaduafor3@gmail.com]  
**GitHub:** [Your GitHub URL]  


---


*Tested with Python 3.11 · Zero external dependencies*