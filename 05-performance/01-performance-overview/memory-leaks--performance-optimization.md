Video_Links : https://youtu.be/I1b2XF8UJBs?si=0K4WCDYj783iScGP

--------------------------------------------------------------------------

This overview provides a comprehensive and detailed account of the JavaScript performance optimization techniques and popular memory leaks discussed in the source material.

The episode is part of the "Chakde System Design series" and focuses on optimization techniques in JavaScript. It covers popular memory leaks and how to fix them, moving beyond network and asset optimization discussed in previous episodes. The content is aimed at freshers, senior developers, and anyone who believes that Data Structures and Algorithms (DSA) is the only way to achieve memory space optimization, promising to change their perspective.

## 1. Accidental Global Variables (Memory Leak 1)

This is the first memory leak discussed, often occurring unintentionally while writing code.

### Concepts and Mistakes

- **Window Object:** The `window` object is a global object or global reference accessible from anywhere in the application.
- **Accidental Leakage:** If a variable is assigned a value (e.g., `a = 10` or `b = 15`) without being properly declared, it becomes available on the `window` object.
- **Inside Functions:** If a variable (e.g., `C`) is assigned a value inside a function (e.g., `ABC`) but is not declared, it is initialized or declared at its parent scope. In this case, if the function is called in the global space, the variable leaks to the global reference.
- **Using `this`:** Using `this.D` inside a named function (`ABCD`) and calling it globally also results in the value leaking to the global space.
- **Consequences:** The main issues are memory leakage and the fact that anyone can access and overwrite the variable's value accidentally.
- **Named vs. Arrow Functions:** The reference mechanism of named functions differs completely from arrow functions.

### Prevention and Fixes

- **Use Strict:** To avoid accidentally consuming or populating the global space, simply use `"use strict"` at the top of the file. This ensures that variables must be declared, preventing accidental leaks.

## 2. Timers (Set Interval/Timeout) (Memory Leak 2)

This memory leak, involving `setInterval` or `setTimeout`, is one that most people may be aware of.

### The Problem

- **Setup:** A `setInterval` is used to do some operation, such as changing a DOM node's value if it is present. The example uses `JSON.stringify` on an object reference to highlight a complex operation.
- **The Dent:** Although the task (e.g., updating the data) is complete, the `setInterval` keeps trying to update the DOM tree again and again. This repeated, unnecessary job behind the scenes causes a performance dent and a memory leak, especially in big applications.

### The Fix

- **Clear Interval:** Always ensure the timer is cleared once its intended operation is complete.
- **Implementation:** Reference the `setInterval` in a variable (e.g., `timer`). Once the task is done, call `clearInterval` and pass the timer reference. This prevents continuous re-rendering and reduces accidental memory leakage.

## 3. Event Listeners (Memory Leak 3)

Event listeners are frequently added using JavaScript, or behind the scenes in frameworks like React.

### The Problem (Repeated Addition)

- **Scenario:** If an event listener is repeatedly added (e.g., inside a function called by `setInterval` every 500ms, mimicking a scenario perhaps related to `useEffect` in React) without being removed.
- **The Leak:** Every time the listener is added, a new occurrence is registered. Clicking the associated element causes the console log or callback to execute multiple times, corresponding to the number of listeners added (e.g., clicking 3 times might result in 65 console logs).
- **Memory Dent:** If the listeners' callbacks contain references to heavy variables or huge values, the garbage collector will not clean up these memory spaces/references, causing a memory dent.

### Fixes

- **Fix A: Manual Cleanup (Same Reference is Key)**:
    1. Move the function/callback outside so a reference can be maintained.
    2. Create a method to call `removeEventListener`.
    3. **Crucial Step:** When removing a listener, the **same reference** must be used in both the `addEventListener` and `removeEventListener` calls.
    4. In the repeated execution flow, first remove the existing listener, and then add the new one. (Newer browsers might attempt to clean up automatically if a function reference is passed, but manual cleanup is safer).
- **Fix B: Executing Only Once**:
    - If the use case requires the job to run only one time (e.g., closing a dialog/modal), pass the third parameter `{ once: true }` when adding the listener.
    - After the event occurs (e.g., a click), the listener is automatically removed, which also gets rid of the memory space internally.

## 4. DOM References (Memory Leak 4)

This leak involves references to DOM elements that are left behind even after the element is removed from the DOM tree.

### The Problem

- **Scenario:** A reference to a DOM element is stored in a JavaScript variable (e.g., `a`).
- **Removal:** The element is removed from the DOM using `document.body.removeChild`.
- **The Leak:** If the element is accessed again via `document.getElementById`, it returns `null`. However, if the variable `a` is accessed, the DOM element is still visible because that reference is still pointing to the element.
- **The Dent:** If the referenced DOM element was heavy, huge, or nested, the resulting memory dent can be significant.

### The Fix

- **Clean Up References:** Always manually set the stored variable reference to `null` (e.g., `a = null`) after removing the DOM element.
- **Avoid Global Scope:** Operations involving these references should ideally not be executed in the global space. Performing them inside an IIFE (Immediately Invoked Function Expression) or another function scope can help with automatically cleaning up the variables where references are maintained.

## 5. Closures (Memory Leak 5)

