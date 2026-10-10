
- Scaling Web Applications | Part 1
- Video_Links : https://www.youtube.com/watch?v=2qtgegNSUoE

--------------------------------------------------------------------------

This response draws exhaustively on the provided transcript excerpts, ensuring all details regarding the scaling of web applications and frontend architectural approaches are included and comprehensively cited.

## Scaling Web Applications: Frontend Architecture Approaches

The video focuses on approaches to **frontend architecture**, discussing monolithic architecture, micro frontends, and modular monoliths. The second part of the video will address mono and poly repositories.

### I. Context and Definitions of Scaling

#### 1. Backend vs. Frontend Scaling

- **Backend applications** handle many simultaneous user requests, interact with databases, manage load peaks, and perform many other tasks.
- The **frontend** doesn't need to manage this backend complexity. Frontend developers still work with data and face challenges, however.
- The main difference is that the frontend works with **one client, one device**, regardless of the number of users per minute, hour, or day.
- The number of users does not usually affect how the frontend application works, except in cases like **collaboration tools or two-way communication apps**.
- While an increasing number of users can cause longer request times because the server is handling a large load, this is not something changeable on the frontend side. Backend operations are considered a constant for convenience.

#### 2. General Scaling Concepts

- **Scaling** generally refers to the system's ability to handle increased demand for resources as application traffic grows.
- **Backend Scaling Types:**
    - **Vertical Scaling:** Adding more power (CPU, memory, etc.) to a single server.
    - **Horizontal Scaling:** Adding more servers.
- These backend scaling types are **not really related to the frontend platform** because developers cannot upgrade user laptops or buy new ones to distribute the load.

#### 3. Frontend Scaling Definition

- For frontend applications, scaling means the **ability of an application to allow multiple developers or teams to work simultaneously** on a large and complex product.
- This facilitates the addition of changes and the deployment of apps.
- Scaling frontend apps is mentioned as being "a bit easier" than scaling backend apps.

#### 4. The Benefits of Scaling Frontend Applications

Thinking about scaling is necessary because when an application is highly scalable:

- Developers can run the development environment faster.
- Developers can make changes faster.
- Testing the application becomes easier, and tests take less time.
- Deployment to production servers is faster.
- The development process reduces the latency of integrated features and improvements.

#### 5. System Architecture vs. System Design

The speaker uses this terminology for convenience, noting they are not academic definitions. Both architecture and design affect an application’s ability to scale.

- **Architecture:** Includes higher-level concerns related to the **overall structure, communication, and coordination of multiple applications or systems**.
- **Design:** Primarily deals with the **internal structure and organization of code within a single application or system**. Design describes how the application works internally and may be defined in terms of patterns like **Model View Controller (MVC)**.

#### 6. Ideal Architecture and System Description

- There is **no ideal architecture** suitable for almost any frontend application. Each approach has pros and cons and is suitable only in particular situations.
- When describing a system on a high level, the backend app may be included in the design in two scenarios:
    1. **Tightly Coupled:** Frontend and backend are a single unit, often using the same technologies (JavaScript or TypeScript). This scenario implies a full stack development team, and the backend is considered part of the system.
    2. **Out of Responsibility:** The backend is owned by another team or company, and the frontend team does not need to care about it.
- The evolution of frontend architectures is shown starting from the simplest approach: **monolithic application in a monorepository**.

---

### II. Monolithic Architecture

#### 1. Definition and Structure

- A Monolithic architecture is a software development approach where an application is built as a **single, self-contained unit**.
- It is the starting point for many web applications.
- In practice, a monolith has a basic structure, including component and services folders, but **lacks modularity**, meaning all components and services are in one place.
- A monolith is **tested and deployed as a single unit**.

#### 2. Advantages (Pros)

|Advantage|Detail|Citation|
|:--|:--|:--|
|**Easy Deployment**|Deployed as a single unit and delivered as a whole, eliminating synchronization problems between different parts.||
|**Easy Development (Early Stages)**|Requires fewer resources spent on building architecture and communication between models. Suitable for small projects or **MVP (Minimal Viable Product)** stage.||
|**Simplified Testing**|Easier because the application is tested as a whole, simulating real user interactions rather than isolated parts. Easier as a single unit, allowing **end-to-end tests** covering the entire user case.||
|**Easy Debugging**|Simple to trace the root cause because the application in production and the application running locally are pretty much the same, providing the whole picture.||

