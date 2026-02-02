# The HTTP Status Code Handbook

This document is a comprehensive guide to HTTP status codes. Understanding these codes is essential for developers, DevOps engineers, and anyone involved in building or maintaining web applications, as they are the primary language servers use to communicate the result of a client's request.

---

### Introduction: What are HTTP Status Codes?

Every time a client (like a web browser) makes a request to a server, the server includes a three-digit **status code** in its response. This code provides a quick and standardized way to understand what happened with the request.

Status codes are grouped into five classes, identified by their first digit:

*   **`1xx` (Informational):** The request was received and the process is continuing.
*   **`2xx` (Successful):** The request was successfully received, understood, and accepted.
*   **`3xx` (Redirection):** Further action needs to be taken by the client to complete the request.
*   **`4xx` (Client Error):** The request contains bad syntax or cannot be fulfilled. The client is likely at fault.
*   **`5xx` (Server Error):** The server failed to fulfill a valid request. The server is at fault.

---

### `1xx` Informational Responses

These are the least common codes in general browsing. They indicate that a request has been received and the server is thinking, but the final response is not yet ready.

*   #### `100 Continue`
    *   **What it means:** The server has received the request headers and the client should proceed to send the request body.
    *   **Context:** Used to optimize large uploads. A client can send headers with `Expect: 100-continue`, and the server can respond with `100 Continue` to signal that it's okay to send the large file, or it can reject the request (e.g., with a `4xx` error) before the client wastes bandwidth sending the body.

*   #### `101 Switching Protocols`
    *   **What it means:** The server is switching to a different protocol at the client's request.
    *   **Context:** This is the mechanism that powers **WebSockets**. A client sends an HTTP request with an `Upgrade: websocket` header, and the server responds with `101 Switching Protocols` before they begin communicating over the WebSocket protocol on the same connection.

---

### `2xx` Successful Responses

This class indicates that everything went as expected.

*   #### `200 OK`
    *   **What it means:** The standard response for a successful HTTP request. This is the most common status code.
    *   **Context:** The response body will contain the requested resource (e.g., an HTML page, an image, or JSON data).

*   #### `201 Created`
    *   **What it means:** The request has been fulfilled and has resulted in one or more new resources being created.
    *   **Context:** Typically returned after a `POST` or `PUT` request to an API that creates a new object (e.g., creating a new user account). The response usually includes a `Location` header pointing to the URL of the newly created resource.

*   #### `204 No Content`
    *   **What it means:** The server successfully processed the request but is not returning any content.
    *   **Context:** Very common for API `DELETE` requests. The client requested to delete a resource, the server deleted it, and it confirms success by sending `204` without a body. It's also used for `PUT` requests that update a resource in-place.

---

### `3xx` Redirection Messages

This class indicates that the client needs to take additional steps to complete the request, usually by going to a different URL.

*   #### `301 Moved Permanently`
    *   **What it means:** The requested resource has been permanently moved to a new URL, which is given in the `Location` header of the response.
    *   **DevOps/SEO Impact:** This is a critical code for SEO. Search engines will update their index to the new URL. Browsers will cache this response aggressively. Use this for HTTP-to-HTTPS redirection and when you permanently change a site's URL structure.

*   #### `302 Found` (Originally "Moved Temporarily")
    *   **What it means:** The resource is temporarily located at a different URL. The client should use the new URL for this request but continue using the original URL for future requests.
    *   **Context:** Historically used for redirection after a form submission. Modern applications often use `303 See Other` for this purpose. Unlike `301`, search engines do not update their indexes for a `302`.

*   #### `304 Not Modified`
    *   **What it means:** This is a response to a conditional `GET` request. The client already has a copy of the resource in its cache and asked the server if it has changed. This code means it has not.
    *   **Context:** The client sends a request with an `If-None-Match` (containing an ETag) or `If-Modified-Since` header. If the server's resource matches, it sends back `304` with an empty body. This saves significant bandwidth.
    *   **DevOps Impact:** Seeing `304`s in your logs is a good sign that your caching strategy (`ETag`, `Cache-Control` headers) is working correctly.

---

### `4xx` Client Error Responses

This class indicates that the server believes the client has made an error.

*   #### `400 Bad Request`
    *   **What it means:** A generic client-side error. The server cannot process the request due to something it perceives as a client error (e.g., malformed request syntax, invalid JSON in a `POST` body).

*   #### `401 Unauthorized`
    *   **What it means:** The client must authenticate to get the requested response. The server does not know the client's identity.
    *   **Context:** This is the correct code for a "login required" situation. The response will include a `WWW-Authenticate` header specifying how to authenticate.

*   #### `403 Forbidden`
    *   **What it means:** The server understood the request but refuses to authorize it. This is different from `401`.
    *   **Context:** This means "I know who you are (you may or may not be logged in), but you simply do not have permission to access this resource."
    *   **DevOps Impact:** This is often caused by filesystem permissions (the web server user cannot read the file) or an explicit `deny` rule in the web server's configuration.

*   #### `404 Not Found`
    *   **What it means:** The server cannot find the requested resource. This is the most famous status code.
    *   **Context:** The client requested a URL that does not correspond to any resource on the server.

*   #### `429 Too Many Requests`
    *   **What it means:** The user has sent too many requests in a given amount of time.
    *   **Context:** This is the standard code for rate limiting. The response may include a `Retry-After` header indicating how long the client should wait before making a new request.

---

### `5xx` Server Error Responses

This class indicates that the server failed to fulfill a valid request. **These are the codes that wake up DevOps engineers.** A `5xx` error is never the client's fault.

*   #### `500 Internal Server Error`
    *   **What it means:** A generic, unexpected error occurred on the server that prevented it from fulfilling the request.
    *   **DevOps Impact:** This is a catch-all for "something broke." **Immediately check the application logs** (e.g., `catalina.out` for Tomcat) or the web server error logs. This could be a bug in the application code, a database connection failure, a full disk, or countless other issues.

*   #### `502 Bad Gateway`
    *   **What it means:** The server, while acting as a gateway or reverse proxy, received an invalid response from an upstream server.
    *   **DevOps Impact:** This is the classic reverse proxy error. It means your Nginx or Apache server is working, but the backend application it tried to contact (e.g., a Tomcat, Node.js, or Python service) is down, crashing, not listening on the correct port, or sent back a garbled response. **Troubleshoot the backend service, not the proxy.**

*   #### `503 Service Unavailable`
    *   **What it means:** The server is currently unable to handle the request.
    *   **Context:** This is often a temporary condition. Common causes include:
        *   The server is down for maintenance.
        *   The server is overloaded with requests and cannot accept any more.
        *   A load balancer has determined that all of its backend application servers are unhealthy.
    *   **DevOps Impact:** This indicates a capacity or availability problem. You may need to scale up your application servers or investigate why they are unhealthy.

*   #### `504 Gateway Timeout`
    *   **What it means:** The server, while acting as a gateway or reverse proxy, did not receive a timely response from an upstream server.
    *   **DevOps Impact:** Your Nginx or Apache server is working, but the backend application it contacted is taking too long to respond. This points to a severe performance problem in the application, such as a very slow database query, an infinite loop, or resource starvation. You need to investigate the performance of the backend application.
