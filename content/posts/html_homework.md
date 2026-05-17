---
title: "HTML Homework"
date: 2024-05-20
draft: false
---

**Category:** Web / Crypto  
**Difficulty:** Easy  
**Author:** Saurav Sreejith 

<!--more-->
## 📝 Challenge Description
> hey big bro, 
> i just learend html at school today!!!
> i cant wait till tommorow to go school and flex in style. i think i added too much stuff but whatever. 
> 
> - your lil bro
> 
> oh almost forgot! here is the link : `https://sigma-soundboard-rizz.vercel.app/`


## 🕵️‍♂️ Solution Walkthrough

### Step 1: Initial Reconnaissance

![main](/images/main-website.png)

Clicking the provided link takes us to a "Sigma Soundboard" website. While it's fun to click around and listen to the audio files, there are no obvious flags or inputs on the main page. In web challenges, when the front end doesn't give you anything, the next step is always to look under the hood at the source code.

### Step 2: Finding the Hidden CSS (Inspecting the Source)
The challenge description drops a subtle hint with the phrase *"flex in **style**"*. This is a nod to Cascading Style Sheets (`.css`). 

Here is how you can find the `style.css` file:
1. **Right-click** anywhere on the webpage and select **"Inspect"** (or press `F12` / `Ctrl+Shift+I` on your keyboard) to open Developer Tools.
2. Look at the **Elements** tab, which shows the HTML structure. 
3. Expand the `<head>` tag at the top of the HTML. You will see a line linking the stylesheet, looking something like this: `<link rel="stylesheet" href="style.css">`.
4. You can either click on the `style.css` link directly from there, or switch over to the **Sources** (or **Network**) tab in Developer Tools, navigate the file tree on the left, and click on `style.css` to view its contents.

### Step 3: Analyzing the Stylesheet
Looking through the CSS code, we find an interesting comment left behind by the developer:

```css
/* fallback_id: REVEU0VD{340c926071e79ea3c8168726dfb067540e7fdf0476df52a2458ee2480af91ccb} */
```

This string is clearly our flag, but it's obfuscated. We need to break it down into two parts: the text before the curly bracket, and the text inside it.

### Step 4: Decoding the Prefix
The first part is `REVEU0VD`. The string ends with uppercase letters and numbers, which is a common indicator of **Base64** encoding. 

We can decode this using an online tool like CyberChef, or directly in the terminal:
```bash
echo "REVEU0VD" | base64 -d
# Output: DEDSEC
```
So, the standard flag format prefix is `DEDSEC`.

### Step 5: Cracking the Hash
The second part, inside the brackets, is `340c926071e79ea3c8168726dfb067540e7fdf0476df52a2458ee2480af91ccb`. 

Looking at this string, it consists of numbers and lowercase letters (hexadecimal) and is exactly 64 characters long. A 64-character hex string is the hallmark of a **SHA-256** hash. 

Since this is an easy challenge, it's highly likely that the hash is of a common word or phrase. We can take this hash over to [CrackStation.net](https://crackstation.net/), a massive database of pre-computed password hashes. 

![crackstation](/images/crackstation.png)

Pasting the hash into CrackStation instantly yields a result:
* **Hash:** `340c926071e79ea3c8168726dfb067540e7fdf0476df52a2458ee2480af91ccb`
* **Type:** `sha256`
* **Result:** `venivedivici`

### Step 6: Assembling the Flag
Now we just put our decoded Base64 string and our cracked hash back together into the original format!

---

## 🚩 Flag
```text
DEDSEC{venivedivici}
```
