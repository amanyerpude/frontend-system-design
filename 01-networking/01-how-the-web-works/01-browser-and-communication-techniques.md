
Video_Link : https://youtu.be/M2RpzmyKfvQ?si=BOTDNE2tbRJE00GG

--------------------------------------------------------------------------

The following provides a detailed, organized explanation of the content from the source, covering the introduction, browser mechanics, communication techniques, and network concepts discussed in the "Frontend System Design Yatra Season 1" video series.

---

## Frontend System Design Yatra Season 1 Overview

This series, "Frontend System Design Yatra Season 1," serves as an introduction video and outlines the topics covered.

### Series Content and Resources

The series focuses on several key areas:

1. **Browser Mechanics:** How the browser works.
2. **Network Protocols:** Discussion of various network protocols.
3. **HTTP Status Codes**.
4. **Communication Techniques (Hands-on Experience):** How to connect a client and server or connect to backend microservices. Techniques covered include:
    - **REST APIs**.
    - **GraphQL**.
    - **gRPC** (Google Remote Procedure Call).
    - **tRPC** (TypeScript Remote Procedure Call).
    - **WebSockets**.
    - **Polling techniques** (Short polling and Long polling).
    - **WebHooks**.

**Available Resources:** Attendees are provided with several resources:

- A link to the **xcal draw sheet**.
- A link to the **GitHub repository** containing hands-on code for all discussed techniques (GraphQL, gRPC, Long Polling, REST, Short Polling, RPC, and WebSockets).
- References to **many free articles** present on the web, with links embedded in the video description, including references for images used to give credit.

**Content Distribution:** This video was initially planned as a bonus section for the NextG Yatra series, but moving forward, **all bonus content will be uploaded on YouTube free of cost** for everyone.

---

## Browser Components and Functionality

The first module discusses the components of a browser.

### Components of a Browser

The browser is represented as a square box containing various components:

|Component|Function / Description|Details|
|:--|:--|:--|
|**UI Layer**|The visible front end of the browser.|Includes the address bar, back/next buttons, home button, refresh/stop buttons, and bookmark option.|
|**Browser Engine**|Acts as a **mediator/bridge**.|Coordinates between the UI layer and the rendering engine, and also communicates with the data storage component.|
|**Rendering Engine**|The **most crucial part** and sometimes asked about in interviews; considered the heart of the browser.|Responsible for painting content onto the screen.|
|**Data Component (Data Layer)**|Handles everything related to storage.|Includes persistent data (bookmarks) or temporary/partial data (cookies, cache, local storage, sessions).|
|**Network Layer**|Component responsible for network connections.|Makes **HTTP requests, WebSockets, and WebRTC connections**. Uses **Real-time Transport Protocol (RTP)**, which utilizes **UDP** (User Datagram Protocol).|
|**JS Interpreter**|Interprets and executes JavaScript code received from the server.||
|**UI Backend**|Considered the **brain of the UI layer** interface.|Defines the functionality of basic widgets like combo boxes and windows (e.g., defining the functionality of the address bar).|

### How the Browser Renders a Page (Rendering Engine Steps)

The process of how a browser renders a page is complex and involves several crucial steps:

#### 1. Request and Response (The Initial Phase)

When a request is made to the server, the response received consists of three basic blocks: **HTML, CSS, and JavaScript**.

- **CSS Nature:** CSS is **render blocking**.
- **JavaScript Nature:** JavaScript is **parser blocking**.
- Both CSS and JavaScript can trigger blockage at any point during the process, coming into effect as soon as they are found when the code reaches that line.

#### 2. Parsing Step (Building the Models)

This phase involves interpreting the HTML and CSS:

