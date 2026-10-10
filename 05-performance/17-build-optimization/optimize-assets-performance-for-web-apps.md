Video_Link : https://youtu.be/9JDlZxR8gVw?si=dAtEtotN-aiG3IrY

--------------------------------------------------------------------------
## Asset Performance Optimization Overview

The focus of this discussion is on **performance optimization**, specifically **asset optimization**. Assets covered include images, videos, fonts, CSS, and JavaScript. The techniques discussed are intended to help manage these assets and boost the overall web application or web performance.

The source material also recommends reviewing the previous episode on network optimization for additional optimization techniques related to the network layer.

### 1. Image Optimization Techniques

Image optimization is described as one of the most underrated aspects of asset management, often causing heavy load times. Techniques covered include compression, progressive enhancement, responsive and adaptive loading, and visual fallbacks.

#### A. Image Compression

Compression is a very small but powerful technique.

**Benefits of Compression:**

- Reduces website load time.
- Saves bandwidth.
- Reduces server cost, resources, and money required.
- Improves user experience.
- Improves SEO ranking.

**Compression Tools and Rates:**

- You can compress images yourself without depending on the designer.
- Available online compressors include JPG optimizer, Kregen, Tiny PNG, Optimus Zilla, ZPEC IO, and compressor.io.
- Compression figures show significant reduction: JPG compression ranges from 13% to approximately **67–68%** (e.g., 66% on an 80kb image). PNG compression starts around 17% and can go up to **80% plus**.
- Choose between **lossy** (more powerful compression rate) and **lossless** compression depending on the use case and required perfection.

#### B. Progressive Enhancement for Images

This involves leveraging different image formats and asking the browser to check support.

- Use the most optimized format first, such as **WebP**.
- Have a fallback, such as JPG or JPEG, if WebP decoding/uncompression is not supported by the browser.
- This is achieved using the **`<picture>` element** with multiple sources and an `<img>` element as a fallback.

#### C. Device Pixel Ratio (DPR)

DPR is a small but powerful technique that ensures a better user experience.

- **Concept:** One CSS pixel is not necessarily equivalent to one pixel of the device; this changes based on the device resolution (e.g., 4K TV).
- **Calculation:** If one CSS pixel equals two device pixels (grids), the DPR is 2; if it equals three device pixels, the DPR is 3.
- **Impact:** If a 500-pixel image is loaded on a device with DPR 2, the image will appear pixelated because 1000 device pixels were required, and the 500-pixel image was stretched.
- **Implementation:** DPR is exposed via the window object (`window.devicePixelRatio`). This value should be considered when requesting image sizes to ensure precise, quality images are shown.

#### D. Responsive and Adaptive Image Loading

This automates the loading of different image sizes depending on the device characteristics.

- **`IMG SRCSET`:** Very powerful for automatically loading different images based on DPR (e.g., loading different parameters for 1x, 2x, or 3x).
- **`sizes` Attribute:** Used when you want to load different images depending on the container size where the image will be rendered. You define multiple widths (e.g., 300, 600, 1200, 2000) and criteria (e.g., minimum 70% of viewport width).
- **Breakpoints:** You can provide multiple sizes related to responsive breakpoints (similar to CSS media queries) to consider the container size, allowing the appropriate image dimension to be loaded.
- **`<picture>` Element:** Allows definition of multiple `<source>` elements, incorporating logic for media (breakpoints) and DPR, with an `<img>` fallback, enabling complex and creative loading scenarios.

#### E. Adaptive Media Loading (Advanced)

This involves the application adopting its behavior based on the scenario or environment it is loading in.

**Adaptation Criteria:**

1. **Network Speed:** Adapt based on slow, medium, or fast network conditions.
2. **Device Memory:** Adapt based on the device's memory size (e.g., if memory size is low).
3. **Hardware Concurrency:** Adapt based on the number of cores available in the machine.

**Tools for Decision Making (via Navigator object):**

