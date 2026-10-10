
Video_Links : https://youtu.be/44mOnnt5pic?si=YVl-k-WEQDm4SUs5

--------------------------------------------------------------------------

This response draws exhaustively on the provided transcript excerpts from the video "Frontend System Design Components | Chakde System Design (English Subtitles) Ep. 3."

The episode, presented by Chirag Goel, discusses the complexities of Frontend system design, moving beyond the initial idea that the job is merely about creating a cool UI. The components are explained through the business evolution of Motu Bhai's small Indian Samosa Valai stall into an international brand.

---

## I. Initial Frontend Development Perspective and Scope

A Frontend developer or UI engineer often starts their career thinking they will be making cool UI, some animation, or some game. They are appealed by and wish to develop cool web apps or mobile apps.

However, through discussions with seniors or during interviews, they realize that Frontend development is much more than just developing a cool UI. The job encompasses many elements, including:

- **Architectural Patterns:** Deciding the architectural pattern.
- **Protocols:** Handling different kinds of protocols.
- **Core Concepts:** Availability, consistency, accessibility, credibility, trust.
- **Infrastructure:** Logging, monitoring, databases, caching, security, performance, and optimization.
- **Quality:** Testing.

Key topics that "chip into" the development process include:

- Micro frontend.
- Maintaining accessibility.
- Securing the application.
- Caching.
- Optimization.
- Performance.
- Search Engine Optimization (SEO).
- 100% availability.
- Providing offline support.

Many non-frontend people incorrectly believe the UI's purpose is simply to "Get the API data and show it here".

---

## II. System Design Components

### 1. Micro Frontend

Motu Bhai wanted to make his setting area more innovative to meet the requirements of different customers, such as those needing a separate area for chit chat/meetings, those who want to "hit the road," and competitive types who have fountain or swimming pool areas.

#### Monolithic Front End (Motu Bhai's Desi Jugaad)

- Motu Bhai tried to fit everything (private area, water inside, smoking area) into one spot.
- This approach resulted in "crap" and "a piece of shit".
- In frontend, this is when developers assume all use cases will fit using the same technology/pattern, often rushed by managers to deliver.
- This pattern is called **Monolithic Front End**.
- It **does not scale** and **does not serve different use cases**.

#### Micro Front End

