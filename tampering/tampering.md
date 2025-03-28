# Tampering

This example demonstrates tampering through script injection.

## Steps to reproduce

1. Install all dependencies

    `npm install`

2. Start the **insecure.ts** server

    `npx ts-node insecure.ts`

3. In the browser, type a potentially malicious script in the name field of the form

    ```
        <script> document.body.innerHTML = "<a href='https://google.com'> Gotcha </a>"</script>
    ```

4. Do you see the potentially malicious hyperlink being injected into the form?

## For you to do

Answer the following:

1. Briefly explain the potential vulnerabilities in **insecure.ts**
2. Briefly explain how a malicious attacker can exploit them.
3. Briefly explain why **secure.ts** does not have the same vulnerabilties?

1. Briefly explain the potential vulnerabilities in insecure.ts

The main vulnerability in insecure.ts is a Cross-Site Scripting (XSS) vulnerability. When a user submits a name through the registration form, the application takes this input and stores it in the session without any sanitization. Later, when rendering the homepage, the unsanitized user input is directly inserted into the HTML response:

app.post("/register", (req: Request, res: Response) => {
  req.session.user = req.body.name.trim();
  res.send(`<p>Thank you</p> <a href="/">Back home</a>`);
});

Then, in the homepage route, this unsanitized input is inserted directly into the HTML:

app.get("/", (req: Request, res: Response) => {
  let name = "Guest";
  if (req.session.user) name = req.session.user;
  res.send(`
  <h1>Welcome, ${name}</h1>
  ...
  `);
});

This allows any HTML or JavaScript code submitted as the name to be directly executed when the page renders, without any protection.

2. Briefly explain how a malicious attacker can exploit them.

A malicious attacker can exploit this vulnerability by submitting JavaScript code in the name field, such as:

<script> document.body.innerHTML = "<a href='https://google.com'> Gotcha </a>"</script>

When this malicious input is stored in the session and then rendered in the welcome message, the browser will interpret and execute it as actual JavaScript rather than displaying it as text. This allows an attacker to:

Manipulate the page content (replacing the entire body with their own content)
Steal sensitive information such as cookies or session data
Redirect users to phishing websites
Perform actions on behalf of the user without their knowledge
Insert malicious forms to capture user credentials
Execute arbitrary JavaScript code in the context of the victim's browser session

This is particularly dangerous because the script executes with the privileges of the original website, potentially allowing access to sensitive data that would normally be protected by same-origin policy.

3. Briefly explain why secure.ts does not have the same vulnerabilities?

The secure.ts version prevents this vulnerability by implementing proper input sanitization before storing user input in the session and rendering it on the page:

app.post("/register", (req: Request, res: Response) => {
  const sanitizedName = escapeHTML(req.body.name.trim());
  req.session.user = sanitizedName;
  res.send(`<p>Thank you</p> <a href="/">Back home</a>`);
});

The key difference is the implementation of the escapeHTML() function:

function escapeHTML(input: string): string {
  return input
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#39;");
}

This function replaces special HTML characters with their encoded entities:

< becomes &lt;
> becomes &gt;
& becomes &amp;
" becomes &quot;
' becomes &#39;

By encoding these special characters, the browser will display them as literal text rather than interpreting them as HTML or JavaScript. So instead of executing the malicious script, the browser would simply display the script tags and their content as plain text on the screen. This effectively neutralizes XSS attacks by ensuring that user input is treated as data, not as executable code.RetryClaude can make mistakes. Please double-check responses.