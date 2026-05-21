# Git in the Disk — picoCTF Forensics

### 📋 Challenge Info

| Field | Details |
| :--- | :--- |
| **Challenge** | Git in the Disk |
| **Category** | Forensics / Disk Analysis / Git |
| **Difficulty** | 🟡 Medium |
| **Flag** | `picoCTF{g17_1n_7h3_d15k_041217d8}` |
| **Tools** | `binwalk`, `7z`, The Sleuth Kit (`fls`, `tsk_recover`), `git` |

### 📝 Description

"The target's system was abruptly compromised while deleting disk data. Find a way to recover the hidden Git repository to investigate the secret."  
**Given file:** `disk.img` (Raw disk image)

---

### 🔎 Analysis & Practice

**Step 1 — Disk Structure Survey** Use `binwalk` to analyze the `disk.img` file. We discover the system uses an MBR partition table and contains Linux ext4 partitions.

```bash
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             MBR boot signature
...
584056832     0x22D00000      Linux ext filesystem, blocks count: 122880
              ^^  ^^
              Main partition containing user data (ext4)
```
**Step 2 — Hunt for the hidden .git treasure
Extract the outer MBR layer using 7z (resulting in 0.img, 1.img, 2.img), where 2.img is the largest partition. Avoid using the mount command as it often causes system errors; instead, use fls (The Sleuth Kit) to recursively scan for hidden/deleted directories.
# Extract the outer shell
7z x disk.img

# Search for the .git directory inside the 2.img partition
fls -r 2.img | grep -i ".git"

Plaintext
```
Root directory
 └── home/
      └── ctf-player/
           └── Code/
                └── secrets/
                     └── .git/  <-- Jackpot hit at Inode 65665!

```
Step 3 — Git Timeline Extract the secrets directory using tsk_recover. Checking the history with git log reveals the author left a clear message in the latest commit.
```Bash
# Extract all data to the 'extracted_data' directory
tsk_recover -r 2.img extracted_data

# Navigate to the treasure location and view the history
cd extracted_data/home/ctf-player/Code/secrets/
git log --all --oneline --graph
```
```Plaintext
Commit Hash | Message
------------+-------------------------------------------------------------
* 327681b   | (HEAD -> master) Wrap this phrase in the flag format: g17_1n_7h3_d15k_041217d8
* 177789a   | Add flag
```
🛠️ Solution (Quick Commands)
Method 1 — The Sleuth Kit (Recommended)
```Bash
7z x disk.img
fls -r 2.img | grep -i ".git"
tsk_recover -r 2.img extracted_data
cd extracted_data/home/ctf-player/Code/secrets/
git log --all --oneline --graph
```
```Bash
7z x disk.img -o/tmp/disk
cd /tmp/disk
7z x 2.img -o/tmp/data
cd /tmp/data/home/ctf-player/Code/secrets/
git log --all --oneline --graph
```
### ⚠️ Troubleshooting (Common Errors)

| Error Message | Cause | Solution |
| :--- | :--- | :--- |
| `mount: failed to...` | Conflict with security/Symlinks. | Do NOT use `mount`. Use `tsk_recover` or `7z` for Raw Extraction. |
| `fatal: not a git repo` | Running `git` in the wrong directory. | Use `cd` to navigate into the specific folder containing `.git`. |
| `No such file...` | `7z` path typo or filename error. | Check with `pwd`. Use absolute paths and no spaces after `-o`. |
| Code directory empty | Working Tree deleted; only Metadata remains. | This is normal! Proceed with `git log --all`. |

🏁 Flag 
```
picoCTF{g17_1n_7h3_d15k_041217d8}
```
💡 Key Takeaways
Deleted is not gone: The rm command or deleting a commit merely destroys Metadata. Raw binary objects remain on the disk.

The power of --all: git log --all helps you see through hidden branches or orphan commits.


