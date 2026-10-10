VIdeo_Link : https://youtu.be/D809Vcki39U?si=3CUJrCjJV5bBMnAT

--------------------------------------------------------------------------

This detailed explanation draws on all the content provided in the sources regarding lazy loading images in HTML, formatted to enhance understanding.

---

## Lazy Loading Images: System Design Concepts for Frontend Engineers

### I. The Problem and Use Case

The concept of lazy loading images addresses significant performance and resource consumption issues that occur when a website contains a large number of images.

1. **Scenario:** If a website has around **20 to 30 images**, when a user loads the site, **all** of these images are downloaded onto the user's machine.
2. **Resource Wastage:** A user might only view five of those images, meaning the remaining **15 to 25 images** are downloaded unnecessarily and never even looked at. This results in significant **memory consumption** and **data wastage**.
3. **User Experience (UX) Issues:** This traditional approach is not user-friendly, especially if the user is operating on a slow network, such as a **3G or 2G network**. Loading many images requires many seconds to render the pages, thereby blocking the rendering process for the user.

### II. The Default Behavior: Eager Loading

The default way a browser handles images defined in HTML is called **eager loading**.

- **Mechanism:** As soon as JavaScript encounters the **`src` tag** within the image attribute (e.g., the `<img>` tag), it **immediately downloads that image file** at that moment.
- **Blocking:** During the time of parsing, if JavaScript encounters 20 or 30 images, it will keep downloading them. This downloading process is considered a **blocker** for the user.
- **DOM Construction:** The browser must create the **DOM tree** along with the assets. If there are many images, this process takes a lot of time for the particular user to load the website.
- **Default Setting:** The value **`eager`** is the default setting for the loading attribute.

### III. The Solution: Implementing Lazy Loading

To avoid the performance issues caused by default eager loading, a special HTML attribute called **`loading`** can be used to easily lazy load images.

#### How Lazy Loading Works:

1. **The Attribute Value:** To activate this mechanism, the `loading` attribute is assigned the value **`lazy`**.
2. **Conditional Download:** The moment JavaScript encounters an image tag with `loading="lazy"`, it ensures that the image is **not downloaded** if that particular image is **not currently in the user's viewport**.
3. **Network Call Trigger:** A network call happens, and the image gets downloaded **only** when the user has arrived or is **about to arrive** on that particular image in the viewport. Images are loaded "on the flight" as the user scrolls.
4. **Demonstration Example:** In a scenario demonstrating lazy loading with perhaps **100 images**, refreshing the page only resulted in the initial download of **four or five images** (specifically mentioned as 400 through 404 in the example network calls). Scrolling down subsequently triggered the loading of other images (e.g., 405, 406, 407).

### IV. Benefits and Optimization

Implementing lazy loading provides crucial benefits related to resource management and network performance:

- **Bandwidth Conservation:** Lazy loading prevents the downloading of **unnecessary images** in the user's browser, which helps in **saving the user's bandwidth**.
- **Performance Optimization:** In the example of 100 images, the initial loading amount was shrunk to just four or five images, optimizing network calls for the user.
- **Crucial Implementation:** This attribute is essential for developers to implement, especially when dealing with a large volume of images in the code.

### V. Best Practices and Future Concepts

While lazy loading is highly beneficial, it must be used selectively.

- **Do Not Lazy Load Everything:** It is important not to lazy load _every_ image.
- **Crucial Assets:** Images that are very crucial for the user to see immediately upon landing on the first page—such as a **logo**, a **header image**, or a **carousel**—should be loaded on **priority**.
- **Targeting:** Developers should aim to **eagerly download assets** that are crucial for the user's first view, while reserving lazy loading for images that are deep down inside the DOM tree.

**Related System Design Concepts:**

The sources mention that in future videos, further network optimization concepts will be explored, as they are crucial for system design interview rounds. These related concepts include:

- Prioritizing images.
- The **`preload`** attribute.
- The **`pre-connect`** attribute.
- Understanding the differences between lazy loading and eager loading.

--------------------------------------------------------------------------