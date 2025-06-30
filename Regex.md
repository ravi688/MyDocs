Regex (short for **Regular Expressions**) is a powerful tool used for pattern matching and text processing. It allows you to search, match, extract, and manipulate strings using patterns defined by a special syntax.

---

## 🔹 1. What is Regex?

Regular expressions are sequences of characters that form search patterns. You can use regex in many programming languages like Python, JavaScript, Java, and tools like `grep`, `sed`, `awk`, and text editors like VS Code and Vim.

---

## 🔹 2. Basic Syntax and Symbols

| Symbol  | Meaning                             | Example   | Matches                            |                |                |
| ------- | ----------------------------------- | --------- | ---------------------------------- | -------------- | -------------- |
| `.`     | Any single character except newline | `c.t`     | `cat`, `cut`, `c9t`                |                |                |
| `^`     | Start of string                     | `^Hi`     | Matches if string starts with "Hi" |                |                |
| `$`     | End of string                       | `end$`    | Matches if string ends with "end"  |                |                |
| `*`     | 0 or more repetitions               | `lo*l`    | `ll`, `lol`, `lool`, etc.          |                |                |
| `+`     | 1 or more repetitions               | `lo+l`    | `lol`, `lool` but not `ll`         |                |                |
| `?`     | 0 or 1 repetition (optional)        | `colou?r` | `color`, `colour`                  |                |                |
| `[]`    | Character set                       | `[aeiou]` | Any vowel                          |                |                |
| `[^]`   | Negated set                         | `[^0-9]`  | Any non-digit                      |                |                |
| \`      | \`                                  | OR        | \`cat                              | dog\`          | `cat` or `dog` |
| `()`    | Grouping                            | \`gr(a    | e)y\`                              | `gray`, `grey` |                |
| `{n}`   | Exactly n times                     | `a{3}`    | `aaa`                              |                |                |
| `{n,}`  | n or more times                     | `a{2,}`   | `aa`, `aaa`, etc.                  |                |                |
| `{n,m}` | Between n and m times               | `a{2,4}`  | `aa`, `aaa`, `aaaa`                |                |                |

---

## 🔹 3. Character Classes (Shorthand)

| Syntax | Meaning                            | Equivalent       |
| ------ | ---------------------------------- | ---------------- |
| `\d`   | Digit                              | `[0-9]`          |
| `\D`   | Non-digit                          | `[^0-9]`         |
| `\w`   | Word character (letter, digit, \_) | `[a-zA-Z0-9_]`   |
| `\W`   | Non-word character                 | `[^a-zA-Z0-9_]`  |
| `\s`   | Whitespace (space, tab, newline)   | `[ \t\n\r\f\v]`  |
| `\S`   | Non-whitespace                     | `[^ \t\n\r\f\v]` |

---

## 🔹 4. Examples

### ✅ Matching a valid email:

```regex
^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-z]{2,}$
```

Matches: `john.doe@example.com`

### ✅ Extracting numbers from a string:

```regex
\d+
```

Matches: `123` from `"Order #123"`

### ✅ Validating a phone number (simple US format):

```regex
^\(\d{3}\) \d{3}-\d{4}$
```

Matches: `(123) 456-7890`

---

## 🔹 5. Regex in Programming

### 📌 Python Example:

```python
import re

text = "My phone number is (123) 456-7890"
match = re.search(r"\(\d{3}\) \d{3}-\d{4}", text)
if match:
    print("Phone:", match.group())
