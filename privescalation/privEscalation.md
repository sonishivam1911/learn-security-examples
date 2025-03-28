# Privilege Escalation

The example demonstrates a privilege escalation vulnerability and how to exploit it.

## Steps to reproduce

1. Install all dependencies

    `$ npm install`

2. Start the **insecure.ts** server

    `$ npx ts-node insecure.ts`

3. In the browser, send a GET request

    ```
        http://localhost:3000/send-form
    ```

4. Try different UserIds and see which one gives you authorized access to change the role of that user.

## For you to do

Answer the following:

1. Briefly explain the potential vulnerabilities in **insecure.ts**
2. Briefly explain how a malicious attacker can exploit them.
3. Briefly explain the defensive techniques used in **secure.ts** to prevent the privilege escalation vulnerability?



1. Briefly explain the potential vulnerabilities in insecure.ts
The primary vulnerability in insecure.ts is an improper authentication and authorization mechanism that can lead to privilege escalation. Specifically:

The system relies on client-provided userId in the request body to determine which user is making the request, without any session management or authentication tokens
The authorization check only verifies if the user identified by the provided userId has admin privileges
There's no verification that the person making the request is actually logged in as that user
The server doesn't maintain any server-side session state to track authenticated users
There's no protection against request forgery or parameter manipulation

This creates a fundamental security flaw where the application accepts the user identity from the request itself without proper verification, essentially letting the requester choose whose identity they want to assume.


2. Briefly explain how a malicious attacker can exploit them.
A malicious attacker can exploit this vulnerability in several ways:

Identity Spoofing: By simply changing the userId parameter in their request to the ID of an admin user (in this case, userId=1), an attacker can make requests that appear to come from the admin user.
Privilege Escalation: Once the attacker has identified that userId=1 belongs to an admin, they can send requests to the /update-role endpoint with this ID and gain the ability to modify user roles throughout the system.
Lateral Movement: The attacker could potentially elevate the privileges of other accounts they control to admin level, creating multiple admin accounts for persistent access.
Unauthorized Access: With admin privileges, the attacker could potentially access sensitive data or perform administrative actions they shouldn't be able to.

For example, a simple attack might involve sending a POST request to /update-role with the body { "userId": 1, "newRole": "admin" }, even if the attacker is not logged in as user 1 (the admin).


3. Briefly explain the defensive techniques used in secure.ts to prevent the privilege escalation vulnerability?
secure.ts implements several defensive techniques to prevent privilege escalation:

Session Management: Implements express-session to maintain server-side session state, which tracks the authenticated user's identity securely across requests.
Proper Authentication Flow: Instead of trusting user-provided identity in each request, the system associates the user's ID with their session upon login (though the login code isn't shown, we can see the session setup).
Session-Based Authorization: Decisions about what a user can do are based on the identity stored in their session (req.session.userId), not on parameters they provide in requests.
Two-Step Authorization Process:

First verifies the user is authenticated at all (if (!req.session.userId))
Then verifies the authenticated user has the required role for the action (loggedInUser.role !== 'admin')


Secure Cookie Configuration:

httpOnly: true - Prevents JavaScript access to the cookie, protecting against XSS attacks
sameSite: 'strict' - Prevents the cookie from being sent in cross-site requests, protecting against CSRF attacks


Separation of Identities: Clearly separates the identity of the user making the request (req.session.userId) from the target of the action (userId in the request body).

These protections ensure that users can only perform actions they're authorized for based on their actual authenticated identity, not based on identity information they provide in each request.