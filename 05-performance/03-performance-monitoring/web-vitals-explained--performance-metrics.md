
Video_Link : https://youtu.be/KY6Cr6t-cuA?si=wj9OEdvf1KrapZ7C


--------------------------------------------------------------------------

The source provides an in-depth explanation of Web Vitals, including definitions, measurements, causes of poor scores, and specific optimization techniques based on the speaker's personal experience.

The following details cover every point mentioned in the source:

## I. Introduction to Web Vitals and Context

- **Key Terms:** Web Vitals are performance words often heard, including **FCP, LCP, INP, FTFB,** and **OMG** (as a general exclamation).
- **Measurement Tools:** These performance numbers are provided when running a website through tools like **Page Speed** or when looking at performance in **Dev tools**.
- **Personal Website Improvement:** The speaker has been moving their personal website and seeking to improve its performance.
    - **Initial Score:** The website initially scored around **65** without any changes.
    - **Improved Score:** The speaker achieved a score of **95** by adding and tweaking small elements ("little nuggets").
    - **Maximum Observation:** The speaker notes they have never seen a website score higher than **95**. The score can be improved slightly higher by removing content.
- **Lighthouse Score Philosophy:** A tweet suggested that the only way to get a high Lighthouse score (a 100 performance Lighthouse score) is by **shipping absolutely nothing**.
    - This is acknowledged as "so true" but **not practical** for most websites, which must include elements like images.
    - The speaker notes that people often ship "really boring websites" in pursuit of high scores, and suggests sites should be livened up.

## II. Specific Score Details and Trade-offs

- **Desktop Score:** The speaker's website achieves **100** on desktop.
- **Mobile Score:** On mobile, specifically a "slow 4G" connection on a "crappy Android phone," the score is **95**.
- **Contentful Paint Example:** The speaker found that a small visual element, **"little grunge speckles over top of my text,"** was causing a later Contentful Paint.
- **The Content/Score Balance:** The score could be higher by stripping out all images and removing the "charm," but the speaker is happy with the score because the website is "nice looking".
- **Security in Score:** If a developer knows why their score is low (e.g., an image) and determines that cause is **not affecting the user's experience**, they can be "secure" in a score that is not 100.
- **Ego Fixation:** The speaker joked they are almost at the point of wanting to **user agent detect** the Page Speed Insight website and **remove those specific items** only for that user agent, just to prevent people on Twitter from "trying to dunk" on them.
- **Current Personal Site Score:** The speaker's personal site is currently at **96**, which they acknowledge is an "ego fix" and likely not affecting actual visitors, but they are curious to gain a few more points.

## III. Core Web Vitals Explained

The speaker references **Sentry web vitals.com** as a resource that does a great job of explaining and visualizing the vitals.

### 1. LCP (Largest Contentful Paint)

- **Measures:** The time it takes for the **largest text or image element to render** on the web page.
- **Definition Summary:** Waiting for the website to **stop making huge changes**.
- **Causes of Poor LCP (Frustrating Experiences):**
    - Seeing content load, then something else loads and renders again.
    - A component (like a banner) loading late and **pushing the website down**.
    - A **fallback font** rendering first, and then the web font loading and causing a re-render/repaint.
    - The page constantly moving while the user attempts to interact with it.

### 2. FCP (First Contentful Paint)

- **Measures:** The time from when a page starts loading to when **any part of the page's content is first displayed**.
- **Optimization Techniques/Hacks:** People may try to improve FCP instantly using **skeleton screens** (especially in client-side rendering).
- **Causes of Poor FCP:**
    - A slow server response.
    - Blocking CSS or HTML that prevents the actual HTML from loading.
- **Personal Fix (FCP):** The speaker achieved a huge improvement when they **flipped caching on** (it had been off during development), which reduced response times for dynamic elements, significantly lowering the FCP score.

### 3. CLS (Cumulative Layout Shift)

- **Measures:** The **total amount of unexpected layout shifts** that occur during the entire lifespan of a web page.
- **User Impact:** CLS is described as one of the **top two most frustrating things** for a user.
- **Primary Causes:**
    - An image tag that **does not have the width and height encoded**.
    - **Late resource loading**, typically an ad, which causes the page to move, resulting in the user clicking on the wrong thing.
- **Personal Fix (Images):** The speaker's logo initially lacked width/height, causing it to load as zero pixels high, then **bump everything down**.
    - The speaker wrote a **script that runs on build** to figure out the aspect ratio of every image and encode that data into the image tag.
- **Tools:** The **Next.js image tag** performs this action and is very handy.

### 4. INP (Interaction to Next Paint)

- **Measures:** The time it takes for an **interaction** (click, touch, or scroll) to when the **actual paint happens** and the user sees the result.
- **Visibility:** INP **is not seen** when using Page Speed Insights because it requires user interaction.
- **Speed Metrics:**
    - A slow INP example can be 1 second or 400 milliseconds.
    - **136 milliseconds** is considered a fast INP.
    - **500 milliseconds** is considered slow INP.
    - Over 500 milliseconds is very, very slow.
- **Causes of Poor INP:**
    - **Faulty code** that blocks the event (e.g., faulty click event capturing down).
    - **Not great programming** within React components, where a lot of JavaScript runs before the UI updates.
    - Waiting for a server response.
