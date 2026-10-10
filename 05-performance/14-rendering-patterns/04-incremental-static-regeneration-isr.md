Video_link : https://youtu.be/a96xrB27jj4?si=ZUDoMrUxFCE6hIiE

--------------------------------------------------------------------------
## Detailed Video Description: Incremental Static Regeneration (iSG)

The video is the final episode of the Rendering Pattern Series and focuses on **Incremental Static Regeneration (iSG)**. The instructor confirms that previous episodes covered Client-Side Rendering (CSR), Server-Side Rendering (SSR), and Static Site Generation (SSG). The main goal is to provide a clear understanding of iSG so viewers can determine the best rendering pattern for their specific project conditions.

iSG is discussed in two primary scenarios: **Auto Revalidate** and **Revalidate On Demand**.

---

### 1. Next.js iSG Overview

Incremental Static Regeneration (iSG) is a feature Next.js offers that allows developers to **create and update static pages _after_ the site has already been built**.

**Key Advantages over SSG:**

- iSG enables static generation on a **per-page basis** without requiring the entire site to be rebuilt.
- It avoids the "time digging process" of manually running `next build` and `next export` every time there is a small data change, a significant hassle associated with basic SSG.
- iSG accomplishes this by using the concept of **revalidations**.

---

### 2. Scenario A: Auto Revalidate

Auto Revalidation automatically re-evaluates and updates a page's content after a specified time interval.

|Implementation Detail|Description|
|:--|:--|
|**Conversion from SSG**|A simple SSG page (e.g., `my post` page) is converted to iSG by introducing the `revalidate` property.|
|**Code Structure**|The `revalidate` property is introduced as a sibling to the `props` property within `getStaticProps`.|
|**Functionality**|Setting `revalidate: 5` means the page will re-evaluate itself after 5 seconds. The general principle is re-evaluating after $n$ number of seconds.|
|**Server Command**|The server is run using `next start`, which consumes the production build created by `next build`.|
|**Process upon Revalidation**|When the page is re-evaluated, a network call is made to fetch new data, and a new HTML file is generated.|
|**Serving Mechanism**|Until the new HTML file is successfully generated, users are served the **older version** of the page.|
|**Scope**|Only the specific page where `revalidate` is present (e.g., `my post.js`) will be re-evaluated; other static pages without this property are unaffected.|

**Drawback (Wastage):** Auto revalidation is considered an **expensive operation** because it consumes server resources by re-evaluating the page constantly, even if the underlying data has not changed. It is estimated that 80% of the time, the data may not be updated, leading to resource wastage.

**Suitable Use Case:** This method is deemed perfect only when the developer is **sure that the data is changing frequently** and predictably after $n$ seconds, such as in the case of constantly updating financial charts (like Nifty or Sensex values).

---

### 3. Scenario B: On-Demand Revalidation

On-demand revalidation addresses the wastage issue of auto revalidate by allowing the site to be updated manually, specifically when content in a headless CMS or e-commerce data changes.

**The Core Problem Solved:** How do we tell Next.js that the API data has updated and that it needs to revalidate or re-evaluate the specific page?

#### Mechanism: API Routes

1. **API Folder:** Next.js allows developers to build APIs using the `pages/api` folder.
2. **API Endpoints:** Any file inside `pages/api` (e.g., `revalidate.js`) is mapped to `/api/filename` and is treated as an API endpoint, not a regular page.
3. **API Structure:** These API routes use a default function handler (similar to Node.js/Express.js requests).

#### Implementing the Revalidation API

The implementation requires creating a dedicated API endpoint (e.g., `/api/revalidate`).

|Component/Step|Detail|
|:--|:--|
|**Security (Secret)**|A **secret key** (e.g., `JS Cafe`) is added to the API code to protect it from external intruders or malicious requests. The API will only process the request if the secret matches.|
|**The Revalidation Command**|The key command is `await response.revalidate(path)`.|
|**Functionality**|When this command is hit, Next.js finds the page associated with the provided route (e.g., `/my-post`), automatically runs its `getStaticProps` function, fetches the new data, and updates the page.|
|**Dynamic Input**|The path to be revalidated is passed dynamically via the request body (e.g., a JSON object with a key like `path_to_revalidate`).|
|**Outcome**|When the revalidation API is successfully hit (sending a `status 200` with `revalidated: true`), the corresponding front-end page instantly shows the new data upon refresh, as the HTML response has been updated.|

#### Avoiding Manual Work

While the demonstration involves hitting the revalidation API manually via Postman, the power of on-demand iSG lies in **automation**.

- The revalidation API should be triggered by the backend service.
- For example, after a user updates a blog post, once the database transaction is successful, the backend automatically hits the front-end revalidation API.
- This process tells the Next.js server to update the specific page, eliminating any manual intervention.

---

### 4. Importance and Conclusion

The instructor stresses that understanding these four rendering patterns (CSR, SSR, SSG, and iSG) is **very important for interviews**, especially Low-Level Design (LLD) or High-Level Design (HLD) interviews, where candidates are often asked to justify which pattern they would select for a project.

The video is intended to be simple and clear so that even those without prior Next.js knowledge can understand the concepts.

--------------------------------------------------------------------------

