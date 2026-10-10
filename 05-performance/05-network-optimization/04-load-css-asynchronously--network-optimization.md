Video_Link : https://youtu.be/D809Vcki39U?si=n2hNygwwyTcAXT7h

--------------------------------------------------------------------------

This detailed response draws exclusively from the provided source material, tallying all relevant concepts and examples related to optimizing CSS loading to prevent render blocking.

---

### Optimization Strategies for Asynchronous CSS Loading

The sources discuss methods for loading CSS asynchronously to optimize front-end performance, specifically addressing the issue of CSS being a render-blocking resource.

#### 1. Understanding the Initial Problem: Render Blocking

By default, CSS is treated as a **rendering blocking resource**.

- **Definition:** The browser will not render any processed content until the **CSS Object Model (CSSOM)** is constructed.
- **CSSOM:** This is essentially the DOM (Document Object Model) of CSS.
- **Blocking Resources:** Both HTML and CSS are rendering blocking resources, meaning that until both the HTML is parsed and the CSS is parsed, the user will not see anything on their screen because the render is blocked.
- **Impact:** The larger the CSS file, the more time it will take to render the page.

#### 2. Analysis of the Initial Scenario (Basic Syntax)

A basic scenario using standard link tag syntax included three stylesheets: one critical style sheet and two non-critical ones.

- **Syntax:** Using the very basic syntax to include external stylesheets led to performance issues.
- **Initial Waterfall Order:**
    1. HTML file (loaded at the highest priority).
    2. Non-critical CSS.
    3. Style 1 CSS.
    4. Critical CSS.
- **Delay:** In this basic setup, **rendering was blocked for 30 seconds** (10 plus 10 plus 10).
- **Undesirable Outcome:** The critical CSS was loaded after 20 milliseconds, which is not what is desired.

#### 3. Strategy 1: Dividing Styles

It is advised to **divide styles into multiple style sheets**.

- **Critical Styles:** Create a single style sheet containing styles critical for the website's first rendering (e.g., basic UI, navigation, buttons, and basic styles).
- **Non-Critical Styles:** Treat whatever is left as non-critical CSS, potentially dividing it into multiple style sheets according to modules.

#### 4. Strategy 2: Loading Non-Critical CSS Asynchronously (Media Trick)

The core asynchronous loading technique involves manipulating the `media` attribute and using the `onload` event for non-critical stylesheets.

**The Mechanism:**

|Attribute/Property|Purpose|Details|
|:--|:--|:--|
|**`media` attribute**|Initially set to `print` (or a non-`all` value).|By default, `media` is set to `all`. When the browser finds a link tag, it usually blocks the render. If `media` is set to `print`, the browser interprets this as styling for a different purpose (like printing the webpage). **The browser automatically loads this CSS asynchronously and does not block the render**.|
|**`onload` property**|Changes the `media` value after loading is complete.|This event listener is triggered once the CSS file has finished loading. Inside the `onload` function, the `media` property is changed to `all`. This signals to the browser that the CSS should now be applied to the main page the user is viewing.|

**Optimized Waterfall Result (Strategy 2):**

- The non-critical and Style 1 CSS files, although declared above `critical.css`, were loaded _after_ it.
- **`critical.css` was loaded first in four milliseconds**.
- This approach ensures that rendering is not blocked and crucial styles are loaded first. This is the required approach when optimizing CSS assets to minimize render blocking.

#### 5. Strategy 3: Advanced Optimization using Preload and Async

A further improvisation involves using `rel="preload"` along with asynchronous loading.

**The Mechanism:**

1. **`rel="preload"`:** This is set initially. It instructs the browser to download the style sheet and sets its **priority to high/highest**.
2. **`onload` handler:** Once the asset is loaded, the `onload` function is used to **change the `rel` attribute to `stylesheet`**.
3. **Consumption:** Changing the `rel` value is necessary to make the preloaded asset usable and consumable by the browser, allowing it to utilize the classes or IDs inside the CSS file.

**Optimized Waterfall Result (Strategy 3):**

- **`style2.css` (the preloaded asset) was given very high/highest priority and loaded first**.
- `critical.css` was loaded next.
- Non-critical and `style1.css` were loaded last, with the **lowest priority**.

#### 6. Summary and Importance

These small techniques for loading critical and non-critical CSS assets are very powerful. They make a **huge impact on the website** and significantly improve performance and network load time, especially in large applications. These methods are often topics in system design rounds or general front-end interview questions regarding how to avoid render blocking.

--------------------------------------------------------------------------