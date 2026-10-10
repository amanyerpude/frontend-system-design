Video_Link : https://youtu.be/XSVkWiW-t4k?si=U_3EMTSVYFExT7yE

--------------------------------------------------------------------------

This response details the content provided in the source transcript regarding network performance optimization for web applications, formatted for easier reading and understanding.

---

## Network Performance Optimization: Overview and Scope

This episode of the "chakde system design series" focuses on **network optimization**, a critical topic for web developers and a popular interview question that helps developers achieve senior status. Optimization is essential given the demands of modern web applications.

The overall plan for optimization (covered across multiple episodes) includes:

- **Asset Optimization** (JavaScript, CSS, images, and more).
- **Network Optimization** (how to utilize the network layer).
- **Build Optimization**.
- **Monitoring** all optimization efforts.

### Topics Covered in Network Optimization (This Episode)

This particular episode covers the following techniques in depth:

- Lazy loading.
- Loading JavaScript in an async fashion.
- Content visibility (a very important and cool technique).
- Serving critical CSS and the techniques to achieve it.
- Resource hints.
- Caching layers.
- Caching using Service Worker.
- Caching using CDN and beneficial headers.
- Client-Side Rendering (CSR) and Server-Side Rendering (SSR).
- Compression techniques.
- Techniques to avoid layout shift and repaint.

---

## 1. Optimizing JavaScript Loading (Async/Defer)

When a browser encounters a basic `<script>` tag in the HTML head, it performs a detrimental sequence of operations:

|Standard Script Loading (Blocking)|Async Attribute (`<script async>`)|Defer Attribute (`<script defer>`)|
|:--|:--|:--|
|**Parsing stops** when the script tag is encountered.|HTML parsing continues.|HTML parsing continues and **completes** without being halted.|
|Script is downloaded.|Script downloads in **parallel**.|Script downloads in **parallel**.|
|After downloading, parsing remains stopped, and the script is immediately **executed**.|Once downloaded, script execution **halts** the HTML parsing.|Script execution happens **only after** the HTML parsing is complete.|
|Nothing is visible on the webpage until all scripts are downloaded and executed.|Content visibility is still potentially delayed by execution halt.|Content is visible as soon as possible.|
|_Analogy:_ Must clean bike and fill petrol before leaving the house.|_Analogy:_ Can clean bike on the way, but must fill petrol before leaving for the party (still causes a certain halt).|_Analogy:_ Vehicle is cleaned on the way, and petrol is filled upon returning home.|

This optimization helps leverage the network layer and makes the web application fast.

---

## 2. Lazy Loading Techniques

Lazy loading ensures that work is done in parallel and does not hold up other tasks. It is primarily used for content that is **behind the fold** (not visible in the current viewport).

### A. Using the `loading` Attribute (Images/Iframes)

A new attribute, `loading`, can be used on images (`<img>`) or iframes (`<iframe>`):

- **`loading="lazy"`:** Used for assets behind the fold. It loads the asset in an async fashion and **does not halt** the parsing or rendering of the webpage.
- **`loading="eager"`:** Used for critical assets visible on the first screen (e.g., logo, hero images). This makes the resource load quickly but ensures it does not halt rendering.

### B. Setting Priorities (Using `priority` attribute)

Even among elements loading eagerly (on the first fold), you can set priorities. You can use an extra attribute called `priority` and set it to `low`. For instance, you might eagerly load a logo but set certain other eager images to a lower priority.

### C. Using the Intersection Observer API

This is a Web API provided by the browser (available via `window.intersectionObserver`). It is useful for high-content pages, such as those with infinite scroll or many images (like Google Photos or Flipkart product pages).

**Mechanism:**

1. A **sentinel element** (an element at the end of a container/div) is added to the DOM.
2. An instance of the Intersection Observer is created.
3. The browser observes the sentinel element.
4. When scrolling causes the sentinel element to intersect the viewport, a **callback** is triggered. This avoids the need to rely on scroll events, which are known to be glitchy in terms of performance.
5. Based on the callback, a decision (like making an API call to fetch the next set of images) is taken.

**Configuration Options (via `options` parameter):**

- **`root`:** Allows you to define scrolling within a specific container (e.g., a suggestion list in a search pop-up) rather than the entire viewport/document.
- **`threshold`:** Defines the percentage of the sentinel element that must be visible to trigger the callback (e.g., 50%). This is important to ensure the next batch of content loads before the user hits the end, avoiding a loading experience.

---

## 3. Content Visibility (CSS Property)

This is a powerful lazy loading technique requiring very minimal code changes—just one CSS property.

