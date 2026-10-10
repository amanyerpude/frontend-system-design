
Video_Link : https://youtu.be/CegvBL1L8qA?si=QdifYk0VnheBLNmn

--------------------------------------------------------------------------

This response draws exhaustively upon the provided source transcript, tallying all details mentioned regarding web storages and related concepts.

---

## I. Introduction and Scope

This is the second video in the system design Building Block series. The series aims to discuss various front-end topics that help developers ace system design interviews, even when designing unfamiliar applications.

**Topics to be Discussed:** The video focuses on web storages, specifically discussing:

- Local storages.
- Session storages.
- IndexedDB (referred to as indexp) and others.
- When data should be moved to the server instead of using client-side storages.

**Storages Covered:**

- URL.
- Cookies.
- Session storage.
- Local storage.
- Cache API.
- IndexedDB.
- Remote databases.

**Problem Solved:** It is often difficult for developers to choose the correct location for data storage. Local storage is frequently overused as the default, while storages like Session Storage and IndexedDB are relatively less popular. This video helps determine the best tool for a particular case.

**Storages Left Out of Scope:**

- **File System API:** Due to very poor browser support.
- **WebSQL:** Because it is now deprecated and has been replaced by IndexedDB.

## II. Foundational Concepts

### Storage Definition

- A storage is defined as a place where data is stored.
- The simplest storage is JavaScript memory, where the values of variables can be stored.
- Storage is needed in any application requiring data storage.
- Storage is _not_ needed for simple static sites, which may not even require JavaScript, consisting only of simple HTML pages.

### Origin Definition

It is important to understand the term **Origin** as it will be frequently referenced. An Origin is a combination of:

1. **Protocol:** Might be HTTP or HTTPS.
2. **Host name:** The address of the site.
3. **Port:** The port on the server addressed to receive a website (usually hidden because the default port is used). _Example:_ `example.com` and `example.com/app/index.html` have the same Origins.

### Storage vs. Storage Manager

Storage is not equal to a storage manager.

|Feature|Storage|Storage Manager (e.g., Redux)|
|:--|:--|:--|
|**Physical Space**|Takes up physical space on the disk.|Does _not_ provide physical disk space.|
|**API/Function**|Provides an API for interactions. Stores data. Persistent storage stores data forever until manually removed.|Works on top of existing storage. Provides an interface for working with storage. Facilitates data operations (CRUD, subscriptions for updates).|
|**Data Lifetime**|Varies (capacity and lifetime).|Can provide temporary or permanent storage (e.g., Redux offers both).|

## III. Storage Types and Characteristics

### 1. URL Search Parameters (Query String)

Considering the URL as a storage mechanism is not obvious. It utilizes search parameters or a query string, often thought of as query parameters.

|Feature|Details/Behavior|
|:--|:--|
|**Data Retention**|Survives a page refresh. **Lost** when the page is closed. Cannot be considered a safe place to store data.|
|**Advantages (Unique Feature)**|Easy to share with other users. Can be saved in browser bookmarks. Other storages (e.g., local storage) do not have this easy page state sharing feature.|
|**Drawbacks**|Fragile (easily removed or lost). Not very secure (all data is visible). Small capacity (e.g., limited to 2,000 symbols in Microsoft Edge). Increases URL length.|
|**Suitable Data**|Short and primitive data like string, number, or boolean values.|
|**Use Cases**|Current page state (ideal for sharing state, like a configured dashboard view or map). Navigation state (e.g., page number, filter, and search query in a table component). Sharing a page with configured parameters (e.g., an online clothes shop).|

### 2. Cookies

Cookies are distinct from other storages.

|Feature|Details/Behavior|
|:--|:--|
|**Unique Features**|Can be set by the server. Automatically sent to the server with every HTTP request.|
|**General**|Widely supported and easy to use.|
|**Capacity**|Limited storage capacity: about **4 kilobytes per origin**.|
|**Security**|Provides better protection against Cross-Site Scripting (XSS) attacks by using the `HTTP only` parameter to hide cookies from attackers.|
|**Drawbacks**|Sent with every HTTP request, increasing HTTP overhead. Not suitable as a complete storage solution for data.|
|**Quota System**|Has an individual quota.|

**Use Cases for Cookies:** Cookies are very useful in a limited number of scenarios:

1. **Session Management (Ideal Tool):** Because they are server-set and automatically sent, they are ideal for managing user sessions without requiring JavaScript management.
    - _Process:_ User sends credentials -> Server validates and sends success response with a token in cookies -> Browser automatically sends cookies with subsequent requests -> Server validates the token and authorizes the user.