- **`navigator.connection.saveData`:** Check if power save mode or data saving is enabled to avoid heavy lifting or background jobs.
- **`device memory`:** Check memory size to make decisions on heavy computation or leveraging storage technologies like IndexedDB.
- **`hardware concurrency`:** Get the number of cores to know how many workers/threads can be spawned for multi-threading.

**Adaptive Use Cases:**

- Loading smaller images on a slow network or larger images on a fast network (possibly including a video preview play button).
- Applications like Tinder load fewer images on a slow network (3G).
- Only performing prefetching inside the viewport link if the connection is good (e.g., 4G) and not in data saving mode.

#### F. Visual Fallback Techniques

These techniques ensure a visually appealing loading state.

- **Small Images with Blur Effect:** Instead of showing a grayish background, load a very small (e.g., one or two pixel) placeholder image, stretch it to cover the space, and apply a blur effect.
    - This technique (used by Medium) uses `background-cover` and CSS `filter: blur(5px)` for a smooth, non-pixelated appearance, matching the color tone of the final image.
    - WebP can also be used with this effect.
- **Solid Primary Color Technique:** A fallback where a primary color (often generated at upload time and stored as metadata) is shown as a background until the actual image loads (used by Google Images).

#### G. CSS Sprites

This technique, used by major companies like Facebook, Google, Netflix, and YouTube, involves clubbing multiple small images (icons, thumbnails) into a **single sprite image** to save size and reduce the number of requests.

- Example: Loading 20–30 individual 4KB images (80 KB total) requires multiple requests; using a sprite reduces the size to approximately **10 KB or less** in a single request.
- Implementation: CSS logic is used to reposition the single sprite image so that only the desired icon is visible within a specific window or container.

#### H. Lazy Loading Images

Ensures only images visible in the current viewport are loaded immediately.

- **Browser Attribute:** Use the `loading` attribute on the `<img>` element with values:
    - **`lazy`**: Browser handles the decision of when to load, without hampering rendering performance.
    - **`eager`**: Loads on high priority.
- **Custom Logic/Intersection Observer:** For custom control, one can write custom logic or use an Intersection Observer.
    - The common technique involves initially using a `data-src` attribute for the image URL instead of `src`.
    - When the image needs to be loaded, custom logic or the observer converts the `data-src` value into the actual `src` attribute, triggering the browser load.

### 2. Video Optimization Techniques

Video optimization is crucial for platforms like YouTube and Netflix.

#### A. Progressive Enhancement for Videos

Load the most performant and optimized file format first (e.g., **WebM**). If the browser doesn't support it, fall back to a heavier format like MP4.

#### B. Replace GIFs with Videos

For animations or previews, use HTML5 video players instead of GIFs.

- HTML5 videos (WebM/MP4) offer better quality and control compared to GIFs.
- HTML5 videos can mimic GIFs using the `autoplay` property.
- Browser dev tools often suggest replacing GIFs with video formats because GIFs take a significant amount of loading time.

#### C. Responsive Poster Images

The static image shown before a video starts playing is called a poster image.

- Use the **`poster` attribute** in the `<video>` tag to specify the image.
- Make the poster responsive using CSS properties like `background-position: center` and `background-size: cover`.
- Different poster images can be defined for different resolutions/breakpoints (e.g., using `75rem` breakpoint).

#### D. Streaming

Instead of downloading the entire file at once, send the video in chunks or packets from the server to the client, where they are clubbed together for playback. This is covered in detail in the Netflix system design discussion.

#### E. Video Audio Optimization

- **Remove Audio:** If the video is playing in the background and audio is not required, remove the audio to reduce cost and optimize performance.
- **Separate Audio/Video:** To support multiple languages, keep the video and audio separate. Download localized audio streams and sync them locally on the client using a local library.

#### F. Platform-Based Video Dimension

This technique involves loading different videos based on the resolution or platform they are going to be loaded in.

