# WEB_CTF_2025

**Room Creator:** Sachintha K.D.P

**Short description**
This room is designed for second-year university students to practice web security in a safe environment. It was created for a university assignment and contains progressive challenges mapped to the OWASP Top 10. The lab focuses on practical, logic-based vulnerabilities (no brute forcing required).

---

## Table of Contents
1. [Overview](#overview)
2. [Prerequisites](#prerequisites)
3. [How to use this repository](#how-to-use-this-repository)
4. [Tasks / Challenges](#tasks--challenges)
   - [Task 1 — Start the Machine & Identify the Challenge Portal](#task-1-—-start-the-machine--identify-the-challenge-portal)
   - [Task 2 — Broken Access Control — Unauthorized Data Access](#task-2-—-broken-access-control-—-unauthorized-data-access)
   - [Task 3 — Stored Cross-Site Scripting (XSS) — Phishing Redirect](#task-3-—-stored-cross-site-scripting-xss-—-phishing-redirect)
   - [Task 4 — Malicious URL Injection (Path Traversal)](#task-4-—-malicious-url-injection-path-traversal)
   - [Task 5 — Malicious File Uploads](#task-5-—-malicious-file-uploads)
5. [Flags / Submission](#flags--submission)
6. [Hints & Learning Objectives](#hints--learning-objectives)
7. [Responsible Use & Safety](#responsible-use--safety)
8. [License](#license)

---

## Overview
`WEB_CTF_2025` is a hands-on CTF-style room that walks students through common web vulnerabilities. Each level gradually increases in complexity and maps to OWASP Top 10 categories. The goal is to teach identification, exploitation (in a controlled lab), and mitigation concepts.

This repository contains a polished writeup students can use as a walkthrough or instructor reference. It does **not** host the vulnerable service — the room runs on a TryHackMe-style VM.

---

## Prerequisites
- Basic Linux knowledge and ability to start/connect to a VM.
- A browser and optionally intercepting proxy (Burp Suite, OWASP ZAP) for analysis.
- Access to the lab environment (e.g., TryHackMe machine or instructor-provided VM).
- For some tasks, use an isolated environment (incognito/private browsing) to avoid redirect loops.

---

## How to use this repository
1. Clone or browse this repo.
2. Follow the task list below while connected to the lab VM.
3. Use the hints provided if you get stuck. Instructor or TA may set the exact flag values per lab deployment.

---

## Tasks / Challenges

### Task 1 — Start the Machine & Identify the Challenge Portal
**Objective**: Deploy the VM and confirm access to the challenge portal by identifying the header title on the index page.

**Quick steps**:
1. Start the provided VM in the TryHackMe interface (or lab host).
2. Copy the machine IP and open it in your browser: `http://<IP>`.
3. Verify the index page loads and note the large header text at the top of the page.

**Question**: What is the header name shown on the index page?  
**Answer**: `WEB_CTF_2025`

---

### Task 2 — Broken Access Control — Unauthorized Data Access
**Objective**: Learn what Broken Access Control is and perform a simple lab demonstrating client-side logic flaws that expose sensitive data.

**Scenario**: A simulated banking portal allows users to view profiles. The app is intended to restrict viewing other users' data and admin-only pages, but carelessly relies on client-side controls and guessable identifiers.

**Provided credentials**: `alice` / `alice123`

**Quick path & hints**:
- Log in as `alice`.
- Inspect requests made when viewing profile pages (observe user IDs or filename patterns in URLs).
- Try manipulating the identifier (query parameter, path segment, or JSON body) to access Bob's NIC or admin panels.
- Hidden form fields or direct requests to `admin.php` may be accessible due to insufficient server-side authorization checks.

**Questions**:
- Q1: Can you access Bob's NIC number by manipulating the request?
- Q2: Can you access `admin.php` without admin privileges?
- Q3: Can you modify your role and successfully log in as an admin?

**Notes**:
- This lab demonstrates logic-based authorization failures rather than brute force.
- Use your browser devtools or an intercepting proxy to observe and modify requests.

---

### Task 3 — Stored Cross-Site Scripting (XSS) — Phishing Redirect
**Objective**: Understand stored XSS and how unsanitized input in persistent areas (comments) can be abused to perform phishing-style redirects.

**Scenario**: The comment section saves submitted comments and displays them without sanitization. Injected JavaScript executes in other users' browsers and can redirect victims to a convincing fake page.

**Quick steps**:
1. Use an incognito tab for testing (to avoid persistent redirect loops in your session).
2. Visit the comment section and test if simple payloads execute, e.g. `<script>alert(1)</script>`.
3. Submit a redirect payload: `<script>window.location.href='fake.php';</script>`.
4. When a user visits the page, the script will redirect them to `fake.php`. The fake page simulates a phishing page that includes a prize amount (the flag).

**Important technical note**: The server app may automatically rewrite occurrences of `fake.php` to include a session token. Do **not** try to open `fake.php` directly — it is intentionally blocked.

**Question**:
- Q1: What is the prize money (flag) shown on the phishing page after the redirect?

**Hints**:
- Use simple `window.location.href` redirects.
- If your browser is immediately redirected incorrectly, open a fresh incognito session to recover and try again.

---

### Task 4 — Malicious URL Injection (Path Traversal)
**Objective**: Learn about path traversal and bypass techniques when straightforward `../` sequences are filtered.

**Scenario**: The profile picture endpoint loads files based on a user-supplied parameter. The server attempts to remove `../` but is not fully canonicalizing or handling obfuscated traversal inputs.

**Quick steps**:
1. Log in as `alice` (`alice` / `alice123`) and view the profile picture request (look at the `img` parameter or the image URL).
2. Try to fetch `secrets/secret.key` by manipulating the parameter to traverse directories.
3. If `../` is filtered, try alternative encodings or obfuscations (URL-encoding, double-encoding, or dot-obfuscations like `....//`) to bypass naive filters.

**File layout (reference)**:
```
level3/
├─ uploads/
│  └─ profile_pics/
│     ├─ user_1_profile_pic.png
│     └─ user_2_profile_pic.png
└─ secrets/
   └─ secret.key  # flag file
```

**Questions**:
- Q1: What bypass method allowed you to read `secret.key`?
- Q2: What is the flag contained in `secret.key`?

---

### Task 5 — Malicious File Uploads
**Objective**: Understand malicious file upload vulnerabilities and how insecure file validation can lead to remote code execution / data disclosure.

**Scenario**: The profile picture upload only checks file extensions (not content). Attackers can upload a file that looks like an image but contains PHP code; when accessed, the server executes that code.

**Quick steps**:
1. Go to level 4 (you will be automatically logged in as Bob in this level).
2. Use the Update Profile Picture feature to upload a crafted file whose filename looks like an image (e.g., `shell.png`) but contains PHP code.
3. After uploading, right-click the displayed profile picture and open it in a new tab. If the server executes the PHP embedded in the uploaded file, it will display the output.

**Example test payloads**:
- Basic test: `<?php echo "check......"; ?>`
- Read the provided lab flag: `<?php echo file_get_contents(__DIR__ . '/lab4flag'); ?>`
- Read the server password file using traversal: `<?php echo file_get_contents('/etc/password'); ?>` (adjust path as required by the lab environment)

**File layout (reference)**:
```
/var/www/html/level4/
├─ uploads/
│  └─ profile_pics/
│     ├─ user_1_profile_pic.png
│     ├─ user_2_profile_pic.png
│     └─ lab4flag
/etc
└─ password
```

**Questions**:
- Q1: Enter the flag you obtained from `lab4flag`.
- Q2: What is the password you retrieved from the `password` file using your uploaded payload?

**Notes**:
- The upload form only validates the extension. Craft the content of your file carefully so it passes the extension check.

---

## Flags / Submission
- Instructor or lab host will supply the exact flag values for your deployment. This walkthrough documents where to find them and how to retrieve them during the lab.
- When submitting answers on the TryHackMe platform, submit only the flag strings as required by that task.

---

## Hints & Learning Objectives
- Understand why server-side authorization checks are necessary — never rely solely on client-side controls.
- Recognize that naive string filtering (e.g. removing `../`) is insufficient; canonicalize and validate paths on the server.
- Stored XSS is powerful because it executes in the context of victim browsers — escaping and context-aware sanitization is critical.
- File uploads must be validated both by extension and by content type and stored in non-executable directories when possible.

---

## Responsible Use & Safety
This repository is for education and defensive learning. Do **not** deploy or run attacks against systems you do not own or have explicit permission to test. Always follow your institution's and local laws and responsible disclosure policies.

If you plan to extend this room or publish it publicly, remove any secrets and ensure the lab uses ephemeral VMs and isolated networks.

---

## License
This writeup is provided for educational purposes. You may reuse and adapt it for teaching under the Creative Commons Attribution-ShareAlike (CC BY-SA) license. Modify the license if your institution requires a different one.

---

*Prepared for university assignment — WEB_CTF_2025.*

