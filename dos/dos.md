# Denial-of-Service (DoS)

This example demonstrates DoS vulnerabilities and how they can be exploited.

## Steps to reproduce

1. Install all dependencies

    `$ npm install`

2. Ignore if you have already done this once. Insert test data in the MongoDB database. Make sure the mongod is up and running by typing the `mongosh` command in the termainal. If mongod process is up then you will see that the connection was successful. Command to insert test data:

    `$ npx ts-node insert-test-users.ts`

This will create a database in MongoDB called __infodisclosure__. Verify its presence by connecting with mongosh and running the command `show dbs;`.

2. Start the **insecure.ts** server

    `$ npx ts-node insecure.ts`

3. In the browser, pretend to be a hacker and type a malicious request

    ```
        http://localhost:3000/userinfo?id[$ne]=
    ```

4. Do you see the server crashing?

## For you to do

Answer the following:

1. Briefly explain the potential vulnerabilities in **insecure.ts** that can lead to a DoS attack.
2. Briefly explain how a malicious attacker can exploit them.
3. Briefly explain the defensive techniques used in **secure.ts** to prevent the DoS vulnerability?


1. Briefly explain the potential vulnerabilities in insecure.ts that can lead to a DoS attack.
The main vulnerabilities in insecure.ts that can lead to a Denial-of-Service attack include:

No Rate Limiting: The server doesn't implement any mechanism to limit the number of requests a client can make in a given time period. This allows attackers to flood the server with multiple requests.
No Error Handling: The code lacks proper error handling for database queries. If a query fails (for example, due to an invalid or malicious input), the server might crash or hang.
Lack of Input Validation: The server directly uses the user-provided id parameter in MongoDB queries without validation. This allows attackers to craft queries that might be resource-intensive or cause the MongoDB driver to throw errors.
Unhandled Promise Rejections: The asynchronous database operation doesn't have proper error handling with try/catch blocks, meaning errors could cause the application to crash.
No Query Timeout: There's no timeout set for database operations, so a carefully crafted query could potentially run for extended periods, consuming server resources.




2. Briefly explain how a malicious attacker can exploit them.
A malicious attacker can exploit these vulnerabilities in several ways:

Query Flooding: By sending a large number of concurrent requests to the /userinfo endpoint, an attacker can overwhelm the server and potentially exhaust available connections to MongoDB or server resources.
Malformed Query Parameters: Using NoSQL injection techniques like in the example (id[$ne]=), an attacker can potentially cause the MongoDB query to fail in a way that crashes the server.
Resource Exhaustion: By crafting queries that require significant processing power or memory, an attacker can deplete server resources. This includes queries with complex operators or ones that would return large result sets.
Automated Attack Scripts: Attackers can use scripts to send thousands of requests per second to the vulnerable endpoint, causing legitimate users to experience significant delays or inability to access the service.
Long-Running Queries: Constructing queries that take a long time to execute can tie up database connections and processing resources, effectively denying service to other users.

For example, the malicious request example (http://localhost:3000/userinfo?id[$ne]=) might cause an error in MongoDB's attempt to process an invalid ID format, potentially crashing the Node.js process if not properly handled.


3. Briefly explain the defensive techniques used in secure.ts to prevent the DoS vulnerability?
The secure.ts version implements several defensive techniques to prevent DoS attacks:

Rate Limiting: Implements express-rate-limit middleware to restrict each IP address to 1 request per 5 seconds:

const limiter = rateLimit({
  windowMs: 5 * 1000, // 5 seconds
  max: 1, // Limit each IP to 1 request per windowMs
  message: 'Server is busy, please try again later.',
});

This prevents attackers from overwhelming the server with too many requests.

Proper Error Handling: Uses try/catch blocks around database operations to gracefully handle errors without crashing the server:

try {
  // Database query code
} catch (error) {
  console.error('Error querying database:', error);
  res.status(500).send('Internal server error');
}

Graceful Error Responses: Instead of allowing errors to crash the server, it returns appropriate error status codes (401, 500) with informative but non-revealing error messages.
Error Logging: Errors are logged for administrators to review, enabling them to identify and address potential attack patterns.
These defensive techniques ensure that even when receiving malicious requests, the server remains operational and continues to serve legitimate users. The rate limiter is particularly effective as it prevents attack scripts from sending large volumes of requests, while the error handling ensures individual malformed requests don't bring down the entire service.