Closures are powerful but tricky and can result in memory leaks if not handled with care.

### The Problem

- **Standard Operation:** A global variable (`thing`) is repeatedly overridden inside a function called by `setInterval`. The function creates a new, heavy object reference (containing a "long string") each time it runs. The memory stabilizes somewhat as the garbage collector cleans up old references.
- **The Leak Mechanism (Unused Method):** A memory leak is caused when an internal variable (e.g., `original_thing`), which references the heavy object (`thing`), is used inside an _unused_ method that is declared within the scope of the repeating function.
- **The Dent:** The garbage collector assumes the reference might still be used because the internal method was created. Despite the inner method being unused, the memory consumption increases dramatically (e.g., surpassing 200 MB), potentially leading to system crashes or "out of memory" errors.

### The Fix

- **Handle with Caution:** Developers must handle closures with extreme caution. Always check the browser's memory tab if a memory glitch is suspected. Avoid creating internal references to heavy objects within functions that are perpetually active, especially if those references are captured by unused or unnecessary closures.

## 6. References (Nested Objects) (Memory Leak 6)

This leak focuses on how references, particularly with nested objects, behave, contrasting them with primitive data structures.

### The Problem

- **Reference Persistence:** When an object `x` is referenced by `y` (`y = x`), and then `x` is set to a new value (e.g., `x = 1`), `y` still holds the original object reference.
- **Nested Reference Persistence:** If a variable `z` references a _nested_ object within `y` (e.g., `z = y.a`), and then `y` is reset, `z` retains the reference to that specific internal object.
- **The Dent:** These persistent references to internal objects prevent the memory from being freed, as the object and its structure are technically still reachable within the code base.

### The Fix

- **Complete Cleanup:** When working with nested objects and references, ensure that all variables referencing the object, including those referencing internal or nested parts (e.g., `x`, `y`, and `z`), are explicitly cleaned up or set to `null`. Only when all references are gone can the garbage collector clean up the complex object and its nested structures.

## 7. Detached Window (Memory Leak 7)

This leak involves scenarios where a new window or tab is opened and later closed.

### The Problem

- **Scenario:** A new window is opened (e.g., via a button click). Memory consumption (JS heap size) increases.
- **Closing Issue:** When the new window/tab is closed (e.g., using a parent function call), the memory size remains almost the same as when it was opened; the memory is not sufficiently cleaned up.
- **The Leak:** The window itself was closed, but the memory leak occurs because the _reference_ to that window was not explicitly removed.

### The Fix

- **Nullify the Reference:** Explicitly set the window reference variable (e.g., `nodes_window`) to `null`.
- **Result:** Doing this ensures that after the window is closed, the memory size is significantly reduced, returning close to the starting value. When playing with windows, cleaning up the memory reference is essential, not just closing the window.

## 8. Forget to Stop Listening (Promises/Observables) (Memory Leak 8)

This leak applies to observables, event emitters, and promises, where a response or settlement is mandatory.

### The Problem

- **Promise Expectation:** Promises should always result in success (resolve) or failure (reject).
- **The Leak:** Glitches can occur, causing a memory leak if a promise is forgotten, specifically if it fails to respond or settle.
- **Scenario:** Using `Promise.race` where one promise resolves (potentially doing heavy computation) but another promise is _unsettled_ (not resolving or rejecting).
- **The Dent:** The unsettled promise may leak memory internally, causing the overall memory size to increase to a large extent.

### The Fix

- **Consumption:** Never forget to consume or settle promises; ensure they either resolve or reject.

## 9. Observers (Forget to Disconnect) (Memory Leak 9)

When listening or observing activity, it is crucial to ensure that the process is terminated.

### Observers Mentioned

- Resize Observer
- Intersection Observer
- Mutation Observer

### The Problem

- If a user navigates to a different page or the observed section is unmounted (e.g., if using a library that handles this), forgetting to call `unobserve` or the dedicated library function `disconnect` will leave a memory leak behind.

### The Fix

- Always call `unobserve` or `disconnect` whenever these observers are used.

## 10. Infinite Scroll and Large Data Handling (Memory Leak 10)

This final point discusses two types of memory leaks related to large lists and data management.

### Leak Type A: DOM Element Appendage

- **Problem:** For infinite scroll or huge lists (thousands/millions of items), continuously appending all those DOM elements directly to the DOM tree.
- **Mitigation (Virtualization):** Smarter developers use **windowing logic or virtualization** to ensure only a limited set of DOM elements (e.g., 20 or 30 items) are present in the main DOM tree, manipulating only that data.

### Leak Type B: Internal Memory (State)

- **Problem:** Even when virtualization is used, developers forget to handle the same cleanup problem in the browser's internal memory. They continue to hit API calls, collect data (which may include heavy data like binary or Base64 images), and keep appending this data to internal state management systems (like Redux or browser memory) without cleaning it up.
- **The Dent:** Eventually, the memory size of the browser is consumed significantly.

### General Recommendations

- Always look after the API calls being made and the amount of data being retrieved.
- Be mindful of DOM manipulation, the number of DOM elements created, and avoid performing individual operations in N number of iterations.
- Prefer using batch or bulk calls.

--------------------------------------------------------------------------