- **Property Used:** `content-visibility: auto`.
- **Impact:** Significantly reduces rendering time (e.g., in one example, rendering time reduced from 232 ms to 30 ms).
- **Behavior:** By setting this property, anything **not visible** in the viewport is subjected to **lazy rendering**. The content behind the fold is loaded and rendered in an asynchronous/lazy fashion.
- **Other Properties:** Values like `hidden` or `visible` can also be set. Certain attribute settings can prevent screen readers from reading content until the user scrolls to it.
- **Enhancement:** To make the browser's optimization decision better, you can set another property to predict the size of the content that will be loaded.

---

## 4. Serving Critical CSS

Since CSS files can be large (KBs or even MBs if unoptimized) and can halt rendering, only critical CSS should be loaded first.

**Technique:**

1. **Segregation:** Break CSS into two files: `critical.css` (above-the-fold CSS) and `full.css` (non-critical CSS).
2. **Blocking CSS:** Link `critical.css`. This file will block rendering until downloaded and parsed.
3. **Non-Blocking CSS (The Trick):** Link `full.css` initially setting its `media` type to `print` (which is non-critical).
4. Use an `onload` event to make the file available once the document loads.
5. The `onload` script changes the `media` type from `print` to `all`, making the CSS available for subsequent rendering and content below the fold.
6. This technique makes the non-critical CSS download asynchronously, leading to a major performance boost.

**Caution:** Any critical element needed for the first fold (like a sidebar) must be kept in `critical.css` to prevent parts of the application from breaking.

---

## 5. Resource Hinting

Resource hints allow developers to provide information to the browser so it can prioritize network calls and processing.

|Hint|Purpose and Behavior|Use Case|
|:--|:--|:--|
|**`preload`**|Hints the browser to download a critical resource immediately, so it doesn't wait during rendering/parsing. Used relation `REL="preload"`.|Critical JS/CSS required immediately after the page loads. Preloading lazy-loaded chunks to prevent displaying a loader experience.|
|**`prefetch`**|Fetches the required content but **does not execute it**. Downloads the asset and puts it into the cache.|Assets required later in the user journey, but not right away (avoiding unnecessary execution).|
|**`preconnect`**|Initiates the necessary DNS resolution, TCP handshake, and TLS handshake _pre-hand_ with a specific domain/subdomain.|Used for known external resources (e.g., Google fonts) to save time when the resource is actually requested.|
|**`dns-prefetch`**|Similar to `preconnect` (makes the handshake).|Used as a fallback strategy for older browsers.|
|**`prerender`**|Extremely powerful. Instructs the browser to create a **hidden tab content** and fully load the target page, including JS execution.|Pages highly likely to be navigated to next (e.g., dashboard after login). If the user navigates, the pre-rendered page is shown instantly.|
|**`modulepreload`**|Used for ES modules. Downloads, caches, and compiles the JS module code as soon as possible. Used relation `REL="modulepreload"`.|ES module loading. Can also be leveraged for modules required within a service worker by adding the hint `service worker`.|

---

## 6. Caching using Service Worker (SW)

Service Workers are powerful for establishing caching strategies and building Progressive Web Apps (PWA).

**Definition and Role:**

- SWs act as a **proxy server** sitting between the browser/web page and the internet.
- They intercept any network request going out.
- They can interact with the browser's cache and IndexedDB.
- SWs help provide a good experience even without internet connectivity (offline mode), as used by applications like Google Docs.

**Service Worker Life Cycle:**

1. **Installing:** The SW file is registered and installed.
2. **Installed (Waiting):** The new SW waits for older clients using the previous SW instance to close.
3. **Activating:** The SW is ready to listen to events. This stage is typically used for cleaning up previous caches.
4. **Activated:** The SW is fully operational and ready to listen to all events.
5. **Redundant:** Occurs if the SW is replaced by a newer service worker.

**Core Events and Strategies:**

|Event/Logic|Description|
|:--|:--|
|**Registration**|Must check if Service Worker is available in the web API before registering the SW file.|
|**Installation**|Used to define the initial caching step, such as opening a cache (with a specific name) and adding files to it.|
|**Fetch**|This is the most important event where the SW intercepts requests. Logic is defined here (using `self.addEventListener('fetch', ...)` to implement strategies like **Cache First** or **Network First**).|
|**Cache First**|SW looks for the requested asset in the cache; if not found, it makes a network call.|
|**Network First**|SW fetches from the network first. If successful, it puts the data into the cache. If the network call results in an error, it can fall back to the cache.|

**Workbox:** For complex scenarios, Workbox is a library that provides out-of-the-box capabilities, reducing the need to write extensive boilerplate code.

- Workbox features include prefetching, defining runtime cache strategies (like Cache only, Network only, or Cache and Network), routing requests, handling background sync processes, and providing good debugging capabilities.

---

## 7. Rendering Mechanisms (SSR, CSR, Static, and Pre-rendering)

Different rendering strategies impact costs, performance, and implementation complexity.

