Video_Link : https://youtu.be/5kBI1LadXVA?si=l1HaTyd9o0i-JA_m

--------------------------------------------------------------------------

This response goes through the source in meticulous detail, ensuring all concepts, examples, warnings, and opinions provided in the text are included and cited accurately.

---

## I. Introduction and Context

This video is part of the System Design Building Block Series. The topic is protocols and patterns used to transfer data between the client and the server.

**Importance of the Topic:**

- Communication between client and server is knowledge needed in possibly every interview, regardless of the subject.
- The information is useful for frontend developers and mobile developers.
- The planned information was extensive, so the content was divided into three videos.
- This first video looks at the general problem of client-server communication and discusses **REST** (Representational State Transfer), which is described as the most distributed pattern for such communications.

## II. Types of Communication and API Usage

Before beginning, it is important to understand what types of communications exist. Classification is based on the consumer of the API.

### A. Communication Types

1. **Client-to-Server Communication:**
    - Involves a client (e.g., a browser, PWA, or mobile application) and a remote server providing the API.
2. **Service-to-Service Communication:**
    - The term "service to service" is preferred over "server to server" because a server is a physical computer that may host one or more services.
    - When the speaker refers to a "service," they mean the provided API, which may be hosted on one or many servers or in the cloud.

### B. Distinction: Client vs. Service

- The main difference is that a **service is not a human user**.
- On the client side, many people may need to read the response (e.g., programmers or the testing team).
- In client-server communications, **human ability and convenience** of using the API are sometimes more important than performance.

### C. Scope of API Usage

The next classification question is whether the API will be used only inside the company or if it is for public usage.

- **Private/First Party:** Used only inside the company.
- **Public/Third Party:** Used for public usage.
- The discussion primarily focuses on **client-server communication**, which can be public or private, and will cover which approach is more efficient in both cases.

## III. Protocols and Patterns (General Overview)

Available options for communication include, but are not limited to, these popular choices:

- **REST**
- **GraphQL**
- **gRPC**
- **tRPC**
- **WebSockets**
- **Others**

## IV. The Interview Signal and Choosing Protocols

For many developers, especially those who are not senior, using REST looks like the default option for API communication. They immediately jump into implementation details when asked to design an API.

**Goal in an Interview:**

- The main goal is to provide a clear signal.
- This signal must show that you know alternatives to REST.
- You understand the advantages and disadvantages of each approach.
- You know when to apply a protocol or pattern and when you shouldn't.
- The only exclusion is if the interviewer specifically asks about implementation details, in which case you should provide them. Otherwise, you risk getting deeply absorbed in nuances because the interviewer timeline is limited.

**Making an Explicit Choice:**

- REST is not necessarily a bad choice.
- What most developers call REST can be a good choice in many cases.
- Regardless of whether you choose REST, GraphQL, or something else, you must do it **explicitly**.
- This means telling the interviewer _why_ you chose that option, why it is better for the specific case, and don't forget to highlight its disadvantages and how you plan to manage them.

## V. Protocols Not Discussed in Detail

The following will not be discussed in this video:

1. **SOAP (Simple Object Access Protocol):**
    - A protocol that was popular before REST.
    - It is left out of scope due to its low popularity nowadays; the speaker knows zero applications using it, though they may exist.
2. **WebSockets:**
    - A protocol for **bidirectional transferring of data**.
    - It is separated because real-time messaging is a huge topic.
    - The speaker has a separate video discussing different options for real-time communications and WebSockets, linked on the top of the screen and in the description.

## VI. REST: Definition, Principles, and Architecture

### A. Definition

- REST stands for **Representational State Transfer**.
- It is an **architecture style** for designing networked applications.
- Its principles were outlined by **Roy Felding** in his doctoral dissertation (a link to which is provided in the description).

### B. The Six Principles of REST

Many developers use REST but do not know these principles, which the speaker calls a "first warning signal".

The principles are:

1. **Uniform Interface**
2. **Statelessness**
3. **Cacheability**
4. **Client-Server Architecture**
5. **Layered System**
6. **Code on Demand (Optional principle)**

#### 1. Uniform Interface

The goal is to make the API more discoverable and less coupled with the backend.

Implementation distinguishes three points:

- **Resource Identification:** REST APIs use Uniform Resource Identifiers (URIs) to address resources (e.g., a user identified by a URI like `api.example.com/users/123`).
- **Self-Descriptive Messages:** The server includes headers (like `Content-Type`) in the response, which is needed to correctly parse the message. The client does not need to know how to parse the message in advance, as all information is obtainable from the response.
- **HATEOAS (Hypermedia As The Engine Of Application State):** The server response includes hypermedia links (e.g., `api.site.com/user123`) guiding the client on how to perform further actions.
    - **Purpose of HATEOAS:** To provide not just the requested data, but also links or instructions on what actions can be taken next.
    - **Typical vs. HATEOAS:** Typically, developers request data, parse the response to get an ID, and send another request. HATEOAS implies returning a link to an entity instead of just its ID.
    - **HATEOAS Importance:** Makes the API more discoverable, reduces dependency on fixed URLs, and allows for dynamic changes without breaking client functionality.
    - **Client Independence:** With HATEOAS, the client is less dependent on the backend and documentation. Theoretically, a client can request a list of users and then request any other data by jumping by links in the responses without documentation.
    - **Requirement:** HATEOAS must be included in a **correctly built REST API**.
    - _Example:_ On a blog platform, the client requests user data (Name, ID, and links to articles); the client requests the list of articles via the link; the article response includes links to comments; the client receives the list of comments—all in three steps without referring to documentation.

