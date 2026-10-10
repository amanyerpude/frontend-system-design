Video_Links : https://youtu.be/KUdqbIHn8Ic?si=1FMzAM0pL2ED_4i3

--------------------------------------------------------------------------

This response draws exhaustively on the provided source transcript, following the requirement to detail every point mentioned and format the information for clarity and understanding.

---

## The 2025 Web Performance Roadmap: Comprehensive Overview

The purpose of this discussion is to cover **web performance**, what it entails, and how to build fast web applications. This information is useful for preparing for system design interviews and enhancing skills as a web developer.

### I. Defining Web Performance

Web performance must be evaluated in two primary directions:

1. **How fast the application loads**.
2. **How fast the application works** (UI speed).

An application is considered **fully performant only if it loads quickly and runs fast**.

If an application takes five minutes to load but then works fast, it cannot be called fast. Conversely, if an application loads very fast but works slowly, it also cannot be called performant.

### II. Balancing Trade-offs and Prioritization

Software development involves balancing trade-offs and benefits. If resources are unlimited, work should proceed in both loading time and UI speed directions; however, in reality, resources are limited, and one direction may be more important than the other in different cases.

#### A. When Loading Time is Critical

Loading time should be seriously prioritized in several critical scenarios:

|Factor|Description|
|:--|:--|
|**Competitive Market**|Cannot afford to be slower than competitors; users will easily find replacements if the site takes too long to load.|
|**Mobile Users**|Mobile users often have poor connections, and slow loading severely worsens the experience.|
|**Conversion Rate Dependency**|If revenue depends on loading speed, this must be taken very seriously.|
|**Impulse Purchases**|For websites where users gravitate toward impulse purchases (e.g., e-commerce), the path from idea to purchase must be as smooth as possible, as facing obstacles makes it easy to drop the idea.|
|**High Traffic Volume**|With high daily users or expected future growth, the cost of slow loading becomes significant (e.g., 3% of users leaving is a serious issue if there are millions of users, as each lost user is a potential customer).|

**Examples & Studies Highlighting Loading Speed Criticality:**

- **Amazon:** Combines all these trade-offs. According to Amazon's research, a **100 millisecond delay in loading time leads to a 1% drop in sales**.
- **Google:** Research shows that **53% of mobile users abandon a site that takes longer than 3 seconds to load**.
- **Walmart:** Found that a **1 second improvement in speed increased conversions by 2%**.

#### B. When Loading Time is Less Critical

There are cases where loading speed can be less demanding, meaning resources can be shifted to focus on other areas, such as UI speed:

- **Lower Card Abandonment Risk:** When developing a **service** (rather than a marketplace), users usually stick with it for features that satisfy their needs, and are likely to buy or subscribe even if loading is slightly slower.
- **Low Power Devices (Optimization Skip):** If the target segment uses at least **mid-level devices**, optimizations for low-end devices can be skipped (e.g., video editors like Clipchamp do not anticipate users on old Android phones).
- **B2B Focus:** For Business-to-Business services, employees are expected to have at least mid-range devices (e.g., modern Windows or Mac OS laptops). The app's feature set usually outweighs minor delays.
- **Complex Applications:** If an application is complex, has many features, or handles large data sets, users will intuitively tolerate a longer loading time. In contrast, a simple website (like for a local pet shop) is expected to load quickly, and if it does not, users perceive it as poorly designed.

> **Note:** Even when loading speed is deprioritized, it must **still be good enough**.

#### C. When UI Speed is Critical

UI speed (how fast the application works) becomes critical when users interact with the app actively in a **workflow**:

- **Task-Oriented Applications:** Apps designed for users to complete specific tasks (e.g., Google Sheets, Figma) require smooth, fast interfaces. While users tolerate slow loading for a one-time action, they will not accept it if the application requires thousands of interactions during normal usage.
- **Long Session Time:** The importance of UI smoothness increases with session length; long sessions magnify the negative effects of a slow interface.
- **B2B Applications (Revenue Impact):** While e-commerce revenue depends on loading speed, B2B application revenue depends on **UI speed**. If employees cannot work effectively due to a slow, unresponsive UI, the client company will lose money and likely replace the product.
- **Frequent Interactions:** Apps requiring frequent interactions prioritize UI speed.