|Mechanism|Description|Example Users|Cost & Performance|
|:--|:--|:--|:--|
|**Server-Side Rendering (SSR)**|Server processes request for every route, generates HTML/JS, and ships the final rendered page.|Older applications.|Very high infra cost (server handles every request). TTFB (Time to First Byte) is very fast.|
|**Client-Side Rendering (CSR)**|Server provides basic JS/CSS; the browser handles routing, rendering, downloading, parsing, and execution using local chunks.|Most applications, actively used by Gmail.|100% JavaScript heavy, as everything is done on the client. TTI (Time to Interactivity) and FCP (First Contentful Paint) are very close.|
|**Static SSR (Static Rendering)**|Webpages (e.g., 100 blog pages) are pre-generated during the build phase (`npm run build`) and served as static HTML directly from the server.|Netflix (for static pages).|Storage cost for keeping pre-built assets. Avoids dynamic API calls.|
|**SSR with Rehydration**|Server generates basic HTML, but the browser must subsequently bind client-side events (rehydrate) to make the page interactive.|Next.js, Remix. Very popular approach.|High infra cost (requests hit the server) and GS size concerns.|
|**CSR with Pre-rendering**|Server provides a basic shell with fundamental HTML rendered. Dependencies (JS/CSS) required for fast rendering are injected in advance. The client handles the rest of the rendering.|Gatsby, VuePress. Very popular approach.|JavaScript cost is high. Flexible approach for optimization.|

---

## 8. Compression Techniques

Compression is a small but powerful technique used to reduce the size of assets (JS, CSS, images) shipped over the network.

- **Mechanism:** The server compresses the file using encoding, and the browser must understand how to decode it (similar to how air or water might uncompress materials).
- **Impact:** Reduces file size dramatically (e.g., 300 KB to 60 KB).
- **Algorithms:** The two popular strategies understood by browsers are **Brotli** and **Gzip** (Z zip). Brotli is often superior to Gzip (e.g., achieving an 8 KB reduction where Gzip might not).
- **Backward Compatibility:** To support legacy browsers that don't understand Brotli, the browser tells the server which compression methods it accepts. The server then serves the best available compressed version (Brotli if supported, otherwise Gzip).

---

## 9. Avoiding Layout Shift and Repaint

Performance is heavily impacted by the browser's rendering cycle, which must be completed quickly to avoid visual glitches or screen hangs.

### The Browser Rendering Cycle

1. **Request Page** (Get HTML response).
2. **Download Assets** (CSS/JS). _Note: CSS blocks rendering, while HTML/JS in the head block parsing._
3. **DOM Creation** (DOM Tree is created).
4. **Recalculate Styles/CSSOM** (CSS rules are generated).
5. **JS Execution**.
6. **Render Tree Creation** (DOM + CSSOM merge).
7. **Layouting (Reflow/Relayout):** Determining element geometry and position (the "building structure/pillars").
8. **Painting:** Filling pixels (the "walls, marble, paint").
9. **Compositing:** Placing final pixel-perfect items in layers (e.g., modals, tooltips, `position: absolute`).

### Why Timing Matters (The Frame Budget)

To ensure smooth scrolling and interaction (avoiding a "Shaktimaan" or "flash" experience), the browser must refresh the screen quickly.

- **Standard Refresh Rate:** Most devices operate at **60 Hertz** (60 refreshes per second).
- **Time Per Frame:** To maintain 60 frames per second (FPS), the browser must complete the entire rendering cycle within **16.67 milliseconds** (ms).
- **Actual Budget:** Due to background tasks, the browser typically has only **10 ms** to render a single frame successfully.
- **Failure:** If the browser cannot complete the required work within this budget, the application may hang or scroll glitchily.

### Costly vs. Efficient CSS Changes

Changes that trigger the entire rendering cycle (especially Layouting) are extremely costly in terms of computation time.

|Cost Level|Phases Triggered|Costly Properties (Avoid)|Efficient Properties (Repaint Only)|Most Efficient Properties (Compositing Only)|
|:--|:--|:--|:--|:--|
|**High Cost** (Layout Shift)|Layouting, Painting, Compositing.|**Dimension related:** `height`, `width`, `offset` properties, `client height`. **Interaction related:** Scroll properties (`scroll X/Y`, `scroll to top`).|N/A|N/A|
|**Medium Cost** (Repaint)|Painting, Compositing (skips Layout).|N/A|`background-color`, `text-color`, `shadows`, `visibility`.|N/A|
|**Lowest Cost** (Composition)|Compositing Only (skips Layout and Paint).|N/A|N/A|**`transform`** and **`opacity`**. Recommended for animations where dimensions change.|

**Final Tips to Avoid Layout Shifts:**

- Set the **height and width of images** when they render for the first time.
- Use **SVG placeholders** for images and videos to reserve the required display box.
- Club multiple fonts together for a single load/render.

--------------------------------------------------------------------------