---
title: "All Roads Lead to Rome"
date: 2024-05-20
draft: false
---

**Category:** OSINT  
**Difficulty:** Easy  
**Author:** Saurav Sreejith 
<!--more-->

## 📝 Challenge Description

![TICK TOCK](/images/tick-tock.jpg)

Participants were provided with a single image containing the following cryptic text:
> **"ALL ROADS LEAD TO 6JWRGWPX+56"**

The goal was to track down the exact location, follow the breadcrumbs left by the creator, and retrieve the hidden flag.

---

## 🕵️‍♂️ Solution Walkthrough

### Step 1: Identifying the Code
The first step is analyzing the string `6JWRGWPX+56`. In the OSINT world, strings formatted with a `+` in this specific alphanumeric pattern are highly recognizable as **Google Plus Codes** (Open Location Codes). 

![PLUS CODE](/images/pluscode.png)

**Rabbit Hole Warning:** A common mistake participants made here was pasting this code directly into the standard Google Maps search bar. Depending on the region, Google Maps can sometimes struggle to pinpoint the exact location for shorter Plus Codes, leading players to a general area rather than a specific building. 

To get the exact pinpoint, the best approach is to use the official Plus Codes website: [plus.codes](https://plus.codes/). 

### Step 2: Investigating the Location
Entering `6JWRGWPX+56` into the Plus Codes site drops a pin directly onto an apartment building in Trivandrum named **"Rome by Sree Dhanya"**. 

This perfectly matches the challenge name, confirming we are looking at the right place. But there's no flag on the map. Where do we go from here? 

### Step 3: Digging into the Reviews
When doing OSINT on a physical location, checking the Google Maps reviews is a standard procedure. Navigating to the Google Maps page for *Sree Dhanya Rome* and sorting through the reviews, we stumble upon a highly suspicious 5-star review left by a user named **Tristan Morrison**:

![review](/images/review.png)

> *"Sree Dhanya Rome is a beautiful apartment project with a unique and eye-catching style of architecture in a peaceful community in Trivandrum. The way it has been built truly matches its name and the monument it represents. For the **paste** few months I have **bin** looking for a property with this level of artistry. Thank you **architect_arun_67** for telling me about this place. The quality of the construction is outstanding, and the amenities are top-notch. Kudos to the engineering team!"*

### Step 4: Connecting the Dots
Reading closely, the review contains intentional grammatical errors and a specific username:
* "past" is misspelled as **paste**
* "been" is misspelled as **bin**
* It mentions a specific user: **architect_arun_67**

Putting **paste** + **bin** together, it's clear the author is pointing us toward [Pastebin.com](https://pastebin.com/). 

*(Note: Even if participants missed the spelling hints, searching the username `architect_arun_67` using an OSINT username enumeration tool like Sherlock or WhatsMyName would also lead them directly to the Pastebin account).*

### Step 5: Retrieving the Flag
Going to Pastebin and searching for the user `architect_arun_67`, we find their profile. They have a single public paste titled `howdidyoufindme` located at the following URL:
🔗 `https://pastebin.com/iUtmHL0R`

Opening the paste reveals the final flag!

---

## 🚩 Flag
```text
DEDSEC{Al1_R0ad5_Le4$_H3r3}
```
