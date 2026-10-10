Video_link : https://youtu.be/zAEy0jchvH8?si=lkp6bYHG302ThU8P

--------------------------------------------------------------------------

This response draws exhaustively on the provided transcript excerpts from the YouTube video "Frontend System Design Yatra Season 2 | Performance & Optimization Techniques 🔥".

---

## Frontend System Design Yatra Season 2 Overview

This video series, hosted on the JsCafe channel, focuses on frontend system design and performance optimization techniques.

### Curriculum and Modules

The series covers a total of **11 modules**:

1. Code Splitting and Bundle Splitting.
2. How to compress JavaScript bundles (Compression).
3. Import on Interaction.
4. Import on Visibility.
5. Prefetch.
6. Preload.
7. Tree Shaking.
8. Virtual List (Virtualization).
9. Core Web Vitals.
10. Interaction to Next Paint (INP).
11. Cumulative Layout Shift (CLS).
12. Largest Contentful Paint (LCP).
    - _Note: LCP, INP, and CLS are the three pillars of Core Web Vitals_.

### Prerequisites and Audience

- **Prerequisites:** Only basic front-end development knowledge is required.
- **Framework Agnostic Content:** Most topics are framework agnostic.
- **Implementation Focus:** The performance approaches are often routed through **Webpack**, chosen because it can be configured in both React and Angular, making it easy for anyone to adapt.
- **Examples Used:** React examples are used in the tutorial, but similar use cases or packages exist for Angular or Vue applications.
- **Core Concept Focus:** Viewers are advised to understand the core concepts and not just the working of things around certain frameworks.
- **Audience:** The video is for everyone, including college students (in any year, as long as they know front-end development), entry-level professionals, and seniors. Seniors might already know some concepts but may learn something new.

### Resources and Support

- **Hands-on Experience:** Each content and topic has an accompanying hands-on experience.
- **Repository:** The speaker created a new folder called "performance" inside the previous season's repo (where networking was discussed). This folder contains hands-on experiences for all 11 modules.
- **Content Curation:** The whole series is curated using **free online articles**.
- **Referenced Articles:** The curriculum source includes links to articles, such as those found on **patterns.dev**, which provides good examples of bundle splitting and compression.
- **Season 1 Success:** By the time the video was recorded, Season 1 had crossed **more than 16k views** and **more than 1.2k likes**.

### Sponsor: Zik Cloud

Zik Cloud is a Global Communication service provider offering developer-friendly and powerful SDKs and APIs to build communication features like video calls, chat, video conferencing, and live streams.

- **UI Kits:** They provide a bunch of UI kits (build components) that help developers build faster and ship more, without building from the ground up.
    - Zik Cloud has **more than 20 UI kits** and **more than 50 components**.
    - These components are open source, cross-platform, easy communication, and ready to use.
- **Voice Call SDK:** Used to build apps in low code, offering crystal clear voice conversation.
    - Features include direct call, group call, call invitation, live video streaming, spatial audio, screen sharing, fun and engaging audio effects, AI noise deduction, intelligent voice processing, recording, call quality monitoring, and multi-device support.
- **Integration:** Interactive real-life audio can be integrated into aspects of life such as social, in-game voice, audio conferencing, and IoT devices.
- **Free Minutes:** Users can register at Zik Cloud to receive **10,000 free minutes**.

---

## Detailed Module Breakdown

### Module 1: Code Splitting (Bundle Splitting)

**Definition and Goal:** Code splitting is a technique where code is separated into several packages or components that are loaded on demand or in parallel. The goal is to prevent these packages from being loaded into the main JS file or network until they are needed, thus saving user bandwidth and loading only what is required.

**Implementation Steps (React/Vite Example):**