- **Building the DOM (Document Object Model):** The HTML is parsed first to build the DOM, which is a **tree-like structure** representing the HTML elements (e.g., `<html>`, `<head>`, `<body>`, tags, nesting levels).
- **Building the CSSOM (CSS Object Model):** Once the HTML is parsed, the browser starts parsing the CSS. The CSSOM is created by attaching styles to the DOM nodes. Styles can be **inherited** (e.g., font size from the body to children) or **directly attached**.
- **Parser Blocking (JavaScript Effect):** If the browser encounters a JavaScript file during the parsing step, it will **stop the parsing** and execute the JavaScript first. This is why heavy JavaScript files can delay screen display.

#### 3. Combining Models (Render Tree)

The DOM and CSSOM are combined to form the final **Render Tree**.

- **Optimization:** The Render Tree is an **optimized version** of the DOM and CSSOM combined.
- **Exclusions:** It lets go of unnecessary trees (or leaves) which are not required for rendering, such as `<head>`, `<meta>`, and `<link>` tags.
- **Display None:** Elements that have the `display: none` CSS property applied are also excluded from the Render Tree, even if they contain content.
- **Specificity:** It superimposes required styles, determining the final styling based on CSS specificity (e.g., a directly assigned font size takes precedence over an inherited one).
- **Render Blocking (CSS Effect):** If, during the rendering process (building the Render Tree and subsequent layout steps), new CSS overrides are encountered, the rendering process will pause, requiring the CSSOM to be rebuilt and the Render Tree recreated.

#### 4. Rendering Process (Layout and Paint)

- **Layout (Wireframes/Blueprint):** Using the Render Tree as a guide, the browser starts building the layout, creating **wireframes** or a blueprint of the page. This step involves reserving specific spaces (in forms of boxes with heights and widths) in the viewport for elements (like P tags and Div tags). This is analogous to creating stencils on a canvas.
- **Paint Step (Painting):** The next step is the **paint step** (or paint tree), where the wireframes (stencils) are filled with colors. This is the part where items start becoming **visually visible** on the screen.
- **Composting/Layering:** If there are multiple layers, such as modals, ads, or elements with Z-index properties, the browser performs a **composting step** (or layering). This involves creating multiple wireframes and then superimposing them on top of each other according to the required configuration.

---

## Network Protocols and Status Codes

### Network Protocols

The source discusses eight popular network protocols:

1. **HTTP (Hypertext Transfer Protocol):** Ideal for web browsing.
    - Requires establishing an initial **TCP connection**.
    - Data exchange happens in the form of **data packets**.
2. **TCP (Transmission Control Protocol):** Establishes a connection using a **sync and acknowledgement** process (client sends sync, server replies sync+acknowledge, client replies acknowledge).
    - Data is broken down into smaller packets.
    - Offers a **very low chance of losing a data packet**.
3. **UDP (User Datagram Protocol):** Used by devices requiring **very high internet bandwidth** (e.g., IoT, virtual reality, video conferencing).
    - Involves a direct connection between client and server.
    - **High chances that data may get lost** in between, as it does not guarantee 100% delivery.
    - Video conferencing often chooses **speed over guaranteeing data packets** (allowing momentary frame freezing or jank).
4. **HTTP/3:** Uses **UDP connection** instead of TCP to ensure fast data transmission, typically for high-bandwidth networks like IoT and VR.
5. **SMTP (Simple Mail Transfer Protocol):** Used for sending mails; the client sends mail to the SMTP server, which forwards it to the receivers.
6. **FTP (File Transfer Protocol):** Used for transferring files, such as uploading projects or deploying them onto servers (e.g., using FileZilla).
7. **WebSockets:** Establishes a **full duplex connection**.
    - First initiates an HTTP connection, then keeps that **single connection open**.
    - Allows sending and receiving data in **real-time** over that single connection.

### HTTP Status Codes

HTTP status codes are categorized into five pillars based on the outcome of a request:

