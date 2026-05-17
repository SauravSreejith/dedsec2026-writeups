---
title: "Yapcord"
date: 2024-05-20
draft: false
---

**Category:** Web / Steganography  
**Difficulty:** Easy  
**Author:** Saurav Sreejith 
<!--more-->

## 📝 Challenge Description
> Man they really made a discord clone, it's filled with a bunch of weirdos.
> 
> `https://yapcord.vercel.app/`

---

## 🕵️‍♂️ Solution Walkthrough

### Step 1: Web Reconnaissance & The Network Tab

![yapcord](/images/yapcord.png)

Visiting the provided link, we are greeted with "Yapcord," a simulated chatroom filled with various eccentric characters spamming messages. 

We have a few ways to analyze the chat:
1. **The Slow Way:** Sit and read the messages as they appear on the screen, hoping to spot a clue.
2. **The Hacker Way:** Open Developer Tools (`F12`), navigate to the **Network** tab, and refresh the page. We can immediately spot the frontend fetching a file called `chat.json` to populate the chatroom. *(Alternatively, participants could use directory fuzzing tools like DirBuster or ffuf to discover the `/chat.json` endpoint).*

![chat](/images/chat.png)

Looking directly at the `chat.json` response, we get the full, unredacted list of every user and their automated messages.

### Step 2: Following the Clues
Scanning through the `chat.json` data, we can notice a recurring theme hidden among the spam. Several different bots specifically mention "grass":
* **Conspiracy_Carl:** *"who is Grass really?", "the grass is a plant... literally"*
* **xX_Shadow_Edge_Xx:** *"touch some grass Shadow says"*
* **Toxic_Avenger:** *"wait, the grass is right there"*
* **AI_Overlord_2045:** *"GRASS IS 99% GREEN"*

This heavily implies that the user named **"Grass"** is our target. 

### Step 3: Inspecting the Avatar
Let's look at the profile picture for the Grass user (`grass.png`). At first glance, it looks like a piece of abstract pixel art—a brown background scattered with seemingly random green pixels. 

![grass](/images/grass.png)

However, looking closely at the pixel distribution, the chaotic layout of the green pixels looks suspiciously similar to the data matrix of a **QR Code**. The only problem? It's missing the three large, distinct square boxes in the corners (known as "Finder Patterns") that allow a camera or software to recognize it as a QR code.

### Step 4: Reconstructing the QR Code
Because the finder patterns are missing, standard QR scanners will completely ignore the image. We need to play the role of a digital artist and fix it.

1. Download `grass.png` and open it in any image editing software (Photoshop, GIMP, Photopea, or even MS Paint).
2. A standard QR code requires three positional squares located at the **Top-Left**, **Top-Right**, and **Bottom-Left** corners.
3. You can manually draw these boxes (a dark outer square, a light inner boundary, and a dark center square), or simply search Google Images for a "blank QR code", copy the three corner squares, and paste them perfectly over the corners of `grass.png`.

### Step 5: Scanning for the Flag
Once the three corner boxes are restored, the image transforms back into a valid, readable QR code. 

![grass-fixed](/images/grass-fixed.png)

Scanning the newly fixed image with a phone camera or an online QR code reader instantly decodes the hidden text, revealing the flag!

---

## 🚩 Flag
```text
DEDSEC{p1c4ss0_0f_qr_c0d3s}
```