```

### 📌 JavaScript Example:

```javascript
let text = "Contact: john.doe@example.com";
let match = text.match(/[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-z]{2,}/);
console.log("Email:", match[0]);
```

---

## 🔹 6. Greedy vs Non-Greedy

By default, quantifiers are greedy — they match as much as possible.

```regex
".*"
```

Matches: `"Hello World"` (entire quoted string)

```regex
".*?"
```

Matches: `"Hello"` from `"Hello" "World"` (shortest match)

---

## 🔹 7. Lookahead and Lookbehind

### 🔸 Positive Lookahead `(?=...)`

```regex
foo(?=bar)
```

Matches `foo` only if it is followed by `bar`

### 🔸 Negative Lookahead `(?!...)`

```regex
foo(?!bar)
```

Matches `foo` only if it is NOT followed by `bar`

### 🔸 Positive Lookbehind `(?<=...)`

```regex
(?<=\$)\d+
```

Matches digits preceded by a `$` (like `$100` → matches `100`)

### 🔸 Negative Lookbehind `(?<!...)`

```regex
(?<!\$)\d+
```

Matches digits NOT preceded by `$`

---

## 🔹 8. Tools to Test Regex

* [regex101.com](https://regex101.com) – Online editor for testing regex (Python, JavaScript, PHP, etc.)
* `grep`, `egrep`, `sed`, `awk` on Linux
* VS Code, Sublime Text (Find using regex mode)

---

## 🔹 9. Practical Use Cases

* **Search and Replace**: Replace phone numbers, emails, etc.
* **Validation**: Emails, IPs, URLs, passwords.
* **Scraping**: Extract data from HTML, logs, etc.
* **Log Analysis**: Parse and extract structured info from logs.

---

## 🔹 10. Tips & Best Practices

* Test patterns on real data before using.
* Use raw strings in Python: `r"..."` to avoid escaping `\`.
* Use named groups: `(?P<name>...)` in Python for clarity.
* Avoid overly complex regex — readability matters.
* Consider regex performance for large data.

---

If you'd like, I can walk you through **regex exercises**, build **real-world use cases**, or **analyze your existing regex** step by step.

Great! Let's walk through **regex step-by-step** with **interactive exercises** and **real-world use cases**.

---

# 🔹 Part 1: Foundations with Practice

## 1️⃣ **Simple Pattern Matching**

> **Goal:** Match the word `cat`.

### Regex:

```regex
cat
```

### Test Cases:

* ✅ `cat` → ✅ match
* ❌ `cater` → ✅ match (partial)
* ❌ `dog` → ❌ no match

> This matches any substring `"cat"` in the input.

---

## 2️⃣ **Match Any Character**

> Match any 3-character word that starts with `c` and ends with `t`.

### Regex:

```regex
c.t
```

### Test Cases:

* ✅ `cat`
* ✅ `cut`
* ✅ `c9t`
* ❌ `coat` (too long)

---

## 3️⃣ **Anchors: Start `^` and End `$`**

> Match string that is **exactly** `hello`.

### Regex:

```regex
^hello$
```

### Test Cases:

* ✅ `hello` → ✅
* ❌ `hello there` → ❌
* ❌ `say hello` → ❌

---

## 4️⃣ **Character Sets**

> Match any lowercase vowel.

### Regex:

```regex
[aeiou]
```

### Test Cases:

* `apple` → ✅ `a`, `e`
* `sky` → ❌

---

## 5️⃣ **Ranges and Negations**

> Match any **single digit**:

```regex
[0-9]
```

> Match any **non-digit**:

```regex
[^0-9]
```

---

## 6️⃣ **Repetitions**

| Regex    | Meaning       | Example             |
| -------- | ------------- | ------------------- |
| `a*`     | 0 or more a’s | `""`, `a`, `aaa`    |
| `a+`     | 1 or more a’s | `a`, `aaa`          |
| `a?`     | 0 or 1 a      | `""`, `a`           |
| `a{3}`   | exactly 3 a’s | `aaa`               |
| `a{2,4}` | 2 to 4 a’s    | `aa`, `aaa`, `aaaa` |

---

## ✅ Mini Exercise: Match words that begin with `gr` and end in `y`, with either `a` or `e` in the middle

### Regex:

```regex
gr(a|e)y
```

Matches: `gray`, `grey`

---

# 🔹 Part 2: Intermediate Patterns

## 1️⃣ **Extract Emails**

```regex
[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-z]{2,}
```

Matches:

* `test@example.com`
* `john.doe@mail.co.uk`

✅ Works well for most emails.

---

## 2️⃣ **Extract Dates (dd/mm/yyyy)**

```regex
\d{2}/\d{2}/\d{4}
```

Matches: `31/12/2025`

---

## 3️⃣ **Extract Numbers from Text**

```regex
\d+
```

Example: `"Price is 299 dollars"` → matches `299`

---

## 4️⃣ **Extracting Named Groups (Python)**

```python
import re

text = "Name: John, Age: 30"
match = re.search(r"Name: (?P<name>\w+), Age: (?P<age>\d+)", text)
print(match.group("name"))  # John
print(match.group("age"))   # 30
```

---

# 🔹 Part 3: Advanced Patterns

## 1️⃣ **Lookaheads**

> Match "foo" only when followed by "bar":

```regex
foo(?=bar)
```

`foobar` → ✅ `foo`

---

## 2️⃣ **Lookbehinds** (in languages that support them, like Python)

> Match digits preceded by `$`:

```regex
(?<=\$)\d+
```

`Total: $99` → ✅ `99`

---

# 🔹 Part 4: Real-world Case Studies

## ✅ 1. Extract all links from HTML

```regex
href="([^"]+)"
```

Input:

```html
<a href="https://google.com">Google</a>
```

Match: `https://google.com`

