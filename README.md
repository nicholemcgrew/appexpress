# AppExpress

AppExpress is a lightweight framework (0 dependencies) inspired by `express.js` designed specifically for Appwrite
Functions. It simplifies creating server-like functionalities by providing an intuitive API for routing, middleware
integration, and more.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Getting Started](#getting-started)
    - [Initialize](#initialize)
    - [Routes](#routes)
    - [Middlewares](#middlewares)
    - [Parameters and Wildcards](#parameters-body-and-wildcards)
    - [Dependency Injection](#dependency-injection)
    - [Rendering Content](#rendering-content)
- [Requests and Response](#requests-and-response)

## Features

- **Custom Routing**: Supported routes include `GET`, `POST`, `PUT`, `PATCH`, `DELETE`, `OPTIONS`, and `ALL`.
- **Middleware Support**: Seamlessly integrate custom middleware to augment the functionality of your routes.
- **Parameter Extraction**: Directly parse parameters embedded in URLs.
- **Wildcard Paths**: Implement wildcard routing for advanced path matching.
- **Dependency Injection**: Facilitate cleaner code and modular architecture through dependency management.

## Installation

```shell
npm install @itznotabug/appexpress
```

## Getting Started

### Initialize

Create a new instance of `AppExpress`:

```javascript
// import AppExpress here.
const appExpress = new AppExpress();
```

**Important**: Now attach it to the function entrypoint :

```javascript
export default async (context) => await appExpress.attach(context);
```

### Routes

Define and handle routes similar to `express.js`:

```javascript
// Simple GET route
appExpress.get('/', (request, response) => {
    response.send('Welcome to AppExpress!');
});

// JSON response
appExpress.get('/hello', (request, response) => {
    response.json({ message: 'Hello World' });
});

// Route with a handler function
const homePageHandler = (request, response) => {
    response.send('Home Page Content');
};

appExpress.get('/home', homePageHandler);
```

### Middlewares

Incorporate middleware to process requests:

```javascript
// Logging middleware
appExpress.use((request, _, log) => {
    log('Requested Path:', request.path);
    // no need to return anything from here
});

// Middleware for analytics
const analyticsMiddleware = (request, _, log, error) => {
    // Implement analytics logic here
    try {
        analyticsSingleton.log('path', request.path);
        log(`recorded '${request.path}' for analytics`);
    } catch (err) {
        error(`Error logging to analytics: ${err.message}`);
    }
};

appExpress.use(analyticsMiddleware);
```

### Parameters, Body and Wildcards

Capture URL parameters and utilize wildcard routes:

```javascript
// Parameterized route
appExpress.get('/user/:id/:transactionID', async (request, response) => {
    const { id, transactionID } = request.params;

    // perform some validation here...
    const billing = new BillingClient();
    const result = await billing.verify(id, transactionID);

    response.send(result.message);
});

appExpress.post('/verify', (request, response) => {
    const { sourceKey } = request.body;
    // perform some checks with the `sourceKey`
    response.json({ status: 'ok' });
});

// Wildcard route for 404 errors
appExpress.get('*', (_, response) => response.send('404 Not Found'));
```

### Dependency Injection

Manage dependencies effectively within your application:

```javascript
// Inject a repository
const repository = new AppwriteRepository();
appExpress.inject(repository);

// Retrieve
import AppwriteRepository from '../data/repository.js'; // import for passing type.
appExpress.get('/user/auth/:userId', async (request, response) => {
    const { userId } = request.params;
    const repository = request.retrieve(AppwriteRepository);
    const result = await repository.performAuth(userId);
});
```

If you have multiple variables of the same class types, make sure to use an `identifier`:

```javascript
// Inject repositories
appExpress.inject(sourceRepository, 'source');
appExpress.inject(destinationRepository, 'destination');

// Retrieve
appExpress.post('/migration', async (request, response) => {
    const { sourceKey, destinationKey } = request.body;
    // perform some checks with the API Keys
    const source = request.retrieve(Repository, 'source');
    const destination = request.retrieve(Repository, 'destination');

    // perform a migration
    const migration = new Migration(source, destination);
    const result = await migration.mirrorDatabases();

    response.json({ result })
});
```

### Rendering Content

1. Serve HTML from `file` (recommended):
    ```javascript
    // Set the directory for views
    appExpress.views('views/');
    
    // Route to serve an HTML file from path `views/index.html`
    appExpress.get('/', (request, response) => response.htmlFromFile('index.html'));
    ```

2. Serve HTML from `string`:
    ```javascript
    const htmlString = '...';
    appExpress.get('/', (request, response) => response.html(htmlString));
    ```

## Requests and Response

### `AppExpressResponse` Methods

The `AppExpressResponse` object provides several methods to manage the HTTP responses effectively:

| Method                                   | Description                                                      | Default Values                             |
|------------------------------------------|------------------------------------------------------------------|--------------------------------------------|
| `empty()`                                | Return no content.                                               |                                            |
| `redirect(url)`                          | Redirect to a specified URL. Use complete URL including schemes. |                                            |
| `send(message, statusCode, contentType)` | Return a plain text response.                                    | statusCode: 200, contentType: 'text/plain' |
| `json(jsonObject)`                       | Return a JSON object.                                            |                                            |
| `html(stringHtml, statusCode)`           | Return HTML content.                                             | statusCode: 200                            |
| `htmlFromFile(filePath, statusCode)`     | Return HTML content from a local file.                           | statusCode: 200                            |

### `AppExpressRequest` Methods

The `AppExpressRequest` object allows access to incoming HTTP request data:

| Property/Method              | Description                                                  | Type    |
|------------------------------|--------------------------------------------------------------|---------|
| `bodyRaw`                    | Raw body as a string.                                        | String  |
| `body`                       | Parsed body as JSON.                                         | JSON    |
| `headers`                    | HTTP request headers.                                        | JSON    |
| `scheme`                     | URL scheme (e.g., http or https).                            | String  |
| `method`                     | HTTP method (e.g., GET, POST).                               | String  |
| `url`                        | Full URL of the request.                                     | String  |
| `host`                       | Hostname from the URL.                                       | String  |
| `port`                       | Port number from the URL.                                    | Integer |
| `path`                       | Path component of the URL.                                   | String  |
| `queryString`                | Raw query string from the URL.                               | String  |
| `params`                     | URL parameters as JSON.                                      | JSON    |
| `query`                      | Query parameters as JSON.                                    | JSON    |
| `triggeredType`              | Type of function trigger.                                    | String  |
| `events`                     | Event mapping based on the event type.                       | JSON    |
| `eventType`                  | Specific form of the parsed event type.                      | String  |
| `retrieve(type, identifier)` | Retrieves an instance based on type and optional identifier. | T       |