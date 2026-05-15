🔍 Enhance! — picoCTF Forensics
📋 Challenge Info
FieldDetailsChallengeEnhance!CategoryForensicsDifficulty🟢 EasyFlagpicoCTF{r3st0r1ng_th3_by73s_2326ca93}Toolshexed.it

📝 Description

"Maybe a couple of bytes could make all the difference."

Given file: file (no extension)

🔎 Analysis
Step 1: Open the file with hexed.it
Go to https://hexed.it → open the file → look at the first bytes:
Offset   | 00 01 02 03 04 05 06 07 ...
---------+----------------------------
00000000 | 5C 78 FF E0 00 10 4A 46 ...
           ^^  ^^
           Wrong! Wrong!
Step 2: Identify the problem — Corrupted Magic Bytes
Every valid JPEG file must start with the magic bytes FF D8:
Offset   | 00 01 02 03 04 05 06 07 ...
---------+----------------------------
00000000 | FF D8 FF E0 00 10 4A 46 ...   ← Valid JPEG
           ^^  ^^
           Correct!
Comparing the original file with the standard:
PositionOriginalShould BeExplanationByte 05CFF5C = ASCII character \Byte 178D878 = ASCII character xByte 2+FF E0 ...FF E0 ...✅ Already correct
→ Conclusion: The first 2 bytes were stored as the text string \x instead of the actual bytes FF D8.
Step 3: How do we know it's a JPEG?
Looking further into the file: 4A 46 49 46 = the text JFIF, clearly visible in the right-side text column of hexed.it.
Common file magic bytes for reference:
File TypeMagic Bytes (Hex)NotesJPEG/JFIFFF D8 FF E0Contains "JFIF" at bytes 6-9JPEG/ExifFF D8 FF E1Contains "Exif" at bytes 6-9PNG89 50 4E 47Literally ‰PNGPDF25 50 44 46Literally %PDFZIP50 4B 03 04Literally PKGIF47 49 46 38Literally GIF8
📌 Full reference: https://en.wikipedia.org/wiki/List_of_file_signatures

🛠️ Solution
Method 1: Using hexed.it (no installation needed)
Step 1: Go to https://hexed.it → click "Open file" → select the file
Step 2: Click on the 5C byte (first byte, top-left corner)
Step 3: Type FF → press Tab to move to the next byte
Step 4: Type D8
Result after the fix:
Before: 5C 78 FF E0 00 10 4A 46 49 46 ...
After:  FF D8 FF E0 00 10 4A 46 49 46 ...
Step 5: Click "Save as" → save as fixed.jpg
Step 6: Open fixed.jpg → see the image and the flag! 🎉

Method 2: Using Python
pythonwith open('file', 'rb') as f:
    data = f.read()

# Fix the first 2 bytes: 5C 78 -> FF D8
fixed = b'\xff\xd8' + data[2:]

with open('fixed.jpg', 'wb') as f:
    f.write(fixed)

print("Done! Open fixed.jpg to see the flag.")

🏁 Flag
picoCTF{r3st0r1ng_th3_by73s_2326ca93}

💡 Key Takeaways

Magic bytes are the first few bytes of a file used to identify its format — completely independent of the file name or extension.
When a file appears "broken", the first step is always to open it in hexed.it and check the magic bytes.
The text string \x equals 5C 78 in hex — a common mistake when copy-pasting binary data.
Look up magic bytes at: https://www.garykessler.net/library/file_sigs.html
