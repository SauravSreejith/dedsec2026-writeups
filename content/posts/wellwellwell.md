---
title: "Well Well Well"
date: 2024-05-20
draft: false
---

**Category:** Forensics / Steganography  
**Difficulty:** Easy   
**Author:** Saurav Sreejith 
<!--more-->

## 📝 Challenge Description
> Oh, Father, tell me
> Do we get what we deserve?
> Oh, we get what we deserve
> **And way down we go**

**Attached File:** `abyss.png` (A picture of a well)

---

## 🕵️‍♂️ Solution Walkthrough

### Step 1: Initial Reconnaissance

![alt text](/images/abyss.png)

We are given an image named `abyss.png`. The visual of a deep well, combined with the challenge name ("Well Well Well") and the lyrics to KALEO's song *Way Down We Go*, all heavily imply that we need to look *deeper* or go *down*. 

In image forensics, the first step is almost always to check the metadata. Let's run `exiftool` on the image:

```bash
└─[$] <> exiftool abyss.png
ExifTool Version Number         : 13.55
File Name                       : abyss.png
...
Image Width                     : 832
Image Height                    : 832
...
Artist                          : Robert_Wadlow_3464
Image Size                      : 832x832
Megapixels                      : 0.692
```

### Step 2: Decoding the Metadata Clues
Scanning the EXIF data, one specific line stands out as highly unusual:
`Artist : Robert_Wadlow_3464`

Who is Robert Wadlow? A quick Google search reveals that Robert Wadlow is the **tallest person in recorded human history**. This is a massive hint that the image's *height* has been tampered with! The numbers appended to his name, **`3464`**, indicate the true, hidden height of the image. 

Currently, the image header claims the height is `832`. Because standard image viewers only read the height specified in the PNG's `IHDR` chunk, they stop rendering at 832 pixels, hiding the rest of the underlying image data that is still intact in the file.

### Step 3: Fixing the IHDR Chunk
To reveal the hidden bottom of the well (where the flag resides), we need to modify the PNG's `IHDR` chunk to change the height to `3464`. 

However, we can't just change the bytes in a hex editor and save it. PNG chunks use a **CRC32 Checksum** to ensure data integrity. If we change the height without recalculating and updating the CRC checksum, image viewers will see the file as corrupted and refuse to open it.

We can write a simple Python script to locate the IHDR chunk, patch the height to `3464`, recalculate the CRC32 hash, and output the fixed image:

```python
import zlib
import struct

INPUT_FILE = "abyss.png"  
OUTPUT_FILE = "flag.png"      
TARGET_HEIGHT = 3464          

def fix_png_height(input_path, output_path, new_height):
    with open(input_path, "rb") as f:
        data = bytearray(f.read())

    # Find the IHDR chunk
    ihdr_start = data.find(b"IHDR")
    
    if ihdr_start == -1:
        print("[-] Error: Could not find IHDR chunk. Is this a valid PNG?")
        return
    print("[+] Found IHDR chunk.")

    # 1. Update the Height
    # The height is 4 bytes long, located 8 bytes after the start of "IHDR"
    height_start = ihdr_start + 8
    
    # Pack the new height as a 4-byte big-endian integer (>I)
    new_height_bytes = struct.pack(">I", new_height)
    data[height_start : height_start + 4] = new_height_bytes
    print(f"[+] Changed height to {new_height} (Hex: {new_height_bytes.hex()}).")

    # 2. Fix the CRC Checksum
    # The CRC is calculated over the Chunk Type ("IHDR") + Chunk Data (13 bytes)
    crc_data_start = ihdr_start
    crc_data_end = ihdr_start + 17
    
    ihdr_data = data[crc_data_start : crc_data_end]
    new_crc = zlib.crc32(ihdr_data)
    
    # The CRC checksum itself is located immediately after the IHDR data
    crc_start = crc_data_end
    data[crc_start : crc_start + 4] = struct.pack(">I", new_crc)
    print(f"[+] Fixed CRC checksum: {hex(new_crc)}.")

    # 3. Save the modified image
    with open(output_path, "wb") as f:
        f.write(data)
        
    print(f"[+] Success! Image saved as '{output_path}'.")

if __name__ == "__main__":
    fix_png_height(INPUT_FILE, OUTPUT_FILE, TARGET_HEIGHT)
```

### Step 4: Extracting the Flag
Running the script yields a successful output:
```bash
[+] Found IHDR chunk.
[+] Changed height to 3464 (Hex: 00000d88).
[+] Fixed CRC checksum: 0x...
[+] Success! Image saved as 'flag.png'.
```

![alt text](/images/flag.png)

Opening the newly generated `flag.png`, the image now renders all the way to the bottom of the well, revealing the hidden flag written in the newly exposed space!

---

## 🚩 Flag
```text
DEDSEC{y0u_f0und_m3_cuti3}
```
