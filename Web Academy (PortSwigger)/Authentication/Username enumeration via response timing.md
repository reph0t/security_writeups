**PortSwigger Web Security Academy — Authentication / Password-Based Login** 

Lab link: https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-response-timing

---

## 1. Lab Objective

This lab is vulnerable to a username enumeration technique using timing attacks. It also has a brute-force protection mechanism that can be bypassed by manipulating HTTP request headers. To solve the lab, enumerate a valid username, bypass the IP-based brute-force protection, brute-force this user's password, then access their account page.

---

## 2. Environment / Tools

- **Burp Suite** — Community Edition
- **Burp Intruder** — Sniper attack (username enumeration), Pitchfork attack (password brute-force + IP header bypass)
- Candidate username and password wordlists

---

## 3. Walkthrough

### Capturing the Login Request

- I submitted an invalid username and password to generate a `POST /login` request and sent it to Burp Intruder.

### Enumerating the Username via Response Timing

Unlike the previous two labs, this application returns an **identical response body and length** regardless of whether the username is valid — there's no visible or byte-level content difference to compare. The only distinguishing signal is how long the server takes to respond.

- In the Positions tab, I marked the `username` parameter as the payload position.
- Critically, I set the `password` parameter to an **excessively long static string** (several thousand characters). This is the key to making the timing difference measurable: if the application only checks the password hash after confirming the username exists, a very long password exaggerates the processing time for valid usernames specifically, while invalid usernames get rejected immediately regardless of password length.

<img width="961" height="644" alt="image" src="https://github.com/user-attachments/assets/5beab137-969f-44f9-8567-223fcea9c78d" />


- In the Payloads panel, I loaded the candidate usernames list as a Simple list, and started the attack.
- Once finished, I enabled and sorted by the **Response received** column (measured in milliseconds).
- One username stood out with a noticeably higher response time than the rest of the list, and — as an additional confirming signal — also showed a different value in the **Length** column compared to every other entry. Having two independent signals (timing _and_ length) agree made this a reliable identification rather than a guess based on a marginal timing gap.
- I noted this username (`arkansas`) as the valid one.

<img width="949" height="165" alt="image" src="https://github.com/user-attachments/assets/56fc8152-3551-4d86-a517-a59c908fa181" />

### Bypassing IP-Based Brute-Force Protection

When I attempted to brute-force the password for the identified username, the application's brute-force protection blocked repeated attempts based on IP address. The lab hint indicated this could be bypassed by manipulating HTTP headers.

- I added a new header to the request: `X-Forwarded-For: §1§`, marking the value as a second payload position. Applications sitting behind a proxy or load balancer sometimes trust this header to determine the "real" client IP for rate-limiting purposes — but since it's just a client-supplied header, it's trivially spoofable.
- Because I now had **two independent payload positions** (the password guess and the fake IP) that needed to advance together, one pairing per request, I switched the attack type from Sniper to **Pitchfork**.

<img width="943" height="102" alt="image" src="https://github.com/user-attachments/assets/71947773-3889-4960-9a1d-7144103e14e0" />

- **Payload set 1** (`password`): the candidate passwords wordlist, Simple list type.

<img width="322" height="457" alt="image" src="https://github.com/user-attachments/assets/fa5153e9-749c-4e33-b006-0199802cd770" />
- **Payload set 2** (`X-Forwarded-For`): the **Numbers** payload type, generating a unique sequential value per request (e.g., 1, 2, 3...) so each login attempt appeared to originate from a different IP address, preventing the block from ever accumulating enough failed attempts against a single source.
<img width="332" height="406" alt="image" src="https://github.com/user-attachments/assets/5137ba3a-7c0b-4bdf-8f7a-c39278f0e91a" />


### Brute-Forcing the Password

- With `username=arkansas` set as static and the two payload positions configured, I started the Pitchfork attack.

<img width="1339" height="836" alt="image" src="https://github.com/user-attachments/assets/db8042a3-8dc8-481a-b5da-0b3ede268cf1" />

- I checked the **Status code** column and found one request that returned `302` instead of the usual `200` — indicating a successful login. I noted the corresponding password.

<img width="1477" height="831" alt="image" src="https://github.com/user-attachments/assets/5aaa49b8-3173-403f-98cf-130373580dc2" />


### Solving the Lab

- I logged in using the identified username and password, and successfully accessed the user account page, solving the lab.

<img width="1716" height="837" alt="image" src="https://github.com/user-attachments/assets/3a0f03a8-3057-4bc7-89af-34fe83af46ed" />

---

## 4. Root Cause / Vulnerability Explanation

This lab combines two separate weaknesses:

1. **Timing-based username enumeration:** Even though the application returns a generic, identical error message and length for invalid logins, the underlying processing time still differs based on whether the username is valid — the app performs extra work (e.g., comparing the submitted password against a stored hash) only when the username exists. This timing side-channel leaks the same information a content-based enumeration flaw would, just through a different channel. It's a good example of why **content-matching alone is not sufficient** to fix enumeration — response time and length must also remain constant regardless of input validity.
    
2. **Trusting a client-controlled header for brute-force protection:** The application's rate-limiting/IP-blocking logic relies on the `X-Forwarded-For` header to identify the requesting client, rather than the actual TCP connection source or a header explicitly set by a trusted, known proxy. Since `X-Forwarded-For` is fully attacker-controlled, this allows trivial bypass of any protection built on top of it.
    

---

## 5. Result

Successfully identified a valid username (`arkansas`) using response-timing analysis (confirmed by an independent length discrepancy), bypassed the application's IP-based brute-force protection by spoofing the `X-Forwarded-For` header with a unique value per request via a Pitchfork attack, brute-forced the corresponding password using the `302` status code as the success indicator, logged in, and accessed the account page.

<img width="1477" height="831" alt="image" src="https://github.com/user-attachments/assets/6400350e-2dc5-48be-a872-7255fd953b96" />

---

## 6. Remediation

- Ensure **response time is constant** regardless of whether the username is valid — e.g., by always performing a password hash comparison (even against a dummy hash) rather than short-circuiting the check when the username doesn't exist.
- **Never trust client-supplied headers** like `X-Forwarded-For` for security-critical decisions (rate limiting, IP blocking, access control) unless the request is coming through a trusted, known proxy that is configured to strip and reset this header from untrusted external input.
- If IP-based protections are needed behind a proxy, derive the client IP from the proxy's own trusted connection metadata, not from a header the client can set directly.
- Layer brute-force protections — combine account-based lockout, CAPTCHA, and properly-sourced IP rate limiting — so that bypassing one mechanism doesn't fully defeat all protection.

---

_Lab status: Solved ✅_
