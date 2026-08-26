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

<img width="492" height="452" alt="image" src="https://github.com/user-attachments/assets/2f915154-9114-4d5f-8cbf-ff7e28cbeaaf" />

### Identifying the Valid Username

- Once the attack finished, the results table included an additional column showing the extracted error message text for each response.
- Sorting by this column revealed that one entry was **subtly different** from the rest.
- Looking closer, the odd-one-out response contained a **typo**: instead of ending in a full stop/period, it had a **trailing space**. This is a classic example of a difference that's invisible when rendered on the page but detectable at the raw response level.
- I noted the username associated with this response.

<img width="1418" height="220" alt="image" src="https://github.com/user-attachments/assets/238b4101-4df3-4920-b475-605b80a1b147" />

<img width="1150" height="464" alt="image" src="https://github.com/user-attachments/assets/d4f01650-c7a5-4aa9-b089-81623158a2b1" />

### Brute-Forcing the Password

- Back in the Intruder tab, I set the `username` parameter to the identified username and added a payload position to the `password` parameter instead:
    
    ```
    username=puppet&password=§invalid-password§
    ```

<img width="1111" height="526" alt="image" src="https://github.com/user-attachments/assets/58d67bdf-5422-4172-a173-daf7a1033e93" />

- In the **Payloads** side panel, I replaced the username list with the list of candidate passwords and started the attack.

<img width="456" height="451" alt="image" src="https://github.com/user-attachments/assets/fad73ccb-8ab6-40d2-9d53-78d035e1d886" />
- Once finished, I checked the **Status** column and found one request that returned a `302` response instead of the usual `200` — indicating a successful login. I noted the corresponding password.
    

<img width="1137" height="676" alt="image" src="https://github.com/user-attachments/assets/360a67a6-9245-4ad1-9e94-cfd823d0beb9" />

### Solving the Lab

- I logged in using the identified username and password, and successfully accessed the user account page, solving the lab.


<img width="1789" height="915" alt="image" src="https://github.com/user-attachments/assets/29e7c793-f7a0-4628-bb83-77bfdd3383ac" />

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


