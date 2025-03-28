# Repudiation

The example demonstrates a vulnerability that can lead to repudiation by malicious users attempting to access the services provided by a server.

## Steps to reproduce

1. Install all dependencies

    `$ npm install`

2. Run the server __insecure.ts__.

3. Pretend to be a malicous user and interact with the services by sending requests from the browser.

4. Do you think your actions can be repudiated?

## For you to do

1. Briefly explain the vulnerability.
2. Briefly explain why the vulnerability is addressed in __secure.ts__.
3. Which design pattern is used in the secure version to address the vulnerability? Briefly explain how it works?

1. Briefly explain the vulnerability.
The vulnerability in the insecure.ts version is a lack of proper logging and accountability mechanisms, which creates a repudiation risk. Specifically:

The server doesn't record detailed information about who is performing actions
There's no logging of user interactions, IP addresses, or timestamps
The system doesn't track or verify user identities when they access resources
There's no record of which user sent which message beyond the self-reported "user" field
The server logs minimal information and only to the console, which isn't permanent

This means that if a malicious user performs actions on the system, they could later deny having done so, and the system would have no way to prove otherwise. Since the only record of who sent a message is the user-provided name field, users can easily claim someone else used their name or that they never sent particular messages.


2. Briefly explain why the vulnerability is addressed in secure.ts.
The secure.ts version addresses the repudiation vulnerability through comprehensive logging and better accountability:

It implements a complete logging system that records:

Timestamps with precise date and time
IP addresses of users making requests
Method and URL of each request
User identifiers for authenticated actions
Content of messages being sent


It writes logs to a persistent file (server.log) rather than just the console
It logs error events with detailed information
It logs all access to sensitive information (when messages are retrieved)
It has hooks for proper authentication (though currently using a placeholder)
It implements structured exception handling with error logging

These measures create an auditable trail of actions that makes it much harder for users to deny their actions later, as the system maintains evidence of who did what and when.

3. Which design pattern is used in the secure version to address the vulnerability? Briefly explain how it works?

The secure version implements the Decorator pattern in the form of middleware to address the repudiation vulnerability. Specifically:
The middleware pattern in Express.js allows the application to "decorate" the request-response cycle with additional functionality. 
In this case:
app.use((req: Request, res: Response, next: NextFunction) => {
    const logEntry = `[${getCurrentDateTime()}] ${req.method} ${req.url} - ${req.ip}`;
    logStream.write(logEntry + '\n');
    next();
});
This middleware:

Intercepts every request before it reaches the route handlers
Logs important metadata (timestamp, HTTP method, URL, IP address)
Writes this information to a persistent log file
Passes control to the next middleware or route handler with next()

Additionally, the error handling middleware decorates the application with centralized error logging:

app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
    console.error(err.stack);
    const logEntry = `[${getCurrentDateTime()}] Error: ${err.message}`;
    logStream.write(logEntry + '\n');
    res.status(500).send(`Something went wrong: ${err.message}`);
});

This pattern allows the application to separate core functionality from cross-cutting concerns like logging and error handling, making the code more maintainable while ensuring that all interactions are properly recorded to prevent repudiation.