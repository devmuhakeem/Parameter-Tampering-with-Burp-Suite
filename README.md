# Parameter Tampering with Burp Suite

A hands-on lab demonstrating an Insecure Direct Object Reference (IDOR) vulnerability: intercepting and modifying an HTTP request's `id` parameter with Burp Suite to access another user's account data without their credentials, against a sandboxed target web application.

## Scenario
A target web application let a logged-in user view their own profile via a URL parameter (`id`). The question was whether the server verified that the requesting user actually owned that profile, or simply trusted whatever `id` value arrived in the request.

## What I did

### 1. Set up Burp Suite as an intercepting proxy
- Configured Firefox's manual proxy settings to route HTTP and HTTPS traffic through `127.0.0.1:8080`
- Launched Burp Suite Community Edition, started a temporary project with default settings
- Confirmed Intercept was active on the Proxy tab, so every outgoing browser request would pause in Burp before reaching the server

### 2. Captured a legitimate login and profile request
- Logged into the target site as a registered user (`sam`)
- Watched the login POST request land in Burp's Intercept tab before forwarding it through
- Navigated to the account's View Profile page, catching that request in Burp as well

### 3. Identified and tampered with the vulnerable parameter
- In the Params tab, found a URL parameter `id` set to `1`, matching the logged-in user's account
- Changed the value from `1` to `2` directly in Burp before forwarding the request
- Turned interception off to let the modified request complete

### 4. Confirmed the impact
- The browser displayed a different user's profile information — an account the logged-in session had no legitimate access to
- Confirmed this works for any sequential ID value, allowing enumeration of every account in the system

## Key takeaways
- This is a textbook IDOR: the app verifies *who* is logged in but never checks *what that user is authorized to request* — an authorization failure, not an authentication one
- Sequential, predictable resource IDs turn a theoretical IDOR into trivially walkable enumeration — no guessing required, just incrementing
- Any client-supplied value (URL params, hidden fields, cookies) must be treated as attacker-controlled; the server has to independently verify ownership/authorization on every request
- A proxy tool is what makes this class of bug visible and testable at all — the vulnerability is invisible just by looking at the rendered page
- Fixes: enforce per-request server-side authorization checks consistently across every endpoint, prefer non-sequential identifiers (UUIDs) for sensitive resources, and monitor for ID-enumeration patterns in traffic

## Tools
Burp Suite Community Edition, Mozilla Firefox, Parrot Security OS

---
*Completed in an isolated EC-Council training lab environment against a designated practice web application.*