2. **Personalization:** Since cookies are sent with each request, the server can display a personalized page immediately.
    - _Process:_ First visit, user selects settings -> Server sets cookies. Next visit, browser sends request + cookies automatically -> Server reads cookies and returns the personalized page. This avoids a two-step process of requesting a default page, determining language, and then requesting the correct page.
3. **Tracking User Behavior:** Used by advertising providers.
    - _Process:_ Ad script on Site A sends user info to provider server -> Provider server generates a unique User ID and sets cookies in the browser. User visits Site B connected to the same provider -> Ad script sends cookies/User ID -> Provider server uses the ID to look up past visited sites to show personalized advertisements.

### 3. Local Storage

Local Storage is the first real candidate discussed for data storage, whereas Cookies and URLs have dedicated purposes.

|Feature|Details/Behavior|
|:--|:--|
|**Persistence**|Persistent storage (data will not be deleted after a certain time, provided quotas are respected). **Caveat:** Not completely persistent; data can still be lost, so it should not be used for important data.|
|**Ease of Use**|Simple, not very powerful, and easy to start using.|
|**Capacity**|**Limited capacity** (typically 5 MB per origin). In WebKit-based browsers, this may only be 2.5 MB because JavaScript stores strings in UTF-16 format. The capacity is 5 to 10 MB or the size of the shared quota.|
|**API**|**Slow synchronous API**. Operations occupy the main thread, potentially slowing down the application.|
|**Availability**|**Not available** inside Web or Service Workers due to its synchronous API. This is inconvenient for offline features.|
|**Data Types**|Supports **only strings**. Developers must encode/decode other data types, which requires resources and may slow the application.|
|**Quota System**|Shares the same quota as IndexedDB and Cache API.|

**Use Cases for Local Storage:**

- Persistent client storage: Storing various user settings and application data (selected theme, language, etc.). Useful when a database is not desired or possible.
- Offline usage: The easiest solution for storing data for a simple offline mode. Can save results of offline operations to send to the server later. Used for developing simple offline-first applications or temporary storage during internet loss.
- Saving drafts: Common in applications with lengthy user input (e.g., blogging platforms).

**Note on Criticism:** Local storage is often criticized for small capacity and slow performance. While not scalable, it is effective and convenient for scenarios dealing with small amounts of data.

### 4. Session Storage

A session begins when a user opens a web page in a browser.

|Feature|Details/Behavior|
|:--|:--|
|**Data Retention**|Retains data during tab refreshes and restores, even if the browser is closed, provided it's within the same session.|
|**Session End/Clearing**|Data is **automatically cleared** when the user closes a tab, or when it is closed by the browser and not restored. This automatic cleanup removes the need for manual data deletion.|
|**Tab Scope**|Does not transfer data between tabs, even if they share the same origin (one tab = one session).|
|**Tab Duplication**|**Exception:** If a tab has been duplicated, the original and the duplicated tab will share the same session store.|
|**Capacity**|Some browsers (e.g., Internet Explorer) offer more space than local storage. Capacity ranges from **5 to 15 megabytes**.|
|**Drawbacks**|Shares the same drawbacks as local storage (inherited limitations).|
|**Quota System**|Operates under a separate quota system (individual quota).|

**Practical Uses for Session Storage:**

- Caching: Storing server-retrieved data once and reusing it throughout the session.
- Data sharing across screens: Ideal for transferring user or system data across different screens within the same session, especially useful for multi-step processes like Wizards.
- Storing global configuration: A good alternative to global variables or storage managers for application settings.

### 5. Cache API

The Cache API is distinct from browser cache. It is dedicated to storing **request/response pairs**.

|Feature|Details/Behavior|
|:--|:--|
|**Purpose**|Specifically for storing server responses, not for general storage (unlike Local or Session Storage).|
|**Availability**|Accessible in **Web and Service Workers** (a unique storage option).|
|**Mechanism**|Service Worker intercepts requests; if the response is not found, it requests it from the server and stores it in the Cache API for later retrieval. Subsequent requests are returned from the cache without hitting the server.|
|**Quota System**|Shares the same quota as IndexedDB and Local Storage. Maximum capacity is equal to the size of the shared quota.|

**Use Cases for Cache API:**

