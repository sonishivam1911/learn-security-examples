# Tampering

This example demonstrates information disclosure by injecting malicious query objects to a NoSQL database.

## Steps to reproduce

1. Install all dependencies

    `$ npm install`

2. Insert test data in the MongoDB database. Make sure the mongod is up and running by typing the `mongosh` command in the termainal. If mongod process is up then you will see that the connection was successful. Command to insert test data:

    `$ npx ts-node insert-test-users.ts`

This will create a database in MongoDB called __infodisclosure__. Verify its presence by connecting with mongosh and running the command `show dbs;`.

2. Start the **insecure.ts** server

    `$ npx ts-node insecure.ts`

3. In the browser, pretend to be a hacker and type a malicious request

    ```
        http://localhost:3000/userinfo?username[$ne]=
    ```

4. Do you see user information being displayed despite the malicious request not having a valid username in the request?

## For you to do

Answer the following:

1. Briefly explain the potential vulnerabilities in **insecure.ts**
2. Briefly explain how a malicious attacker can exploit them.
3. Briefly explain the defensive techniques used in **secure.ts** to prevent the information disclosure vulnerability?


1. Briefly explain the potential vulnerabilities in insecure.ts
The main vulnerability in insecure.ts is a NoSQL injection vulnerability that can lead to information disclosure. Specifically:

The application directly uses the user-provided username query parameter in the MongoDB query without any validation or sanitization
The code takes the raw input from req.query.username and passes it directly to User.findOne()
There's no type checking to ensure the username is actually a string
There's no sanitization to prevent NoSQL query operators or other special characters
Error handling is minimal, potentially allowing error messages to leak information
The full user object (including password) is sent in the response if a user is found

This creates a situation where an attacker can manipulate the query structure by using MongoDB operators in the input instead of providing a simple string value.


2. Briefly explain how a malicious attacker can exploit them.
A malicious attacker can exploit this vulnerability in several ways:

Data Enumeration: Using the $ne (not equal) operator as demonstrated in the example, an attacker can retrieve all users by sending a request like username[$ne]= which translates to "find users where username is not equal to an empty string" - effectively matching all users.
Bypassing Authentication: If this were an authentication endpoint, an attacker could use MongoDB operators to create a query that always returns a user regardless of the credentials provided.
Data Exfiltration: Since the full user object (including the password) is returned in the response, an attacker can steal sensitive information from the database.
Pattern Matching: Using operators like $regex, an attacker could search for usernames matching certain patterns (e.g., starting with 'a', containing 'admin', etc.).
Manipulating Query Structure: An attacker could inject more complex MongoDB query operators to manipulate the query structure and access unauthorized information.

For example, the URL in the exercise (http://localhost:3000/userinfo?username[$ne]=) makes MongoDB interpret the query as { username: { $ne: '' } } instead of the expected { username: 'some_value' }, causing it to return all users in the database.


3. Briefly explain the defensive techniques used in secure.ts to prevent the information disclosure vulnerability?
secure.ts implements several defensive techniques to prevent NoSQL injection and information disclosure:

-Input Type Validation: It checks that the username is a string before processing it:

if (typeof username !== 'string') {
  return res.status(400).send('Invalid username format');
}

-Input Sanitization: It removes any non-alphanumeric characters that could be used for injection:

const sanitizedUsername = username.replace(/[^\w\s]/gi, '');

This regex specifically strips out characters like $, [, and ] that could be used to construct MongoDB operators.


-Error Handling: The code implements proper error handling with try/catch blocks to prevent detailed error messages from being exposed to users:

try {
  // Query code here
} catch (error) {
  console.error('Error querying database:', error);
  res.status(500).send('Internal server error');
}

Explicit Error Messages: The error messages are generic and don't reveal specific details about the failure (e.g., "Invalid username" instead of details about why authentication failed).
These defensive techniques ensure that user input cannot alter the structure of the database query and that sensitive information is not inadvertently disclosed even when errors occur.