#### 3. Disadvantages (Cons)

|Disadvantage|Detail|Citation|
|:--|:--|:--|
|**Slower Development Speed**|Only true for early stages; as the application grows bigger and the team grows, development becomes harder.||
|**Lack of Scalability**|Monolithic applications **do not scale at all** because they are a single unit and cannot be separated.||
|**Reliability Issues**|An error in any module could affect the application's internal availability because there are **no clear error boundaries**.||
|**Barriers to Technology Adoption**|Changes to the framework or language affect internal parts, making them expensive and time-consuming. Not all technology is suitable for all internal services.||
|**Lack of Flexibility**|Constrained by existing technologies. It is hard to use different frameworks for different parts of the application.||
|**Slow Deployment**|A small change requires the **redeployment of the entire monolith**. This issue worsens with application growth.||

#### 4. Use Cases (When to Use Monolith)

- **Small project** with relatively straightforward requirements (facilitates development in early stages). Some projects remain small forever.
- **Prototyping/MVP development** where time is more important than quality, allowing rapid prototyping.
- Projects with a **small team of maintainers** (one or two developers), where separating by modules may not be beneficial.

#### 5. Anti-Use Cases (When Not to Use Monolith)

- **Large scale Enterprise applications** with complex and evolving requirements.
- Projects where different parts require **different technologies or scaling strategies** (due to tight coupling). Migration between technologies is very difficult.
- Projects where **many people are working on it**, as changes may conflict, increasing complexity and time.

#### 6. Monolith Summary

- Monoliths are suitable mostly for **small or medium-sized applications**.
- The two main reasons to drop the Monolithic approach are **the team gets bigger or the application gets bigger**.
- A monolithic architecture is a valid choice and is often considered the **sensible default choice** as an architectural style.

---

### III. Micro Frontends

#### 1. Definition and Context

- Micro frontends are the logical next step when a monolithic application becomes insufficiently scalable.
- The idea originated from microservices, sharing the same core concept.
- **Definition:** An architecture style where **independently deliverable frontend applications are composed into a greater whole**. It is the opposite of monolithic architecture.
- For end users, the application remains a single unit.
- **Transition:** Scaling challenges from a monolith (increasing services and team members) necessitate separating responsibilities, leading to more flexible and faster deployment. Maintaining code in separate repositories helps transition from a monolith to micro frontends.
- **Key to Independence:** The critical element is **clear boundaries** between modules.

#### 2. Implementation: Slicing Approaches

|Approach|Description|Suitability/Examples|Citation|
|:--|:--|:--|:--|
|**Vertical Slices (Separating by Domains/Pages)**|Dividing the system by domain, where a domain is responsible for one business feature (e.g., Authentication model, To-do model). Teams are responsible for their domain and its internal architecture (e.g., MVC).|Better suited for **web applications** (e.g., Google Calendar, Skype, Figma) which have a limited number of pages dedicated to a single domain.||
|**Horizontal Slices (Separating by Components)**|Dividing the system by components (e.g., Team A handles Header/Footer, Team B handles the Main component).|Better suited for projects containing many almost identical pages, such as **e-commerce sites and catalogs** (e.g., Amazon).||

- A hybrid approach is also possible.

#### 3. Advantages (Pros)

|Advantage|Detail|Citation|
|:--|:--|:--|
|**Incremental Upgrades**|Ability to evolve a large code base without rewriting the whole application at once. Only one part needs to be rewritten (e.g., migration from React to Vue can be done in incremental steps).||
|**Independent Deployment**|Reduces the scope and risk of each deployment. Changes made by one team in one model do not affect other models, reducing errors and deployment time.||
|**Simpler/Decoupled Code Base**|Easier to manage large applications split into smaller pieces.||
|**Autonomous Teams**|Decoupling of code base and release cycles leads to a greater level of autonomy.||
|**Reduction of Cognitive Load**|New developers can concentrate solely on their part initially. Experienced members do not have to keep the whole application in mind constantly.||
|**Independent Deployment Coordination**|Teams do not have to coordinate deployment across multiple teams, provided the **input and output events are not changing**. Internal behavior and technical details are irrelevant to other teams.||

