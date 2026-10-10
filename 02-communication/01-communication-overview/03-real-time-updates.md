

Video_Link : https://youtu.be/8Uyka3fzXek?si=U07GGRPjvaCR8ihN

--------------------------------------------------------------------------

This response draws exhaustively from the provided source material, detailing every concept, strategy, advantage, and disadvantage related to designing real-time updates for frontend systems.

---

## I. Introduction and Foundational Concepts

### A. Context and Goals

This video initiates a series focused on system design building blocks. These building blocks are likened to a **Constructor kit**, allowing developers to build any application, preparing them for diverse system design topics. The series begins with **real-time updates**.

**Real-time updates** generally mean updating the user interface (UI) in real-time, near real-time, or sometimes periodically. The goal for frontend developers is to update the UI to display new data to the user.

### B. Typical Applications Requiring Real-Time Updates

Applications where real-time updates are needed typically include:

- Chat applications
- Messengers
- Online games
- Stock applications
- Collaborative tools (e.g., Google documents, Figma)

### C. Key Strategies: Pull vs. Push

There are two primary strategies for delivering updates:

|Strategy|Definition|Initiator|Examples|
|:--|:--|:--|:--|
|**Pull**|The **client requests** updates from the server.|The client.|Short Polling and Long Polling.|
|**Push**|The **server sends** messages to the client.|The server.|Server Sent Events (SSE) and WebSockets.|

### D. Comet

**Comet** is an **umbrella term** encompassing techniques and technologies that create a long-held HTTP connection to push messages from the server to the client. Comet includes Long Polling and Server Sent Events (SSE), among others.

### E. The WebSockets Dilemma and Trade-offs

Articles often simplify the choice, stating that short polling is ineffective, long polling should only be used if the browser lacks WebSocket support, and WebSockets should be used for real-time updates everywhere because they are progressive and fast.

However, the source notes that the issue is often more complex:

- WebSockets offer a **bidirectional connection** primarily designed for specific cases.
- It is **not a one-size-fits-all solution**.
- While using WebSockets every time is possible, it might not be the wisest choice.
- Designing web applications involves **trade-offs**, focusing not just on efficiency but also on resources required for **development and maintenance**.
- _Analogy:_ Building a static one-page site in React is possible but generally not a good choice.

**Real-World Examples of Complex Choices:**

- **LinkedIn** uses Server Sent Events for instant messaging (raising the question of why they didn't use WebSockets).
- **Stack Overflow** uses WebSockets to update the interface for new posts in the feed (raising the question of why not SSE or polling, as a bidirectional connection is not needed).

### F. Interview Preparation Focus

In an interview setting, what matters most is **not what solution is chosen, but why the choice is made and why other options are rejected**. The interviewer expects the candidate to be knowledgeable about alternative options, potential trade-offs, and effective ways to navigate them. It is suggested that choosing a solution the interviewer might disagree with, but backing it up with solid arguments and demonstrating extensive knowledge, is preferable.

---

## II. Detailed Examination of Real-Time Update Protocols

The discussion proceeds through four methods: Short Polling, Long Polling, Server Sent Events, and WebSockets.

### A. Short Polling (Pull Strategy)

**Definition:** Short polling is a process where the client **periodically asks the server for updates** (e.g., every 5 seconds).

#### Key Characteristics

|Advantages|Disadvantages|
|:--|:--|
|**Simplicity** (No additional server-side work required).|**Inefficient solution** in many cases (serious trade-off).|
|Operates over HTTP.|Leads to the occurrence of **useless requests** when no data updates exist.|
|Does not require a persistent connection.|**Updates during the waiting period will be skipped**.|
||Does not provide true **real-time updates**.|

#### Use Cases

1. **Fixed Intervals:** Suitable when data needs updates at a static interval (e.g., every 5 minutes, every 30 seconds, or 1 minute) and true real-time updates are not necessary.
2. **External Backends:** When the client does not have access to the server's code (e.g., using a deprecated third-party backend that doesn't support real-time updates by design).
3. **Simple/Temporary Solution:** When the simplest and most cost-effective solution is needed (e.g., logging developer resources, uncertainty about app evolution).
4. **Example Apps:** Analytic apps, stock applications.

#### When to Avoid Short Polling

1. When **real-time updates are necessary** (it may only _appear_ to provide real-time updates if the request interval is short, but this is coincidental).
2. When updates occur **rarely or randomly**, making the majority of requests useless and the approach highly inefficient.

#### Scaling Short Polling

Scaling for short polling is **straightforward** and aligns with its simplicity. There are no unique problems related to scaling short polling.

- **Scaling Types:**
    - **Vertical Scaling:** Adding resources (like CPU and memory) to a single machine.
    - **Horizontal Scaling:** Adding more servers to the server pool.
    - **Grid Scaling:** Combines both vertical and horizontal approaches.
