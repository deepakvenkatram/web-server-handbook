# Production Troubleshooting Handbook for Web Servers

This document is a production-grade, quick-reference guide for operating and troubleshooting the most common web and application servers. It is intended for DevOps, SRE, and Cloud specialists.

---

## Part 1: Nginx

Nginx is favored for its high performance as a reverse proxy and static content server. Troubleshooting often involves checking the connection between Nginx and its backend services.

### Chapter 1: Core Operational Details

*   **Service Management (systemd):**
    *   `sudo systemctl status nginx`: Check if the service is active and running.
    *   `sudo systemctl start | stop | restart nginx`: Basic service controls.
    *   `sudo systemctl reload nginx`: **Gracefully reloads configuration without dropping connections.** This is the preferred way to apply config changes.

*   **Configuration & Validation:**
    *   **Main Config:** `/etc/nginx/nginx.conf`
    *   **Site Configs:** `/etc/nginx/sites-available/` and `/etc/nginx/sites-enabled/`
    *   **Validate Syntax:** `sudo nginx -t`. **Always run this before reloading.** It checks all included files for errors and tells you the exact line where the error occurred.

*   **Default User:**
    *   Debian/Ubuntu: `www-data`
    *   RHEL/CentOS: `nginx`

*   **Default Port:** `80`. Changed with the `listen` directive in a `server` block.

*   **Log Locations:**
    *   **Error Log:** `/var/log/nginx/error.log`. **This is your most important troubleshooting tool.**
    *   **Access Log:** `/var/log/nginx/access.log`

### Chapter 2: Production Troubleshooting Scenarios

#### Scenario 1: You just applied a new configuration and the site is down.

1.  **Don't Panic.** If you used `systemctl restart`, the service may have failed to start.
2.  Check the service status: `sudo systemctl status nginx`. It will likely show a failed state and provide an error message or suggest running `journalctl -xe`.
3.  Run the configuration test: `sudo nginx -t`. This will almost always point to a syntax error in your `.conf` files (e.g., a missing semicolon, an unknown directive).
4.  Fix the error identified in the test and run `sudo nginx -t` again to confirm.
5.  Once the test passes, start the service: `sudo systemctl start nginx`.

#### Scenario 2: Users are reporting a "502 Bad Gateway" error.

**Meaning:** Nginx is successfully receiving the request but cannot communicate with the backend service defined in `proxy_pass`.

1.  **Is your backend application running?**
    *   For a Node.js app: `pm2 status` or `sudo systemctl status my-node-app`.
    *   For a Java app: `sudo systemctl status tomcat` or `ps -ef | grep java`.
    *   For a Python app: `sudo systemctl status gunicorn`.
2.  **Is it listening on the correct port/socket?**
    *   In your Nginx config, find the `proxy_pass http://localhost:3000;` line.
    *   On the server, run `sudo netstat -tulnp | grep 3000`. If you see no output, the application is not listening on that port.
3.  **Is a firewall blocking the connection?**
    *   Even on `localhost`, firewalls or security groups (in cloud environments) can block connections. Temporarily disable the firewall (`sudo ufw disable`) to test if it's the culprit. If so, add a rule to allow traffic on the specific port.
4.  **Check the Nginx error log:** `tail -f /var/log/nginx/error.log`. It will often contain a more specific message, like `(111: Connection refused) while connecting to upstream`.

#### Scenario 3: Users are reporting a "403 Forbidden" error.

**Meaning:** Nginx understands the request but is forbidden from accessing the requested file or directory. This is almost always a **filesystem permission issue**.