|Category (XXX)|Description|Example Use Case|
|:--|:--|:--|
|**1xx**|**Informational codes**.|Server has acknowledged and is processing the request.|
|**2xx**|**Success codes**.|Server has received, understood, and processed the request (typically sending back data).|
|**3xx**|**Redirectional codes**.|Server received the request but there is a redirect to somewhere else; may require additional action.|
|**4xx**|**Client error codes**.|Errors that occurred on the client side (e.g., 404 page not found, 403 authorization error).|
|**5xx**|**Server error codes**.|Client made a valid request, but the server failed to complete the request.|

---

## Communication Techniques (REST, GraphQL, tRPC, gRPC)

### 1. REST APIs (Representational State Transfer)

REST is defined by specific rules or specifications:

- **Client-Server Independence:** The client application and server application must be able to operate **independently** of each other; there is no dependency or linkage.
- **Self-Contained Requests:** Each request from the client must contain **all information needed** to understand and process the request.
- **Cacheable Responses:** Responses should be defined as cacheable or non-cacheable (optional).

**Implementation Paradigm (RESTful Architecture Rule):** In a RESTful architecture, the endpoint (e.g., `/API/problems`) often **remains the same**, and only the **type of request changes** (GET, POST, PUT, PATCH, DELETE).

|REST Method|Purpose|Implementation Detail|
|:--|:--|:--|
|**GET**|Retrieving data from a server.|Endpoint typically does not require a body.|
|**POST**|Creating a new resource.|Data is passed in the body of the request.|
|**PUT**|**Completely updating** a specific resource.|Requires an ID in the endpoint to identify the resource (`/API/problems/:ID`).|
|**PATCH**|**Partially updating** some resource (e.g., only updating the title or description).|Similar structure to PUT, requiring an ID and updated data in the body.|
|**DELETE**|Deleting a specific resource.|Requires an ID in the endpoint; typically no body is required.|

### 2. GraphQL (Graph Query Language)

GraphQL allows clients to request and receive **only the specific data required**, avoiding over- or under-fetching.

- **Single Endpoint:** Instead of separate API endpoints for different types of data (e.g., players, teams, matches), GraphQL uses a **single endpoint** that listens for all queries.
- **Flexibility:** Clients can fetch any combination of data (e.g., players and matches but not teams) by changing the query sent to the server.

**Key Architectural Concepts:**

|Concept|Description|
|:--|:--|
|**Schema/Type Definitions**|Defines the structure of the data (e.g., `type User`, `type Problem`) and the relations between them (e.g., `User` has an array of `Problem`, `Problem` has an array of `User` solvers). This uses the `gql` imported utility.|
|**Query Type**|Defines the API endpoints the client is allowed to run (e.g., `users` returning an array of `User`, or `problems` returning an array of `Problem`).|
|**Resolvers**|Functions that define what query or database call needs to run when a client requests a specific field. Resolvers are crucial for generating data for **relational fields** (like populating problems within a user object).|

### 3. tRPC (TypeScript Remote Procedure Call)

tRPC is used to connect a client and a server, leveraging **TypeScript heavily**.

- **TypeScript Requirement:** It requires both the client and the server to use TypeScript.
- **Goal:** To ensure that if there is an error in the API schema on the server, the client is **automatically notified** by TypeScript before compilation, allowing for faster development ("move fast break nothing").
- **Mechanism:** tRPC works best when the client and server reside in the **same repository** so they can share the TypeScript methods (the `appRouter`).
- **Procedures:** Uses `query` procedures to fetch data and `mutation` procedures to update data.

### 4. gRPC (Google Remote Procedure Call)

gRPC was primarily developed by Google to **connect two microservices together** (e.g., a User service communicating with E-commerce, Prime Video, and Alexa services).