1. Initial setup used **Vite** and a React JavaScript project.
2. Several components (Navbar, About, Contact, FAQ, Login, Profile) were created inside an `SRC/Pages` folder.
3. **Barrel Exports** (exporting all components from a single index file) were initially used.
4. **React Router Dom** was installed to manage routes.
5. **Initial Problem:** Even when navigating to a single route (e.g., `/login`), the network tab showed that all components (Profile, Contact, About, FAQ) were loaded, costing extra bandwidth (e.g., 3 KB per unnecessary file).
6. **Failed Attempt (Segregation):** Routes were segregated into private routes (Profile, About, Contact, FAQ) and public routes (Login), based on an `isAuthenticated` boolean flag, but this separation alone did not fix the loading issue; all components were still loaded. The speaker notes this segregation was not code splitting.
7. **Successful Solution (Lazy Loading):** The solution involves leveraging **`lazy` and `suspense`** from React.
    - Components were lazy loaded instead of direct importing.
    - `lazy` imports function like a promise; therefore, the components must be wrapped in a **`Suspense`** boundary with a `fallback` (e.g., "dot dot").
    - **Crucial Fix:** The named exports were changed to **`export default`** for each component, as `lazy` expects a default export.
8. **Result:** After implementation, only `login`, `navbar`, and `app.jsx` were loaded initially. When `isAuthenticated` was set to `true`, only the necessary route (e.g., Profile) was loaded on demand.

### Module 2: Import on Interaction

**Context:** This is an extended version/variation of code splitting applied at the **component level**, rather than the route level.

**Definition:** Lazy loading non-critical resources only when a user interacts with the UI, requiring that resource.

**Use Case Example (Chat Application):** In a chat application containing a message UI and an Emoji Picker component, the heavy Emoji package component should be lazy loaded and imported only when the user clicks the emoji icon.

**Implementation Steps:**

1. A Vite/React application was set up.
2. `Message.jsx` and `Emoji.jsx` components were created.
3. **Initial Problem:** A simple conditional rendering based on a toggle button (`showEmoji`) prevented the Emoji component from rendering (saving Heap memory) but _did not_ prevent the `Emoji.jsx` file from being loaded in the network on the initial refresh, costing 3.3 KB of bandwidth.
4. **Successful Solution:** The component was fixed by using **`lazy` and `suspense`**.
    - The `Emoji` component was lazily imported.
    - The lazy-loaded component was wrapped in a `Suspense` boundary with a `fallback` (e.g., "loading").
    - The suspense boundary is necessary because while the component is loading (takes milliseconds), React needs something to show.
5. **Result:** This technique saved both Heap memory (by not rendering) and Network bandwidth (by not loading the file initially). The Emoji file request was only made upon clicking the "Show Emoji" button.

### Module 3: Import on Visibility

**Context:** This is another variation of code splitting and import on interaction.

**Difference from Interaction:** On interaction, the import happens upon an action (like clicking); on visibility, the import happens when the component is detected in the view.

**Mechanism:** It uses the **Intersection Observer API** to detect if a component is present in the viewport (view).

**Use Cases and Tradeoffs:**

- This is useful for optimizing components that are in the second half of the fold (below the initial screen view).
- **Scenario Based:** The choice depends on the application use case.
    - **Import on View Makes Sense:** Snapchat or WhatsApp, where users tend to use stickers/emojis/GIFs often.
    - **Import on Click Makes Sense:** Instagram, where users are not interacting with emojis as often.

**Implementation Steps (Emoji Example):**

1. A Vite/React application was set up.
2. The package **`react-intersection-observer`** was installed.
3. The `useInView` hook (from the package) is used, passing a `ref` to the component container.
4. The `Emoji` component was lazy loaded and suspended, similar to the previous module.
5. A series of break tags were added to move the Emoji component below the first fold so that the `isInView` value was initially `false`.
6. The Emoji component was rendered only if the `isInView` variable was `true`.
7. **Result:** When the user scrolled down and the element came into the view, `isInView` became `true`, the network call for the Emoji file was made, and the component became visible.

### Module 4: Compression (Gzip and Brotli)

**Goal:** Compression applies algorithms on top of the production build code (which is already minified) to compress the bundle further by **15% to 20%**.

**Algorithms:**

- **Gzip and Brotli (Br):** These are the most common compression methods and are widely supported by modern browsers.
- **Brotli Advantage:** Brotli offers **better compression** than gzip.
- **Industry Results:** Uber saw a 15% to 20% reduction, and Wix saw a 21% to 25% reduction in file sizes after switching to Brotli compression instead of gzip.
- **Framework Behavior:** Next.js performs gzip compression by default. For React/Webpack applications, compression must be explicitly set up in the webpack config.