#### D. Case Study Comparison: Amazon vs. Figma

|Feature/Metric|Amazon (E-commerce)|Figma (Task-Oriented B2B)|
|:--|:--|:--|
|**Session Length**|Short (open site, add items, purchase).|Long (10 minutes to several hours; user opens app and works for hours).|
|**Interactions**|Short sessions, few interactions.|Long sessions, many interactions (critical to workflow).|
|**Market Type**|Competitive; clone is not difficult.|Unique features; difficult for competitors to overtake/for customers to find replacement.|
|**Design Focus**|Convenience, best possible user experience (both loading time and UI speed); avoids fancy animations.|Users tolerate longer initial loading because they know the eventual quality; UI speed is paramount.|
|**Revenue Correlation**|Directly affected by loading speed.|Less correlated with load times (users pay for subscriptions).|
|**Prioritization**|Loading Time is More Important.|UI Speed is More Important.|

### III. Single Page Applications (SPA) vs. Multi-Page Applications (MPA)

|Type|Characteristics|Trade-offs|
|:--|:--|:--|
|**SPA (Client-Side Rendering, CSR)**|Downloads the entire application (or chunks). Uses client-side routing without backend page requests, resulting in a **smoother UI**.|Downside: Loading time, Time to Interactive (TTI), and SEO suffer without techniques like code splitting.|
|**MPA (Server-Side Rendering, SSR)**|Usually utilizes SSR.|Code splitting is not applicable in the case of SSR.|

> **Note:** Techniques like adding a CDN are useful for both SPA and MPA applications.

### IV. Improving Loading Time: Metrics and Optimizations

The loading process involves the browser sending a request, the server handling it, and sending the page, which the browser then parses and renders. Our goal is to improve the metrics tied to these steps.

#### A. Key Performance Metrics (Google Lighthouse Benchmarks)

The following scale helps evaluate performance, with benchmarks provided by Google Lighthouse:

|Metric|Definition|Good Time Benchmark|
|:--|:--|:--|
|**TTFB (Time to First Byte)**|Time until the browser receives the first byte of data after sending the request and the server handles it.|**Under 200 milliseconds**.|
|**FCP (First Contentful Paint)**|Time when the browser renders the first visible element on the page.|**Less than 1.8 seconds**.|
|**LCP (Largest Contentful Paint)**|Time when the browser renders the largest visible content on the page.|**Under 2.5 seconds**.|
|**TTI (Time to Interactive)**|Time when the page becomes fully interactive (users can click buttons, links, etc.).|**Less than 3.8 seconds**.|

---

#### B. Optimizing Time to First Byte (TTFB)

The main goals for TTFB are processing the request on the server and delivering data to the client as quickly as possible.

**General Steps to Improve TTFB:**

1. Use a Content Delivery Network (CDN) to serve static files.
2. Optimize server-side processing (e.g., database queries).
3. Optimize DNS lookup time with DNS prefetching.
4. Enable compression to reduce response size.
5. Use caches (e.g., Redis) for responses.
6. Use HTTP/2 or higher for multiplexing and lower latency.
7. Reduce HTTP overhead by minimizing headers and cookies.

##### 1. Content Delivery Network (CDN)

Adding a CDN is the first and most important step to significantly improve TTFB and overall loading time.

- **Benefit:** Eliminates the need for requests to travel all the way to the origin server, mitigating the limitation of the speed of light by placing servers closer to the end user. This also reduces load on the origin server.
- **Advanced CDN Features:**
    - **Compression:** Automatically compresses files, reducing size and load time, and reducing load on origin servers.
    - **Image Optimizations:** Processes and serves images in optimized formats (e.g., WebP) and resolutions tailored to the user’s device/network, with fallbacks (PNG/JPEG).
    - **Minification:** Can minify CSS and JavaScript files on the fly.
    - **Optimize Cache Hit Ratios (CHR):** Intelligently caches content across edge servers.
