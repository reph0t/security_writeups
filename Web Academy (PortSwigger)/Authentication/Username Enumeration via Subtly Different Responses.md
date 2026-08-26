**PortSwigger Web Security Academy — Authentication / Password-Based Login** 

Lab link: https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-subtly-different-responses

---

## 1. Lab Objective

This lab is subtly vulnerable to username enumeration. To exploit this vulnerability and solve the lab, enumerate a valid username, brute-force this user's password, then access their account page.

---

## 2. Environment / Tools

- **Burp Suite** — Community Edition
- **Burp Intruder** — Sniper attack, Simple list payload type
- **Grep - Extract** (Intruder Settings) — to isolate the error message text for comparison
- Candidate username and password wordlists

---

## 3. Walkthrough

### Capturing the Login Request

- With Burp running, I submitted an invalid username and password to generate a `POST /login` request.
- I highlighted the `username` parameter in the request and sent it to Burp Intruder.

### Configuring the Enumeration Attack

- In Intruder, the `username` parameter was automatically marked as a payload position.
- In the **Payloads** side panel, I selected the **Simple list** payload type and added the list of candidate usernames.

### Isolating the Error Message with Grep - Extract

This lab's error message is visually identical across every response, so comparing by eye or by length alone isn't reliable enough on its own. To make the subtle difference easier to spot:

- I opened the **Settings** tab in Intruder and, under **Grep - Extract**, clicked **Add**.
- In the dialog, I scrolled through a sample response to find the error message `Invalid username or password.` and highlighted its text content. Burp automatically configured the extraction settings based on the highlighted text.
- I clicked **OK**, then started the attack.

<img width="492" height="452" alt="image" src="https://github.com/user-attachments/assets/90ecc0dc-eed3-4014-ba6d-7af0bb536fcc" />

### Identifying the Valid Username

- Once the attack finished, the results table included an additional column showing the extracted error message text for each response.
- Sorting by this column revealed that one entry was **subtly different** from the rest.
- Looking closer, the odd-one-out response contained a **typo**: instead of ending in a full stop/period, it had a **trailing space**. This is a classic example of a difference that's invisible when rendered on the page but detectable at the raw response level.
- I noted the username associated with this response.

<img width="1418" height="220" alt="image" src="https://github.com/user-attachments/assets/e24ce6b8-361d-4218-b17d-463f3f17af5f" />

<img width="1150" height="464" alt="image" src="https://github.com/user-attachments/assets/216383a5-bcbd-4139-8a00-9008a5955ce1" />

### Brute-Forcing the Password

- Back in the Intruder tab, I set the `username` parameter to the identified username and added a payload position to the `password` parameter instead:
    
    ```
    username=puppet&password=§invalid-password§
    ```

<img width="1111" height="526" alt="image" src="https://github.com/user-attachments/assets/d0fad88d-1c67-42a4-a2a0-1217035b7e18" />

- In the **Payloads** side panel, I replaced the username list with the list of candidate passwords and started the attack.

<img width="456" height="451" alt="image" src="https://github.com/user-attachments/assets/9d655bd6-eb3e-4325-901e-5b4c44586fc9" />

- Once finished, I checked the **Status** column and found one request that returned a `302` response instead of the usual `200` — indicating a successful login. I noted the corresponding password.
    

<img width="1137" height="676" alt="image" src="https://github.com/user-attachments/assets/1423d138-243e-4695-85ea-2175380d5aa8" />

### Solving the Lab

- I logged in using the identified username and password, and successfully accessed the user account page, solving the lab.

<img width="1789" height="915" alt="image" src="https://github.com/user-attachments/assets/6cf40647-d6dd-4194-942a-03e9fbb8ae14" />

---

## 4. Root Cause / Vulnerability Explanation

Unlike the previous lab, this application's error message _appears_ identical for both invalid-username and invalid-password cases — a naive attempt at fixing the enumeration flaw. However, the fix was incomplete: a **typo in the error-handling logic** (a trailing space instead of a period) produces a response that is byte-for-byte different from the standard message, even though it renders identically in the browser.

This demonstrates an important lesson: **generic error messages are only a real fix if they are truly identical at the byte level.** Any inconsistency — even one invisible to the human eye — can still be detected programmatically (via response length, Grep - Extract, or diffing tools) and exploited exactly like an overtly different message would be.

---

## 5. Result

Successfully identified a valid username by extracting and comparing the error message text across all responses using Burp Intruder's Grep - Extract feature, spotting a single-character discrepancy (trailing space vs. period) invisible in the rendered page. Brute-forced the corresponding password via the `302` status code, logged in, and accessed the account page.

---

## 6. Remediation

- Ensure error messages are **byte-for-byte identical** for both invalid-username and invalid-password cases — not just visually similar. Generate the message from a single shared code path rather than two separate strings that can drift out of sync.
- Add **automated testing** (e.g., response diffing) to CI/CD pipelines to catch unintended discrepancies in security-relevant response text before deployment.
- Ensure **response length and timing** are also consistent, since either can independently leak the same information even if the message text is perfectly identical.
- Implement brute-force protections (rate limiting, lockout, CAPTCHA) as defense-in-depth, so that even if enumeration succeeds, password brute-forcing is still meaningfully hindered.

---

_Lab status: Solved ✅_

> [!note] Note
> _It's also possible to brute-force this login using a single Cluster Bomb attack (usernames × passwords simultaneously). However, enumerating a valid username first — as done above — is significantly more efficient, since it turns a two-variable brute-force problem into two much smaller single-variable ones.