**Industry Examples (Response Headers Check):**

- **Hotstar:** Uses **Brotli** (`content-encoding: br`) compression.
- **Coursera:** Also uses **Brotli** compression.
- **Browser Support:** Modern browsers support multiple encodings, including `gzip`, `deflate`, `br` (Brotli), and `zd` (Zang).

**Hands-on Implementation (Gzip via Webpack/Nginx):**

1. **Setup:** A React application was set up using **Webpack** and **Babel**. Webpack is the bundler responsible for generating production builds.
2. **Webpack Configuration:**
    - Entry point: `src/index.js`.
    - Output: `dist` folder.
    - `html-webpack-plugin`: Used for injecting the script tag into `index.html`.
    - Babel: Used as a transpiler to convert advanced JavaScript into plain JavaScript that the browser understands, configured with `preset-env` and `preset-react` (with `runtime: automatic`).
3. **Initial Production Build:** Running `npm run build` generated a minified `main.js` file. The size was measured at **140 KB** (Production build size).
4. **Enabling Gzip Build:** The **`compression-webpack-plugin`** package was installed.
    - The plugin was added to the `plugins` array in `webpack.config.js`, specifying the gzip algorithm.
5. **Gzipped Build Result:** The build generated a gzipped file. The size came down to **45 KB**, representing roughly **100 KB of savings**.
6. **Serving Gzip Files (Nginx Setup on Amazon EC2):**
    - The goal was to serve the gzipped file instead of the 140 KB file.
    - An Ubuntu machine was launched on Amazon EC2.
    - **Nginx** was installed using `sudo apt install nginx`.
    - Port 80 (HTTP) was enabled in the EC2 security group rules (Inbound Rules).
    - The production build code was cloned into the Nginx serving directory (`/var/www/html/`).
    - **Enabling Gzip in Nginx:** The Nginx configuration file was modified (`sudo nano /etc/nginx/nginx.conf`).
        - The **gzip settings** were found and uncommented.
        - The Nginx server was restarted (`sudo systemctl restart nginx`).
7. **Final Result:** The application began serving the **45.5 KB gzipped version** of the file, verified by checking the `content-encoding: gzip` header.

### Module 5: Prefetch

**Definition:** Prefetch is an early fetch technique where an asset or resource is fetched and **cached locally** by the browser because it may be requested _sometime soon, but not immediately_.

**Timing:** A prefetched resource is fetched when the browser is **ideal** and has calculated it has enough bandwidth.

**Contrast with Preload:** Prefetch happens during ideal time.

**Use Case Example:** Instagram, where users are not likely to open the Emoji tab very often.

**Implementation (React/Webpack):**

1. The setup was based on the previous Webpack configuration.
2. The implementation started similar to "Import on Interaction," where the Emoji asset was loaded only upon button click. Initially, the file (e.g., `988.js`) was only fetched _after_ the click (on interaction).
3. **Magic Comment:** To enable prefetching, a **magic comment** was added before the dynamic import statement:
    
    ```
    /* webpackPrefetch: true */
    ```
    
4. **Optional: Chunk Naming:** A chunk name can also be added (e.g., `webpackChunkName: "Emoji"`) to rename the output file (e.g., `Emoji.js`).
5. **Result:** With the magic comment, the file was fetched when the browser was ideal and cached. Upon clicking the button, the file was loaded immediately from the **prefetch cache** instead of hitting the server (though a network call indication might still occur, the data is served from the cache). The header indicates the purpose as `prefetch`.

### Module 6: Preload

**Definition:** Preload is an early fetch technique used for **critical resources** required for the current navigation, such as fonts, images, or assets instantly visible on the landing page.

**Timing:** Preload occurs **in parallel** with the main bundle fetch; the browser does not wait for an ideal period.

**Contrast with Prefetch:** Preload fetches critical resources instantly/in parallel. Prefetch fetches non-critical resources during ideal time. Anything used within 3 seconds of the initial load should be preloaded.

**Use Case Example:** WhatsApp or Snapchat, where users often use stickers as soon as they open chats.

**Implementation Challenge:**

