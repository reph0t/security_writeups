**PortSwigger Web Security Academy — Authentication / Password-Based Login** 

Lab link: [https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses)
## 1. Lab Description

This lab is vulnerable to username enumeration and password brute-force attacks. It has an account with a predictable username and password, which can be found in the following wordlists:

- [Candidate usernames](https://portswigger.net/web-security/authentication/auth-lab-usernames)
- [Candidate passwords](https://portswigger.net/web-security/authentication/auth-lab-passwords)

**Goal:** Enumerate a valid username, brute-force that user's password, then access their account page.

---
## 2. Environment / Tools

- **Burp Suite** — Community Edition
- **Burp Intruder** — Sniper attack, Simple list payload type
- Candidate username and password wordlists (linked above)

---
# 3.  Walkthrough

### Logging in
In this lab we are given a link to a website where we can use username enumeration and password brute-force attacks.

<img width="1215" height="721" alt="image" src="https://github.com/user-attachments/assets/772bd61a-5a43-497f-b063-c0ae2006b9b1" />

We then access `My Account`, which directs to a login page

<img width="956" height="449" alt="image" src="https://github.com/user-attachments/assets/64b65ced-a24c-47cf-a515-0c6d62469e70" />

### Capturing the Login Request

- I entered an invalid username and password to trigger and capture the `POST /login` request.
- I went to **Proxy > HTTP history** and located the `POST /login` request.

<img width="1295" height="352" alt="image" src="https://github.com/user-attachments/assets/4c0268fc-30ac-4f3c-81af-24b59fef85f8" />

- I sent the request to **Burp Intruder**.

<img width="787" height="545" alt="image" src="https://github.com/user-attachments/assets/2a47d0af-1fb3-43b1-bbf6-717933c4c80c" />

- ### Enumerating the Username

- In Burp Intruder, the `username` parameter was automatically marked as a payload position by Burp's auto-detection of likely injectable fields in the request body. This position is indicated by two `§` symbols, e.g. `username=§user§&password=password`.
- I kept the `password` field static for this step, since I was only testing which usernames triggered a different server response — the password value itself didn't matter yet.
- I confirmed the **Sniper** attack type was selected.
- In the **Payloads** side panel, I set the payload type to **Simple list**.
- Under Payload configuration, I pasted the list of candidate usernames.

<img width="1432" height="755" alt="image" src="https://github.com/user-attachments/assets/4a668d74-8042-4fb0-b47f-962353e716e4" />

<img width="451" height="456" alt="image" src="https://github.com/user-attachments/assets/8b78f2d2-df26-41db-9bc1-d31944cdaf69" />

- I clicked **Start Attack**. The attack ran in a new window.
- Once finished, I examined the **Length** column in the results table (sorting by clicking the column header). One entry stood out with a noticeably longer response length than the rest.
- I compared the response body for that entry against the others: most responses contained the message `Invalid username`, while the standout response said `Incorrect password` instead — confirming that username was valid. I noted this username for the next step.


<img width="1125" height="319" alt="image" src="https://github.com/user-attachments/assets/a8d1380d-fa68-4f42-b725-39f68d961a4f" />

<img width="614" height="376" alt="image" src="https://github.com/user-attachments/assets/99a61777-e2a5-4aaa-afbe-9ba0b7f339dc" />

- ### Brute-Forcing the Password

- Back in the Intruder tab, I cleared the existing `§` markers, set the `username` parameter to the confirmed valid username, and added a payload position to the `password` parameter instead:

```
  username=identified-user&password=§invalid-password§
```

- In the **Payloads** side panel, I replaced the username list with the list of candidate passwords, then clicked **Start attack**.

<img width="435" height="436" alt="image" src="https://github.com/user-attachments/assets/84058a33-bd43-48ef-aca8-87f8416bc630" />

- Once finished, I checked the **Status** column. Every request returned a `200` status code except for one, which returned a `302` — a redirect typically indicating a successful login. I noted the corresponding password.

<img width="1170" height="349" alt="image" src="https://github.com/user-attachments/assets/8958fc64-ee70-4f46-8b84-722d757aae7f" />

<img width="746" height="247" alt="image" src="https://github.com/user-attachments/assets/39146b04-84af-4a7a-ac1d-7aa1e29e919f" />


### Solving the Lab

- I logged in using the identified username and password combination and successfully accessed the user account page, solving the lab.

--- 
## 4. Root Cause / Vulnerability Explanation

The application returns **different error messages** depending on whether the _username_ or the _password_ was incorrect — `Invalid username` versus `Incorrect password`. This behavioral difference (also reflected in response length) allows an attacker to determine whether a given username exists **without needing the correct password at all**.

This is a textbook **username enumeration** vulnerability. It significantly reduces the effort required for a full account compromise, because it lets an attacker split the attack into two much easier stages:

1. Enumerate valid usernames using generic/incorrect passwords.
2. Brute-force only the password for a _confirmed_ valid username.

Without this flaw, an attacker would need to guess valid username/password **pairs** simultaneously, which is exponentially harder.

A secure implementation should return an **identical, generic error message** (e.g., "Invalid username or password") regardless of which field was incorrect, and ideally keep response length/timing consistent between the two cases as well.

---

## 5. Result

Successfully identified a valid username via response-message differences, brute-forced the corresponding password via HTTP status code differences (`200` vs `302`), logged in, and accessed the account page — confirming both the username enumeration vulnerability and the lack of protection against password brute-forcing.

---

## 6. Remediation

- Return a **generic, identical error message** for both invalid username and invalid password cases (e.g., "Invalid username or password").
- Ensure **response length and timing are consistent** regardless of whether the username or password was the incorrect field.
- Implement **brute-force protection** such as account lockout, rate limiting, or CAPTCHA after repeated failed attempts, ideally combined with monitoring for anomalous login patterns.
- Consider **multi-factor authentication (MFA)** to reduce the impact even if credentials are compromised via enumeration/brute-force.
