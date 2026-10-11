
- Deep dive into HTTP 
	- Video_Link :  https://youtu.be/gxF9fLo5XQw?si=ykqnifO9duH4L4Ck

--------------------------------------------------------------------------

This response draws comprehensively upon the source material provided, detailing every mention of structure, history, terminology, and concept related to the HTTP protocol and its evolution.

The discussion is split into two videos; the first video provides a general overview of the HTTP protocol, its position in web architecture, deficiencies of early versions, and performance tricks engineers use.

---

## I. Overview of Web Mechanics and Architecture

### A. Initial Web Request Steps

When a user wants to fire up a browser request (e.g., to `front-end engineer.io`), the following general steps occur:

1. **DNS Translation:** The browser requests the real address from a **Domain Name System (DNS) server**. The DNS server translates the human-friendly name (`front-end engineer.io`) to the **machine-friendly IP address**.
2. **TCP Connection:** The web browser asks the computer to open a **Transmission Control Protocol (TCP) connection** over the IP address.
    - This typically happens over **Standard Web Port 80** if the connection is HTTP.
    - If the connection is HTTPS, it happens over the **Standard Security Web Port 443**.
3. **HTTP Request:** Once the browser has a connection to the web server, it can start asking for the website. This step is where the **HTTP protocol comes in**.
    - If HTTPS is used, extra steps are required to set up encryption, but this part is skipped in the discussion for simplicity.

### B. Page Rendering Process

After the initial connection and request:

- The **initial page gets rendered** (e.g., `index.html`).
- Rendering begins while the browser is **parsing the page**.
- The browser may encounter the need to load **additional resources** like JavaScript files, CSS, images, and so on.
- **Critical Resources:** The page can only be rendered when all critical resources are fully loaded.
- **Post-Rendering:** After initial rendering is complete, background requests can be made, such as those initiated by the `onload` event or other asynchronous requests for additional resources.
- This schema repeats each time a new web page is requested.

### C. Definition and Requirements of HTTP

- **HTTP Definition:** HTTP stands for **Hypertext Transfer Protocol**.
- **Initial Intent:** It was initially intended to transfer **hypertext documents** (documents that contain links to other documents). The first version did not support anything else.
- **Evolution:** As the web progressed rapidly, developers realized other types of resources could be transferred, such as images, free files, and binary data.
- **Protocol Type:** HTTP is the **requester response protocol**. The web browser makes a request using HTTP syntax to the web server, which sends a message containing the requested resource.
- **Networking:** HTTP requires a network connection and works over **TCP/IP**. TCP/IP represents the physical connection, such as an Ethernet cable or Wi-Fi.

---

## II. HTTP and the OSI Model

The **OSI Model** (Open Systems Interconnection Model) is a conceptual model often used to describe the layered approach of networking, consisting of seven layers.

|Layer|Name|Description/Relevance|
|:--|:--|:--|
|**Layer 1**|Application Layer|Many applications are built on top of HTTP; this refers more to networking layers than to the JavaScript application itself.|
|**Layer 2**|Presentation Layer|Used to describe the response; determines what formats (e.g., JavaScript file, CSS, PNG) are supported and can be transferred/presented.|
|**Layer 3**|Session Layer|Provides security for the connection. This layer activates when working with HTTPS, setting up encryption for the connection.|
|**Layer 4**|Transport Layer|Represents the protocol used to transfer data (HTTP works over TCP).|
|**Layer 5**|Network Layer|Represents the Internet Protocol (IP) itself (TCP works over IP).|
|**Layer 6**|Data Link Layer|Represents how the physical signal is transferred to the device (via cable or Wi-Fi).|
|**Layer 7**|Physical Layer|Represents how the physical signal is transferred to the device (via cable or Wi-Fi).|

_Note: The discussion skips the Network, Data Link, and Physical layers (Layers 5, 6, and 7) as they are not relevant to the HTTP discussion_.

---

## III. Evolution of the HTTP Protocol

### A. HTTP 0.9

The protocol started "super simple".

|Feature|Detail|
|:--|:--|
|**Command**|The first and only possible command was the **`GET` request**.|
|**Syntax**|Simple `GET` request with a single parameter representing the path to the page.|
|**Response**|The response was represented by a **stream of ASCII characters**.|
|**Error Status**|**No error state** existed to determine if the response was successful or finished with an error (introduced in later versions).|
|**Immutability**|All requests were **idempotent**, meaning all requests were not mutable, and the same request always yielded the same response.|
|**Headers**|**No header support**.|