#### 4. Disadvantages (Cons)

|Disadvantage|Detail|Citation|
|:--|:--|:--|
|**Potential Performance Issues**|Increased number of technologies can increase the bundle size.||
|**Lack of Communication**|Can lead to **duplication of specific implementation methods** and waste time.||
|**Complex Architecture/Deployment**|Architecture maintenance and deployment are more complex compared to the simplicity of a monolithic application.||
|**UX/UI Incompatibility**|Consistency is hard to ensure until services are deployed to production. Local and production versions may not match because they run in different environments.||
|**Increased Management Overhead**|Leads to more things to manage: repositories, tools, builds, deploy pipelines, servers, domains, etc.. Managing many models is more difficult than a single monolithic process for deployment, testing, and synchronization.||

#### 5. Use Cases (When to Use Micro Frontends)

- **Big project with multiple teams:** Micro frontends function as an organizational tool, accelerating development, changes, deployment, and delivery.
- **Need independent development life cycle/release cadence:** Allows parts with different deployment frequencies (e.g., frequently updated Calendar page vs. infrequently updated Unification page) to be managed independently, simplifying releases and reducing bottlenecks.
- **Integration of new technologies/features:** Allows incremental migration by separating the application by modules first and then rewriting them sequentially.
- **Cross-functional teams:** Useful when developers are responsible for the **end-to-end experience** (from UI creation to delivery). Such a unit often includes the frontend application and the backend application that handles its requests, allowing one team to work on both.

#### 6. Anti-Use Cases (When Not to Use Micro Frontends)

- **Small projects** (and sometimes medium size) with limited complexity; the overhead of implementation outweighs the benefits.
- **Tightly integrated teams** working closely together on all parts.
- **Small teams,** as micro frontends are a tool for sharing responsibility.
- **Limited budget or resources,** due to the additional effort required for setup, maintenance, and coordination.
- **Lack of automation/poor infrastructure,** making handling multiple applications problematic.
- **Websites with static content** or minimal interactivity/dynamic features, where simpler architectures (static site generators, server-side rendering) are more appropriate.
- **Conclusion on Usage:** Micro frontends should only be considered when **scalability becomes a real problem**. Implementing them simply because they are popular is a common mistake.

### IV. Micro Frontend Implementation Approaches (Composition)

Micro frontends must be collected together to work as a whole on the user's device. There are three main composition approaches:

#### 1. Server Side Template Composition

- **Mechanism:** The application is built on the server using templates and components. An `index.html` file contains common elements (header, footer, navigation). Based on the URL, the server plugs in the content of a specific page using HTML fragment files.
- **Process:** Client requests page by URL $\rightarrow$ Server retrieves template $\rightarrow$ Picks model based on URL $\rightarrow$ Integrates model into template $\rightarrow$ Returns the full page to the client.
- **Use Case:** When the site already uses server-side rendering (e.g., an e-commerce site).

#### 2. Run Time Integration (Client Side Rendering)

If client-side rendering is used, developers must choose between four runtime approaches.

|Approach|Method/Details|Pros (Advantages)|Cons (Disadvantages)|Choice Context|
|:--|:--|:--|:--|:--|
|**iFrames**|Template (shell) integrates models using iFrames. Each iFrame is a separate document.|Provides **good isolation** for models (isolates scripts and styles).|Difficult communication (requires `postMessage` or URL). Performance degradation due to several documents. Accessibility problems (screen readers struggle). Tricky E2E testing. Cannot share packages, potentially loading libraries twice.|Pick if **maximum isolation** is required.|
|**JavaScript Integration**|Downloads modules and integrates them into the page.|Better performance than iFrames. Easier direct communication (single document).|**Poor module isolation**; higher chance of leaking styles or global scripts.|Pick if communication, performance, and testability are important.|
|**Web Components**|Custom elements used as wrappers; similar integration to JavaScript.|Provides **good style isolation** (Shadow DOM encapsulates styles).|**Worse browser support**.|Choice often depends on preference compared to JavaScript integration.|
|**Webpack Module Federation**|Allows JavaScript chunks to load synchronously or asynchronously.|Allows **Library Sharing** across micro frontends (Huge Advantage). Smooth developer experience (easy importing/composing).|Leads to micro frontends not being fully isolated; undisciplined sharing results in complicated maintenance architecture.|Pick if communication, performance, and testability are important.|