- The expected **magic comment** (`webpackPreload: true`) was implemented but was observed **not to be working** in the tutorial.

**Successful Implementation (Plugin Support):**

1. Since the magic comment failed, support was taken from a Webpack plugin.
2. The plugin **`preload-webpack-plugin`** is archived, so the dedicated alternative plugin (which is highly recommended) was installed.
3. The plugin was imported and utilized in the Webpack configuration.
4. **Result:** The `Emoji.js` asset was fetched **in parallel** to the main bundle file (visible in the waterfall chart). Clicking the "Show Emoji" button resulted in _no network call_ because the asset was already loaded.
    - _Note: This specific plugin may preload any JavaScript asset it finds, even without the magic comment_.

**Caution:** Developers must be **careful** when using preload because prefetching/preloading additional bundles along with the main bundle can hurt performance by splitting network resources.

### Module 7: Tree Shaking

**Definition:** Tree shaking is the process of **eliminating dead code** before including it in the final bundle.

**Analogy:** Shaking a tree causes all the dead leaves to fall out; similarly, shaking the application removes all the dead code.

**Use Case Example:** If a `math.js` file contains `sum` and `multiply` functions, but only `multiply` is used in the app, tree shaking ensures that only the `multiply` function is present in the final bundle, excluding `sum`.

**Implementation (Vanilla JS/Webpack):**

1. A non-React application using Webpack was set up.
2. A `math.js` file was created with exported `sum(a, b)` and `multiply(a, b)` functions.
3. The main index file only imported and used the `multiply` function.
4. **Initial Problem:** When checking the production build (`main.js`) without tree shaking enabled, both the `multiply` function and the `sum` function definitions were present.
5. **Successful Solution (Webpack Optimization):** The following optimization flag was added to the Webpack config:
    
    ```
    optimization: {
        useExports: true
    }
    ```
    
6. **Result:** When the build was regenerated, the unused `sum` function definition was now marked internally by Webpack in the build file as **`unused Harmony export sum function`**. This indicates that the dead code has been identified and will be removed/ignored during the bundling process.

### Module 8: Virtual List (Virtualization)

**Definition:** Virtualization is an optimization technique where you only **render what is visible** in the current viewport.

**Motivation:** Rendering thousands of rows (e.g., 500 or 1,000) on a single web page is very costly for the browser's rendering performance, leading to sluggishness.

**Goal:** Implementation of a virtual list can help achieve performance near **60 FPS**.

**Concept/Mechanism:**

- If the screen size can only display 50 items (the viewport), the system might render a slightly larger buffer, perhaps 60 items (5 items above and 5 items below the viewport).
- As the user scrolls, the items are dynamically updated, but the total number of children/elements rendered remains constant.

**Hotstar Example (Living Room Platform):**

- Hotstar implements virtualization on their living room platform (Android TV, iOS TV, Web OS devices).
- They use a **virtual tray wrapper**.
- Instead of keeping a heavy DOM element for every movie tray, they replace the tray with a **virtual tray** which is very **less in size**, improving rendering performance.
- As the user scrolls down, the number of virtual trays increases dynamically (e.g., from one to two, then three, then four).

**Hands-on Implementation:**

1. A React/Vite application was set up.
2. The package **`react-window`** was installed.
3. A `List` component from `react-window` was used, configured with:
    - Overall height (e.g., 150).
    - Item count (e.g., 1,000).
    - Item size (e.g., 35 pixels height per item).
4. **Result:** Although 1,000 elements were specified, only 7 items (in this example) were rendered depending on the height and size settings.

### Module 9: Core Web Vitals (LCP)

**Web Vitals Initiative:** An initiative by Google to provide guidance for quality signals essential for delivering a great user experience on the web.

**Core Web Vitals (CWV):** A subset of Web Vitals that apply to all web pages, should be measured by all site owners, and are surfaced across all Google tools. These three metrics are crucial for performance.

**The Three Pillars of CWV:**

1. Largest Contentful Paint (LCP) (Loading performance).
2. Interaction to Next Paint (INP).
3. Cumulative Layout Shift (CLS).

#### Largest Contentful Paint (LCP)

