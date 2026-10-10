Video_Link : https://youtu.be/jrJX8PdU4rE?si=004m04E7xxRt3HLt

--------------------------------------------------------------------------

This response provides a highly detailed summary of the source material, ensuring that every piece of information mentioned in the transcript excerpts is included and correctly cited.

## Web Communication Protocols: Summary and Concepts

The source material represents the third and final part of a series discussing web communication protocols. This final part aims to summarize previous learning, include practice examples, and determine which protocol/pattern should be applied to a real-life application. Previous parts discussed REST, GraphQL, and RPC approaches.

### The Concept of Coupling

Coupling describes how strongly the client is connected with the backend. A crucial principle is that connection generally cannot be efficient and flexible at the same time. If endpoints are static, they can be optimized for the best performance, but they will lack flexibility.

|Protocol|Level of Coupling|Characteristics|
|:--|:--|:--|
|**tRPC**|**Maximum Coupling**|All complexity is encapsulated inside the procedures. Web developers work only with these procedures. Both teams _must_ use TypeScript; switching to another language is impossible.|
|**REST**|**Less Coupling**|Allows choosing any programming language. Allows choosing more other things because it is protocol independent, though the discussion primarily stays within the combination of HTTP and JSON.|
|**GraphQL**|**Most Freedom**|Bound to the query language, but allows choosing exactly what data the client wants to get.|

**Implication for API Design:** The fact that REST and GraphQL are less coupled to the backend makes them more suitable for **public APIs**. If backend and frontend teams are situated in different rooms or even different companies, REST and GraphQL should be considered as API solutions.

### Summary of Technology Choice Based on API Type

The source provides a short summary to help choose technology based on the type of API:

|API Type / Architecture|Recommended Protocols / Solutions|
|:--|:--|
|**Private API**|REST, GraphQL, gRPC, or tRPC.|
|**Public API**|REST or GraphQL.|
|**Microservices Architecture**|Choose between gRPC and GraphQL.|
|**Microservices Network Organization**|Consider **Supergraph** as an alternative to gRPC. A Supergraph is defined as a unified network of a company's data microservices that serves as a composition layer for the whole organization.|

### Client-Server Architecture Protocol Choices (Detailed Criteria)

For a standard client-server architecture, the following guidelines apply:

#### 1. REST Usage

REST should beAl used for:

- Simple APIs.
- Binary data transfer.
- Streaming.
- CRDT.
- Authentication (audification).
- When resources are limited and the easiest solution is needed.

#### 2. GraphQL Usage

GraphQL should be used for:

- Multiple versions of an application, especially mobile versions that need different datasets.
- Complex responses that combine fields from many entities.

#### 3. tRPC Usage

tRPC should be considered for:

- Teams that work together or full stack teams.
- A monorepository setup.
- When both client and server are written in TypeScript.

#### 4. gRPC Usage

gRPC should be considered when:

- High performance is critical.
- Usability and development speed can be sacrificed.

---

## Examples and Case Studies

The source provides two practical examples: Stack Overflow and Netflix.

### Example 1: Stack Overflow Clone Design

The goal is to design a clone of Stack Overflow, allocating three main features within the interview timeline: **search**, **showing posts and posts feed**, and **creating new posts**. These features align perfectly with CRUD (Create, Read, Update, Delete) operations.

#### Requirements Assumption (for the clone):

1. Backend will use TypeScript (though the actual Stack Overflow backend is unknown).
2. Front-end and backend code are **not** in a monorepository (as monorepos are generally not popular nowadays).
3. No dedicated mobile version or application; usage is assumed to be mostly on desktops.
4. Usual performance requirements (application should work fast and smooth).

#### Protocol Comparison:

|Protocol|Analysis|Feasibility|
|:--|:--|:--|
|**tRPC**|Backend uses TypeScript, but the lack of a monorepository makes it difficult to share types between front-end and back-end teams.|Not very beneficial.|
|**gRPC**|Specific speed requirements were not highlighted. Requires sacrificing API usability and increasing maintenance and infrastructure costs.|Not feasible.|
|**GraphQL**|Can offer advantages, particularly by solving potential underfetching. For example, fetching a typical post with comments might take five REST requests but only one GraphQL request.|Possible, but complex.|

#### The REST vs. GraphQL Underfetching Trade-Off:

While GraphQL solves underfetching (e.g., reducing 5 requests to 1), the solution is not straightforward. If the need for different data sets in different places were a requirement (which it currently isn't), REST could handle this using query parameters, field sets, or creating multiple endpoints.

**MVP Focus:** The current goal is designing an MVP, not covering all possible future cases (like places needing different data sets or a mobile version).

#### Final Recommendation: **REST**

**Why REST?**

- Other core operations (adding, editing, deleting posts) fit within CRUD.
- Provides familiarity for most developers, resulting in fast implementation.
- Allows easy construction of efficient endpoints, leading to fast implementation and maintenance.
- Offers a flexible API, eliminating the need to launch additional APIs for tasks like authentication or sending binary data, unlike the requirements often associated with GraphQL.

### Example 2: Netflix Clone Design

This application (Search, Movie Info, Recommendations List) is considered more interesting to discuss.

#### Requirements:

1. The application has a mobile website version and a mobile application.
2. The major part of the audience uses the application from cell phones.
3. Mobile traffic is limited, necessitating minimization of Under- and Overfetching of data.
4. Clients fetch different sets of fields based on screen size.

#### Client Experience Comparison:

|Client Type|Screen Size|Traffic|Data Redundancy|
|:--|:--|:--|:--|
|**Desktop**|Bigger screen (can display more data).|Unlimited traffic.|Redundant data is not a critical problem.|
|**Mobile**|Small screen (can show less data).|Limited traffic.|Must load only the necessary data.|

#### Protocol Comparison:

|Protocol|Analysis|Feasibility|
|:--|:--|:--|
|**tRPC**|High probability the backend is _not_ written in TypeScript. Does not solve the problem of multiple clients with different requirements.|Unsuitable.|
|**gRPC**|Does not solve the problem of multiple clients. Adds additional complexity.|Unsuitable.|
|**REST**|Does not perform anything special to solve the underfetching problem that alternative approaches can't handle. However, it is not limited to TypeScript and is familiar to frontend developers, allowing faster application development.|Unsuitable due to core requirements.|

#### Data Set Requirements (Illustrating the need for field selection):

The requirement involves many places needing the same base data but with different sets of fields.

|Data/Location|Desktop Needs|Mobile Needs|
|:--|:--|:--|
|**Movie (Homepage)**|Title, poster, title.|Less data than desktop.|
|**Movie (Hovering Card)**|ID, poster, title, short description, rating, and channel.|Less data than desktop.|
|**Movie (Movies Page)**|All data.|Less data (e.g., short description instead of full description).|
|**Persona (Page)**|ID, name, and photo.|Same data as desktop, but less.|
|**Persona (Hovering Card)**|ID, name, recent movies, country of origin.|Same data as desktop, but less.|
|**Persona (Personal Page)**|All data.|Same data as desktop, but less.|

(A Persona might be an actor, director, script writer, or other person related to creating movies).

#### Final Recommendation: **GraphQL**

**Why GraphQL?**

- Looks like the most suitable choice based on initial requirements.
- Solves the problem of Under- and Over-data fetching for current clients.
- Allows independence of the backend and multiple front-end teams.
- Allows launching a new version of an application or new pages without needing to worry about fetching data.

#### Note on Decision Making

The goal in a system design interview is not necessarily making "good or bad decisions" (which can only be seen in the future), but working with requirements and finding a balance between benefits and trade-offs. Any choice might well be chosen if it can be defended with solid arguments.

---

## System Design Interview Tips

The source concludes with several tips useful in a system design interview setting:

### Tip 1: Always Describe Your Choices

- Developing an API does not mandate using only REST; other options exist.
- Regardless of the technology or pattern chosen, the decision must be described.
- **Pro Tip:** Explain not only _why_ a particular technology or pattern was chosen, but also _why others were not_.

### Tip 2: Don't Forget About Trade-offs

- An engineer's skill is determined largely by their horizons (how many technologies they know) and their ability to think about negative scenarios.
- Ask: **"What trade-offs does the technology entail?"**.
- Be sure to discuss possible risks with the interviewer.

### Tip 3: Don't Forget About Other Teams and Business Requirements

- While focusing on the UX and frontend team requirements is key, this should not lead to ignoring factors like the cost of the solution and the complexities of implementation and support for other teams.

### Tip 4: Don't Dive Into Details If You Don't Need To

- Describing the API in detail takes significant time and does not reveal much about the engineer's overall capability.
- Focus on the **high-level architecture**.
- Only dive into the details if specifically asked to do so.

### Bonus Tip: Know What REST and RESTful Are

- If you choose not to follow all REST principles (e.g., HATEOAS, which can sometimes be tricky), make it clear to the interviewer that you understand the API is **RESTful** rather than strictly **REST**.

--------------------------------------------------------------------------