- Similar to CSS breakpoints, use media attributes in the video sources to define different media types (e.g., `all`, `portrait`) and breakpoints.
- The browser is responsible for deciding which video source to load based on these definitions.

#### G. Preload Video/Metadata

Allows preloading the video or just its metadata for critical areas like banners or previews (e.g., Netflix homepage banners).

- Use the `preload` attribute with options:
    - **`none`**: Do not preload.
    - **`auto`**: Let the browser decide.
    - **`metadata`**: Only preload the metadata information, not the entire video.

### 3. Font Optimization Techniques

Font optimization is important, especially for websites (like blogging sites) that rely heavily on custom fonts, as poor optimization can compromise initial load, text visibility, and styling.

#### A. Font Display Descriptor

This descriptor defines how the browser should behave when custom fonts are loading.

- **FOUT (Flash of Unstyled Text):** The browser renders the text using an unstyled (system) font first, and then swaps to the actual custom font once it's loaded.
- **FOIT (Flash of Invisible Text):** The browser shows nothing (invisible text) until the custom font is fully loaded, compromising content visibility.
- **`font-display: swap`:** This behavior swaps the font once the actual fonts are ready, rendering the unstyled fonts on the first cut (achieving the FOUT behavior).

#### B. Progressive Enhancement for Fonts

Always load the most optimized font format first (e.g., **WOFF2**). If not supported, fall back to other formats (e.g., WOFF). It is important to have multiple font formats for backup across different browsers.

#### C. Font Loading Trigger Trick

A font defined using `@font-face` will not load unless that font family is used somewhere in the stylings or HTML.

- **Trick:** Programmatically (using JavaScript) create a `div` or add a style to the body that uses the dependent font family. This action triggers the loading of the required font.
- Some browsers require the container to have content, so dummy content might be needed.

#### D. Data URI

Allows embedding font data (base64 encoded) directly within the URL/CSS instead of relying on an external dependency.

- Use this technique for **small** font families to avoid making another request.
- **Caution:** Using base64 encoding increases the file size, so it must be used intelligently.

#### E. Preload

Use the preload technique for critical fonts that need to be available when the page loads.

- Implementation: Use `<link rel="preload" href="font URL" as="font" type="font type" crossorigin>`.

#### F. Subset of Web Fonts

Avoid clubbing all font weights, versions (like Roman, Italic), and formats into a single large font file.

- Create subsets (lighter versions) of the required fonts using multiple tools.
- These subsets can also be used as data URIs or hosted elsewhere.

#### G. Asynchronous Loading of Dependent CSS

If a style sheet containing font dependencies is not critical for the first fold, delay its loading.

- Initial method: Define the CSS link with a non-critical media attribute (e.g., `media="print"`).
- Once loaded, use a small technique (often JS) to convert the media attribute to `media="all"`, which triggers parsing and loading of the dependent font asynchronously.

#### H. Combined Loading Strategy

For easy and effective font service, combine multiple approaches:

1. **Pre-connect** to the domain where the font is hosted.
2. **Preload** the critical font.
3. **Swap the media** to `all` once the style sheets are ready.
4. Have a **backup** link that loads the font directly if JavaScript is unavailable.

#### I. Font Face Observer (Programmatic Control)

A package that provides programmatic control over when and what font to load.

- It utilizes the loading trigger trick.
- Method: Initialize content with a fallback font family (e.g., sans-serif). Use the `font face observer` package to wait for the custom font to load. Once loaded, a callback is triggered, allowing you to programmatically add a class (e.g., `fonts-loaded`) to the document body, which then applies the desired font family via CSS.

### 4. CSS Optimization Techniques

CSS is render blocking and must be optimized to prevent compromising web vitals and page performance.

#### A. Critical CSS Rendering

Ensure that CSS critical for the application's first fold is loaded immediately.

