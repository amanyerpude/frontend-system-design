Video_link : https://youtu.be/HLz3aZHOjjk?si=YSW1BLpybIRF3jDq

- Static Site Generation (SSG)
	- With Data
	- Without Data
		- No Data
		- Fetch Data on client

--------------------------------------------------------------------------
## Overview of Static Site Generation (SSG) in Next.js

The video, "Ep3 Static site generation (SSG) with and without data | Rendering Patterns | Next.js | ReactJs," follows a previous discussion on implementing Client-Side Rendering (CSR) and Server-Side Rendering (SSR).

### I. Rationale for SSG (Why Not Just Use SSR?)

When implementing SSR (using `getServerSideProps`), the Next.js server generates the complete HTML _every time_ a URL is requested.

- **The Problem with High Traffic:** If a website is highly popular and handles many requests, regenerating HTML constantly becomes a heavy operation on the server, potentially leading to memory errors.
- **The SSG Solution:** Static Site Generation (SSG) addresses this by building the entire HTML _once_ during the build time. This static build is then deployed.
- **Nature of SSG Output:** The resulting files are static HTML files that require **no computation** on the server. They function like simple HTML files containing standard HTML and script tags.

### II. Implementing the SSG Build Process

To implement SSG in Next.js, specific functions and commands are utilized:

- **Key Function:** The function central to SSG is **`getStaticProps`**.
- **Build Commands:** To generate a static build for production, the video defines an `export` command which uses two sequential Next.js commands:
    1. `next build`: Creates the primary build.
    2. `next export`: Exports that build into static files.
- **Output Folder:** The `next export` command generates an `out` folder that contains the static HTML files.

### III. Conflict: SSR vs. SSG Export

An attempt is made to export a page (`my-post.js`) that currently uses SSR (meaning it uses `getServerSideProps`).

- **Identification Symbols:** In the terminal output, a Lambda symbol ($\lambda$) indicates a server-side rendered page (rendered at runtime), while a circle ($\circ$) indicates a static page.
- **The Error:** Next.js throws an error because a page using `getServerSideProps` **cannot be exported**.
- **Reason:** `getServerSideProps` is designed to generate HTML during the **runtime**, not the build time, preventing its static export.

_(Note: An initial issue regarding Next.js image optimization incompatibility with `next export` required removing the `next/image` component for the demonstration to proceed)._ 

### IV. Scenario 1: SSG Without Data

This scenario simplifies the code by commenting out data fetching, generating a static site from basic content (e.g., "These are my post no data").

- **Success:** Running `npm run export` successfully generates `my-post.html` in the `out` folder.
- **Data Structure:** The generated HTML file still contains some JSON structure (e.g., `pageProps`), which is crucial for data handling in SSG with data. This JSON is where data would be added if API calls were made during the build.
- **Serving and Caching:** The static `out` folder can be hosted directly. Since the HTML is built once, it doesn't need to be regenerated for every request, and the files can be cached on a CDN (Content Delivery Network) closer to the customer, making serving very fast.
- **Ideal Use Case:** SSG without data is highly useful for **less dynamic websites**, such as a video blog, where content, once written, changes very infrequently and is served identically to every user.

### V. Scenario 2: SSG With Data (Build-Time Fetching)

The goal here is to populate the HTML with data fetched from an API during the build process.

- **Implementation:** The code is updated to replace `getServerSideProps` with **`getStaticProps`**.
- **`getStaticProps` Definition:** This function instructs Next.js to pre-render the page at **build time** using the data (props) it returns.
- **Build Output:** The export process succeeds, generating "static HTML + JSON".
- **Data Handling:** During the build:
    1. The API calls inside `getStaticProps` are executed.
    2. The server waits for the data to be fetched.
    3. The prefetched data is embedded directly into the HTML file's props (the associated JSON).
    4. When the hosted HTML file loads, the data is served from this embedded JSON, resulting in a **super fast** load time because no runtime API calls are needed for the initial render.
- **Major Drawback:** The process must **wait** for _all_ API calls to finish during the build. If APIs are slow, this significantly increases the total build time of the application.

### VI. Scenario 3: SSG Fetching Data on the Client

This approach generates a static HTML file but avoids waiting for APIs during the build time.

- **Implementation:** `getStaticProps` is commented out. The page uses basic React code (e.g., `useEffect`) to fetch data _after_ the page component has loaded on the client's browser.
- **Behavior:** The initial HTML served is static (e.g., showing a "loading from API" message or a skeleton loader). After the static HTML is displayed, the client's browser makes a separate API call to fetch and display the data.
- **Advantages:**
    - It reduces build time significantly by removing API waits.
    - It allows for showing skeleton loaders during the client-side data fetching.
- **Ideal Use Case:** This method is necessary for data that is **highly dynamic** or changes frequently (partially dynamic websites).
    - **Example:** A property listing website where the number of available properties changes constantly (e.g., one property gets sold).
    - **Efficiency:** Instead of having to generate and deploy a whole new build every time a small piece of data changes, the client makes a fresh API call on load, retrieving the updated data from the backend database.

### VII. Conclusion on Rendering Patterns

The video emphasizes that every rendering pattern has its own use case, advantages, and disadvantages.

- **Bias Avoidance:** It is unfair to claim one pattern is universally better; developers must be clear about the specific scenarios where a pattern will be beneficial.
- **SSG Limitation:** SSG is generally **not suitable** for websites that are **highly dynamic** (like social media feeds). SSG with client-side fetching is suitable for partially dynamic or less dynamic sites.
- **Addressing Data Stale Issue:** A major plot point remaining for SSG is how to handle situations where statically generated content (like a blog post) is updated by an author after the initial build. Rebuilding the entire application for one small change is inefficient.
- **Next Topic:** This issue leads directly to the discussion of **Incremental Site Regeneration (ISR)**, which will be covered in the subsequent video.

--------------------------------------------------------------------------

