
Video_Link : https://youtu.be/5kBI1LadXVA?si=Mwb3sKN2TtyZmdNX

--------------------------------------------------------------------------

This response draws upon the provided source material, discussing the web communication protocols GraphQL, gRPC, and tRPC, including their definitions, advantages, disadvantages, specific problems, and use cases.

The content presented is the second part of a video series about web communication protocols.

---

## I. GraphQL: Query Language and Runtime

**GraphQL Definition and Functionality**

GraphQL is defined as a query language and runtime for APIs. Simply put, it allows clients to query the server for specific data they need and receive _only_ that data in response, unlike the fixed set of data returned by the traditional REST approach. Clients can specify the desired data structure.

### A. Advantages of GraphQL

1. **Reduced Resource Consumption:** Clients get only the data they need in a single query.
2. **Solves Over and Under Fetching:** This efficiency eliminates the problems of over-fetching (getting unnecessary data) and under-fetching (needing multiple queries).
3. **Usability:** Developers do not need to memorize hundreds of endpoints and parameters; there is only one endpoint, and the client describes exactly what is needed.
4. **Static Analysis:** GraphQL is schema-based, meaning developers always know exactly what they receive from the backend, eliminating the need to validate responses.
5. **Automatically Generated Documentation:** GraphQL solves the problem of data documentation and provides a rich link structure.

### B. Trade-offs and Problems of GraphQL

While GraphQL solves some problems associated with REST, it introduces its own trade-offs.

|Trade-off/Problem|Description|Source|
|:--|:--|:--|
|**Learning Curve**|It is a relatively new approach compared to REST, requiring additional resources for implementation and maintenance because developers may not be familiar with the technology or best practices.||
|**Boilerplate and Bundle Size**|GraphQL, especially with frameworks like Apollo, requires more code, leading to a larger bundle size.||
|**Network Level Cache**|GraphQL typically uses a POST request to a single endpoint, meaning browser caching is lost.||
|**Security Issues**|GraphQL APIs are vulnerable to denial of service (DoS) attacks due to their flexible nature.||
|**Complex Backend Implementation**|It is much more difficult on the backend side to create a database query that is both performant and flexible.||

### C. Detailed Problem 1: Building Queries on the Backend (The N+1 Problem)

GraphQL is more difficult to implement on the backend because backend developers must solve the **N+1 problem** when building queries to the database.

- **Rest (Dedicated Endpoints):** With REST, backend developers can request all data in one request to the database because all requirements are known in advance, allowing for optimized data fetching. However, these endpoints are not flexible; they cannot query more or less data than what they are designed to provide. In REST, a request for an article, comments, and a user would be three different endpoints (e.g., `get article`, `get comment`, `get user`).
- **GraphQL (Single Endpoint):** All requests (e.g., article with fields and comments; only comments; only a user) go through a single endpoint. Backend developers cannot predict which data will be requested, preventing them from optimizing data fetching as they would with dedicated endpoints.

**Impact on Frontend Developers:** If the N+1 problem is not solved, it leads to slower handling of requests and a worse user experience. If the backend manages the problem, it requires additional resources to solve.

#### Solution to the N+1 Problem: Batching

The solution is using **batching**.

- Batching defers resolving queries to a later stage when all necessary data is available to perform a batch query.
- A batch query is a group of queries made in one SQL request.
- Instead of making multiple (e.g., three) requests to the database, the server waits until all needed data about the request is received, and then executes a single request to the database.

### D. Detailed Problem 2: Security Issues (Denial of Service)

If GraphQL is public, deeply nested requests are possible. Such requests, especially if run in parallel, can slow down the CPU or crash the application. This is called a **denial of service (DoS) attack**. This problem is related to GraphQL's flexible nature.

- _Contrast with REST:_ With REST, DoS attacks based on complex queries are avoided because each endpoint is static and returns only a predefined set of data.

#### Solutions to Security Issues

1. **Avoid Using GraphQL for Third-Party APIs:** Instead, use REST, as REST endpoints are designed to be cheap in terms of resources (even though they are not flexible).
2. **Use Quotas:** Limit deep requests.
3. **Use Pagination:** Limit the amount of data requested (e.g., the YouTube API uses quotas to limit data).
4. **Apply Throttling:** Prevent users from sending requests too frequently.

### E. Detailed Problem 3: Network Level Caching (Lack of it)

GraphQL is protocol independent but is mostly used over HTTP. When used with HTTP, it is often utilized through a single POST endpoint, making network-level caching difficult.

- _Contrast with REST:_ REST uses the method and URL as a unique identifier to bind a response to a request.
- _GraphQL Issue:_ This is not possible in GraphQL because the server sees only one POST method and a single endpoint, so all requests look the same to the server.

