**PortSwigger Web Security Academy — File Path Traversal** 

Lab link: https://portswigger.net/web-security/file-path-traversal/lab-simple

---

## 1. Lab Objective

This lab contains a path traversal vulnerability in the display of product images. To solve the lab, retrieve the contents of the `/etc/passwd` file on the server.

---

## 2. Environment / Tools

- **Burp Suite** — Community Edition
- **Burp Proxy** — HTTP history, filtered to show image requests
- **Burp Repeater** — to modify and resend the image request

---

## 3. Walkthrough

### Locating Image Requests

- I browsed the shopping application and used Burp's Proxy HTTP history filter to isolate image requests, making it easier to identify the relevant traffic among the noise of CSS, fonts, and other assets.

<img width="1370" height="602" alt="image" src="https://github.com/user-attachments/assets/fa917fbf-8bc5-4464-8e7d-e6824b97a72f" />

- I intercepted the `GET` requests for the product images, each following the pattern `/image?filename=<name>.jpg`.

<img width="1127" height="385" alt="image" src="https://github.com/user-attachments/assets/7376369a-dff6-488b-9b50-5cbc42bbde38" />

### Testing for Path Traversal

- I selected one of the image requests, right-clicked, and sent it to **Burp Repeater**.
- The request's `filename` parameter (`GET /image?filename=<name>.jpg`) was the injection point to test. Since the images are known to be stored in `/var/www/images/`, the goal was to traverse upward out of that directory to reach the filesystem root, then access `/etc/passwd`.
- I modified the `filename` parameter to:
    
    ```
    ../../../etc/passwd
    ```
    
    Each `../` sequence instructs the filesystem to step up one directory level. Three consecutive `../` sequences step up from `/var/www/images/` past the web root and up to the filesystem root (`/`), from which `etc/passwd` resolves correctly.
- I sent the modified request and received a `200 OK` response containing the full contents of `/etc/passwd`, confirming the traversal was successful.

<img width="622" height="608" alt="image" src="https://github.com/user-attachments/assets/03bf1a51-6fd0-49cf-a3b5-a80c8958754a" />

### Solving the Lab

- By modifying the relative path in the `filename` parameter, I was able to escape the intended `/var/www/images/` directory and read `/etc/passwd` directly from the server's filesystem, solving the lab.

---

## 4. Root Cause / Vulnerability Explanation

The root cause of this vulnerability is that the application performs **no input validation or sanitization** on the `filename` parameter before using it to build a filesystem path. The server simply concatenates the user-supplied value onto its base image directory (`/var/www/images/`) and reads whatever file results — including sequences like `../` that are valid filesystem syntax but were never intended to be accepted from user input.

This is fundamentally an **input validation failure**, not an authentication or access-control issue — the application never checks whether the resulting file path stays within the intended directory. As a result, any file readable by the web server process can potentially be retrieved, not just images.

---

## 5. Result

Successfully exploited the path traversal vulnerability in the `filename` parameter of the image-loading endpoint to read the contents of `/etc/passwd`, confirming arbitrary file read access on the server's filesystem.

---

## 6. Remediation

- **Validate and sanitize user input** — reject any `filename` value containing directory traversal sequences (`../`, `..\`) or other unexpected path characters before it's used to construct a filesystem path.
- **Use an allow-list approach** — rather than trying to block bad input, validate that the requested filename matches an expected pattern (e.g., a simple filename with no path separators) or exists in a known list of valid files.
- **Canonicalize the path and verify containment** — resolve the final path to its absolute form and confirm it still falls within the intended base directory (`/var/www/images/`) before reading the file; reject the request otherwise.
- **Run the web server process with least privilege**, limiting which files it can read at the OS level, so that even a successful traversal has minimal impact.

---

_Lab status: Solved ✅_