- Offline Mode: Plays a crucial role in enabling offline mode when combined with Service Workers, by caching assets (HTML, CSS, static files).
- Performance Improvement: Reduces network requests and leads to faster page load times by reusing cached assets retrieved from disk.
- _Real-life example:_ Slack uses Service Workers and Cache API to cache assets (HTML, JavaScript, CSS) and data (in IndexedDB) to enable quick app startups (warm boots) and usability without an internet connection.

#### Cache API vs. Browser Cache

|Feature|Cache API (Developer Controlled)|Browser Cache (Browser Controlled)|
|:--|:--|:--|
|**Control**|Controlled by developers.|Controlled by the browser.|
|**HTTP Header Dependency**|Does _not_ depend on the `Cache-Control` HTTP header.|Controlled based on the `Cache-Control` HTTP header.|
|**Flexibility**|Very flexible; allows manual data writing.|Not very flexible; manual reading or writing is impossible.|
|**Primary Goal**|Primarily used for offline accessibility and resource reusing.|Mainly aims to reduce network traffic for the user.|
|**Quota System**|Shares quota with IndexedDB.|Has an individual quota; controlled by the browser (developers shouldn't care).|

**Browser Caching Scenarios (When the browser receives a cacheable response):**

1. **Cache Control Enabled:** The file is stored on disk in the browser's cache. The next time, it is retrieved from disk without a server request. This allows offline use and saves resources when online. _Caution:_ Cache control settings must be carefully adjusted to avoid returning outdated content.
2. **Cache Control Disabled:** The browser must make a request to the server. Usually, the request goes to the CDN. If the file is relevant, the CDN returns a `304` status code and an empty body. The browser then retrieves the resource from the local cache. This method requires an additional request but decreases the risk of outdated files.

### 6. IndexedDB (Indexp)

IndexedDB is a NoSQL database available in browsers.

|Feature|Details/Behavior|
|:--|:--|
|**Structure**|Allows storing large amounts of structured data. Supports almost all JavaScript types, except for some (e.g., functions).|
|**Capacity**|Large capacity. Maximum capacity is equal to the size of the shared quota.|
|**Power/Features**|Very powerful; supports advanced features such as indexing, transactions, and versioning.|
|**API**|Fast because it supports **asynchronous operations**.|
|**Availability**|Available in Web and Service Workers.|
|**Complexity**|**Steeper learning curve** (more difficult to develop and maintain). Wrapper libraries like `idb` are recommended to hide complexity and increase convenience.|
|**Quota System**|Shares the same quota as Local Storage and Cache API.|

**IndexedDB Reliability Concerns (Problems often acutely discussed for IndexedDB):**

1. **Intelligent Tracking Prevention (ITP):** Safari deletes data (including Local Storage and IndexedDB) after 7 days of inactivity. This means browser storage is generally only a one-time cache and not suitable for long-term user data storage.
2. **Private Mode:** IndexedDB does not work in private mode in Firefox. There is no API to detect private mode; the only solution is to attempt to open a connection with IndexedDB and show a possible solution if it fails.
3. **Confusing Quotas:** Quotas vary from browser to browser (e.g., Firefox quota is shared among all subdomains). There is no predictable way to handle quota exceeded errors across all browsers.
    - _Note:_ These ITP and quota issues also affect other storages like Local and Session Storages, but they are critical for IndexedDB because it is used when developers rely on storage for a long period.

**Use Cases for IndexedDB (When IndexedDB should be considered):**

- Storing large amounts of data on the client side (due to high capacity and fast asynchronous API).
- Storing data with complex structure (due to broad support for JavaScript types).
- Storing binary files (e.g., media content for offline usage or uploading media files).
- Offline first applications (must work as well offline as online, or applications that rely solely on client-side storages with no backend).

### 7. Remote Databases (Server-side)

Remote databases reside on the server and are part of the backend architecture. Sometimes it is smarter to move data to the server.

|Feature|Details/Behavior|
|:--|:--|
|**Reliability**|Provides the **highest reliability** compared to client-side storages.|
|**Security**|Most secure for sensitive data, such as personal information or credentials. Client-side storages are vulnerable to attacks like XSS.|
|**Data Sharing**|Facilitates **data sharing across devices**.|
|**Complexity**|Requires additional team or external resources.|
|**Cost**|High costs (maintaining database, server disk space, handling data requests, scaling infrastructure).|
|**Interface Speed**|Slower interface: Every data interaction requires server requests, potentially slowing the UI, especially for users with slower internet connections.|
|**Offline Use**|Cannot be accessed offline; application function is significantly reduced or becomes useless without internet.|

**Use Cases for Remote Databases:**

- Sensitive data.
- Data that needs to be shared between devices.
- Important data.
- Server-side databases are commonly used in almost every web application as they are secure, reliable, and do not face client-side quota limitations.

## IV. Quota Systems and Eviction Policies

### Shared Quota

Disk space allocated by the browser for storing data. The following storages share the same quota:

- Local Storage
- IndexedDB
- Cache API
- WebSQL
- File System API

**Impact:** If one shared storage (e.g., IndexedDB) consumes the entire quota, other shared storages (e.g., Local Storage) cannot write data, activating the eviction policy.

**Shared Quota Size by Browser:**

- **Chrome-based browsers:** 60% of the total disk space per origin.
- **Safari:** 20% of the total disk space per origin.
- **Firefox:** 10% of the total disk space per group (domain + subdomains), with a maximum of 10 GB in best effort mode.

### Individual Quotas

The following storages have individual quotas:

- **Cookies:** 4 kilobytes.
- **Session Storage:** 5 to 15 megabytes.
- **Browser Cache:** Controlled by the browser (developers should not worry about it).

### Eviction Policy

Even persistent storages can delete data, managed by the eviction policy. Browsers use one of two policies:

1. **Best Effort (Default):**
    - If the quota is exceeded, the browser automatically removes data.
    - Uses the Least Recently Used (LRU) policy.
2. **Persistent Mode:**
    - The browser will **not** automatically delete data when the quota is exceeded.
    - Write operations will throw a `QuotaExceededError`.
    - Requires wrapping write operations in a `try-catch` block.
    - Needed when the application is highly dependent on offline storage (e.g., offline-first applications).

**Switching to Persistent Mode:**

- **Firefox:** Requires asking for user permissions.
- **Safari and Chrome-based browsers:** Happens automatically based on user behavior (Allow or Disallow).

## V. Applying Storage Decisions (Facebook Example)

The example uses Facebook components and follows functional requirements. The decision-making process relies on the data's **lifetime** and **importance**.

|Data Characteristic|Recommended Storage|
|:--|:--|
|**Short-Lived, Not Important**|Session storage.|
|**Long-Lived**|Database (client-side storage is unreliable).|
|**Long-Lived/Important (Offline Exception)**|IndexedDB (if application is used only or mostly offline).|
|**Sensitive Data (Passwords, Tokens)**|Remote Database (must not be stored client-side due to XSS vulnerability).|

### 1. Search Parameters (Filters, Sorting)

- **Data Type:** Search parameters (name, city, age, sorting options).
- **Requirement Analysis:** Not very important; data loss is not a big problem; short-lived. User may want to save filter configurations in bookmarks or share with friends.
- **Decision:** **URL search parameters** are a good choice.

### 2. Post Drafts

Draft creation (title, text, attachments) may take time, and users need to save and return to them.

**Possible Locations for Draft Storage:**

1. **Local Storage:** Simplest solution, suitable if few drafts are needed and the draft is plain text.
2. **IndexedDB:** More complex but offers scalability; suitable for a large number of drafts, especially those with a complex structure (title, content, text, attachments).
3. **Database:** Most robust solution; allows data sharing between devices; covers all cases. Drawbacks include maintenance difficulty and requiring server requests for reading/writing.

**Optimal Solution Analysis:**

- A database is the best solution for reliability and allowing users to continue drafting on another device.
- However, if the post is expected to be relatively short and completed on one device, IndexDB or Local Storage can be used, which is quicker as it doesn't involve the backend team.

**Why Local Storage is _Not_ Recommended for Complex Drafts:**

- Drafts often have complex structure (title, text, files).
- Posts can be long, and Local Storage's synchronous API can slow down the application when reading/writing large amounts of data.
- Local storage does not support binary files, which is necessary if users want to add media files offline.

**Recommended Strategy (Remote DB Primary, IndexedDB Secondary):** The final recommendation is to use a **Remote Database as the primary storage** and **IndexedDB for offline mode only**.

- **Why Remote DB over IndexedDB as primary?** IndexedDB fails two criteria important for drafts: it does not allow data sharing between devices, and it is not completely reliable (data loss is likely).

**Synchronization Strategy:**

1. **User is Online:** Simply save the drafts to the remote database.
2. **User is Offline:** Save the drafts to IndexedDB.
3. **User Comes Back Online:** Synchronize IndexedDB and the remote DB.
--------------------------------------------------------------------------