- **Problems with CDN Usage:**
    - **CDN Costs:** Can be expensive, especially for advanced features.
    - **Geographical Limitations:** Lack of sufficient coverage in key user regions impacts performance.
    - **Security Risks:** Potential vulnerabilities like cache poisoning or misconfigurations.
    - **Data Privacy:** Must configure the CDN to avoid caching sensitive data, especially due to GDPR and other laws.

|Caching Strategy|Type of Content|TTL Recommendation|Rationale/Example|
|:--|:--|:--|:--|
|**Cache**|Public Resources|Short TTL for frequently updated assets.|Do not contain user-specific information.|
|**Cache**|Static Content|Long TTL.|Infrequently changing assets (images, CSS, JS).|
|**Sometimes Cache**|Dynamic Content|Short period (e.g., 5 seconds) during heavy traffic.|API responses; reduces load on origin server.|
|**NEVER Cache**|Private Resources|N/A|Contain data intended for a single user (e.g., resources requiring authentication).|

**Cache Hit Ratio (CHR):**

- **Definition:** Measurement of how many content requests a cache fulfills successfully (cache hit) versus how many requests it receives (cache miss, requiring request to origin).
- **Goal:** Aim for the highest possible CHR (95–99% for mostly static sites).
- **Improving CHR:**
    - Handle query parameters effectively (normalize or ignore them if results are the same).
    - Avoid unnecessary cookies, as CDNs typically do not cache responses containing cookies (due to privacy concerns).
    - Use shorter TTL or scheduled cache cleaning for regularly updated content.
    - Apply long TTLs for static assets that rarely change.
- **Choosing a Good CDN (Parameters):** Geographical distribution/PoPs, feature set (security, analytics, compression), pricing, support quality, and complexity of onboarding.

##### 2. Server Side Processing (SSP)

SSP is crucial for dynamically generated pages (SSR/Hydration).

- **Improvements:**
    - **Add Indexes:** Use appropriate indexes for frequently queried columns to speed up lookups, while avoiding over-indexing.
    - **Batch Queries:** Fetch data in bulk instead of executing multiple smaller queries.
    - **Caching:** Use caching servers (e.g., Redis or Memcached) to store frequently accessed data or complex database queries.
    - **Dynamic Scaling:** Scale server resources based on load using platforms like AWS autoscaling or Kubernetes HPA.

##### 3. DNS Lookup Optimization

DNS lookup translates a domain name (e.g., google.com) into an IP address via a DNS server. This lookup increases loading time and occurs every time a resource is requested from a different domain, unless previously cached.

- **Technique:** **DNS Prefetching** allows the browser to resolve domain names in advance.
- **Application:** Apply prefetching carefully for critical resources (scripts, fonts, stylesheets). Avoid prefetching too many domains.

##### 4. Compression

Compression helps reduce loading time by shrinking asset size over the network.

- **Formats:**
    - **Gzip (GZ):** Widely supported, ideal for text-based files (HTML, CSS, JS).
    - **Brotli:** Newer, more effective, offers better compression rates and faster decompression, outperforming Gzip. Use Brotli as primary, Gzip as a fallback.
- **Compression Ways:**
    - **Manual Compression:** Origin server compresses responses before sending them (e.g., during CICD pipeline). Ensure files are compressed before reaching the CDN.
    - **Automatic Compression (on CDN):** CDN uses fast algorithms (e.g., Brotli 5) for quick initial delivery and slower ones (Brotli 11) for subsequent requests. This works for both cashable and non-cashable resources.

##### 5. Caching Responses

Responses can be cached in tools like Redis or Memcached, which use RAM for fast operations. RAM is limited, so it is unsuitable for large volumes of data.

- **Responses to Cache:**
    - **Heavy Database Responses:** Frequently accessed data that remains mostly the same (e.g., product lists, user profiles), especially for queries with many search parameters (filtering, sorting).
    - **Third-Party API Responses:** Data from APIs with rate limits, high latency, or paid access (e.g., weather data, stock prices).