#### 3. Build Time Integration

- **Method:** Integrating models as an external library.
- **Advantage:** **Discoverability** (easy to see the list of models in the `package.json` file).
- **Disadvantage:** Contradicts independence; requires the entire application to be **recompiled whenever any micro frontend is changed**.
- **Conclusion:** Generally **not used** because the need to recompile contradicts the idea of micro frontend independence.

---

### V. Modular Monolith Architecture

#### 1. Definition and Rationale

- The Modular Monolith finds a balance between the monolithic approach and micro frontends.
- **Definition:** Combines the benefits of **modular design** with the **simplicity of a monolithic architecture**.
- **Mechanism:** Divides the system into a set of **loosely coupled modules**, each with well-defined boundaries and explicit dependencies.
- **Deployment:** The system still **builds and deploys a single application**, just like a traditional monolithic architecture.
- Teams work on their models independently, but the overall application remains monolithic.

#### 2. Modularity and Transition

- A Modular Monolith may serve as a **transition stage to micro frontends**.
- It allows for easier extraction of single models into micro frontends later if they become too big, which would be difficult starting from a traditional monolith.
- **Rules for Modularity:** Modularity is achieved through the input and output of each model.
    1. Each model **must have input and output**.
    2. Models must have **minimum side effects**.
- These rules ensure isolation, making extraction into a separate micro frontend possible. In reality, teams strive to minimize connections, as full isolation is often hard to achieve.

#### 3. Advantages

- **Encapsulation:** Features or pages are encapsulated in modules.
- **Reusable Code:** Increases code reusability for large development teams.
- **Better Organized Dependencies:** Dependencies are more organized and visible.
- **Less Complex than Micro Frontends:** Easier to manage due to a **single deployment process**.
- **Faster Time to Market:** Provides modular benefits without the deployment/infrastructure management overhead of micro frontends.

#### 4. Challenges/Key Characteristics

- **Technology Limitation:** Harder to diversify technology and languages compared to micro frontends.
- **No Independent Deployment:** Since it is a single unit, models cannot be deployed independently.
- **Isolation:** Modules are isolated and independent, but **not completely** as they are in micro frontends.
- **Scaling:** Scaling is possible, unlike a traditional monolithic architecture.
- **Communication:** Communication between models is easier than in micro frontends.
- It can be a transitional stage or a permanent architectural style.

#### 5. Internal Workings: Application Shell

The core concept is the **Application Shell**.

- **Definition:** The fundamental structure or framework providing the overall layout, navigation, and basic functionality of the application (it is not a static template). It controls model integration.
- **Shell Components:** The application shell consists of the **main module** (responsible for loading all modules and navigation), and **shared libraries**. Sub-modules of the shell include the header, navigation, and footer.
- **Modules:** Bring the application content, assets, and dependencies (e.g., Authentication model, To-dos model, Settings model).

#### 6. Use Cases (When to Use Modular Monolith)

- **Cross-functional teams:** Well suited when one unit is responsible for end-to-end maintenance (including front-end and back-end parts of the model).
- **Small to medium-sized applications:** When massive scaling is not expected. Provides necessary modularity without the complexity and expense of micro frontends.
- **Time to Market is crucial** and quick iteration is needed, as features are tested and deployed more easily within a single code base.
- **Small or less experienced teams:** Simplifies development, deployment, and maintenance compared to complex distributed systems.
- **Operational Simplicity is crucial:** When the operational overhead of managing multiple services (micro frontends) outweighs the benefits.

#### 7. Anti-Use Cases (When Not to Use Modular Monolith)

- **Highly distributed teams:** Coordination is challenging across different geographical locations/organizational boundaries. Micro frontends are more suitable.
- **Need independent deployment and scaling:** Micro frontends offer granular control over deployment/scaling based on varying demands, which the single-unit modular monolith cannot provide.
- **Need technology diversity:** Applications requiring different technologies, languages, and frameworks are better suited to micro frontends.
- **Simple applications:** A standard monolithic architecture offers simplicity and ease of development for straightforward apps (CRUD, blogs, basic CMS).
- **Prototyping/MVP:** A standard monolithic architecture allows for faster building and iteration for quick validation.

### VI. Conclusion of Part 1

