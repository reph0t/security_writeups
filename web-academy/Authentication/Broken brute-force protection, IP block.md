
**PortSwigger Web Security Academy — Authentication / Password-Based Login** 

Lab link: https://portswigger.net/web-security/authentication/password-based/lab-broken-bruteforce-protection-ip-block

---

## 1. Lab Objective

This lab is vulnerable due to a logic flaw in its password brute-force protection. To solve the lab, brute-force the victim's password, then log in and access their account page.

---

## 2. Environment / Tools

- **Burp Suite** — Community Edition
- **Burp Intruder** — Pitchfork attack, Resource Pool (max concurrent requests: 1)
- **Python 3** — used to generate aligned username/password payload lists
- Candidate password wordlist (`passwords.txt`)

---

## 3. Automation Script

The core of this lab's bypass relies on interleaving the victim's username (`carlos`) with the tester's own valid username (`wiener`) at a fixed ratio, so that a successful login resets the server's failed-attempt counter before it reaches the lockout threshold. Rather than build this list by hand, I wrote a short Python script to generate two aligned payload lists — one for `username`, one for `password` — ready to paste directly into Burp Intruder's Payloads panel.

```python
# Generate the username list: alternates "wiener" (own account) with "carlos" (target)
print("################ Usernames ################")
for i in range(150):
    if i % 3:
        print("carlos")
    else:
        print("wiener")

# Generate the password list: pairs each "wiener" entry with the known-good
# password ("peter"), and each "carlos" entry with the next candidate password
print("################ Passwords ################")
with open('passwords.txt', 'r') as f:
    lines = f.readlines()

i = 0
for pwd in lines:
    if i % 3:
        print(pwd.strip('\n'))
    else:
        print("peter")
        print(pwd.strip('\n'))
    i += 1
```

**Output:** the script prints two aligned lists — one of usernames, one of passwords — matched row-for-row, ready to be pasted into the two Intruder payload sets.

<img width="1312" height="619" alt="image" src="https://github.com/user-attachments/assets/3feb6b2f-88ed-4635-95bd-ea2d6c1a06fd" />

<img width="825" height="660" alt="image" src="https://github.com/user-attachments/assets/1d0abc7f-5fdb-429c-bd18-6b247cc3050d" />



---
## 4. Walkthrough

### Understanding the Defense

This lab enforces an IP-based lockout after 3 incorrect login attempts. However, the lockout counter resets whenever a successful login occurs from that IP — a flaw that can be exploited by interleaving genuine successful logins between guesses. The lab provides the victim's username (`carlos`) and the tester's own valid credentials (`wiener`); the goal is to brute-force `carlos`'s password without ever accumulating 3 consecutive failures.

### Capturing the Login Request

- I submitted an invalid username and password to generate the `POST /login` request and sent it to Burp Intruder.

### Configuring the Attack

- In Intruder, I set the attack type to **Pitchfork**, with two payload positions: `username` and `password`.

<img width="1276" height="502" alt="image" src="https://github.com/user-attachments/assets/817efae8-4395-4331-ab28-6c098289be05" />

- I opened the **Resource Pool** side panel and assigned the attack to a resource pool with **Maximum concurrent requests set to 1**. This ensures requests are sent one at a time, in strict order — critical here, since the interleaved "reset" logins must land in the correct sequence relative to the guesses, and concurrent requests could arrive out of order and break the pattern.

<img width="328" height="680" alt="image" src="https://github.com/user-attachments/assets/d9dede73-3f28-4f56-a8d6-ce030d6d4c07" />

- In the **Payloads** panel, I set **Payload set 1** (`username`) to the generated list, which alternates between `wiener` (my own account, appearing first and roughly every third entry) and `carlos` (the target, repeated across the remaining candidate attempts).

<img width="283" height="455" alt="image" src="https://github.com/user-attachments/assets/80f42660-24f9-40d3-a1f7-568e658783ac" />

- I set **Payload set 2** (`password`) to the generated list, aligned row-for-row with the usernames: `peter` (my own valid password) paired with each `wiener` entry, and each candidate password paired with the corresponding `carlos` entry.

<img width="286" height="463" alt="image" src="https://github.com/user-attachments/assets/f91a92a2-877f-4d4c-8bca-1cb652a750a5" />

### Running the Attack and Identifying the Password

- I started the attack.
- Once finished, I filtered out responses with a `200` status code and sorted the remaining results by username.
- Among the `carlos` entries, one request returned a `302` response — indicating a successful login. I noted the corresponding password from the Payload 2 column.

<img width="996" height="228" alt="image" src="https://github.com/user-attachments/assets/ad22a220-a60e-454c-85f2-64d918f856f0" />

### Solving the Lab

- I logged in to Carlos's account using the identified password and successfully accessed his account page, solving the lab.

---

## 5. Root Cause / Vulnerability Explanation

The application's brute-force protection locks an IP address after 3 consecutive failed login attempts — but the failed-attempt counter is reset whenever a _successful_ login occurs from that same IP. This logic assumes that a successful login is only ever performed by a legitimate user, but it doesn't account for an attacker who also has access to _any_ valid account (their own) on the same system.

By interleaving guesses against the victim's account with periodic logins to their own account, an attacker resets the counter before it reaches the lockout threshold — effectively neutralizing the IP-block defense entirely, regardless of how many total guesses are made over time.

---

## 6. Result

Successfully bypassed the IP-based brute-force lockout by interleaving valid logins to a known account with password guesses against the target account (`carlos`), using a Python script to generate correctly aligned payload lists and a single-threaded Pitchfork attack in Burp Intruder to preserve request ordering. Identified the correct password via the `302` status code, logged in, and accessed the account page.

---

## 7. Remediation

- Do **not** reset the failed-attempt counter based on any successful login from the same IP — track failed attempts **per target account**, not per source IP, so that a successful login to an unrelated account has no effect on another account's lockout state.
- Consider combining **per-account lockout** with **per-IP rate limiting** as independent, non-interacting controls — a flaw in the logic connecting the two should never be able to fully cancel out the other.
- Enforce **CAPTCHA** or **progressive delays** after repeated failures, which are harder to reset with a single well-timed success.
- Monitor and alert on **anomalous login patterns** from a single IP (e.g., frequent alternation between successful and failed logins), which can indicate exactly this type of bypass attempt in progress.

---

_Lab status: Solved ✅_
**Results:**