- This pattern is the better way.
- Different areas are designed, developed, and maintained by their respective experts.
- It allows different technologies to be involved in a single UI (e.g., one part in React, another in Vue.js, another in old jQuery technology) to serve different business use cases.
- It helps scale the team and allows each team to take decisions without impacting others (e.g., changes in the "beach area" won't impact the "open space area").

#### Technical Achievement Methods (Overview)

1. **iFrame way** (very famous).
2. **Web Component way** (very famous).
3. **Module Federation** (charming and popular way, especially if using React).
4. **Micro apps** or **route based** (trendy and seen for a long time).

### 2. Communication Protocol

The analogy used is communication between the user (client) and the server (kitchen).

|Protocol|Analogy/How it Works|Technical Features|Usage/Application|
|:--|:--|:--|:--|
|**Long Polling**|The waiter (client) is sent repeatedly to ask if the order is ready, bringing back partial data. This takes time, disturbs the kitchen, and misuses the waiter.|Request sent, and client keeps on asking until the entire data is received.|Generally used for **analytics and data display**.|
|**WebSocket**|The waiter sends the order, and the kitchen (server) promises to send updates when the food is ready, automatically.|Creates a persistent, **dual bi-directional connection**. Client sends a request and can get multiple updates in respect of it.|Anything which is **more real time** (e.g., chat applications or Google Doc where multiple users consuming/changing data see updates instantly).|
|**Server Sent Event (SSE)**|Client sends the order (requirement) once, and the server says it will keep sending the data once ready until all data is sent. The client cannot make another call or request.|Server sends data to the client once it is ready at the server.|Mostly used for **notifications** (e.g., Facebook notifications for friend requests, shares, or likes).|

### 3. Availability (Offline Support)

Motu Bhai wanted his business not to sleep even when he was sleeping ("I should sleep well but my business shouldn't be sleeping").

- **Business Goal:** Keep customers engaged even if the full service (like hot samosas from the closed kitchen) is unavailable.
- **Motu Bhai's Solution:** He provides a local version (like a refrigerator) with items like cold coffee and packed samosas. Customers who were authorized can access the data/things they had accessed during the day.
- **Technical Context:** This applies when the application is in an **offline mode** (network gone, connection not made with the server, or server is down).
- **Goal:** Show pages or static information that was available when online to keep the customer engaged.
- **Examples:** Opening google.com when internet is down provides a game. Google Doc allows modification in offline mode.
- **Technical Achievement:** This offline behavior is achieved using **Service Worker**.

### 4. Accessibility

As Motu Bhai grew his business, he faced a variety of customers (Chinese, Indian, Desi style) with different requirements, understanding skills, and challenges.

- **Definition:** Accessibility means the application "can be easily consumable by everyone".
- **Challenges/Disabilities Motu Bhai Encountered:**
    - People unable to understand English (menu/waiter speaks English).
    - People with speech disabilities (cannot speak properly).
    - People with hearing disabilities (cannot hear properly).
    - People unable to differentiate between colors (difficulty reading the menu).
- **Motu Bhai's Solutions:**
    - Provide menu card in **different languages**.
    - Keep the menu card **simple** to avoid color combination problems.
    - Waiters are able to understand and communicate using **sign language**.
- **Technical Solutions for Web Applications:**
    - Utilize **keyboard accessibility**, ensuring the tab key navigates input fields in a proper order.
    - Utilize **screen reader** support by defining things in a better format and using **HTML5 semantics**. This helps people with vision problems get an audio experience.
    - Define colors and **contrast** of the web application properly for people with vision challenges.

### 5. Consistency

Motu Bhai opened branches in different locations (Connaught Place, Kashmiri Gate, Chandni Chowk).

- **Challenge:** Maintaining the same quality of food items (samosas, tea) everywhere.
- **Analogy:** The branches represent different **browsers** (Chrome, Firefox, Opera).
- **Technical Challenges:**
    - **CSS consistency**.
    - JavaScript functionalities work in one browser but break in another.
- **Solutions:**
    - Changing **CSS properties** to ensure they work at different browser levels.
    - Writing **polyfills** for JS features not available by default in browsers.
    - Using auto-generating tools like **Style Components** (for React) to manage CSS differences.
    - Other polyfill solutions like **dynamic polyfill** and **modernizer**.

### 6. Design System (Scaling Consistency)

Motu Bhai aimed to expand the business further and become an **international brand**.

- **Problem:** Manual effort, testing effort, and quality check effort increases significantly with scale.
- **Goal:** Bring all standards and guidelines to a ** central place**.
- **Motu Bhai's Solution:** Set rules/guidelines, centralize core ingredients, and create documentation on how to consume them.
- **Technical Context:** Setting standards is difficult due to the variety of browsers, browser versions, operating systems (Linux, Windows, Mac OS, and their versions), and different hardware across the globe.
- **Solution:** Creating a **Design System**.
- **Functionality:** The design system contains atomic behaviors/ingredients (like how a modal or form looks) that are centralized as a standard/guideline.
- **Benefit:** Consumers follow the guidelines and do not have to worry about support across different platforms, as that is the responsibility of the team maintaining the design system.
- **Popular Design Systems:**
    - **Material UI** (Google).
    - **Atlantean design system**.
    - **Fluent UI** (Microsoft).
    - **Semantic** (Amazon).
    - And design.

### 7. Credibility and Trust (Search Engine Optimization - SEO)

After expanding internationally, Motu Bhai needs customers. He wants to build brand value so shop owners divert customers to his restaurant. This is achieved through quality promotion, self-promotion, and mouth advertisement, similar to paid advertising.

- **Digital Method:** **Search Engine Optimization (SEO) techniques**.
- **Two Types of SEO:**
    1. **On-page optimization**.
    2. **Off-page techniques**.

#### On-page Optimization

- It is crucial to showcase the content (like variety of samosas).
- Explicitly telling search engines what the content is, is better than having Google peep inside the website.
- **Key components of a web page:**
    - **Head:** Never underestimate the power of the head.
    - **Title**.
    - **Description**.
    - **Meta** tags (important for social media sharing).
    - **Content:** Must be done in the best possible way.
    - **Performance:** Very important; slow performance worries customers and makes them walk away.

#### Off-page Techniques

1. **Back links:** People directing users to the good quality service (credibility check).
2. **Ads:** Using platforms like **Google Tag Manager**, **Google AdWords**, or **Facebook Ads** to advertise the web application.

### 8. Logging and Monitoring

To maintain scale and prevent customer attrition, Motu Bhai must track likes/dislikes (reviews, feedback, suggestions) and maintain quality and hygiene. Challenges include damaged items (cracked glass), small sitting areas, and unavailable food.

- **Logging:** Keeping a record of every transaction, feedback, and stuff happening in the system.
- **Monitoring:** Monitoring what is happening all around the restaurant.

#### Technical Logging and Monitoring Categories

1. **Error logging (very important):** Tracking application failures (e.g., app not loading, features broken). This allows faster decision making since many customers walk away without complaining.
2. **User tracking:** Understanding users (where they come from, preferences, age group, language).
3. **User activity:** Tracking what users are liking and ordering.
4. **Feature/Content Utilization:** Monitoring which features/content are being utilized. This prevents investing time on unpopular dishes.
5. **Infra/Capacity Monitoring:** Monitoring if capacity is full (e.g., system down due to traffic, people not getting space to sit). Necessary to track capacity utilization so scaling decisions can be made before load comes.

#### Handy Tools

- _Error logging (UI):_ **Sentry**, **Trac** (or **TrackJS**).
- _User Tracking/Activity:_ **Amplitude**, **Log Rocket** (to see user actions and platform consumption).

### 9. Client Side Databases and Caching

Motu Bhai aims for better user experience by reducing server round trips, realizing information doesn't change frequently.

- **Caching Analogy:** Keeping readily available items like cutleries, tissue papers, or water bottles locally.
- **Database Analogy:** Setting up a buffet-type system (client side database) where frequently used, prepared items are kept, allowing local interactions.
- **Scenarios/Challenges:**
    - If data is available locally, it is served; if not, it is ordered from the server (kitchen).
    - Must handle **stale data** (or stale food), which cannot be consumed and must be updated from the server.

#### Technical Caching and Storage Methods

1. **HTTP Caching:** Caching static assets (images, CSS, JavaScript) that don't change frequently.
2. **In-memory caching:** Data is stored in memory, and policies decide how long the data lasts.
3. **Apollo Caching:** If using GraphQL/Apollo, decisions are made on whether to make a network request or retrieve data from the cache.
4. **State Management:** Libraries like **Redux** or **Context API** (in React) are popular for storing normalized or denormalized data in browser storage for consumption.
5. **Browser Databases/Storage:**
    - **Local storage**.
    - **Session storage**.
    - **Cookies** (for small information pieces, maintaining identity for client-server communication).
    - **Index DB**.

- **Benefit:** These techniques help achieve better optimization and performance on the client side.

### 10. Security

Security must be taken care of, regardless of business size or customer trust.

|Threat/Attack|Analogy|Technical Term/Solution|
|:--|:--|:--|
|**DDoS**|One customer placing many large orders at once (10 samosas, 20 pakodas), clogging the kitchen, then refusing to pay.|DDoS (Distributed Denial of Service).|
|**Impersonation/Unauthorized Access**|A smart bistro opens nearby, sets up a UI interface, starts ordering from Motu Bhai’s kitchen, but keeps the money.|**Authentication and Authorization** are essential before allowing people into the system.|
|**iFrame/Brand Hijacking**|A competitor puts Motu Bhai's website in an iFrame and uses his brand/web application as their own.|**Content Security Policies (CSPs)**: Configures who can access, who can inject things, and prevents unauthorized use of the brand.|
|**Cross-Origin Issues**|Configuring rules for who can access the service.|**CORS (Cross-Origin Request)**: Decide which domain/subdomain/browser can accept requests.|
|**Man in the Middle (MITM)**|Someone intercepts the order (samosa) by claiming it is for their table.|Must be avoided using various techniques.|

### 11. Performance and Optimization

This is an "all time favorite and hot topic" both in interviews and implementation.

- **Goal:** Provide better user experience by doing things as soon as possible.
- **Motu Bhai's Strategies:**
    - **Immediate Serving:** Serve water, provide menu card, and take the order immediately.
    - **Variety Serving:** Do not wait for the entire order to finish; serve items variety-wise as they are prepared. Waiting too long is not a good idea.
    - **Prioritization:** Serve items in a logical order (e.g., starters first, finger bowls at the end).
    - **Pre-preparation:** Prepare high-demand ingredients (like Chole) separately to speed up assembly and service.
    - **Size Optimization:** Optimize the size of the serving apparatus (e.g., use a small plate for tea, not a big, slow tray).

#### Technical Optimization Techniques

1. **Optimize Assets:** Optimize JS, CSS, and images. Prioritize and send only what is needed. Consideration must be given to "below the fold," "above the fold," and blocking/non-blocking behavior.
2. **Delivery Option:** Choose the best delivery mechanism based on use case (e.g., WebSocket, SSE). Keep non-server assets (like water containers) on the client side.
3. **Build Assets:** Optimize the bundle size. Ensure the asset contains only relevant information to speed up transfer and reduce resource waste.
4. **SSR (Server-Side Rendering):** Helps speed up processes by pre-cooking/optimizing data on the server side. It sends useful information in one shot to the client.
5. **Service Worker:** Helps in caching and other optimizations.

- **Monitoring:** Performance can be monitored using **Web Vitals** to check if standards are met.
- **Perceived Performance (Most Important):** Optimization must be **visible** to the end user. If the user does not feel happy, the optimization is largely a waste.

### 12. Testing

Testing is required to ensure the product, content, feature, and functionality work correctly.

|Type of Testing|Analogy (Motu Bhai)|Technical Terminology|Purpose/Tools|
|:--|:--|:--|:--|
|**Individual Testing**|Samosa must be individually tasty; empty samosa tastes good.|**Unit Testing**. Testing low-level programs and functions (e.g., logic to identify sour/sweet chutney).|Tools: **Jest**, **Mocha**.|
|**Combination Testing**|Samosa and chole taste good together; tea and samosa combination tastes good.|**Integration Testing**. Testing how prepared items look together and if they fit.|Tools: **Cypress** (can do integration testing).|
|**Entire System Testing**|The entire flow of consuming samosa, chola, and tea.|**End to End (E2E) Testing**. Testing the entire flow from preparation to delivery, consumption, and feedback.|Tools: **Cypress**, **Selenium** (popular), **Protector** (famous once), **Playwright** (from Microsoft, currently famous).|

---

_Note: The speaker mentions that technical details of many components (Micro Frontend methods, communication protocols, Service Workers, Design Systems, Caching policies, Security techniques, Performance techniques, and Testing frameworks) will be covered in upcoming episodes._

--------------------------------------------------------------------------