- **Load Balancer (LB):** A component responsible for distributing incoming traffic based on factors such as server load.
- **Data Storage:** If data (e.g., stock prices) is saved to a shared database, subsequent requests can be redirected by the load balancer to a different server without issue, as all servers access the same information. Storing data in the server cache could cause problems, but this is generally not relevant for short polling.
- **Sticky Sessions:** Issues related to sticky sessions are generally not applicable to short polling because it is used in stateless scenarios (e.g., analytics) where data is updated via the database, not stored on the server.

### B. Long Polling (Pull Strategy / Comet Technique)

**Definition:** Long polling is a **technique** (not a technology). The client sends a request, but the server **holds the request** until new data becomes available, at which point the data is sent to the client.

#### Key Characteristics

|Advantages|Disadvantages|
|:--|:--|
|**More resource efficient** than short polling (solves the useless requests problem).|Requires a **persistent connection**, adding complexity.|
|Offers **nearly real-time updates**.|Infrastructure (proxy servers, elements) might interrupt the connection due to configurations favoring short-lived connections.|
|**Broadest browser support** (uses simple XML HTTP requests).|Requires **additional work on the backend side**.|
|Used as a **fallback** for older browsers by WebSockets and SSE libraries.|**Horizontal scaling is more challenging** (moving connections between servers is complex).|
|Operates over standard HTTP (inherits HTTP/2 features).|Inherits HTTP-related issues, such as **overhead**.|
|Allows the use of HTTP headers and avoids authentication issues.|**Not true real-time updates**; potential delays occur due to required reconnections.|

#### Details on Delay and Reconnection

Long polling is _perceived_ as persistent, but reconnections occur. When an update happens, the server sends it, and the connection closes. If a second update occurs simultaneously, the server **cannot immediately send it**; it must wait for the client to **reestablish the connection**. While this delay is usually not critical, it is problematic if minimal latency is required. To avoid losing updates that occur during reconnection, a **message queue** needs to be implemented.

#### Use Cases

1. **Broad Browser Support:** Required when real-time updates must work for all browsers, including older ones.
2. **Infrequent Messages:** If server messages are infrequent, the slight delay is not a problem.
3. **Simplicity/Cost:** When keeping the connection simple and cheap is smarter and doesn't significantly impact performance.
4. **Example Apps:** Analytic apps, stock apps, social networks.

#### When to Avoid Long Polling

1. When an **efficient bidirectional channel** is needed (WebSockets are better for minimal latency).
2. When the infrastructure is not suited for multiple persistent connections (e.g., short polling is better for updates occurring every 5 minutes from a third party).
3. When **HTTP overhead** is an issue (switch to SSE).
4. When **minimal latency** is critical (due to reconnection delays).

#### Scaling Challenges: Stateful vs. Stateless Architectures

Scaling long polling is a more interesting challenge.

1. **Stateless Architecture:** No data is stored on the server. Offers flexibility (easy replacement, request redirection).
2. **Stateful Architecture:** Data is kept on the server (e.g., loading progress maintained in server memory).
    - **The Problem:** If the Load Balancer (LB) redirects a specific client's subsequent request to a different server, the new server will not know about the ongoing process (the state).

**Solutions for Stateful Architectures:**

|Solution|Mechanism|Advantages|Disadvantages|
|:--|:--|:--|:--|
|**Sticky Sessions**|Forces the LB to consistently redirect a client's requests to the same server based on Client ID (LB maintains a client/server map).|Easier to implement. Suitable for **short-lived or non-critical data** (e.g., file uploads).|Can lead to **uneven distribution** of connections across servers/hosts. Data loss possible if the server goes down. Presents a vulnerability to **targeted attacks**.|
|**Shared Data Storage**|Use an additional place (e.g., database, cache) to store data, making the servers stateless.|Does **not affect load balancing**. Safer, as data is centrally stored. Perfect for **long-lived or critical data** (e.g., user data). More reliable storage solution.|More challenging to implement and maintain.|

A hybrid approach (shared storage for critical data, sticky sessions for short-lived data) can also be considered. Long polling is frequently used in stateful contexts (like tracking real-time file uploads).

### C. Server Sent Events (SSE) (Push Technology / Comet Implementation)

**Definition:** SSE is an implementation of the Comet pattern and a **real technology** (not a hack). The client establishes a connection to a server, allowing the server to **initiate message sending** directly to the client. It is a **push technology**.

#### Key Characteristics

