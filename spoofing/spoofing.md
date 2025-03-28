# Spoofing

This example demonstrates spoofind through two ways -- Stealing cookies programmatically and cross site request forgery (CSRF).

## Steps to reproduce the vulnerability

1. Install dependencies

    `$ npx install`

2. Start the **insecure.ts** server

    `npx ts-node insecure.ts`

3. Start the malicious server **mal.ts**

    `npx ts-node mal.ts`

4. Open __http://localhost:8000__ in a browser, type a name and Submit.

5. Open the __Application__ tab in the Browser's inspect pane. Find the __Cookies__ under __Storage__. You should see a __connect.sid__ cookie being set.

6. Open the HTML file __mal-steal-cookie.html__ file in the same browser (different tab). Open inspect and view the console.

7. Click the link in the HTML file. Do you see the cookie being stolen in the console?

8. Open the HTML file __mal-csrf.html__ file in the same browser (different tab). What do you see if the user has not logged out of **insecure.ts**? What do you see if the user has logged out? 


## For you to answer

1. Briefly explain the spoofing vulnerability in **insecure.ts**.
2. Briefly explain different ways in which vulnerability can be exploited.
3. Briefly explain why **secure.ts** does not have the spoofing vulnerability in **insecure.ts**.


1. Briefly explain the spoofing vulnerability in insecure.ts

The spoofing vulnerability in insecure.ts is primarily due to insecure session cookie configuration. The server sets up session cookies with httpOnly: false, which means client-side JavaScript can access these cookies. The code also lacks other critical security configurations:

session({
  secret: "SOMESECRET",
  cookie: { httpOnly: false },  // Allows JavaScript to access cookies
  resave: false,
  saveUninitialized: false,
})


Additionally, the session secret is hardcoded directly in the code rather than being provided as an environment variable, which is another security weakness. There's also no SameSite attribute configured for the cookies, making them vulnerable to cross-site request forgery attacks.


2. Briefly explain different ways in which vulnerability can be exploited
This vulnerability can be exploited in several ways:

Session Hijacking via JavaScript: Since httpOnly is set to false, any malicious script that runs in the user's browser can access the session cookie via document.cookie. The attacker can then use this stolen cookie to impersonate the user.
Cross-Site Request Forgery (CSRF): Without SameSite protection, an attacker can trick a user who is already authenticated on the target site into performing unwanted actions. For example, a malicious site could submit a form that posts to the /sensitive endpoint, and the browser would automatically include the session cookie.
Session Fixation: Without proper session management, an attacker might be able to set a known session ID for a victim before they authenticate, allowing the attacker to gain authenticated access once the victim logs in.
Man-in-the-Middle Attacks: Without the Secure flag, cookies may be transmitted over unencrypted HTTP connections, making them vulnerable to interception.

3. Briefly explain why secure.ts does not have the spoofing vulnerability in insecure.ts
The secure.ts version addresses these vulnerabilities through several improvements:

HttpOnly Flag: Sets httpOnly: true, preventing client-side JavaScript from accessing the cookie:

cookie: {
  httpOnly: true,
  sameSite: true,
}

SameSite Attribute: Implements the SameSite attribute, which prevents the browser from sending cookies in cross-site requests, mitigating CSRF attacks.

Secret Management: The session secret is provided as a command-line argument rather than being hardcoded, improving security:

const secret: string = process.argv[2];

Cookie Security: The overall cookie configuration is more robust, with multiple security flags enabled that were missing in the insecure version.

These improvements collectively make it much harder for attackers to steal session cookies or perform cross-site request forgery attacks, significantly reducing the risk of session spoofing.

