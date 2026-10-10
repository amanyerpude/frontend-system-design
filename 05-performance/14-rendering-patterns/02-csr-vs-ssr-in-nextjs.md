Video_link : https://youtu.be/PyY8yI47Kqw?si=Qvn_Xh8bSsV9tpcu

- Server Side Rendering (Pre-render) SEO Friendly
- CSR : Client Side Rendering
	- NOT SEO FRIENDLY :The web-crawler will not be able to crawler the API call data as the page will fully render on the Client side and crawler does not have a client side.
- SSR : Server Side Rendering
	- If you export a function called 'getServerSideProps' (SSR) from a page. Next.js will pre-render this page on each request using the data returned by 'getServerSideProps'.
	- SEO FRIENDLY : Now the crawler will be able crawler all the API request data on the page.

--------------------------------------------------------------------------
## Overview of Client-Side Rendering (CSR) and Server-Side Rendering (SSR)

The video is the **second episode of the Rendering Pattern Series**. It focuses on **Server-Side Rendering (SSR)** as an example of **pre-rendering**, a concept discussed in the previous episode. The purpose is to compare how pre-rendering (SSR) looks versus no pre-rendering (CSR) by examining the **network tabs** in the browser.

### I. Definitions and Next.js Context

- **Next.js:** Next.js is a **production framework** built on top of **React.js**. It adds extra functionalities that help developers achieve SSR, Static Site Generation (SSG), and other features. The speaker noted they were impressed by the **level of control** Next.js provides over application rendering.
- **Client-Side Rendering (CSR):**
    - A basic React application is CSR.
    - By default, a simple Next.js application is also CSR if one does not utilize the framework's new features (e.g., by not using `getServerSideProps`).
    - In CSR, the content only becomes visible **when the JavaScript (JS) is loaded on the browser**.
- **Server-Side Rendering (SSR):** This is a **different case** from CSR and is the focus of the pre-rendering demonstration.

### II. Setting Up the Demonstration Application

1. **Project Setup:** The application was created from scratch using a command copied from `nextjs.org/docs`. The project was named "**my app**" and run using `npm run Dev`.
2. **API Data Source:** A basic JSON structure was hosted on **jsonbin.io**.
    - The bin must be **public** to allow API calls.
    - The data consists of an array called `post`, where each object has an **`ID`** and a **`title`**.
3. **Routing:** In Next.js, routes are created by adding a new file inside the **`pages`** folder. The component file created was **`my posts.js`**, which maps to the URL `/my posts`. The goal was to fetch the API data and render the posts as `UL`/`Li` tags.

### III. Demonstration of Client-Side Rendering (CSR)

The first approach implemented fetching using classic React logic (`useEffect` and `useState`).

#### CSR Implementation Details:

- `useEffect` was used to make the `fetch` call upon component mount.
- `useState` was used to store the fetched data (`post`) as an empty array initially.
- The data (located in `data.record.post`) was then mapped to render the `Li` tags displaying `post.title`.

#### CSR Key Observation (Network/Response Tab):

- When the HTML file served from the server was inspected, the visible post content (e.g., "**my post one**") **could not be found** in the initial HTML response.
- The HTML served statically included surrounding text (like "These are my posts"), but the **`UL` tag was empty**.
- The content was rendered on the client only **after the relevant JavaScript was executed** by the browser.

#### The Problem with CSR and SEO:

- Because the content is generated client-side, **web crawlers** (which typically read only the server data and **do not execute client-side code**) receive the empty HTML structure.
- This makes the approach **not SEO friendly**, explaining why most basic React applications have **poor Search Engine Optimization**.

### IV. Converting to Server-Side Rendering (SSR)

Next.js provides a simple method to convert the application to SSR, a procedure which would otherwise be "very painful" in basic React (requiring modification of the whole React config from scratch).

#### SSR Implementation Details:

- The conversion requires exporting an asynchronous function named **`export function getServerSideProps()`** from the page file.
- **Functionality:** Next.js uses this function to **pre-render the page on each request**.
- The API fetching logic (`await fetch`, `await promise.json`) is moved **inside** `getServerSideProps`.
- The fetched data is passed to the main component via a `props` object (e.g., `return { props: { posts: data.record.post } }`).
- The main component receives the data as `props.posts`, eliminating the need for `useState` and `useEffect` for data fetching.

### V. Demonstration of Server-Side Rendering (SSR)

- **Execution:** When the server processes the request and sees `getServerSideProps`, it executes the function _before_ returning the HTML.
- **SSR Key Observation (Network/Response Tab):**
    - When the HTML served from the server is inspected, the visible content ("my post one," "my post two," etc.) **is now present**.
    - The **`UL` tag is not empty**; it contains the `Li` tags with the post data.

#### The Power of SSR and SEO:

- This approach is **extremely powerful** and **very SEO friendly**.
- The visible content (the `Li` tags) is sent immediately with the initial HTML response from the server.
- Web crawlers now receive the actual post content when they hit the server, solving the previous SEO issue.

### VI. Conclusion and Encouragement

The speaker calls Next.js a "**game changer**" and strongly encourages developers who are comfortable with React.js to **move to Next.js** to "uplift their level". The speaker is happy to contribute to viewers' careers and interviews, emphasizing that they record these videos sometimes on Sunday, working extremely hard. The video's target is **50 likes**.

--------------------------------------------------------------------------