The next part of the video will discuss Mono and Poly repositories and look at examples to consolidate the knowledge.

--------------------------------------------------------------------------


- Scaling Web Applications | Part 2
- Video_Links : https://youtu.be/RRcb3ZUOe_w?si=WMqJMgQmL9rDkuu9


--------------------------------------------------------------------------

This response draws exclusively on the provided source material, detailing the concepts, comparisons, examples, and frameworks discussed in the video transcript excerpts.

---

## I. Introduction and Context

This content serves as the second part of a frontend scaling video series. The previous part covered different approaches to frontend architecture, including Monolithic architecture, Micro Frontends, and Modular Monoliths. This part focuses on Monorepositories and Polyrepositories, followed by examples to consolidate knowledge.

## II. Repository Management Strategies: Monorepos vs. Polyrepos

The video clarifies the difference between these two approaches:

### A. Monorepository (Monorepo)

A monorepository is defined as a **single repository** containing multiple projects or components.

- **Benefits:** Simplifies dependency management and improves consistency across the code base.
- **Definition Clarification:** A monorepository is a **Version Control repository management strategy** where multiple projects or components are stored within a single repository.
- **Requirement:** A monorepository **must include more than one application**.

### B. Polyrepositories (Multi-repositories/Polyrepos)

Polyrepositories are defined as **collections of separate repositories**, with each repository containing a single project or component.

- **Benefits:** Provides flexibility and autonomy to individual teams or projects.

### C. Monolith vs. Monorepository (Key Distinction)

Inner developers sometimes confuse these two terms, but they are **not equal**.

|Concept|Description|
|:--|:--|
|**Monolith**|A single application deployed as a single unit. A monolithic application cannot include modules or parts.|
|**Monorepository**|A Version Control repository management strategy. Must include more than one application.|

### D. Examples of Monorepository Content

Other combinations are possible, but typical monorepos might include:

1. A frontend application and a backend application.
2. Two frontend applications and a shared library (likely used by both).
3. Two microservices and shared code.

### E. Tooling Note

The video does **not** cover specific monorepository management tools like NX, Lerna, or Turbo Repo. This is because:

- Companies are typically interested in developers with system design experience, not experience with specific tools.
- Tools become outdated very quickly, and technology popularity can change tomorrow.
- It is recommended to invest time in learning the **core concepts**, not the tools.

## III. Monorepositories in the Context of Micro Frontends

Monorepositories generally do not make much sense for Monolithic and Modular Monolithic architectures, as a monorepository must contain at least two projects (though two monolithic apps can exist in one).

Polyrepositories are a very reasonable option for micro frontends, as micro frontends often involve separate development, testing, and deployment processes. However, a monorepository may be considered as an alternative for the following reasons:

### A. Advantages (Pros) of Using a Monorepository for Micro Frontends