#### Solution to the Caching Problem: Persisted Queries

One solution is using **persisted queries**, often offered by GraphQL servers such as Apollo server.

1. **Initial Request:** The client sends both the query string and a hash to the server.
2. **Server Action:** The server perceives and executes the query string and hash, then returns the result.
3. **Subsequent Request:** After some time, the client sends only the hash of the query string to execute.
4. **Server Response:** The server finds the persisted query string and returns the result.

This improves performance because it avoids an additional request to the database. While browser caching doesn't work for GraphQL due to the issues discussed, there are many solutions to implement client-side caching for GraphQL.

### F. When to Use GraphQL (Use Cases)

GraphQL may be very useful in specific scenarios:

1. **Loosely Coupled Backend and Frontend Teams:** When teams work closely together or are a single full-stack team, GraphQL’s flexibility may be unnecessary because endpoints can be customized easily. The opposite is true: GraphQL is more preferable for a public API, where the backend cannot predict who will use the API or how. However, REST remains the most distributed solution for public APIs as not all customers want to use GraphQL.
2. **Multiple Clients:** When an API server has multiple clients, flexibility is crucial, and using the most flexible solution is advisable since the API cannot be adjusted to be convenient for all clients.
3. **Mobile Clients:** Under- and over-fetching, which might not be a major issue on desktop due to unlimited and fast internet, can be a problem on mobile phones. If a significant portion of customers use the application from mobile phones, GraphQL should be considered.
4. **Applications with Complex Data Relationships:** If using the API becomes difficult, requires many requests to receive data, or necessitates frequent requests to the backend team to adjust endpoints, GraphQL implementation might be beneficial.

### G. When to Choose Something Else (Anti-Use Cases)

In the following cases, choosing another solution is advisable:

|Scenario|Rationale/Alternative|Source|
|:--|:--|:--|
|**Simple API**|If the API mostly adheres to the CRUD paradigm and over/under fetching is not critical, REST adds less extra complexity than GraphQL.||
|**Limited Resources**|GraphQL requires additional knowledge to implement and maintain, making it less cheap in terms of resources.||
|**Files/Binary Data**|GraphQL was not designed for this type of data. Binary data can only be passed as a base64 string, which increases file size and imposes a transfer limit of 380 megabytes.||
|**Conflict-Free Replicated Data Types (CRDTs)**|Collaboration tools like Google Docs or Figma require CRDTs, and GraphQL was not designed to solve these problems.||
|**User Authentication**|Modification includes redirects, cookies, and HTTP headers, all of which are inconsistent with GraphQL's design as a protocol-agnostic solution. Additionally, types are usually not needed for authentication as the request contains only two fields (login and password).||
|**Server-to-Server Communication**|These communications require high speed, low memory, and traffic consumption; gRPC and Protocol Buffers (protobuf) are much more efficient.||
|**Streaming Video/Audio/File Transfers**|These processes do not benefit from GraphQL features.||

---

## II. Remote Procedure Call (RPC) Approaches

**RPC Definition and Concept**

RPC, or **Remote Procedure Call**, is a communication pattern allowing client applications to call procedures or functions on a remote server. RPC is an umbrella term referring to this concept, but it does not define a specific protocol; implementations may use different communication protocols.

### A. RPC vs. REST

The main difference between REST and RPC is how they describe interactions:

- **REST:** Describes _entities_ (e.g., `GET users` to retrieve users).
- **RPC:** Describes _actions_ or _functions_ (e.g., performing `get users` to do the same thing).

### B. General RPC Advantages and Disadvantages

RPC should be considered when frontend and backend teams work together or if it is a single team of full-stack developers.

|Advantage (Compared to REST)|Applicable Implementation(s)|Source|
|:--|:--|:--|
|Improves Performance|gRPC only||
|Improves Reliability|gRPC and tRPC||
|Makes the API more convenient|gRPC and tRPC||
|Reduces Latency|gRPC only||

**Disadvantages:** RPC approaches lack flexibility. This lack of flexibility makes RPC a poor choice for public APIs, though it is acceptable if the API is not used by other teams.

---

## III. gRPC (Google RPC)

**gRPC Definition and Protocol**

gRPC is a specific implementation of RPC developed by Google. It uses **HTTP/2** as a transport protocol and **Protocol Buffers (protobuf)** for data serialization. gRPC uses protobuf instead of JSON.

### A. gRPC Advantages

1. **Better Performance:** Achieved by using protobuf and HTTP/2.
2. **Strong Typing and Validation:** Provided by the protocol.
3. **Code Generation:** Allows for both client-side and server-side code generation.

### B. gRPC Disadvantages

