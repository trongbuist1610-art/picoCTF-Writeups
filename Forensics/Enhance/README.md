#  Enhance! — picoCTF Forensics

## 📋 Challenge Info

| Field | Details |
|-------|---------|
| Challenge | Enhance! |
| Category | Forensics |
| Difficulty | 🟢 Easy |
| Flag | `picoCTF{r3st0r1ng_th3_by73s_2326ca93}` |
| Tools | hexed.it |

---

## 📝 Description

> "Maybe a couple of bytes could make all the difference."

Given file: `file` (no extension)

---

## 🔎 Analysis

### Step 1 — Open the file with hexed.it

Go to [hexed.it](https://hexed.it) → open the file → look at the first bytes:

```
Offset   | 00 01 02 03 04 05 06 07
---------+------------------------
00000000 | 5C 78 FF E0 00 10 4A 46
            ^^  ^^
            Wrong!
```

### Step 2 — Corrupted Magic Bytes

Every valid JPEG must start with `FF D8`:

```
Offset   | 00 01 02 03 04 05 06 07
---------+------------------------
00000000 | FF D8 FF E0 00 10 4A 46
            ^^  ^^
            Correct!
```

| Position | Original | Should Be | Explanation |
|----------|----------|-----------|-------------|
| Byte 0 | `5C` | `FF` | `5C` = ASCII `\` |
| Byte 1 | `78` | `D8` | `78` = ASCII `x` |
| Byte 2+ | `FF E0 ...` | `FF E0 ...` | Already correct |

The first 2 bytes were saved as text `\x` instead of actual bytes `FF D8`.

### Step 3 — How do we know it's a JPEG?

Bytes `4A 46 49 46` = **JFIF** — visible in the text column of hexed.it.

| File Type | Magic Bytes | Notes |
|-----------|-------------|-------|
| JPEG/JFIF | `FF D8 FF E0` | "JFIF" at bytes 6-9 |
| JPEG/Exif | `FF D8 FF E1` | "Exif" at bytes 6-9 |
| PNG | `89 50 4E 47` | Literally `PNG` |
| PDF | `25 50 44 46` | Literally `%PDF` |
| ZIP | `50 4B 03 04` | Literally `PK` |
| GIF | `47 49 46 38` | Literally `GIF8` |

Reference: [List of file signatures](https://en.wikipedia.org/wiki/List_of_file_signatures)

---

## 🛠️ Solution

### Method 1 — hexed.it

1. Open [hexed.it](https://hexed.it) → click **Open file**
2. Click byte `5C` (top-left corner)
3. Type `FF` → press **Tab**
4. Type `D8`
5. Click **Save as** → save as `fixed.jpg`
6. Open `fixed.jpg` → flag appears! 🎉

```
Before: 5C 78 FF E0 ...
After:  FF D8 FF E0 ...
```

### Method 2 — Python

```python
with open('file', 'rb') as f:
    data = f.read()

fixed = b'\xff\xd8' + data[2:]

with open('fixed.jpg', 'wb') as f:
    f.write(fixed)
```

---

## 🏁 Flag

```
picoCTF{r3st0r1ng_th3_by73s_2326ca93}
```

---

## 💡 Key Takeaways

- Magic bytes identify a file's format — independent of filename or extension.
- When a file is broken, always check magic bytes first using hexed.it.
- Text `\x` = `5C 78` in hex — common error when copy-pasting binary data.
- Reference: [Gary Kessler's file signatures](https://www.garykessler.net/library/file_sigs.html)
