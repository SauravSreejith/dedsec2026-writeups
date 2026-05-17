---
title: "THE PRIME SUSPECT"
date: 2024-05-20
draft: false
---

**Category:** Forensics / Steganography / Scripting  
**Difficulty:** Hard  
**Author:** Saurav Sreejith 
<!--more-->

## 📝 Challenge Description
We are provided with a zip file containing the remnants of a suspect's digital footprint. According to the `CASE_1866_NOTES.txt`, the suspect initiated a wipe, causing severe logical damage. We are tasked with analyzing the recovered artifacts for actionable intelligence. 

**Contents:**
* `ambulance.jpg`
* `browser_history.csv`
* `config.ini`
* `notes.txt`
* `bash_history_recovered.log`
* `CASE_1866_NOTES.txt`
* `crime_and_punishment.txt`

---

## 🕵️‍♂️ Solution Walkthrough

### Step 1: Triage and Red Herrings
Upon extracting the archive, we see a variety of files. Looking through `bash_history`, `browser_history`, `notes.txt`, and `config.ini` paints a great lore picture (searching for bleach, deleting ledgers, reading about the statute of limitations for homicide), but forensically, these are **red herrings**.

The actual anomalies lie in two files: `ambulance.jpg` and `crime_and_punishment.txt`.

### Step 2: The EXIF Breadcrumbs
Let's analyze `ambulance.jpg`. It's a picture of an ambulance seen through a rearview mirror. Let's check the metadata using `exiftool`:

```bash
└─[$] <> exiftool ambulance.jpg
...
Software                        : 4-Byte Header Studio 24.3
Image Unique ID                 : 0203050711131719
...
```

Three massive clues are hidden here:
1. **The Image Unique ID (`0203050711131719`):** If we break this string apart, we get `02, 03, 05, 07, 11, 13, 17, 19`. This is the mathematical sequence of **Prime Numbers**. This also ties into the challenge title: "The *Prime* Suspect".
2. **The Subject (Ambulance in a mirror):** The word "AMBULANCE" is famously printed backwards on vehicles so it can be read in a rearview mirror. This hints at a **Reverse** or **Backwards** operation.
3. **The Software (`4-Byte Header`):** This hints that whatever data we extract will begin with a 4-byte size header.


![alt text](/images/ambulance.jpg)

### Step 3: Analyzing the Text File
Now we turn to `crime_and_punishment.txt`, a massive file containing the classic novel. When trying to open it in a standard text editor, we get a warning:

![alt text](/images/error.png)

Text editors expect valid text encoding (like ASCII or UTF-8). If they encounter raw binary bytes (`\x00`, `\x89`, etc.), they throw errors. This means binary data is stuffed inside the text. Let's look at a Hex Dump of the first few lines:

```text
00000000: 6372 496d 4520 614e 6420 7055 6e49 7368  crImE aNd pUnIsh
...
00000050: 4a55 6c59 2061 2089 4f75 6e47 206d 614e  JUlY a .OunG maN
00000060: 2043 414d 4520 6f50 744e 6f66 2074 4865   CAME oPtNof tHe
00000070: 2067 6152 7245 7420 494e 2057 6869 6348   gaRrEt IN WhicH
00000080: 2048 6520 6c4f 6467 4544 2047 6e20 532e   He lOdgED Gn S.
```

If we carefully isolate the non-printable and out-of-place characters, a pattern emerges:
* At `00000050`, there is an `89`.
* At `00000060`, the letters `P` and `N` replace normal characters.
* At `00000080`, the letter `G` appears out of nowhere.

`89 50 4E 47` is the magic byte signature for a **PNG Image**. Following closely behind, we can also spot `49 48 44 52` (the `IHDR` chunk). A PNG file has been shattered into pieces and scattered byte-by-byte throughout the novel!

### Step 4: The Extraction Logic
How do we know *which* indices hold the hidden PNG bytes? Let's combine our clues from Step 2:
* **Primes:** The bytes are hidden at prime number indices.
* **Reverse (Ambulance):** The indices are calculated backwards from the end of the file. 

Therefore, the exact position of the hidden bytes is: `File Size - Prime Number`.
Additionally, the EXIF data hinted at a **"4-Byte Header"**, meaning the first 4 bytes we extract will tell us the exact size of the payload. 

We can write a Python script to sieve for primes, calculate the reverse positions, read the 4-byte header, and extract the hidden PNG:

```python
import math

# Generate prime numbers up to n using the Sieve of Eratosthenes
def primes_upto(n):
    sieve = bytearray(b"\x01") * (n + 1)
    sieve[0:2] = b"\x00\x00"
    for i in range(2, int(math.sqrt(n)) + 1):
        if sieve[i]:
            sieve[i*i:n+1:i] = b"\x00" * len(sieve[i*i:n+1:i])
    return [i for i in range(2, n + 1) if sieve[i]]

def extract(embedded_path, output_path):
    with open(embedded_path, "rb") as f:
        story = f.read()

    file_size = len(story)
    
    # Recreate the exact position array: (Total Size - Prime)
    primes = primes_upto(file_size)
    positions = sorted([file_size - p for p in primes if file_size - p > 0])

    # 1. Extract the first 4 bytes to read the size header
    header_bytes = bytearray()
    for i in range(4):
        header_bytes.append(story[positions[i]])
    
    # Decode the Big-Endian 4-byte integer to get payload size
    payload_size = int.from_bytes(header_bytes, byteorder='big')
    print(f"[*] Detected 4-Byte Header...")
    print(f"[*] Payload size to extract: {payload_size} bytes")

    # 2. Extract the actual hidden PNG based on the size
    payload = bytearray()
    for i in range(4, 4 + payload_size):
        payload.append(story[positions[i]])

    # Save to disk
    with open(output_path, "wb") as f:
        f.write(payload)
        
    print(f"[*] Extraction complete! Saved to {output_path}")

extract("crime_and_punishment.txt", "recovered_data.png")
```

### Step 5: Bit Plane Steganography
Running the script successfully extracts `recovered_data.png`. However, when we open the image, it appears to be completely solid black.

![alt text](/images/recovered_data.png)

This is a classic CTF steganography trick. The image isn't actually a single solid color; it contains a QR code drawn with colors that are only 1 bit apart in value (e.g., `#000000` vs `#010101`). To the human eye, it looks blank.

We can analyze the image using a tool like **StegSolve** (a Java-based image analysis tool) or `zsteg`. 
By opening the image in StegSolve and clicking the forward arrow to cycle through the color planes, we isolate the Least Significant Bits (LSB). When we reach `Red plane 0` or `Green plane 0`, the subtle differences in pixel values are highlighted in high contrast, revealing a perfectly intact **QR Code**.

![alt text](/images/solved.png)

### Step 6: Capturing the Flag
Scanning the revealed QR code using a phone or decoding software gives us the final flag.

---

## 🚩 Flag
```text
DEDSEC{TH3_4X3_MURD3R}
```