**Definition:** LCP measures the loading time of the single largest content element (text, image, or video element) visible in the first fold of the web page.

**Good Performance Standard:**

- **Good:** LCP loaded within **2.5 seconds**.
- **Needs Improvement:** 4 seconds.
- **Poor:** More than 4 seconds.

**Measurement Tool:** The **Web Vitals Chrome extension** is used to measure LCP, CLS, and INP. It is advisable to enable console logging in the settings.

**Hands-on Implementation (Improving LCP):**

1. A simple HTML file was created.
2. The application was tested with network throttling set to **Slow 4G**.
3. **Initial Problem:** Using a large image (e.g., Cat 1000) under slow 4G conditions resulted in a very poor LCP measurement, such as **6.5 seconds**.
4. **LCP Element Identification:** The Chrome Performance profiler was used to identify the LCP element (in this case, the cat image).
5. **Successful Solution:** LCP was improved by rendering a smaller version of the image that was required, using **`srcset`**.
    - For example, using Cat 500 (255 KB size) instead of Cat 1000.
    - The LCP time improved just by rendering the smaller version.
    - Preload, prefetch, and preconnect techniques can also be used to improve asset loading.

### Module 10: Interaction to Next Paint (INP)

**Definition:** INP assesses a page's performance when a user interacts with it (e.g., clicking a button) and measures the time it takes for the **next paint** to happen.

**Good Performance Standard:**

- **Good:** **200 milliseconds or less**.
- **Needs Improvement:** 500 milliseconds or less.
- **Poor:** Above 500 milliseconds.

**Cause of Poor INP:** Heavy tasks running on the **main thread** can cause the application to choke and delay the next paint.

**Hands-on Implementation (Simulating Poor INP):**

1. A React application was set up.
2. A button ("Open Modal") toggled a component using state (`showModal`).
3. **Baseline Test:** On a high-performance gaming device, the INP was initially reported as **8 milliseconds** (super fast).
4. **Throttling Test:** By enabling CPU throttling (20x slowdown) and setting Hardware concurrency to 1 in the Performance panel, the INP spiked to **64 milliseconds** upon interaction.
5. **Main Thread Blocking Simulation:** A blocking heavy operation (a `for` loop iterating 10,000,000 times) was placed before the `toggleModal` function.
6. **Result:** Due to the main thread blocking operation, the INP spiked significantly to **712 milliseconds**, landing it in the **very poor** category.

**Use Cases for Improvement:**

- **Scrolling/Re-rendering:** If components re-render heavily upon scrolling (visible using React Profiler's highlight updates), **memoization** can be used to fix individual item re-rendering.
- **Opening Menus/Keyboards:** If opening a menu is sluggish because the system keyboard is opening in parallel, the developer can move the logic for rendering the list under **`startTransition`**.
    - `startTransition` allows state updates without blocking the main UI. Using this, one developer improved INP to 168 ms.

### Module 11: Cumulative Layout Shift (CLS)

**Definition:** CLS is a metric that measures sudden shifts in the layout experienced by the user after the page has loaded.

**CLS Unit:** CLS is a **ratio**, not a definite unit.

**Good Performance Standard:**

- **Good:** **0.1 or less**.
- **Needs Improvement:** 0.25.
- **Poor:** Anything above 0.25.

**Cause of Bad CLS:** Dynamically loaded elements, such as **ads or widgets**, that shift existing content (e.g., an ad suddenly appearing at the top and pushing content down).

**Hands-on Implementation (Breaking and Fixing CLS):**

1. A React application was set up.
2. **Simulation (Bad CLS):** A `useEffect` hook introduced an ad div (with content and a height of 300 pixels) after a short `setTimeout` (100 milliseconds).
3. **Result:** The sudden appearance of the ad caused the CLS to break, measuring **0.11** (requiring improvement, as it is above the 0.1 good threshold).
4. **Successful Solution (Placeholder):** The fix involved creating a **placeholder div** (or skeleton/shimmer loader) that already took up the anticipated amount of space (e.g., 500 pixels height) before the ad image was injected.
5. **Result:** By reserving the space, the layout did not shift when the content loaded, and the CLS returned to **zero**.

--------------------------------------------------------------------------