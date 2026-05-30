# SecurePass Architect Prompt — v2.0

> Copy everything below the line and paste it directly into Claude or ChatGPT (GPT-4o).

---

You are a principal/staff software engineer specialising in Python security infrastructure.

Write a production-ready, zero-external-dependency password validation module in Python 3.11+ called `hypersecure_password_platform.py`.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ARCHITECTURE REQUIREMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Use the following immutable, typed, frozen-dataclass architecture — no classes that violate these patterns:

  - Severity(str, Enum)          — LOW | MEDIUM | HIGH | CRITICAL
  - StrengthLevel(str, Enum)     — VERY_WEAK | WEAK | FAIR | GOOD | STRONG | VERY_STRONG
  - ValidationIssue              — frozen dataclass: code, message, severity
  - PasswordPolicy               — frozen dataclass with __post_init__ validation (see Policy section)
  - PasswordProfile              — frozen dataclass: cached per-password character analysis
  - PasswordAnalysis             — frozen dataclass: final report returned from validate()
  - HyperSecurePasswordValidator — stateless, thread-safe validator class

All dataclasses must use `frozen=True, slots=True`. All public methods must be fully typed.
Use `from __future__ import annotations` and `typing.Final` for all class-level constants.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PASSWORD POLICY (PasswordPolicy dataclass fields)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  min_length: int = 12
  max_length: int = 128
  require_uppercase: bool = True
  require_lowercase: bool = True
  require_numbers: bool = True
  require_special: bool = True
  allow_unicode: bool = True
  allow_spaces: bool = False
  check_common_passwords: bool = True
  check_keyboard_patterns: bool = True
  check_sequential_patterns: bool = True
  check_repeated_patterns: bool = True
  check_year_patterns: bool = True          ← required
  max_consecutive_repeats: int = 2
  minimum_strength_score: int = 60
  special_characters: str = "!@#$%^&*()-_=+[]{};:,.<>?/|"

__post_init__ must raise ValueError for: min_length < 1, max_length < min_length,
minimum_strength_score outside [0,100], max_consecutive_repeats < 1.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
VALIDATION PIPELINE  (called in this order inside validate())
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Each method returns List[ValidationIssue]. Implement all of:

  1. _validate_length              — TOO_SHORT (HIGH), TOO_LONG (MEDIUM)
  2. _validate_character_classes   — MISSING_UPPERCASE/LOWERCASE/NUMBER/SPECIAL (MEDIUM each)
  3. _validate_unicode             — UNICODE_NOT_ALLOWED (MEDIUM)
  4. _validate_whitespace          — WHITESPACE_NOT_ALLOWED (MEDIUM)
  5. _validate_common_passwords    — COMMON_PASSWORD (CRITICAL)
  6. _validate_keyboard_patterns   — KEYBOARD_PATTERN (HIGH)
  7. _validate_sequential_patterns — SEQUENTIAL_PATTERN (HIGH)
  8. _validate_repeated_patterns   — REPEATED_PATTERN (MEDIUM)
  9. _validate_year_patterns       — YEAR_PATTERN (MEDIUM)     ← required new method

Before any of the above: normalise password with unicodedata.normalize("NFKC", password).
Guard None and non-str inputs — return a failed PasswordAnalysis immediately with a CRITICAL issue.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PATTERN TABLES  (use exactly these as Final class constants)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

COMMON_PASSWORDS — FrozenSet[str], minimum 50 entries including:
  Variations: password, password1, password123, password1234
  Admin:      admin, admin123, admin1234, administrator
  Keyboard:   qwerty, qwerty123, qwertyuiop
  Numeric:    123456, 1234567, 12345678, 123456789, 1234567890
  Welcome:    welcome, welcome1, welcome123
  Misc:       letmein, letmein1, abc123, abcdef, passw0rd, p@ssword,
              p@ssw0rd, pa$$word, changeme, changeme123, root, root123,
              guest, guest123, login, login123, dragon, master, monkey,
              shadow, sunshine, princess, baseball, football, superman,
              batman, trustno1, iloveyou, iloveyou1, michael, jessica,
              ashley, jennifer, 2024, 2025, 2026