|Advantage|Detail|
|:--|:--|
|**High Visibility**|Allows a developer working on one micro frontend to look at the code of other micro frontends it calls, helping to understand how they work and determine the source of bugs (local code or another team's micro frontend).|
|**Code Sharing**|Common modules, shared libraries, and helper code stored in a single repository can be easily shared among many micro frontends.|
|**Standardization**|Easier to standardize code and linking across teams. Policies can be created to enforce naming guidelines, include code reviewers, limit access to specific branches, and enforce best practices.|
|**Discoverability**|Offers a single view of the whole codebase and branches. Tracking modifications is much easier than in polyrepositories.|

### B. Disadvantages (Cons) and Problems of Monorepositories

|Problem|Detail|
|:--|:--|
|**Fragile Modules**|Changing shared code can potentially affect many components of the application.|
|**Merge Conflicts**|When multiple engineers work on a large codebase, this can lead to frequent merge conflicts.|
|**Security Concerns**|Monorepositories make it difficult or impossible to easily separate access rights for developers, unlike polyrepositories where each team has access only to its part.|
|**Large Size**|Developers usually do not need to load all micro frontends. Some virtual file systems allow only part of the code to be present locally.|
|**Slow Indexing, Searching, and Discovery**|Since developers may not have all the code locally in a searchable state, the ability to search the entire codebase is required.|

### C. Use Cases Where a Monorepository is Useful

Monorepositories are useful in specific scenarios:

1. **Tightly Integrated Frontends:** If micro frontends are tightly integrated and share dependencies, a monorepo provides a centralized place to manage common codes, libraries, and dependencies. This simplifies development and ensures consistency.
2. **Sharing Code:** Monorepositories facilitate the sharing and reuse of code between micro frontends because all components reside in the same code base.
3. **Applications Must Be Released Together:** If modules should be released together, they should be in the same repo, which makes it easier to manage projects even with different release schedules.

### D. Use Cases Where a Monorepository Should NOT Be Used (Polyrepo Preference)

Monorepositories are generally discouraged in these cases:

1. **Decoupled and Independent Frontends:** Polyrepositories offer greater autonomy and flexibility when micro frontends are designed to be developed independently by different teams.
2. **Large Number and Services of Large Size:** Polyrepositories scale well as the number of micro frontends increases, helping to manage complexity and avoid bottlenecks in development and deployment processes.

- **Resource Warning:** While large tech companies (e.g., Google) implement and maintain monorepositories for many applications, they possess vast resources. Monorepositories are tools that require additional resources to maintain, and this requirement should be considered unless resources are unlimited.

## IV. Fullstack Teams and Monorepositories

Fullstack teams are common for small or middle-sized companies. For small projects, having a team of fullstack developers working simultaneously on frontend and backend parts is more reasonable than specialized developers. A trendy solution is hosting the backend app on Node.js.

### A. Monorepo for Fullstack Projects

A monorepository should be considered if the frontend and backend applications **represent a single entity**.

- **Benefits:** Putting them in a single repo increases visibility and discoverability, easing development. A developer can easily run the entire project locally.
- **Suitability:** Mostly suitable for **small or medium-sized projects**. As the project grows, it becomes difficult to keep the details of both applications in mind.

### B. tRPC (typescript RPC)

When discussing fullstack projects in a monorepo, tRPC (TypeScript RPC) is relevant.

- **Function:** tRPC is an implementation of RPC designed for TypeScript monoliths (or environments where TypeScript is used on both sides).
- **Recommendation:** If TypeScript is used on both the client and the server side, tRPC should be considered for client-server communications.
- **Facilitation:** tRPC facilitates client-server interactions. Frontend developers simply call functions and do not need to worry about implementation details like which REST verb is used or how to define URLs.

## V. Versioning in Monorepositories

When multiple applications are in one repository, three different approaches to versioning exist:

### 1. Semantic Release Versioning

- **Method:** A fully automated versioning method.
- **Process:** Automates the entire package release workflow, including determining the next version number, creating release notes, and publishing the package.

### 2. Manual Versioning

- **Responsibility:** The developer and maintainers are responsible for changing versions.
- **Process:** This can happen entirely manually or by using a tool to determine the next version, but the choice of the next version always rests with the developers.
- **Note:** The logic between manual and semantic release versioning is the same; the difference lies in how the new version is installed.

### 3. Global Versioning

- **Concept:** All packages and services share the same version.
- **Process:** One change in one package increases the version of all other packages and services in the codebase.
- **Popularity:** This approach is very popular and has many advantages, but it can be complicated.
- **Compromises:**
    - **Calculating and Fetching:** The global version source might change while a pipeline is running. This can be fixed if all version changes are committed locally using a tool like Lerna.
    - **Scaling Expense:** When a service or package is upgraded, every item in the repo must also be upgraded. This is very expensive for huge projects. This problem becomes more severe as the number of apps in the repository increases.

## VI. Practice Example 1: Admin Dashboard Architecture

### A. Scenario and Requirements

The goal is to build the architecture for an admin dashboard where clients register and use various services. The services are logically separated, and one client may use one or more services (e.g., Users, Metrics, Budget, Groups).

- **Key Requirements/Conditions:**
    1. Services are not coupled with each other.
    2. Services have a separated deployment process.
    3. Services may share common code but do not communicate with each other.
    4. Each service is maintained by a separate team.
    5. Each service uses a single backend application, which can be shared by multiple services.
    6. The use of different technology stacks is not required; they can be built on the same stack to improve interchangeability between developers.
    7. Basic navigation elements (menus, headers, footers) are common to all services and should always be visible.

### B. Architectural Evaluation

1. **Monolithic Architecture:**
    - **Conclusion:** Not the best idea.
    - **Reasons:** Not scalable enough for medium and large projects. The application consists of separate services managed by separate teams. Deployment processes are independent. Existing services are expected to grow, and new services will be added.
2. **Modular Monolithic Architecture:**
    - **Conclusion:** Much better.
    - **Reasons:** Can scale with application and team growth. Dividing the application into models separates responsibilities, allowing teams to work only on their part.
3. **Micro Frontends:**
    - **Question:** How independent are the services?.
    - **Challenge:** Choosing Micro Frontends introduces challenges in code sharing, communication between modules, and CI/CD processes.
    - **Decision Point:** If high independence (separate deployment) is critical, Micro Frontends are necessary. If not, the Modular Monolith is sufficient.

### C. Repository Choice (If Micro Frontends Are Chosen)

- **Assessment:** The teams work mostly independently on modules.
    
- **Monorepo Pros (e.g., easy code sharing, visibility) are weighed against Cons (e.g., resource requirements).**
    
- **Conclusion:** The advantages of a monorepository are **not crucial** in this case. The monorepo requires additional resources that are not deemed worth the benefit. A multi-repository (polyrepo) structure is implicitly preferred for this decoupled setup.
    
- **Final Structure:** The application consists of five models (Users, Budget, Metrics, Groups, Settings) and the application shell. The core team works on the application shell and settings modules; other modules are distributed across different teams.
    

## VII. Practice Example 2: To-Do Application Architecture

### A. Scenario and Requirements

The To-Do application serves as an example of a simple web application with one main feature: allowing users to create, update, delete, and complete tasks.

- **Key Requirements/Conditions:**
    1. Application is very simple with no special requirements.
    2. Small team consisting of two or three developers.

### B. Architectural Evaluation

1. **Monolithic Architecture:**
    - **Reasons to Stay:**
        - The whole team works on all parts, and separating teams into models adds unnecessary complexity for a small application and team.
        - Ease of development (no need to run multiple apps or establish connections).
        - Easy testing and debugging (run as a whole, easily track the whole stack trace).
        - Consistent UI (application looks the same locally as in production).
        - Easy to start and cheap. It is a good starting point when commercial success is uncertain, and minimal resources should be invested in the architecture.
    - **Reasons to Avoid (Potential Problems):**
        - The application may still have multiple modules (e.g., authentication, settings).
        - Risk of **Peter Pan Syndrome**, where the team assumes the application will never grow and neglects scaling. Architecture choice should consider future development.
2. **Modular Monolithic Architecture:**
    - **Advantages:** Retains the advantages of monolithic architecture. It requires more resources initially to build the architecture.
    - **Benefits:** Establishes clear boundaries between models (which improves further development). Makes the application ready for further scaling and future splitting into micro frontends.
    - **Decision:** If the design is finalized with no expected big future changes, stick to the monolith. If it is a first iteration with plans to expand, and resources/time are available, start with a Modular Monolithic architecture.
3. **Micro Frontends:**
    - **Conclusion:** This application is an example where micro frontends would make things too complicated without providing benefits.

### C. Repository Choice (Fullstack Consideration)

Assuming the application involves both frontend and backend apps (often using the same technology stack like TypeScript for cost reduction) and a small team of fullstack developers:

- **Frontend Code:** Keep the single frontend application code in a single repository.
- **Fullstack Repository Conclusion:** A **monorepository** is recommended.
- **Reasons:** Projects are tightly coupled and likely share a single development cycle. High discoverability of the code increases development speed. tRPC can be applied to facilitate communication between frontend and backend parts.

## VIII. Conclusion: Architecture Framework

The video provides a framework to help determine the appropriate architecture for a project, noting that project and team size usually grow together.

### A. Starting Point

Start with a **Monolithic** architecture and a **Monorepository** (meaning a single repository).

### B. Step Progression

If the answer to one or more listed control questions (not detailed in the provided transcript) is "yes," consider jumping to the next level:

1. **Next Level:** Modular Monolith.
2. **Next Level:** Micro Frontends (if control questions warrant it).

### C. Repository Decision for Micro Frontends

Once Micro Frontends are chosen, the decision must be made whether to stick with a **Monorepository** or switch to **multiple repositories**. Answering the list of control questions (not detailed in the transcript) helps determine the suitable solution.

--------------------------------------------------------------------------