- **Concept:** Break the CSS into smaller files. Load the critical styles (e.g., header, sidebar, top sliders) first.
- **Impact:** If CSS is heavy, the entire rendering blocks. The browser must wait for the CSS to load, parse the CSS, create the CSSOM, and only then proceed to paint the final layout. Loading critical CSS ensures that at least some pieces of the web app are rendered quickly, providing a better user experience.

#### B. Lazy Loading CSS using Media Attribute

Write CSS to support different devices, resolutions, or media types (e.g., printing, portrait, tab support).

- Split CSS files based on media type.
- Add the **`media` attribute** to the `<link>` statement in the HTML.
- The browser automatically loads only the CSS files relevant to the current media type, providing optimization out of the box. If the media attribute is not defined, it defaults to `all`.

#### C. Asynchronous/Non-Blocking CSS Loading

Load non-critical CSS files in a non-blocking fashion.

**Method 1: JavaScript Injection**

- Use JavaScript (e.g., by adding a script tag at the bottom) to dynamically create a link tag and attach the CSS file to the `<head>` element, effectively loading it lazily.

**Method 2: Preload Hack**

- Use `<link>` with `rel="preload"` and `as="style"` to load the CSS file without blocking rendering.
- Use a JavaScript `onload` event to change the `rel` attribute from `preload` to `stylesheet`, telling the browser to consume, parse, and apply the styles.
- **Fallback:** Include a `<noscript>` block with a standard `<link rel="stylesheet">` for users who have JavaScript blocked.

**Future Technique (Build Time Optimization):**

- Using CSS-in-JS formats like **Emotion** or **Styled Components** (often with React) allows inline styles, loading the CSS along with the component instead of loading everything in a single go.

### 5. JavaScript Optimization Techniques

Optimization is necessary for JavaScript (JS), which is parser blocking.

#### A. Loading Optimization (`defer` vs. `async`)

These attributes are the easiest way to optimize JavaScript file loading.

|Script Loading Method|Parsing Behavior|Download Timing|Execution Timing|Impact|
|:--|:--|:--|:--|:--|
|**Normal Script**|Stops (halts) parsing.|Downloads immediately.|Executes immediately once downloaded.|Parsing is blocked until execution is complete.|
|**`defer`**|Continues parsing in parallel.|Downloads in parallel.|Executes only _after_ HTML parsing is complete.|Preferred because it doesn't halt parsing.|
|**`async`**|Continues parsing in parallel.|Downloads in parallel.|Executes immediately once downloaded, temporarily halting parsing.|Halts passing for execution, but less time blocked overall than a normal script.|

#### B. Script with `type="module"`

Used for loading ES6/ES bundles.

- By default, scripts with `type="module"` load using the **`defer`** technique (download parallel, execute after parsing).
- This can also be explicitly set to `async` (where execution halts parsing temporarily).

#### C. Execution Optimization (Web Workers)

Web workers are used to handle heavy computation jobs off the main thread, ensuring the main thread remains free for user interaction and rendering.

**Use Cases for Web Workers (Heavy Computation):**

- Analytics dashboards (data massaging, chart plotting).
- Image processing (UI effects like blur).
- Canvas drawing (whiteboard applications).
- Gaming applications.

**Web Worker Details:**

- **Limitation:** Workers cannot perform DOM manipulation or direct canvas manipulation.
- **Communication:** Main thread and worker thread communicate using **Post Messaging**.
    - The main thread uses `postMessage` to send data.
    - The worker uses the `onmessage` handler to receive and send data back.
- **When to Use:** When code block computation is long, or when there is small input/output for fast communication.
- **When to Avoid:** When dependency on DOM manipulation or near real-time output based on user input is required, as communication takes time.
- **Implementation:** An instance of a worker is created using `new Worker()`.

### Conclusion

This session covered optimization for images, videos, fonts, CSS, and JavaScript. The next session is planned to cover **build optimization**, following the previous session on network optimization.

--------------------------------------------------------------------------