##### 6. Reducing HTTP Overhead

HTTP overhead (unnecessary headers/cookies) increases load time.

- **Steps to Reduce Overhead:**
    - Eliminate unnecessary HTTP headers (e.g., `Server`, `X-Powered-By`).
    - Limit cookies to relevant paths and domains.
    - Avoid sending cookies for static assets.
    - Use `SameSite` and `Secure` attributes.
    - Use HTTP/2 or HTTP/3 to automatically compress headers.

---

#### C. Optimizing First Contentful Paint (FCP)

The goal is to maximize priority by preloading or caching content that is above the fold, and decrease priority (lazy loading) for everything else.

**Steps to Improve FCP:**

- Apply the **PURPLE Pattern** (Preload, Render, Precache, Lazy Load).
- Inline critical CSS.
- Delay loading non-critical resources.
- Use browser caching for static resources.

##### 1. PURPLE Pattern: Preloading

Preloading fetches specific resources in advance. It should only be applied to resources that cannot be automatically detected and loaded by the browser's preload scanner.

- **Preload Scanner:** An internal browser mechanism that identifies assets to load in the background, even when the rendering process is blocked by parsing (CSS, HTML).
- **What to Manually Preload:** Critical resources necessary for the first interaction. This includes lazy loaded images, CSS background images, critical CSS, and fonts defined with the `@font-face` rule.
- **Caution:** Overusing preloading negatively affects performance. Resources like async scripts or images that the preload scanner detects should not be manually preloaded.

##### 2. PURPLE Pattern: Rendering (Above the Fold)

- **Above the Fold (ATF):** The part of the page visible without scrolling. All resources needed to render this section should be loaded as quickly as possible, while everything else below the fold can be loaded later.
- **Quick Initial Rendering (Wider Scope):** Use inline critical CSS and JavaScript for ATF content. For the entire page, use forms of server-side rendering (full SSR, static SSR, or hydration).
    - _Note:_ Inline JS/CSS requires additional development resources. Hydration can negatively impact Time to Interactive (TTI).

##### 3. PURPLE Pattern: Pre-caching (Client-Side Caching)

Client-side caching consumes resources for maintenance, ensuring content isn't outdated, and handling limited client storage.

|Cache Type|Storage and Strategy|Trade-offs|
|:--|:--|:--|
|**Browser Cache**|Automatically managed by the browser for static/frequently updated assets. Use long TTL for static resources.|Reduces bandwidth; enables immediate loading.|
|**JavaScript Memory**|Used by store managers (e.g., Redux) to cache server responses or client-side computations. Session-based.|Simplifies maintenance (cleared on page reload) but limits effectiveness due to short TTL.|
|**Service Worker Cache**|Intercepts requests to serve cached data without contacting the server. Supports offline functionality.|Requires more resources to implement and maintain.|

- **Caching Challenges:**
    - **Stale Content:** Happens when TTL is too long. Solution: **Cache busting** techniques, such as adding version hashes to file names (e.g., `main.abc123.js`), ensuring the browser fetches new files.
    - **Storage Usage:** Browsers limit storage per site. Solution: Set maximum cache sizes and implement cache cleaning projects.

##### 4. PURPLE Pattern: Lazy Loading

Lazy loading is a conditional strategy where application parts are loaded only when specific conditions are met (e.g., when an image is near or visible in the user viewport).

- **Benefits:** Makes the loading of critical resources faster. Reduces load on CDN servers, potentially lowering monthly costs.
- **What Can Be Lazy Loaded:**
    - Images and Videos (most are off-screen initially).
    - Application code (split into chunks/pages using dynamic imports).
    - Third-party libraries (not immediately needed, e.g., text editors).
    - Data fetching (pagination).