1. **Unfamiliar to Frontend Developers:** Requires additional learning time.
2. **Difficult Debugging:** Protobuf is not a human-readable format like JSON.
3. **Browser Support Issues:** gRPC is **not supported by browsers** because it requires access to low-level HTTP/2 elements. Additional resources are needed to make it workable.
4. **Struggles with Underfetching:** Fields and entities cannot easily be added to the response.

### C. Using gRPC in the Browser (Proxy Solution)

While gRPC is not natively supported by browsers, it can be used non-natively via a **proxy** between the client and server.

1. The client sets up a gRPC client, which sends a request to the gRPC web proxy using HTTP/1 or HTTP/2.
2. The proxy is needed to transfer the request to HTTP/2 and send it to the original gRPC backend.

Using gRPC in the browser is possible but difficult, which is one reason it is not widely distributed for client-server communications.

### D. gRPC Use Cases and Anti-Use Cases

|Scenario|Rationale|Source|
|:--|:--|:--|
|**Use Case: Communications between Microservices**|gRPC was designed for these interactions, providing better performance and smaller message sizes.||
|**Use Case: Client-Server with High Data Needs**|Consider gRPC when dealing with high data loads where high data transfer speed is critical.||
|**Anti-Use Case: Public API**|Rest or GraphQL should be used instead due to the lack of flexibility.||
|**Anti-Use Case: Most Client-Server Oriented APIs**|While technically possible, implementing a solution for cases it wasn't designed for can lead to unpredictable challenges (lack of documentation, community support, infrastructure issues).||

---

## IV. tRPC (TypeScript RPC)

**tRPC Definition and Protocol**

tRPC (TypeScript RPC) is an implementation of RPC designed for **TypeScript monorepos**. It uses **JSON RPC**. It facilitates client-server interactions. tRPC relates to features quite opposite to gRPC: it is built on TypeScript, does not imply using other languages, and is mostly used for client-to-server communications.

tRPC is considered a more obvious alternative to REST and GraphQL for certain scenarios. Developers simply call functions, and tRPC handles the complexities of transferring data and receiving responses. The backend team is responsible for preparing data for the client.

### A. tRPC Advantages

1. **Shared Types:** Shared types between the frontend and backend.
2. **Full Static Type Safety:** Provides full static type safety.
3. **Autocompletion:** Offers auto-complete on the client side for inputs, outputs, and errors.
4. **Ease of Development and Maintenance:** It is easy to develop and maintain.
5. **Low Cost:** It is simpler and has much lower implementation costs compared to GraphQL or gRPC.
6. **No Additional Infrastructure/Language Required:** It does not require additional infrastructure (like gRPC) or learning a new query language (like GraphQL).
7. **Increased Reliability:** Shared contracts mean developers do not need to care about contract updating, which is a cheap way to increase application reliability (provided types are used correctly and keywords like `any` are avoided).

### B. tRPC Trade-offs and Limitations

1. **TypeScript Requirement:** Both the client and server must use TypeScript.
2. **Underfetching:** Underfetching is still a problem; fields and entities cannot be added to the response.
3. **No Runtime Validation:** tRPC does not perform runtime validation. TypeScript types disappear after the build process, so tRPC only shares TypeScript contracts between client and server. Using the `any` keyword can still crash the application.

### C. tRPC Use Cases and Anti-Use Cases

|Scenario|Rationale/Alternative|Source|
|:--|:--|:--|
|**Perfect Scenario: TypeScript Monorepos**|The primary requirement is that both client and server must use TypeScript. Convenience deteriorates as client and server become distant from each other.||
|**Anti-Use Case: Public API**|tRPC binds the user to TypeScript, making it inflexible and unsuitable for clients that do not use TypeScript; REST or GraphQL should be used.||
|**Anti-Use Case: Complex Responses**|When working with complex structures and multiple entities, using GraphQL might be preferable.||
|**Anti-Use Case: Simple API**|For simple APIs, the tRPC typing system is not very convenient, and REST should be used instead.||
|**Anti-Use Case: High Efficiency Requirement**|tRPC does not offer speed advantages when high efficiency is very important.||

---

## V. Addressing Underfetching in RPC

Underfetching is a problem for both gRPC and tRPC because the procedures are static and cannot be adjusted to the client's needs.

- **For gRPC:** Over-fetching is mitigated by using **field masks**. A field mask is a set of symbolic field paths (e.g., `article.out.name` returns the name of the article author) used to fetch only the required fields.
- **For tRPC:** Underfetching can be addressed by passing the required fields as a **function parameter** or by adding an **additional method** to return a different set of fields.
- **Contrast with REST:** While requested fields can be put in a REST URL, this may run into URL length limitations. Additionally, GET requests do not allow the use of the body to pass data.

The video concludes by mentioning that the next part will summarize the lessons learned and look at real-life examples.

--------------------------------------------------------------------------