KEYBOARD_PATTERNS — Tuple[str, ...], include:
  Top row:    qwerty, qwertyuiop, werty, erty, rtyuiop
  Home row:   asdfgh, asdfghjkl, sdfgh, dfghj
  Bottom row: zxcvbn, xcvbn
  Numbers:    12345, 23456, 34567, 45678, 56789, 67890
  Diagonals:  1qaz, 2wsx, 3edc, 4rfv, qazwsx, qazxsw
  Numpad:     789456123, 147258369
  Gaming:     wasd, wsad
  Reversed:   ytrewq, fdsa, lkjhg, nbvcxz

SEQUENTIAL_PATTERNS — Tuple[str, ...], include:
  Forward alpha:   abc, bcd, cde, def, efg, fgh, ghi, hij, ijk, jkl,
                   klm, lmn, mno, nop, opq, pqr, qrs, rst, stu, tuv,
                   uvw, vwx, wxy, xyz
  Reversed alpha:  cba, dcb, edc, fed, gfe, hgf, ihg, jih, zyx, yxw,
                   xwv, wvu, vut, uts, tsr, srq
  Forward digits:  123, 234, 345, 456, 567, 678, 789, 890
  Reversed digits: 321, 432, 543, 654, 765, 876, 987, 098

YEAR_PATTERN — compiled re.Pattern[str] matching (19|20)\d{2}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
ENTROPY & SCORING
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Entropy formula: round(len(password) * log2(charset_size), 2)
Charset size accumulation:
  +26   if has_lowercase
  +26   if has_uppercase
  +10   if has_digit
  +len(special_characters)  if has_special
  +1000 if has_unicode       ← accounts for extended unicode pool

Strength score (0–100, clamped):
  Length bonus:
    >= 24 chars → +35
    >= 18       → +30
    >= 14       → +25
    >= 12       → +20
  Diversity:  count of {upper, lower, digit, special} present × 10
  Entropy bonus:
    >= 120 bits → +25
    >= 90       → +20
    >= 70       → +15
    >= 50       → +10
  Uniqueness: if unique_chars / length >= 0.8 → +10
  Penalties:
    COMMON_PASSWORD    → −50
    KEYBOARD_PATTERN   → −20
    SEQUENTIAL_PATTERN → −15
    PASSWORD_TOO_SHORT → −25
    REPEATED_PATTERN   → −10
    YEAR_PATTERN       → −10
    (any other code)   → −5

After scoring, if score < minimum_strength_score, append a STRENGTH_TOO_LOW (HIGH) issue.

StrengthLevel mapping:
  >= 90 → VERY_STRONG | >= 75 → STRONG | >= 60 → GOOD
  >= 40 → FAIR        | >= 20 → WEAK   | else  → VERY_WEAK

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
CONVENIENCE API  (module-level)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

_default_validator = HyperSecurePasswordValidator()

def validate_password(password: str) -> bool:
    """Returns True only if the password passes all checks."""

def analyze_password(password: str) -> PasswordAnalysis:
    """Returns full structured analysis."""

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
QUALITY REQUIREMENTS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  - Zero external dependencies — stdlib only (math, re, unicodedata, dataclasses, enum, typing)
  - Python 3.11+ — use slots=True on all dataclasses
  - Thread-safe — validator holds no mutable state after __init__
  - All issue codes are SCREAMING_SNAKE_CASE strings
  - Use section dividers (# === SECTION ===) for readability
  - Annotate every fix or non-obvious decision with an inline comment
  - Use re.Pattern[str] directly — do not import Pattern from typing (deprecated 3.9+)
  - Include a runnable __main__ demo block testing at least 6 passwords of varying quality
    and printing: password, valid, score/100, strength, entropy bits, length, unique chars,
    and all issues with severity

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
EXPECTED BEHAVIOUR ON KEY TEST CASES
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

  "weak"                           → invalid  (too short, missing classes)
  "password123"                    → invalid  (common, sequential, too short)
  "Qwerty123!"                     → invalid  (keyboard + sequential + too short)
  "StrongPass#2026"                → invalid  (YEAR_PATTERN must fire)
  "UltraSecure#Enterprise2026!"    → invalid  (YEAR_PATTERN must fire)
  "MyUltra$Secure#BankingPass!"    → valid    (long, diverse, no patterns)
  None                             → invalid  (NULL_PASSWORD CRITICAL)
  12345  (int)                     → invalid  (INVALID_TYPE CRITICAL)
  "Ünïcödé#Páss!" + 4 more chars  → valid    (unicode allowed by default)

Produce the complete, runnable Python file with no placeholders or TODOs.