- **Trade-offs/Issues:**
    - **Overuse:** Can degrade performance, making the site feel slow, especially for users with high-power devices. **Content above the fold should never be lazy loaded**.
    - **Quick Scrolling Issues:** Fixed thresholds may fail, especially with infinite scrolling. Solution: Advanced strategies like Dynamic Chunk Loading with **Intersection Observer**.
    - **Time Wasting:** Lazy loading misses opportunities to use idle network time. Solution: Combine lazy loading with smart preloading based on user behavior.
    - **SEO Impact:** Can hinder web crawlers from fully scanning the website. Mitigation: Avoid lazy loading SEO critical content (keywords, business information).

---

#### D. Optimizing Largest Contentful Paint (LCP)

The goal is to display meaningful content quickly while rendering the full page in minimal time, prioritizing content less aggressively than for FCP.

**Steps to Enhance LCP:**

1. Image optimization (modern formats, compression).
2. Lazy loading with care (avoid lazy loading ATF images).
3. JavaScript optimizations (reduce bundle size, defer/async non-critical JS).

##### 1. Modern Media Formats (Images & Video)

|Media Type|Format|Characteristics and Use|
|:--|:--|:--|
|**Image**|**AVIF**|Superior compression/quality; ideal for high-resolution and bandwidth-sensitive content; supports animations, overlays, HDR. Better compressed size for photographic images than WebP.|
|**Image**|**WebP**|Processes faster; broader browser support; smaller uncompressed size than other formats; better for schemas, text images, logos.|
|**Video**|**AV1**|Open source, ideal for streaming 4K/8K content; strong compression efficiency; increasingly supported by major browsers/platforms.|
|**Video**|**VP9**|Google's open source alternative to ABC; widely used; moderate compression efficiency; broader browser support than AV1.|
|**Video Container**|**WebM**|Uses VP9 or AV1 for video, and Opus or Vorbis for audio.|

**Choosing the Right Codec (Parameters):**

1. **Compression Efficiency:** Smaller file size leads to faster loading times.
2. **Quality:** Assess if acceptable quality is retained (tools like Squoosh help compare appearance/compression).
3. **Encode and Decode Speed:** Crucial for platforms processing large media volumes (Instagram, Pinterest); factor in costs for CDN-based processing.
4. **Browser Support:** Newer formats require fallbacks (maintaining multiple versions), consuming storage space.

> **Note:** Media optimization tools offered by CDNs can handle format conversion, resolution optimization, and compression, reducing the need for manual transformation.

##### 2. Font Optimization

Fonts block page rendering to avoid **Flash of Unstyled Text (FOUT)**, which creates poor UX and layout shifts.

- **Recommendations:** Use modern font formats like **WOFF2** (reduces latency and file size). Cache fonts (browser cache or local storage).

##### 3. JavaScript Optimizations

- **Tree Shaking:** Removes unused code during the build process. Relies on ES Modules, which are statically analyzable. If a module has side effects (modifies a global variable), it may be included unless manually marked as side effect-free.
    - _Causes of Unused Code:_ Modular code structure (importing entire library for a few methods), default imports, and third-party libraries.
- **Code Splitting:** Divides the JavaScript bundle into smaller chunks (by pages, components, or dependency). This is especially relevant for CSR SPAs where loading a single large JS file is inefficient.
- **Defer or Async Loading:** Delays loading of non-critical scripts.

---

#### E. Optimizing Time to Interactive (TTI)

For a page to be fully interactive, JavaScript must be loaded, parsed, and executed.

**Goals and Steps for TTI:**

- Render the page on the server if possible (SSR, generation, static rendering).
- Avoid executing heavy JavaScript on the main thread during initial render.
- Break down long-running JavaScript tasks.
- Prioritize loading essential scripts first.
- Minimize third-party scripts that may block interactivity.
- Preload resources based on user behavior.

##### 1. Delegating Work and Breaking Down Tasks

Long-running JavaScript tasks block the main thread, causing the UI to freeze and delaying interactivity.

- **Solutions:**
    - **Web Workers:** Operate in a separate thread for heavy computations, keeping the main thread free. Communication is via `message` events, and should only be used for resource-intensive tasks. (Web Workers are different from Service Workers).
    - **Server-Side Computations:** Offload expensive tasks to the backend, which is typically more powerful, although this increases infrastructure and debugging costs. Suitable tasks include data processing (filtering, searching), aggregation, machine learning, and AI tasks.
    - **Splitting Tasks:** If remaining on the main thread, break tasks into smaller chunks and run each chunk in a new event loop cycle using `setTimeout` or `requestAnimationFrame`.