|Advantages|Disadvantages|
|:--|:--|
|Provides **real-time updates**.|Requires **additional server-side work** (more complex server implementation than polling).|
|Establishes a **true persistent connection**, minimizing delay (no need to reconnect).|**Less browser support** than polling and WebSockets.|
|Operates over **plain HTTP**; works well with most proxy servers and load balancers.|Less efficient traffic consumption compared to WebSockets (which have no HTTP overhead).|
|Relatively **lower HTTP overhead** than polling (approx. 5 bytes vs. 8 bytes).|Cross-domain connection needs appropriate CORS settings.|
|Includes features like **automatic reconnection**, event names, and event IDs.|Limited to **six concurrent connections per browser** below HTTP/2 (more require HTTP/2 or higher).|
|Supports **multiplexing** and other HTTP/2 features (if used over HTTP/2).|**HTTP Overhead**.|
|Supports **connectionless pushes** for battery saving on mobile devices.|Does not support HTTP headers like the authorization header, creating **authentication issues**.|
|Provides a convenient and intuitive **event-driven interface**.||

#### Use Cases

1. **One-Directional Channel:** Required when a one-directional channel for real-time updates is needed.
2. **Medium Frequency Messages:** Suitable for frequent server messages where the HTTP overhead of long polling might be too high.
3. **Battery Life Concerns:** Supports connectionless pushes for mobile devices.
4. **Real-World Examples:** Real-time notifications (news/social feeds), live updates (sports events, crypto prices), real-time analytics, progress updates for long processes, server statistic monitoring (uptime, health).
5. **Example Apps:** Analytic apps, stock apps, social networks, possibly chat applications.

#### When to Avoid SSE

1. When **bidirectional communication** is needed (WebSockets are better, as emulating two-way communication with SSE plus a standard HTTP request is less efficient).
2. For **very frequent server messages** where the HTTP overhead is problematic (switch to WebSockets).

#### Update Frequency Schema Comparison

The source suggests a simple schema for choosing between methods based on update frequency:

- **Low update frequency:** Long Polling.
- **Medium update frequency:** Server Sent Events (offering better traffic efficiency).
- **High update frequency:** WebSockets.

### D. WebSockets (Bidirectional Technology)

**Definition:** WebSocket is a **bidirectional channel** that enables native communication in both directions (client to server and server to client). This capability is unique, although it can be emulated using other methods.

#### Key Characteristics

|Advantages|Disadvantages|
|:--|:--|
|Provides a **bidirectional channel**.|**More complex to implement and maintain** than other methods.|
|Operates on a separate protocol; offers **lower latency** and **reduced message overhead**.|Runs on different protocols (**ws** and **wss**).|
|Better browser support compared to SSE.|Potential issues with proxy servers, firewalls, and load balancers configured only for HTTP.|
|Supports **binary data transfer**, reducing payload size (e.g., media files).|**Lacks support for HTTP features** like multiplexing.|
|Can connect across domains **without CORS requirements**.|Browser support is not as high as for polling methods.|
|Highly efficient in network traffic and minimal delay.|Horizontal scaling involves **additional complexities**.|
||**Locks (Lacks) HTTP/2 core features** like multiplexing.|

#### Use Cases

1. **Bidirectional Channel:** Required if the application genuinely needs this connection.
2. **Maximum Efficiency:** Used to achieve lower latency and reduced overhead, even if bidirectional communication isn't strictly necessary.
3. **Minimal Latency:** Crucial for applications requiring real-time interactions.
4. **Send and Receive:** Applications where users need to both receive and actively send messages.
5. **Example Apps:** Collaborative tools (Google Docs, Figma), multiplayer games, chat applications.

#### When to Avoid WebSockets

1. If **bidirectional communication isn't actually needed** (it may be overkill).
2. If the existing infrastructure **isn't ready** to handle the WebSocket protocol (downgrading to SSE plus HTTP requests might be more feasible).
3. When specific features of **HTTP/2 or SSE are required** (WebSockets miss HTTP/2 benefits because they do not operate over HTTP).
4. If the risks related to using a different protocol within HTTP-designed infrastructure are **not fully understood**.

#### Overhead and Efficiency

WebSockets provide **minimal overhead** compared to polling methods. For highly frequent and intensive interactions (e.g., multiplayer games), even the smallest overhead is critical.

#### Implementation Complexity and Scaling WebSockets

**1. Reconnection Logic:** While Long Polling and SSE inherently handle or allow automatic reconnection, WebSocket reconnection logic **must be manually implemented**.

**2. Load Balancer and Direct Connection:** WebSocket connections are **not maintained through the load balancer**; the LB is used only for establishing the initial connection. Afterward, the client and WebSocket server maintain a **direct connection**. Reconnection must go through the LB again.

**3. Shared Updates:** When using multiple servers, updates must be shared, similar to Long Polling and SSE. This challenge can be solved using the **Pub/Sub pattern**.

