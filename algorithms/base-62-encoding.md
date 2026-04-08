---
title: "Base 62 Encoding"
category: algorithms
summary: "Base 62 encoding converts numbers into strings using 62 characters (0-9, a-z, A-Z) to create compact, URL-safe representations. It's commonly used in URL shorteners to generate short, unique identifiers."
sources:
  - raw/articles/url-shortener-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:49:28.874Z
---

# Base 62 Encoding

> Base 62 encoding converts numbers into strings using 62 characters (0-9, a-z, A-Z) to create compact, URL-safe representations. It's commonly used in URL shorteners to generate short, unique identifiers.

# Base 62 Encoding

Base 62 encoding is a numeral system that represents numbers using 62 different characters: digits `0-9`, lowercase letters `a-z`, and uppercase letters `A-Z`. This encoding scheme is particularly useful for creating compact, URL-safe identifiers.

## Character Set

The Base 62 character set consists of:
- Digits: `0, 1, 2, 3, 4, 5, 6, 7, 8, 9` (10 characters)
- Lowercase: `a, b, c, ..., z` (26 characters)
- Uppercase: `A, B, C, ..., Z` (26 characters)

Total: **62 unique characters**

## Conversion Process

To convert a decimal number to Base 62:
1. Divide the number by 62
2. Map the remainder to the corresponding character
3. Repeat with the quotient until it becomes 0
4. Read the characters in reverse order

**Example:**
Converting `2009215674938` to Base 62:
- `2009215674938 ÷ 62 = 32406704273` remainder `44` → `i`
- `32406704273 ÷ 62 = 522688004` remainder `25` → `p`
- Continue process...
- Result: `zn9edcu`

## Applications

### URL Shorteners
Base 62 encoding is ideal for [[URL Shortener]] systems because:
- Creates short, readable identifiers
- URL-safe (no special characters)
- Supports massive scale (62^7 = 3.5 trillion combinations)
- Deterministic conversion from unique IDs

### Advantages
- **Compact:** More efficient than hexadecimal (Base 16)
- **URL-Safe:** No special characters requiring encoding
- **Human-Readable:** Uses familiar alphanumeric characters
- **Collision-Free:** When based on unique IDs

### Disadvantages
- **Predictable:** Sequential IDs create predictable URLs (security concern)
- **Variable Length:** Length increases with larger numbers
- **Case-Sensitive:** Requires careful handling of uppercase/lowercase

Base 62 encoding provides an elegant solution for creating short, unique identifiers in distributed systems requiring compact representations.

---
*Related: [[URL Shortener]], [[Unique ID Generation]], [[Hash Functions]]*