#### 2. Statelessness

- The server does not store the client state between requests.
- Each request is **independent**.
- **Session Information:** Even if a session is used (which implies state on the server), it does not violate statelessness if session information (in HTTP header or cookies) is included in each request, and each request acts independently and does not refer to previous requests or responses.

#### 3. Cacheability

- Responses from the server should be labeled as **cacheable or non-cacheable**.
- **Cache Control Headers** (e.g., `Cache-Control`) are used to specify caching behavior.
- Caching can be implemented on multiple levels: in browsers, proxy servers, CDNs, etc..
- _Mechanism Example:_ Client requests data -> response might be cached on CDN/reverse proxy layer, proxy layer, and finally on the client layer (browser) if the server sets correct cache headers. (Caching in other layers is a separate topic).

#### 4. Client-Server Architecture

- The design keeps the **website view separated from the data**.
- This makes it easier to use the same view on different devices and helps websites adapt and grow without mixing things up.
- The client does not need to know about the construction of the server to interact with the API, making the client less dependent on backend realization.
- Backend realization is hidden from the clients.

#### 5. Layered System

- When using the internet, the client usually cannot tell if they are talking directly to the website or through an intermediary (like a proxy or load balancer).
- A middleman will not disrupt communication, and the client or website does not have to change anything.
- This principle hides the complex infrastructure between the client and the server and, like the previous principle, helps keep the client **independent from the backend realization**.

#### 6. Code on Demand (Optional)

- Servers can temporarily extend or customize the client's functionality by transferring **executable code** (e.g., JavaScript code).
- The server sends code that must be executed by the client.

## VII. REST vs. RESTful API

The terms REST and RESTful API may sometimes be confused.

|Term|Definition|
|:--|:--|
|**REST**|A pattern that includes the six principles discussed, which an API must be constructed with.|
|**RESTful API**|An API that follows REST principles.|

**Current Reality:**

- Today, REST is mostly a **buzzword**.
- Most APIs do not follow at least the **HATEOAS principle** and therefore cannot be called truly RESTful.

**What Developers Usually Call a "REST API":**

- The API uses **nouns** (not verbs) to describe entities.
- The API uses some semantic features of HTTP (e.g., `GET` for requesting data, `DELETE` for deletion), but sometimes not all (e.g., true REST requires `PUT` and `PATCH` for different cases).
- Data is most often transferred in **JSON format**. REST is protocol agnostic, so JSON or anything else can be used.
- It is not an RPC or GraphQL API.
- Since there is no special word for an API that is not GraphQL/RPC but does not follow all REST principles, it is simply called a REST API.
- _Unofficial Terminology:_ The term **"Rest-foolish"** can describe an API that uses some REST principles but may or may not follow them all.

## VIII. Advantages of REST

It is important to separate the advantages of REST as an architectural pattern from the advantages of REST applied over HTTP.

### A. Advantages of REST as an Architectural Pattern

1. **Flexibility:** Typically works over HTTP but can work over any other protocol.
2. **Compatibility with HTTP:** HTTP was designed to be suitable for REST, making REST very suitable for the web.
3. **Low Coupling:** The client and server are less coupled to each other if the REST API is built correctly.

### B. Advantages of REST over HTTP and JSON

1. **Simplicity:** REST provides a comprehensible set of rules.
2. **Familiarity:** HTTP is a widely distributed and familiar protocol.
3. **Human Readable:** JSON provides human-readable messages.
4. **Ease of Use:** These factors make the combination easy to integrate into an application and maintain.
5. **Infrastructure Support:** Most infrastructure knows how to work with HTTP, which additionally facilitates maintenance.
6. **Automatic Caching:** Browsers support HTTP caching, and if the server puts the correct cache header into the response, caching works automatically without special configuration.
7. **Ease of Debugging:** JSON is a human-readable format, and requests can be sent to the server from any client and tool.
8. **Broad Support:** There are many tools, packages, and libraries available to work with HTTP.

## IX. Disadvantages and Problems of REST

REST is still a popular solution for new projects, but it receives criticism.

