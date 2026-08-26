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

<img width="961" height="644" alt="image" src="https://github.com/user-attachments/assets/77cad2be-db68-4039-a09c-398f8fada7c8" />

- In the Payloads panel, I loaded the candidate usernames list as a Simple list, and started the attack.
- Once finished, I enabled and sorted by the **Response received** column (measured in milliseconds).
- One username stood out with a noticeably higher response time than the rest of the list, and — as an additional confirming signal — also showed a different value in the **Length** column compared to every other entry. Having two independent signals (timing _and_ length) agree made this a reliable identification rather than a guess based on a marginal timing gap.
- I noted this username (`arkansas`) as the valid one.

<img width="949" height="165" alt="image" src="https://github.com/user-attachments/assets/f5c1854e-461c-42c7-8399-936b64a9ee3b" />

### Bypassing IP-Based Brute-Force Protection

When I attempted to brute-force the password for the identified username, the application's brute-force protection blocked repeated attempts based on IP address. The lab hint indicated this could be bypassed by manipulating HTTP headers.

- I added a new header to the request: `X-Forwarded-For: §1§`, marking the value as a second payload position. Applications sitting behind a proxy or load balancer sometimes trust this header to determine the "real" client IP for rate-limiting purposes — but since it's just a client-supplied header, it's trivially spoofable.
- Because I now had **two independent payload positions** (the password guess and the fake IP) that needed to advance together, one pairing per request, I switched the attack type from Sniper to **Pitchfork**.

<img width="943" height="102" alt="image" src="https://github.com/user-attachments/assets/ce1a2841-c43f-4a15-94d0-5aaf1d80f7cc" />

- **Payload set 1** (`password`): the candidate passwords wordlist, Simple list type.

<img width="322" height="457" alt="image" src="https://github.com/user-attachments/assets/a6c71ce1-1462-499b-bfdb-70683816cd4e" />


- **Payload set 2** (`X-Forwarded-For`): the **Numbers** payload type, generating a unique sequential value per request (e.g., 1, 2, 3...) so each login attempt appeared to originate from a different IP address, preventing the block from ever accumulating enough failed attempts against a single source.


<img width="332" height="406" alt="image" src="https://github.com/user-attachments/assets/83bb8d1a-994a-4a05-a70d-8f41bd48b994" />


### Brute-Forcing the Password

- With `username=arkansas` set as static and the two payload positions configured, I started the Pitchfork attack.


<img width="1339" height="836" alt="image" src="https://github.com/user-attachments/assets/e0d97440-2496-45d9-ad2d-9d4c3539d505" />



- I checked the **Status code** column and found one request that returned `302` instead of the usual `200` — indicating a successful login. I noted the corresponding password.


<img width="1477" height="831" alt="image" src="https://github.com/user-attachments/assets/aa8c50d4-07ae-4745-a785-6e6b53b25ce6" />


### Solving the Lab

- I logged in using the identified username and password, and successfully accessed the user account page, solving the lab.

<img width="1716" height="837" alt="image" src="https://github.com/user-attachments/assets/c93c8da8-5e9e-42bb-8a1e-b13398068f41" />

---

## 4. Root Cause / Vulnerability Explanation

This lab combines two separate weaknesses:

1. **Timing-based username enumeration:** Even though the application returns a generic, identical error message and length for invalid logins, the underlying processing time still differs based on whether the username is valid — the app performs extra work (e.g., comparing the submitted password against a stored hash) only when the username exists. This timing side-channel leaks the same information a content-based enumeration flaw would, just through a different channel. It's a good example of why **content-matching alone is not sufficient** to fix enumeration — response time and length must also remain constant regardless of input validity.
    
2. **Trusting a client-controlled header for brute-force protection:** The application's rate-limiting/IP-blocking logic relies on the `X-Forwarded-For` header to identify the requesting client, rather than the actual TCP connection source or a header explicitly set by a trusted, known proxy. Since `X-Forwarded-For` is fully attacker-controlled, this allows trivial bypass of any protection built on top of it.
    

---

## 5. Result

Successfully identified a valid username (`arkansas`) using response-timing analysis (confirmed by an independent length discrepancy), bypassed the application's IP-based brute-force protection by spoofing the `X-Forwarded-For` header with a unique value per request via a Pitchfork attack, brute-forced the corresponding password using the `302` status code as the success indicator, logged in, and accessed the account page.

<img width="1477" height="831" alt="image" src="https://github.com/user-attachments/assets/22996b45-d68c-40b8-b0e2-6f9c75c2b19f" />

---

## 6. Remediation

- Ensure **response time is constant** regardless of whether the username is valid — e.g., by always performing a password hash comparison (even against a dummy hash) rather than short-circuiting the check when the username doesn't exist.
- **Never trust client-supplied headers** like `X-Forwarded-For` for security-critical decisions (rate limiting, IP blocking, access control) unless the request is coming through a trusted, known proxy that is configured to strip and reset this header from untrusted external input.
- If IP-based protections are needed behind a proxy, derive the client IP from the proxy's own trusted connection metadata, not from a header the client can set directly.
- Layer brute-force protections — combine account-based lockout, CAPTCHA, and properly-sourced IP rate limiting — so that bypassing one mechanism doesn't fully defeat all protection.

---

_Lab status: Solved ✅_