### B. HTTP 1.0 (Major Release)

HTTP 1.0 was a measure release that introduced many new features as the protocol became more complex.

|Feature|Detail|
|:--|:--|
|**New Methods**|Introduced `HEAD` and `POST`.|
|**HEAD Method**|Allows a client to fetch meta information for a resource **without downloading the resource itself** (e.g., a browser can get image size ahead of time to reserve page space).|
|**POST Method**|Allows the client to send data to the server, including content as part of the request, also known as a **request body**.|
|**Version Field**|Allowed specifying the protocol version (for backward compatibility, it was set to 0.9 for a while; currently, most clients use 1.1 as default).|
|**HTTP Headers**|Introduced and could be sent with the request and response. Examples of real-world headers include `Accept` (encoding supported) and `Accept-Language` (language supported). Headers are set as a **key-value map**.|
|**Response Codes**|Introduced status codes, enabling conditional requests and error statuses. The code **404** is one of the best-known codes for a resource not found on the server.|

### C. HTTP 1.1 (Refined Version)

HTTP 1.1 is a refined version of HTTP 1.0 and is still used by many servers. The fundamental structure did not change between 1.0 and 1.1. Most features were based on the HTTP headers introduced in 1.0.

The two most notable changes were **mandatory headers** and **persistent connections**.

#### 1. Mandatory Host Header

- **Context:** Historically, it was assumed a web server would host only one website, making the host part of the URL obvious.
- **Modern Need:** Nowadays, web servers host several sites on the same server, a situation known as **virtual hosting**. It is important to tell the server which site the user wants to access.
- **Implementation:** Implemented by adding the **`Host` header** to the request.
- **Requirement:** While optional in HTTP 1.0, HTTP 1.1 made the `Host` header **mandatory**. A request missing this header is technically badly formed.

#### 2. Persistent Connections (Keep Alive)

- **Initial Problem:** Initially, HTTP was a singular request-response protocol; the connection was closed after the response. Since displaying a single page requires several HTTP resources, closing and reopening the connection caused unnecessary delays.
- **Solution:** The **`Connection` HTTP header** could be sent with the request.
    - **Client Action:** By specifying the value `keep-alive` in this header, the client asks the server to keep the connection open for additional requests.
    - **Server Action:** If the server supports persistent connections, it includes the `Connection: keep-alive` header in the response, signaling the client that it can send another request on the same connection once the response is complete.
- **Default Behavior:** HTTP 1.1 made persistent connection the **standard default**. It can be assumed that any HTTP 1.1 connection is using persistent connection, even without the explicit header in the request.
- **Closing Connection:** If the server needs to close the connection, it must **explicitly include the `Connection: close` HTTP header** in the response.

#### 3. Other HTTP 1.1 Features

- New methods such as **`PUT`**, **`OPTIONS`**, and the less used **`CONNECT`**, **`TRACE`**, and **`DELETE`** were introduced.
- **Better caching methods** were added, allowing the server to instruct the client to store resources (like CSS files) in the browser cache for later use. This is known as the **caching header** and must be configured on the server.
- **HTTP cookies** were introduced, allowing HTTP to move from being a kind of stateless protocol.
- New status codes.
- Proxy and authentication support.

---

## IV. Prerequisites and Fundamental Problems of HTTP 1.1

The need for HTTP/2 arose because the committee needed to solve many problems associated with HTTP 1.x.

### A. Context of Web Growth

- **Payload Size:** Average website size grew from **100 kilobytes to 5 megabytes**.
- **Average Requests:** The average number of requests per website is about **70**.
- **Complexity:** Websites became more complex, incorporating multiple frameworks, libraries, and dependencies.
- **Interaction:** Web pages shifted from static pages to interactive pages, making a lot of calls from client-side JavaScript. The HTTP protocol was not designed with this huge increase in resources in mind.

### B. Problem 1: No Parallel Resource Loading

- This problem stems from HTTP starting as a simple request-response protocol for basic HTMLs.
- **Sequential Loading:** The browser sequentially loads every resource (JS, CSS files, etc.) it sees in the HTML.
- **Latency Impact:** The web page is not displayed until all critical resources are loaded. This creates **significant latency** and time to render, causing a huge delay for the user.
- Users spend approximately **80 percent of the time** waiting for messages to travel across the internet.
- The impact is critical if a website contains **50 or more resources**. This was one of the largest prerequisites for HTTP/2.

### C. Problem 2: Potential Large Header Size and Compression

