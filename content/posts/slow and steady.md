---
title: "Slow and Steady Wins the Race"
date: 2024-05-20
draft: false
---

**Category:** Steganography / Audio Forensics  
**Difficulty:** Easy  
**Author:** Saurav Sreejith 
<!--more-->

## 📝 Challenge Description
> `$can you fix the broken?`
> `$can you feel my heart?`

**Attached File:** `chad.wav`

---

## 🕵️‍♂️ Solution Walkthrough

### Step 1: Analyzing the Clues & The Audio
We are given an audio file named `chad.wav`. Listening to the file, we hear absolute silence for the first few seconds, followed immediately by harsh, rhythmic, robotic screeching and beeping sounds. 

Before we jump into tools, let's look at the hints provided by the challenge creator:
1. **The Title:** "*Slow* and steady..."
2. **The Description:** The unusual use of the dollar sign in `$`can points us toward the word **"scan"**. 
3. **The Lore:** The text "can you fix the broken, can you feel my heart" are lyrics from a Bring Me The Horizon song heavily associated with the "Gigachad" internet meme, perfectly matching our filename (`chad.wav`).

Combining **"Slow"** and **"Scan"** with the robotic sounds, we can confidently identify this as **SSTV (Slow-Scan Television)**, a method used by radio amateurs to transmit and receive static pictures via audio.

### Step 2: The Twist (Evading Standard Readers)
Normally, to decode SSTV, you pipe the audio into software like RX-SSTV (Windows), QSSTV (Linux), or a mobile app. 

However, participants who tried to auto-decode this file likely got nothing but a blank screen. Why? 
Standard SSTV audio starts with a specific calibration header (known as the VIS code) that tells the software exactly what mode (e.g., Martin M1, Scottie 1) the image is transmitted in so it can auto-sync. The creator intentionally padded the beginning of the audio with silence, effectively stripping out these headers to break standard auto-detecting readers!

### Step 3: Decoding in Raw Mode
To bypass this trick, we need an SSTV decoder that allows us to force-read the audio stream without relying on the missing headers. 

A highly effective tool for this is the **Robot36** app (available on Android). 
1. Open Robot36.
2. Go to settings and enable **Raw Mode** (or manually select a common SSTV mode like Scottie 1 / Martin 1 depending on the encoding). 
3. Play the `chad.wav` audio out loud from a computer speaker directly into the phone's microphone.

Because Raw Mode forces the app to listen to the audio regardless of the missing calibration header, it begins painting the image line by line.

As the screeching finishes, an image (likely of the Gigachad himself) is decoded on our screen, with the flag written across it.


![sstv](/images/sstv.png)

---

## 🚩 Flag
```text
DEDSEC{4N4L0G_H1D3_4ND_S33K}
```