1.  **Identify the Nginx User:** Remember, it's `www-data` or `nginx`.
2.  **Check File Permissions:** The `nginx` user needs **read** permission on the file itself.
3.  **Check Directory Permissions:** This is the most common mistake. The `nginx` user needs **execute** permission on **every parent directory** in the path leading to the file.
    *   **The Ultimate Debugging Tool:** Use `namei -om /path/to/your/file.html`. This command will list each component of the path and its permissions, making it easy to spot where the permissions are wrong.
    *   **Example:** For a file at `/var/www/mysite/index.html`, the `nginx` user needs `r` on `index.html` and `x` on `/`, `/var`, `/var/www`, and `/var/www/mysite`.
4.  **Check for SELinux/AppArmor:** If permissions seem correct, a security module might be blocking access. Check the audit log with `sudo ausearch -m avc -ts recent` or `dmesg`.

---

## Part 2: Apache HTTP Server (httpd)

Apache is very flexible, but its configuration can be complex. Troubleshooting often involves checking which part of its distributed configuration is causing the issue.

### Chapter 1: Core Operational Details

*   **Service Management (systemd):**
    *   `sudo systemctl status apache2` (or `httpd`)
    *   `sudo systemctl start | stop | restart apache2`
    *   `sudo systemctl reload apache2`: Graceful reload.

*   **Configuration & Validation:**
    *   **Main Config:** `/etc/apache2/apache2.conf` (or `/etc/httpd/conf/httpd.conf`)
    *   **Site Configs:** `/etc/apache2/sites-available/`
    *   **Validate Syntax:** `sudo apache2ctl configtest`. **Always run this before reloading.**

*   **Default User:**
    *   Debian/Ubuntu: `www-data`
    *   RHEL/CentOS: `apache`

*   **Default Port:** `80`. Changed with the `Listen` directive.

*   **Log Locations:**
    *   **Error Log:** `/var/log/apache2/error.log` (or `/var/log/httpd/error_log`)
    *   **Access Log:** `/var/log/apache2/access.log`

### Chapter 2: Production Troubleshooting Scenarios

#### Scenario 1: `.htaccess` file changes are having no effect.

**Meaning:** Apache is ignoring your `.htaccess` files.

1.  **Check `AllowOverride`:** This is the cause 99% of the time. Find the `<Directory /var/www/html>` block in your main server or virtual host configuration that corresponds to your site's `DocumentRoot`.
2.  The `AllowOverride` directive inside that block **must not** be `None`. To enable all directives, set it to `AllowOverride All`.
3.  **Check Module Loading:** If you are using `RewriteRule`, ensure `mod_rewrite` is enabled. On Debian/Ubuntu, run `sudo a2enmod rewrite` and restart Apache.
4.  **Check Syntax:** A syntax error in `.htaccess` will cause a "500 Internal Server Error". Check the Apache error log for details.

#### Scenario 2: You are getting a "500 Internal Server Error".

**Meaning:** A fatal error occurred during request processing. With Apache, this often points to a misconfiguration in `.htaccess` or an error in an external script (like PHP).

1.  **Check the Apache Error Log:** `tail -f /var/log/apache2/error.log`. This is the most important step. It will contain a detailed error message.
2.  **Common Cause 1: Bad `.htaccess` Syntax.** The error log will say something like `... .htaccess: Invalid command 'RewriteRul'`, indicating a typo.
3.  **Common Cause 2: CGI/PHP Script Error.** If you are running a script, the error log might show `Premature end of script headers`. This means your script crashed before it could output valid HTTP headers. You then need to check the script's own error log (e.g., the PHP error log) to find the bug in the code.

---

## Part 3: Apache Tomcat

Tomcat runs Java applications. Troubleshooting is a mix of checking the server itself and debugging the Java application it is hosting.

### Chapter 1: Core Operational Details

*   **Service Management:** Usually run as a `systemd` service.
    *   `sudo systemctl status tomcat`
    *   `sudo systemctl start | stop | restart tomcat`

*   **Configuration Files:**
    *   **Core Server Config:** `$CATALINA_HOME/conf/server.xml`
    *   **JVM Options:** `$CATALINA_HOME/bin/setenv.sh` (you may need to create this file). This is where you set memory options like `-Xmx`.
    *   **Application Defaults:** `$CATALINA_HOME/conf/web.xml`

