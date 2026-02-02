# Web Servers Handbook

This repository is a comprehensive knowledge base for DevOps and Cloud Engineers, providing detailed guides and handbooks on the architecture, configuration, and operation of modern web and application servers. The primary goal is to offer a deep understanding of how these servers work, the problems they solve, and how they fit into a scalable, production-grade infrastructure.

Whether you are troubleshooting a production issue, designing a new system, or simply looking to deepen your understanding of web technologies, this repository serves as a practical, real-world guide.

## Table of Contents

1.  [Introduction to Web & Application Hosting](./intro.md)
2.  [Server Handbooks](#server-handbooks)
    - [Nginx](./nginx.md)
    - [Apache HTTP Server](./apache.md)
    - [Apache Tomcat](./tomcat.md)
    - [Other Web Servers (Caddy, LiteSpeed)](./other.md)
3.  [API Design & Debugging](#api-design--debugging)
    - [Web API & RESTful Design](./web-api.md)
    - [Debugging API Errors](./api-debugging.md)
4.  [Practical Operations & Troubleshooting](#practical-operations--troubleshooting)
    - [Production Troubleshooting Guide](./troubleshooting-web-server.md)
    - [HTTP Status Code Handbook](./status-code.md)

---

## Introduction to Web & Application Hosting

Before diving into specific technologies, the [**Introduction to Modern Web & Application Hosting**](./intro.md) provides a foundational understanding of the client-server model, the critical distinction between web servers and application servers, and the universal **Reverse Proxy Architecture** that underpins modern, scalable deployments.

---

## Server Handbooks

This section contains in-depth guides to the most common servers in the industry.

### [Nginx](./nginx.md)

The Nginx handbook covers its event-driven, asynchronous architecture, which makes it the industry standard for high-performance reverse proxying, load balancing, and static content delivery. It includes practical examples for configuration, SSL/TLS management with Let's Encrypt, and performance tuning.

### [Apache HTTP Server](./apache.md)

This guide explores the "Swiss Army knife" of web servers. It details the Multi-Processing Module (MPM) architecture (`prefork`, `worker`, `event`), the powerful module system, and its use as both a traditional web server and a reverse proxy. It also provides a clear comparison with Nginx, outlining when to choose one over the other.

### [Apache Tomcat](./tomcat.md)

A deep dive into the world of Java application hosting. This handbook clarifies Tomcat's role as a **Java Servlet Container**, not a traditional web server. It covers its core components (`<Connector>`, `<Engine>`, `<Host>`, `<Context>`), application deployment (WAR files), and the best-practice architecture of running Tomcat behind an Nginx reverse proxy.

### [Other Web Servers](./other.md)

An overview of significant alternatives to the mainstays:
- **Caddy:** A modern, secure-by-default web server with automatic HTTPS.
- **LiteSpeed:** A high-performance, drop-in replacement for Apache, famous in the WordPress hosting community.

---

## API Design & Debugging

A well-designed API is crucial for a maintainable system. This section covers the principles of building and troubleshooting them.

### [Web API & RESTful Design](./web-api.md)

This handbook introduces the core principles of the **REST architectural style**, including statelessness, resource-oriented URIs, and the correct use of HTTP methods. It provides practical guidance on designing clean, intuitive, and scalable APIs.

### [Debugging API Errors](./api-debugging.md)

A practical guide to diagnosing common API issues, with a special focus on the dreaded `500 Internal Server Error`. It provides a universal troubleshooting workflow: **always check the logs first**, and shows how to interpret stack traces to find the root cause of server-side problems.

---

## Practical Operations & Troubleshooting

This section is designed for real-world, on-the-job scenarios.

### [Production Troubleshooting Guide](./troubleshooting-web-server.md)

A quick-reference handbook for troubleshooting Nginx, Apache, and Tomcat in a production environment. It covers common error scenarios like "502 Bad Gateway," "403 Forbidden," and application deployment failures, providing step-by-step diagnostic and resolution instructions.

### [HTTP Status Code Handbook](./status-code.md)

A comprehensive reference for all major HTTP status code classes (`1xx` to `5xx`). Understanding these codes is essential for diagnosing issues, as they are the primary way a server communicates the result of a request. This guide explains what each code means and its impact, especially from a DevOps and SEO perspective.