1. **Over fetching:** Fetching too much data; data is included in the response that is not used on the client.
2. **Under fetching:** Not fetching enough data, forcing the client to call a second endpoint.
3. **Versioning Required:** REST requires versioning to ensure backward compatibility, which adds additional cost and complexity.
4. **Uncontrollable Number of Endpoints:** When the API becomes large, it becomes difficult to maintain endpoints and update documentation.
5. **CRUD Focus:** REST is mostly suited for CRUD (Create, Read, Update, Delete) operations. When creating a more complex API than CRUD, it becomes difficult to stay within the REST architecture and provide a clear, usable API.
6. **Historical Context:** Many disadvantages stem from REST being relatively old (introduced in 2000) when the web consisted of websites that were not as complex as modern web applications. This makes it difficult to implement complex scenarios correctly within REST boundaries.
7. **Lack of Validation:** REST doesn't have a mechanism to validate data. Unlike other options that have schemas or data typings, REST lacks a built-in solution to validate what the client receives from the backend.

## X. Detailed Problems: Fetching Issues

Over and under fetching are important problems for the discussion of alternatives.

### A. Over fetching (Fetching Too Much Data)

- **Definition:** Fetching data that is not needed on the client.
- _Example:_ Client needs only the username and profile image for a comment section, but the server sends all user fields when requesting user data by ID.
- **Consequences:** Inefficient use of user traffic, increased data transfer times, and unnecessary consumption of server resources.
- **Mitigation Note:** Over fetching may not be a major problem if the main audience is desktop users and the redundant data is not too large; in such cases, one could simply do nothing.

**Fixing Over fetching:**

1. **Use GraphQL:** Consider GraphQL instead of REST, especially if over fetching becomes a massive problem.
2. **Sparse Field Sets (Query Parameters):** Add required fields to the URL, separated by a comma.
    - _Problem:_ Max URL length (e.g., 2083 characters) might not be enough, especially for complex filtering like on Amazon.
3. **Use POST (Non-RESTful):** If the URL length is insufficient, use `POST` and put the included fields in the body, but this means it is **no longer REST** for data retrieval.
4. **Extra Parameters:** Add an extra parameter (e.g., `short`) to the existing endpoint to get a limited number of fields.
    - _Problem:_ The number of parameters can become too large, making them difficult to remember and documentation hard to keep updated. This solution is also not very flexible if different sets of fields are needed elsewhere.

### B. Under fetching (Not Fetching Enough Data)

- **Definition:** Needing more fields or entities than the backend returns, requiring additional requests.
- _Example:_ Displaying an article, its comments, and the user who left each comment requires three different entities and therefore three different requests (Article, Comments, Users).
- **Consequences:** Incomplete or insufficient data receiving, resulting in extra requests, increased latency, and a worse user experience.

**Fixing Under fetching:**

1. **Add an Additional Endpoint**.
2. **Use GraphQL** instead of REST.
3. **Ask the Backend Team to Add Fields**.
    - This solution has two problems: a. **Increased Coupling:** Increases coupling between frontend and backend teams, which may be undesirable depending on team structure. b. **Reduced Unification:** Makes the endpoint less unified, possibly resulting in over fetching for other teams or multiple places within the same project.
4. **Bonus Solution (Full Stack Team):** If the team is full stack, the fields can be added by the developers themselves, making the problem easier but not eliminating the unification issue.

**Unification Paradox:**

- The main purpose of REST is to be unified, working well with different protocols, teams, and customers.
- This unification is both the main **advantage** and the main **disadvantage** of REST.
- While suitable in many cases, a unified solution cannot truly "shine" in any specific scenario.

## XI. When REST is Recommended (Good Cases)

The speaker will look at approaches that are less flexible but more efficient, but first, cases where REST is good.

1. **Public API:** Most developers are more comfortable with REST, making it the default choice for public APIs.
2. **Simple Application with Mostly CRUD Operations:** REST is the choice if you don't know why you need GraphQL or RPC.
    - This scenario covers a huge part of websites and web applications.
    - Complexity should not be added when it is not needed.
    - Balance simplicity of adding CRUD with overall complexity.

**Warning:** Using REST for complex applications may turn the API into legacy, unsupported code soon after development starts. System design is crucial for considering scalability and maintenance costs before starting development.

## XII. When Alternatives to REST are Recommended (Bad Cases)

1. **Over fetching or Under fetching is a Problem:** These issues lead to increased traffic consumption, which is a problem for mobile devices. **GraphQL** should be used instead.
2. **Complex Response Structure:** If the team must build a complex response structure on the backend side or send many requests on the frontend, switching to **GraphQL** may make sense.
3. **Microservices on the Backend Side:** REST is **not very suitable** for service-to-service communication.

### Reasons Why REST is Slow

- **Priority:** REST was not designed to be efficient; usability was the top priority.
- **Protocol Independence:** REST is never native; it is protocol independent, which is good for variety of applications.
- **Speed Criticality:** When speed is critical, native approaches are usually preferred.

**Alternatives for Speed/Efficiency:**

- **RPC approaches (e.g., gRPC):** Used for synchronous operations because they are usually faster.
- **RabbitMQ or other Queues:** Used for Pub/Sub like operations.

_(Note: While backend is not the responsibility of the client-side developer, it is useful to know when REST is not useful)._

The next video will cover GraphQL and RPC approaches.

--------------------------------------------------------------------------