##### 2. Predictive Prefetching

This technique anticipates user actions and preloads resources before they are needed.

- **Mechanism:** Tools like Guess.js analyze user behavior (navigation patterns, clicks) to predict which pages or assets the user is likely to access next, then preloads those resources (images, scripts, HTML).

### V. Improving UI Speed

When a user interacts with the page (animation, server request, client-side computation), they expect animations to run smoothly, the server to respond quickly, and computations not to block the page.

#### A. GPU Acceleration and Animations

The key to smooth animation is GPU acceleration.

- **Preference:** Prefer **CSS animations over JavaScript animations**, as JS animations shift the workflow to the CPU, which can block the main thread.
- **GPU-Friendly Properties:** Use `transform` and `opacity`, as they can be uploaded to the GPU. Avoid animating properties that trigger layout or calculations (width, height, margin).
- **Benefits of GPU Animations:** Offloads work from the CPU, efficiency in repetitive calculations, and avoidance of relayouts and repaints.

##### The Rendering Pipeline and Composition Layer

The rendering life cycle involves a sequence of steps:

1. **JavaScript:** Dynamic changes to styles.
2. **Style:** Browser calculates final computed styles.
3. **Layout:** Browser computes geometry (size and position) of each element. This step is computationally intensive and should be avoided.
4. **Paint:** Browser fills in visual styles (background color, shadows). Less expensive than layout, but still handled by the CPU.
5. **Composite Layers:** Browser splits painted elements into layers that are merged into a final image. This is a lightweight calculation.

- **Composition Layer:** Layers can be changed independently without triggering the earlier, heavy steps. Only `transform` and `opacity` can be moved directly to the composition layer.
- **Manual Layer Promotion:** Use the CSS property `will-change` to signal that an element's properties (like transform or opacity) are expected to change, prompting the browser to create a composition layer in advance. This helps address animation delays, dynamic property changes, and overlapping elements.
- **Caution with `will-change`:** It is not a silver bullet. Each layer consumes GPU memory and processing power; overuse slows down performance, especially on low-end devices.

**Animation Checklist:**

- Fewer layers and animation effects lead to better performance.
- Try to perform animations on the composition layer.
- If not possible, use animations that only trigger repaints.
- **Avoid animations that trigger relayouts**.
- Use tools like **CSS Triggers**.
- If supporting a wide range of devices, allow users to **control animations** via a settings toggle.

#### B. Unblocking the Main Thread

The main thread must remain free for handling user interactions.

- **Steps:** Avoid heavy computation, divide work into smaller chunks, and minimize/optimize DOM manipulations.
- **Low Priority Tasks:** Use **`requestIdleCallback`** to execute tasks when the browser is idle (after rendering, layout, and painting). Even with `requestIdleCallback`, tasks must be split into small enough chunks to allow the page to render at **60 frames per second** (one frame every 60 milliseconds).

#### C. Minimizing Document Object Model (DOM) Manipulations

Reducing the number of page re-renders is crucial because they are computationally expensive.

- **Strategies:**
    
    - Group multiple DOM changes into a single update to minimize layout recalculations.
    - Avoid layout thrashing.
- **Layout Thrashing:** Occurs when JavaScript reads layout properties (e.g., `offsetHeight`) and immediately modifies the DOM (repeated reading and writing in a loop). This forces the browser to recalculate the layout multiple times in quick succession, severely impacting performance.
    
- **Avoiding Thrashing:** Separate style reads and style writes (e.g., read styles outside of the loop, then write them inside the loop).
    

### VI. Summary

Some optimizations, even those focused on loading time, benefit both loading time and UI performance (e.g., image optimizations). The goal is to apply these techniques, patterns, and tools to achieve a fast, performant web application.

--------------------------------------------------------------------------