- **Language Agnostic:** gRPC can be written in **any language** (e.g., Python, Java, Go, Node.js).
- **Protocol Buffers (Proto Buff):** Uses `.proto` files to define the **contract of the data** (schema) that will be sent or transmitted between services. If a field defined in the `.proto` file (e.g., `description`) is not sent in the request, gRPC will automatically attach the property, possibly leaving it empty.
- **Communication:** Data is structured and serialized using Protocol Buffers, and transmitted using **HTTP/2**.
- **Backend Focus:** gRPC communication occurs at the **backend level** (backend service communicating with another backend service); the client is not necessarily a web browser.

### tRPC vs. gRPC Comparison

|Feature|tRPC|gRPC|
|:--|:--|:--|
|**Focus**|Connecting **client and server** (typescript client to typescript server).|Connecting **two microservices** together.|
|**Language Support**|Requires **TypeScript**.|**Language agnostic** (can be used by any language).|
|**API Contract**|Uses **TypeScript functions** between client and server.|Uses **Protocol Buffers** as the data contract.|
|**Safety/Efficiency**|Uses TypeScript safety for APIs.|Uses binary serialization (often more efficient).|

---

## Communication Techniques (Polling and WebSockets)

### 5. Short Polling

Short polling involves the client initiating requests to the server at **regular, fixed intervals**.

- **Mechanism:** The client makes a request, receives a response (which might contain an update or be empty), and then waits for the defined interval (usually less than a minute) before making the request again.
- **Use Cases:** Best suited for scenarios where the required **screen time is very less** (e.g., Disney Hotstar QR code login screen, where a user only stays briefly).
- **Drawback:** It is **not very scalable** or cost-effective because the client repeatedly makes heavy TCP calls, leading to a high cost on the server (imagine thousands of requests per user).

### 6. Long Polling

Long polling is similar to short polling, but it seeks updates only when an event occurs or the request times out.

- **Mechanism (Blocking Request):** When the client makes the first request, the server **blocks that request** and keeps it open until one of two conditions is met:
    1. The server responds back with new data (the event/update happened).
    2. The request times out.
- **New Request Cycle:** Once a response is sent (or timeout occurs), the client immediately initiates a **new blocking request**.
- **Implementation:** Often implemented using an **event emitter** model (pub/sub pattern).
- **Advantages over Short Polling:** It avoids unnecessary API calls, reducing cost on the server.
- **Drawbacks:** It consumes server bandwidth by maintaining an **always-open TCP connection** with the server.

### 7. WebSockets

WebSockets is an advanced technology enabling a **two-way interactive communication session** between the user's browser (client) and the server.

- **Mechanism:** It establishes a single HTTP connection, which is then kept open to facilitate **bidirectional messaging**.
- **Key Difference from Polling:** WebSockets establish a **single HTTP connection** for two-way communication, whereas polling techniques require creating **multiple HTTP connections** for back-and-forth data exchange.
- **Cost and Scalability:** Because only one persistent connection is established, the **server cost is generally less** compared to polling techniques.
- **Use Cases:** Required for scenarios demanding **real-time data** (e.g., chat applications, stock exchange platforms like Zerodha).
- **Control:** Either the client or the server can close the socket connection, disconnecting the entire HTTP connection.

### 8. WebHooks

A WebHook is an **HTTP request triggered by an event** in a source system and sent to a destination system, often including a data payload.

- **Automation:** WebHooks are **automated**; they are automatically sent out when their corresponding event is fired.
- **Mechanism (Asynchronous API Call):** A WebHook is essentially a **normal API** (often a POST request) on your backend server that is designed to be called by a **third-party vendor**.
    - _Example:_ When a user initiates a payment on an e-commerce platform using Stripe/PhonePe, the platform doesn't wait for confirmation. Once the payment is completed, the vendor (Stripe/PhonePe) calls a specific API endpoint (the WebHook) on the e-commerce server asynchronously, passing transaction data (order ID, user ID, amount).
- **Security:** WebHooks are generally very secure, typically implemented behind **secret keys** (sent in headers for authentication) and **whitelisting** of vendor domains to prevent malicious users from hitting the API.

--------------------------------------------------------------------------