**4. HTTP/TCP Connection Issues:**

- WebSockets do not use HTTP, meaning they miss out on benefits of HTTP/2 and higher, such as **multiplexing**.
- WebSockets require opening a **new TCP connection for each WebSocket connection**. In contrast, HTTP/2 reuses a single TCP connection for multiple HTTP connections. This is a potential inefficiency.

**5. Handling Multiple Tabs:** In scenarios like a chat application where users subscribe to multiple groups across multiple tabs, each tab opens its own independent WebSocket connection and TCP connection. This can lead to significant inefficiency (e.g., 8 TCP connections for 2 tabs, each with 4 groups).

**Methods to Reduce Connections:**

- Switch to Long Polling or SSE (which utilize HTTP/2 multiplexing).
- Use **Shared Workers** (limited browser support).
- Use **Broadcast Channel** (maintaining the WebSocket connection in only one tab and sharing data between tabs).
- Use **browser storage** (session or local storage) combined with a subscription mechanism.
- Simply do nothing if independent TCPs are preferred.

**6. Infrastructure and Authentication Problems:**

- **Infrastructure:** Many components (proxy servers, LBs) are not configured for WebSockets and may block or ignore the protocol.
    - _Bypass:_ Use an **encrypted WebSocket connection (WSS)**, making it invisible to most proxy servers.
- **Authentication (Auth):** The WebSocket API does not allow setting the authorization header with a token. This is also a problem for SSE. Cookie authentication is not always suitable.

**Solutions for WebSocket Authentication:**

|Method|Mechanism|Security/Trade-offs|
|:--|:--|:--|
|**Query Parameters**|Send the access token in the URL query parameters. Connection refused if the token is invalid.|Not the most secure: **Server logs may still record plain text parameters**. Safer if WSS is used, as the URL is encrypted in transit and not visible in the browser URL bar.|
|**Short-Lived Tokens**|A separate server issues a short-lived token to the client before the WebSocket request.|Highly secure: If intercepted, the token's short time-to-live makes it useless. Downside: Requires an **additional service** to handle WebSocket-specific tokens.|
|**Send Token with First Message**|The token is sent via the WebSocket channel after the connection is established; the server then verifies it.|More secure than query parameters, simpler than short-lived tokens. **Vulnerable to DoS attacks**, as the connection is established before the token is checked.|
|**Cookies**|Simplest solution, as the connection begins with an HTTP handshake.|Requires the WebSocket server and web application to be deployed on the **same domain**.|
|**Sec-WebSocket-Protocol Header**|Initial HTTP request includes this header to transmit a token.|Not designed for security (could be read as plain text and logged by proxy servers). Lack of standard handling may cause unpredictable behavior.|

#### Recommendations for WebSockets

1. Use **Frameworks** that offer multiple transport protocols with automatic fallback (e.g., **Socket.IO defaults to Long Polling** if WebSockets are unsupported).
2. Always deploy the real-time backend over an **encrypted TLS connection** (improves security and helps bypass issues with proxies and load balancers).
3. Prefer using a WebSocket framework or library to handle numerous edge cases.
4. Avoid using WebSockets unconsciously; they are best suited for applications requiring **ultra-fast full duplex communications** (like online games).

---

## III. Summary of Approaches

The following table summarizes the key characteristics, use cases, advantages, and disadvantages of the four approaches discussed.

| Approach                     | Use Cases                                                                                                                                                 | Key Advantages                                                                              | Key Disadvantages                                                                                                                                     |
| :--------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Short Polling**            | Updates at fixed intervals (e.g., every 30 seconds or 1 minute). Suitable for **non-real-time requirements**. Example: analytic apps, stock applications. | **Simplicity**.                                                                             | **Inefficient** in terms of network traffic. **Doesn't provide real-time updates**.                                                                   |
| **Long Polling**             | **Near to real-time updates**. Example: analytic apps, stock apps, social networks.                                                                       | Operates over **HTTP** (benefiting from HTTP/2 features). **Broad browser support**.        | Potential **delays between updates**. **HTTP overhead**.                                                                                              |
| **Server Sent Events (SSE)** | **Real-time updates**. Example: analytic apps, stock apps, social networks, possibly chat applications.                                                   | **HTTP based**. Offers unique features (reconnection, event ID, **connectionless pushes**). | **HTTP overhead**.                                                                                                                                    |
| **WebSockets**               | **Bidirectional Communications**. Example: collaborative tools (Google Docs/Figma), multiplayer games, chat applications.                                 | **Highly efficient** in network traffic. **Minimal delay**.                                 | Operates over the **WebSocket protocol** (potentially causing infrastructure compatibility issues). **Lacks HTTP/2 core features** like multiplexing. |

--------------------------------------------------------------------------