- **Format Inefficiency:** While HTTP message bodies can contain binary data, requests and headers must still be in **text format**. Text is great for humans but not optimal for machines.
- **Increased Size and Repetition:** Text format makes HTTP messages larger than necessary due to inefficient encoding and **repeating headers**.
- **Security Headers:** The use of headers has grown, with security headers (like Content Security Policy) producing extremely large headers.
- **Total Overhead:** For many websites made up of 10 or more requests, large HTTP headers can add **hundreds of kilobytes of transferred data**. This is especially critical in countries with high traffic costs.

---

## V. HTTP 1.1 Performance Workarounds

Despite the fundamental problems, workarounds were introduced to decrease the impact of protocol limitations.

### A. Workaround 1: Open More Connections

The easiest way to circumvent the blocking issue is to open more connections to allow parallelization.

1. **Multiple Connections:** Most browsers open **six connections per domain**. However, six parallel connections are often not enough.
2. **Domain Sharding (Follow-up Workaround):**
    - **Technique:** Static assets (images, CSS, JavaScripts) are served from subdomains.
    - **Result:** This allows web browsers to open up to six connections for _each new domain_.
    - **Implementation Note:** These domains are often hosted on the same server, sharing resources but using different domain names to trick the browser.

#### Downsides of Multiple Connections

- **Overhead:** Causes additional overhead for both the client and the server.
- **Resource Consumption:** Starting a TCP connection takes time, and maintaining it requires extra memory and processing costs.
- **TCP Inefficiencies:** The main issue is the significant inefficiency with the underlying TCP protocol.
    - **Guaranteed Protocol:** TCP is a guaranteed protocol that sends packets with unique sequence numbers and requests any lost packets by checking for missing sequence numbers.
    - **Three-Way Handshake Cost:** TCP requires a three-way handshake to set up:
        1. Client sends a **synchronous message (SYN)**, telling the server the sequence number to expect.
        2. Server sends a combined **SYN and acknowledge message (SYN/ACK)**, acknowledging the client's number and sending its own sequence number.
        3. Client sends an **acknowledge message (ACK)**, acknowledging the server's sequence number.
    - **Trip Cost:** This process adds **three network trips** (or one and a half round trips) before a single HTTP request can be sent.
    - **Mobile Impact:** This is a critical downside, especially for mobile users, where messages must travel across multiple cell phone towers, resulting in high latency.
- **Bandwidth Exhaustion:** Using multiple independent connections can result in bandwidth issues, leading to TCP timeouts or retransmissions on other connections.
- **Lack of Prioritization:** Unlike HTTP/2, HTTP 1.1 cannot use bandwidth efficiently because there is **no concept of prioritization** between the traffic on independent connections.
- **Final Costs:** This approach adds infrastructure and maintenance costs, consumes a great chunk of server/client resources, requires specific domain knowledge and expensive specialists, and causes higher battery consumption on mobile devices.

### B. Workaround 2: Make Fewer Requests

This approach focuses on reducing the total number of trips to the server.

|Strategy|Description/Benefit|
|:--|:--|
|**1. Caching**|The browser can cache resources from the server and reuse them later, provided the necessary caching headers are applied.|
|**2. Image Bundling (Spriting)**|Packing all icons into one large file, loaded in one go, with JS and CSS later used to extract the necessary icon. This helps reduce queuing when a website has many images (e.g., hundreds).|
|**3. CSS and JS Bundling**|Merging all critical resources into one file and serving them as a bundle (the reason for tools like webpack). This technique also involves compressing and minimizing the file.|
|**4. Inlining Critical Resources**|Serving CSS styles together with HTML by placing all critical styles required for the page into a `<style>` tag served with the HTML.|

#### Downsides of Fewer Requests (Bundling/Inlining)

- Increased traffic and file sizes.
- Clients with poor internet connections may suffer significant latency time.
- Not all code is used during runtime, leading to serving **unnecessary payloads**.
- **Caching is tricky:** Because bundling produces a new file every time code changes, the entire bundle must be redownloaded even if only small changes occur.

### C. Conclusion on Workarounds

The workarounds take time, money, and understanding to implement and maintain, and they ultimately add their own performance problems. Studies show that slower websites lead to abandonment and loss of visitors and sales. **HTTP/2 was introduced to fix these problems**.

---

## VI. Transition to HTTP/2

The next video in the series will discuss how **HTTP 2.0** is designed, why it is drastically different, and how concepts like **multiplexing** and **bi-stream prioritization** helped resolve the issues discussed in the first video.

--------------------------------------------------------------------------