- **Personal Anecdote (eBay):** Clicking eBay's search bar would not focus for around 200 milliseconds, suggesting blocking code.
- **Native App Standard:** Native applications (on iPhone or Android) almost always do something the moment a user clicks, setting the standard for web apps.
- **Fixes:**
    - If waiting for a server response, use **optimistic UI** so the action paints immediately while background work happens.
    - Ensure a visual signal is shown to the user immediately upon clicking so they know the interaction registered and don't click twice.
- **Measuring INP:** Can be measured in the Dev tools Performance tab by loading the page and clicking on something.
- **Personal Measurement (INP):** The speaker's logo click resulted in an INP of **40 milliseconds**.
- **Framework Advantage:** This fast INP (40ms) is attributed to the framework being used, **Waku (or Waku)**, which preloaded the homepage, making the component swap extremely fast.

### 5. TTFB (Time to First Byte)

- **Measures:** The amount of time between the **request being made** and when the **first byte is delivered** to the browser.
- **Cause:** This metric is entirely determined by **how quickly the server can deliver a response** to the client.
- **Related Issues:** Slow rendering or lack of caching.
- **Personal Anecdote (TTFB):**
    - The uncached server response was **727 milliseconds**.
    - When the function was "warm" (after an immediate refresh), the TTFB was **93 milliseconds**.
- **Future Strategy:** The speaker plans to cache the output and use **`stale-while-validate`** to prevent the user from waiting for server rendering.

## IV. Optimization Techniques and Debugging Tips

### A. Font Fallback Optimization (via Juan Farah)

- **Context:** While using `font-fallback: swap` is recommended (to show text immediately while the custom font loads, avoiding invisible text), a problem arises when the fallback font is a different size than the actual font, causing layout shift.
- **The Solution:** Juan Farah suggested creating a `@font-face` called `fallback` that uses local **Arial**, but then adjusts properties like **`ascent`**, **`descent`**, **`line-gap`**, and **`size-adjust`**.
- **Effect:** This effectively stretches every letter of Arial to match the size of the custom web font (**Red Hat Display**).
- **Attribution:** Juan Farah credited **Roman R. Stefanov** with a write-up on this technique.
- **Tool Assistance:** **Next.js** handles this automatically when using **`next/font`** (it measures all the glyphs and matches them).
- **Result:** This fix ensures that while a paint still occurs when the custom font loads, the page **does not shift** (push up or down).

### B. Font Preloading

- **Implementation:** The speaker added a preload for each font using `<link rel=preload>`.
- **Benefit:** This tells the browser to **start downloading the assets before the opening `<body>` tag** (before the CSS is parsed and realizes the web font is needed), significantly speeding up the process.

### C. Developer Tools and Debugging

- **Performance Tab:** The Dev tools Performance tab is used to gather many metrics, including INP, CLS, and LCP.
- **Usage:** Click **Record** and use the application, or use **Refresh and Record** to capture the initial page load.
- **Intimidation:** The Performance tab is considered underrated but can be intimidating due to the volume of lines and numbers.
- **Information Provided:**
    - A flame graph of running JavaScript functions.
    - Screenshots to see **frame by frame** how things load and if/when things are shifting.
    - Where **60fps is dropping**.
- **Caching Test:** It is useful to test performance with the client **cache turned off and on** to understand the experience of a first-time visitor.

### D. Other Optimization Tricks

- **Background Images:** Juan Farah also found that if the speaker made their background image style **`cover`**, the web vitals would ignore it because it is then considered a background image, not content.

## V. Syntax Website Metrics and Further Improvements

- **Content Load:** The Syntax website loads a lot of content, including a **massive `content.json` file for search** (which loads in the background and is likely ignored by web vitals).
- **Actual User Data:** The speaker notes that using Google Web Master tools (or similar) will show the actual scores for people visiting the site.
- **Syntax Site Page Speed Score:** 61.
- **Field Data:** Most stats seen by users are **in the green**.
- **Yellow Metrics:** The site is "eeking in the yellow" for **FCP** and **TTFB**.
    - Improving server response speed is the key to moving FCP and TTFB into the green.
- **Desktop Score:** 78.
- **Page Speed Recommendations:** Page Speed Insights offers "broken tests" or "possible savings".
- **Specific Actionable Suggestion:** The site is loading **GitHub avatars directly off GitHub**. Since the site owners have no control over caching or compressing these, they could:
    - Place a **proxy** in front of the avatars.
    - Compress the images or convert them to **AVIF or WebP** formats.
- **Cloud Image Service History:** The speaker previously wrote a **Cloudflare worker** that sat in front of Cloudinary to handle resizing and **caching** (to save Cloudinary bandwidth). Currently, the speaker uses **Cloudflare Images**, which handles all tasks automatically.

## VI. Gamification and Prioritization

- **Motivation:** The speaker finds web vitals optimization "addictive" and enjoys the **gamification** provided by the numbers.
- **Sentry's Value:** Sentry provides helpful data by showing the pages that are **hit the most and have the worst scores**.
- **Prioritization Strategy:** This Sentry data allows developers to prioritize fixing the slowest routes first, ensuring the biggest impact on actual users.
- **Ego Fix Warning:** Fixing scores solely to satisfy Lighthouse, without benefiting users, is merely an **"ego fix"**.

--------------------------------------------------------------------------