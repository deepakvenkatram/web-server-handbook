# The Handbook of Web API & RESTful Design

This handbook provides a comprehensive guide to designing and building Web APIs, with a focus on the principles of REST (REpresentational State Transfer). A well-designed API is scalable, maintainable, and easy for client developers to consume.

---

## Part 1: The Core Principles of REST

### Chapter 1: What is REST?

REST is not a strict protocol or standard; it is an **architectural style** for designing networked applications. It's a set of constraints that, when applied, lead to a scalable, fault-tolerant, and easy-to-use system. APIs that adhere to these principles are called "RESTful."

The six guiding constraints of REST are:

1.  **Client-Server Architecture:** This is a fundamental separation of concerns. The client (e.g., a web browser or mobile app) is responsible for the user interface, while the server is responsible for storing and retrieving data. They are independent and can be evolved separately.

2.  **Statelessness:** This is a critical principle. **Every request from a client to the server must contain all the information needed to understand and complete the request.** The server does not store any client "session" state between requests.
    *   **Why it matters:** Statelessness dramatically improves scalability and reliability. Any server instance can handle any request because no session data is stored on a specific server. This makes load balancing and failover trivial.

3.  **Cacheability:** Responses must define themselves as cacheable or not. This allows the client or intermediary proxies to cache responses, which can significantly improve performance and reduce server load. This is typically controlled via `Cache-Control` HTTP headers.

4.  **Uniform Interface:** This principle is the foundation of REST's design and is broken down into four sub-constraints:
    *   **Identification of Resources:** Each resource on the server is uniquely identified by a URI (Uniform Resource Identifier), such as `/users/123`.
    *   **Manipulation of Resources Through Representations:** The client interacts with a *representation* of the resource, not the resource itself. This representation is typically a JSON or XML object.
    *   **Self-Descriptive Messages:** Each request and response contains enough information for the other party to understand it. This is achieved through the use of HTTP methods (`GET`, `POST`), status codes (`200`, `404`), and headers (`Content-Type: application/json`).
    *   **Hypermedia as the Engine of Application State (HATEOAS):** This is the most advanced principle. It states that a client should be able to navigate an entire API just by following links provided in the responses. For example, a response for a user might include a link to retrieve that user's orders.

5.  **Layered System:** A client cannot ordinarily tell whether it is connected directly to the end server or to an intermediary along the way (like a load balancer, cache, or proxy). This allows for a flexible and scalable architecture.

---

## Part 2: Practical API Design

### Chapter 2: Designing Resources and URIs

The core of REST is thinking in terms of **"nouns" (resources), not "verbs" (actions).**

*   **Good (Resource-Oriented):** `/users`, `/orders`, `/products`
*   **Bad (Action-Oriented):** `/getUsers`, `/createOrder`, `/listAllProducts`

**URI Naming Conventions:**

*   **Use Plural Nouns for Collections:**
    *   `GET /users` -> Retrieves a list of all users.
    *   `POST /users` -> Creates a new user.

*   **Use an ID for a Specific Item:**
    *   `GET /users/123` -> Retrieves user with ID 123.
    *   `PUT /users/123` -> Updates user with ID 123.
    *   `DELETE /users/123` -> Deletes user with ID 123.

*   **Use Nested URIs for Related Resources:**
    *   `GET /users/123/orders` -> Retrieves a list of all orders for user 123.
    *   `GET /users/123/orders/5` -> Retrieves order 5 belonging to user 123.

*   **Use Query Parameters for Filtering, Sorting, and Pagination:**
    *   **Filtering:** `GET /orders?status=shipped`
    *   **Sorting:** `GET /products?sort=-price` (the `-` indicates descending order).
    *   **Pagination:** `GET /users?page=2&limit=50`

### Chapter 3: Using HTTP Methods Correctly

REST leverages the standard HTTP methods to perform actions. Understanding **idempotency** is key: an operation is idempotent if making the same request multiple times produces the same result as making it once.

*   `GET`: **Read** a resource. Safe and idempotent.
*   `POST`: **Create** a new resource. **Not idempotent** (making the same request twice will create two new resources).
*   `PUT`: **Update/Replace** an existing resource. The request body should contain the complete representation of the resource. **Idempotent** (making the same request twice will result in the same final state).
*   `PATCH`: **Partially update** an existing resource. The request body only needs to contain the fields that are changing. **Not necessarily idempotent.**
*   `DELETE`: **Delete** a resource. **Idempotent** (the resource is deleted after the first request; subsequent requests will also result in that state, likely returning a `404`).

### Chapter 4: Structuring API Responses

Consistency is key to a good developer experience.

*   **Use JSON:** It is the de facto standard for modern APIs. Always set the `Content-Type: application/json` header.

*   **Use a Consistent Response Envelope:** This makes it easy for clients to parse responses, especially for handling errors.
    ```json
    {
      "status": "success", // or "error"
      "data": {
        "id": 123,
        "name": "John Doe"
      },
      "message": null
    }
    ```

*   **Provide Meaningful Error Responses:** Don't just send a status code. Explain what went wrong.
    ```json
    // HTTP Status: 422 Unprocessable Entity
    {
      "status": "error",
      "message": "Validation failed.",
      "errors": {
        "email": "The email field is required.",
        "password": "Password must be at least 8 characters long."
      }
    }
    ```

*   **Handle Pagination:** For collections, never return all results at once. Include pagination metadata.
    ```json
    {
      "status": "success",
      "data": [ ... ], // Array of user objects
      "pagination": {
        "totalItems": 523,
        "totalPages": 21,
        "currentPage": 2,
        "limit": 25
      }
    }
    ```

*   **HATEOAS in Practice (Optional but good):** Include links to related actions.
    ```json
    {
      "id": 123,
      "name": "John Doe",
      "_links": {
        "self": { "href": "/users/123" },
        "orders": { "href": "/users/123/orders" }
      }
    }
    ```

---

## Part 3: Versioning and Authentication

### Chapter 5: API Versioning

As your application evolves, you will need to make changes to your API. To do this without breaking existing client applications, you must version your API.

*   **URI Path Versioning (Most Common and Recommended):** This is the clearest and most straightforward approach.
    *   `https://api.example.com/v1/users`
    *   `https://api.example.com/v2/users`

*   **Query Parameter Versioning:**
    *   `https://api.example.com/users?api_version=1`

*   **Custom Header Versioning:**
    *   `Accept: application/vnd.myapi.v1+json`

### Chapter 6: Securing Your API

*   **Authentication:** Verifying who a user is.
*   **Authorization:** Verifying what an authenticated user is allowed to do.

**Common Authentication Methods:**

*   **API Keys:** A simple secret token that is passed in an HTTP header (e.g., `X-API-Key: <your-key>`). Best for server-to-server communication or tracking usage, but less secure for client-side applications.

*   **JWT (JSON Web Tokens):** The modern standard for web and mobile apps.
    1.  A user logs in with their credentials.
    2.  The server validates them and returns a signed, stateless JWT.
    3.  The client stores this token and includes it in the `Authorization` header for all subsequent requests: `Authorization: Bearer <token>`.
    4.  The server can verify the token's signature on every request without needing to look up a session in a database, adhering to the REST principle of statelessness.

*   **OAuth2:** A framework for **delegated authorization**. It is not an authentication protocol itself.
    *   **Use Case:** Use OAuth2 when you want to allow a third-party application to access a user's data on your server on their behalf, without giving them the user's password. This is the mechanism behind "Login with Google/Facebook." It is more complex to implement than JWTs and should be used when that specific third-party delegation is required.