*   **Default User:** Typically a dedicated `tomcat` user.

*   **Default Port:** `8080`. Changed in `server.xml` in the `<Connector port="8080" ... />` element.

*   **Log Locations:**
    *   **Primary Log:** `$CATALINA_HOME/logs/catalina.out`. **This is the most critical log.** It captures all standard output and error streams from the Java application, including startup messages and stack traces.

### Chapter 2: Production Troubleshooting Scenarios

#### Scenario 1: Application fails to deploy, returning an HTTP 404.

**Meaning:** Tomcat is running, but your specific application is not available.

1.  **Check `catalina.out` Immediately:** `tail -f $CATALINA_HOME/logs/catalina.out`. Watch this log file as you restart Tomcat.
2.  **Look for Deployment Errors:** Scan the startup logs for `SEVERE` or `ERROR` messages related to your application's context path (e.g., `/myapp`). Common errors include:
    *   `ClassNotFoundException`: A required library (JAR file) is missing from your application's `WEB-INF/lib` directory.
    *   `LifecycleException`: A component of your application failed to start, often due to a misconfiguration in `web.xml` or a failed database connection.
    *   XML Parsing Errors: A syntax error in `web.xml` or `context.xml`.
3.  **Check the Manager App:** If enabled, the web-based Manager application (`/manager/html`) will show a list of all applications and their status. If your app is listed as "stopped," it failed to deploy, and the logs will tell you why.

#### Scenario 2: Application is slow or crashes with `OutOfMemoryError`.

**Meaning:** The Java Virtual Machine (JVM) running Tomcat has exhausted its allocated memory.

1.  **Confirm the Error:** Search `catalina.out` for the string `java.lang.OutOfMemoryError: Java heap space`.
2.  **Increase Heap Size:** You need to allocate more memory to the JVM.
    *   Create or edit the file `$CATALINA_HOME/bin/setenv.sh`.
    *   Add the following line, adjusting the values for your server's available RAM:
        ```bash
        # Set the initial and maximum heap size for the JVM
        # Use 'm' for megabytes, 'g' for gigabytes
        export CATALINA_OPTS="$CATALINA_OPTS -Xms512m -Xmx2g"
        ```
    *   This example sets an initial heap size of 512 MB and allows it to grow to a maximum of 2 GB.
3.  **Restart Tomcat:** `sudo systemctl restart tomcat`. The new memory settings will be applied.
4.  **Analyze Memory Usage:** If the problem persists, you may have a memory leak in your application. Tools like VisualVM or a commercial profiler (like YourKit or JProfiler) are needed to analyze the heap and find the source of the leak.

---

## Part 4: Caddy

Caddy's troubleshooting is often simpler due to its design.

*   **Service Management:** `sudo systemctl status | restart caddy`.
*   **Configuration:** `/etc/caddy/Caddyfile`.
*   **Validation:** `caddy validate --config /etc/caddy/Caddyfile`.
*   **Logs:** Caddy logs directly to the `systemd` journal.
    *   `journalctl -u caddy`: View all logs.
    *   `journalctl -u caddy -f`: Follow logs in real-time.
*   **Common Issue: Automatic HTTPS Fails.**
    *   **Meaning:** Caddy could not get a certificate from Let's Encrypt.
    *   **Troubleshooting:**
        1.  Check the logs: `journalctl -u caddy`. Caddy provides excellent, human-readable error messages.
        2.  **Cause 1: Bad DNS:** The domain name in your `Caddyfile` does not have an 'A' record pointing to your server's public IP address. Let's Encrypt cannot verify your ownership.
        3.  **Cause 2: Firewall:** A firewall is blocking port 80 or 443. The Let's Encrypt server needs to be able to reach your Caddy server on these ports to complete the validation challenge.