---

## ✅ 2. Validate Password (at least 8 chars, one digit, one letter)

```regex
^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d]{8,}$
```

Matches: `abc12345`, `passw0rd`

---

## ✅ 3. Mask Credit Card Numbers

Python:

```python
re.sub(r'\b\d{12}(\d{4})\b', r'************\1', 'Card: 1234567812345678')
```

Output: `Card: ************5678`

---

# 🔹 Part 5: Your Turn to Practice

Try writing regexes for these:

1. Match US ZIP code: `12345` or `12345-6789`
2. Match IPv4 address: `192.168.1.1`
3. Match time: `23:59` or `09:15`

Want help solving these? Want more exercises or real-world examples like logs, emails, or CSV extraction?

Let me know what you'd like to focus on next:
✅ More exercises
✅ Web scraping
✅ Log analysis
✅ Data validation
✅ Regex debugger tools

Perfect! Let's go deeper step-by-step.

---

# 🔸 Part 6: Practice Exercises + Solutions

## ✅ **Exercise 1: Match US ZIP Code**

* Format: `12345` or `12345-6789`

### 🔍 Regex:

```regex
^\d{5}(-\d{4})?$
```

### 💡 Explanation:

* `^\d{5}` – starts with exactly 5 digits
* `(-\d{4})?` – optional dash followed by 4 digits
* `$` – end of string

### ✅ Examples:

* `12345` → ✅
* `12345-6789` → ✅
* `123456` → ❌

---

## ✅ **Exercise 2: Match IPv4 Address**

* Format: `192.168.1.1`

### 🔍 Regex (simple version):

```regex
^(\d{1,3}\.){3}\d{1,3}$
```

### ✅ Examples:

* `192.168.1.1` → ✅
* `10.0.0.1` → ✅

> ⚠️ This does **not** check for valid octets (0–255). For stricter validation:

### 🔍 Regex (strict version):

```regex
^(25[0-5]|2[0-4]\d|1\d{2}|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d{2}|[1-9]?\d)){3}$
```

---

## ✅ **Exercise 3: Match Time (HH\:MM in 24-hour format)**

### 🔍 Regex:

```regex
^([01]\d|2[0-3]):[0-5]\d$
```

### ✅ Examples:

* `23:59` → ✅
* `00:00` → ✅
* `24:00` → ❌

---

# 🔸 Part 7: Real-World Use Cases

---

## ✅ 1. **Extract URLs from text**

### 🔍 Regex:

```regex
https?://[^\s]+
```

Matches:

* `http://example.com`
* `https://openai.com/blog`

---

## ✅ 2. **Parse Apache/Nginx Logs**

Log line:

```
127.0.0.1 - - [10/Oct/2023:13:55:36 +0000] "GET /index.html HTTP/1.1" 200 1043
```

### 🔍 Regex:

```regex
^(\d{1,3}\.){3}\d{1,3} .*? \[(.*?)\] "(.*?)" (\d{3}) (\d+)
```

Groups:

* IP Address
* Timestamp
* Request
* Status Code
* Response Size

---

## ✅ 3. **Extract CSV Fields (comma-separated, possibly quoted)**

### 🔍 Regex:

```regex
(?:\"([^\"]*)\"|([^,]+))(?:,|$)
```

Matches:

* `John,Doe,25`
* `"John, A.",Doe,25`

---

# 🔸 Part 8: Regex Debugging Tools

Use these to **test**, **debug**, and **learn**:

| Tool                                 | Description                          |
| ------------------------------------ | ------------------------------------ |
| [regex101.com](https://regex101.com) | Best regex debugger with explanation |
| [RegExr](https://regexr.com)         | Interactive and educational          |
| [regexr.com](https://regexr.com)     | Friendly UI, live testing            |
| VS Code `Ctrl+F` + .\*               | Search with regex in files           |

---

# 🔸 Part 9: Custom Regex Tasks (optional for you)

Pick any of the following, and I’ll walk you through the solution:

1. 🔐 Password strength checker (uppercase, lowercase, digit, symbol, min 8 chars)
2. 🛒 Extract product name and price from invoices
3. 📧 Validate more email types (including `.co.in`, etc.)
4. 📜 Clean up and reformat logs
5. 📂 Match file extensions like `.pdf`, `.jpg`, `.zip`

Would you like to:

* ✅ Solve these step-by-step with me?
* ✅ Use regex in a programming language like Python, JS, Bash?
* ✅ Learn how to *build your own regex validator tool*?

Let me know what you'd like to do next!
