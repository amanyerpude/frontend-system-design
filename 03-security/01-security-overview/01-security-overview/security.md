
Video_Link : https://www.youtube.com/watch?v=kjv8B-xqsMg

--------------------------------------------------------------------------

This response draws comprehensively upon the source material provided, ensuring that all details mentioned regarding security for frontend developers are included, formatted for clarity, understanding, and learning.

---

## I. Importance and Context of Security

Security is an extremely critical part of any application. Both senior and junior engineers should be knowledgeable about all techniques required to handle and tackle application security.

### A. The Reality of Attacks and Frontend Responsibility

- **Source of Attacks:** The majority of security attacks in the IT industry start from the **UI interface**. Most major attacks start or initiate on the frontend.
- **The Analogy of Wealth:** Security is likened to safeguarding wealth. Earning money is difficult, but losing money is very easy. If you earn 100 rupees and someone steals 80 rupees, you are making extra effort and going into debt.
- **The Cost of Insecurity:** Losing money can happen via government penalties or hackers stealing data, information, and money.
- **Backend Trust:** The backend gives APIs that can be modified. The backend system tends to trust the frontend systems. Sending **malicious data from the front end** can spoil everything inside the entire system.
- **Frontend vs. Backend Security:** Frontend security is declared as important as backend security.
- **Frontend Developer's Role:** Developers must know the necessary security parameters so they can identify and report issues that are dangerous for the application.
- **User Experience:** Security must be put in check so that it is not compromised, but the **user experience remains very smooth**. Too much security can compromise user experience.

### B. Misconceptions and Core Security Principles

- **Developer Misconceptions:** Many frontend developers mistakenly believe that security is something that should be handled mostly on the backend/server side.
- **Underestimation:** Many people underestimate the security concerns that start from the client.
- **The Rule of Trust:** A fundamental security rule is that **the person or session whom you trust the most is often the most dangerous person**.
- **Users as Threats:** Engineers must operate under the lens that anybody who comes onto the website _can be a malicious user_ and potentially break the systems.
- **Authenticated Users:** Even if a user is authenticated or a paid user, developers cannot simply trust them. All APIs and actions performed by that user must be checked to ensure they cannot mess up the system.
- **User Safety:** Engineers also have a responsibility to keep their users safe and ensure they feel safe when using the website.

## II. Security Roadmap, Techniques, and Vulnerabilities

A system must first identify the types of threats it faces. The source outlines a security roadmap covering various techniques.

### A. Core Security Techniques (Roadmap)

The planned topics for discussion include:

1. Putting the website over **HTTPS**.
2. Sanitizing input data (what the user is entering into input boxes).
3. **Authorization and Authentication**.
4. Security Headers.

### B. Input Validation and Sanitization

- **Importance of Input Data:** What data a user enters is crucial.
- **SQL Injection:** SQL injection attacks are often caused by unsanitized input data. (Note: The speaker mentions they haven't heard of many websites currently suffering from SQL injection attacks lately, suggesting it may be less common now).
- **React/JSX Handling:** When using React, **JSX automatically takes care of input sanitization**. This protection is bypassed if the developer uses `dangerouslySetInnerHTML`.
- **Input Sources:** Input must be validated regardless of where it originates, including **forms, URLs, and anything else**.
- **Output vs. Input Sanitization:** Some developers misunderstand the protection provided by React, believing that output sanitization is covered, but they neglect input sanitization (if they take the value as is to the database without rendering it in JSX).
- **Diverse Threats:** Validation must consider different kinds of data, not just simple JavaScript injection, but also potentially SQL or XML hijack kinds of things.
- **Validation Checks:** Engineers need to ensure they are doing the right set and kind of validation.

### C. Authentication and Token Handling

- **Checking Requests:** After securing the site with HTTPS and validating input, the next step is checking if the request is **authenticated** and **authorized**.
- **Tokens:** Authorization and authentication typically happen through **tokens** (though not always).
- **Token Safety:** Tokens must be kept safe on the frontend. If a token is stolen, this is known as **session hijacking**.
- **Storage Mechanisms:** Safe token storage involves consideration of:
    - Local Storage.
    - Session Storage.
    - Cookies.
    - Frontend databases.
- **Expiry and Invalidation:** Tokens must have a clearly set **expiry**. This applies to both the server and the client. When a user clicks "log out," the system must ensure the token is actually invalidated and logged out, rather than just being left in the browser.
- **Critical Storage Error:** It is a grave mistake for a frontend developer to use **local storage to put their authorization token**. This should **never** be done.

### D. Data Protection and PII

- **PII:** Personal Identifiable Information (PII) must be kept very safely.
- **Dumping Data:** Many companies make the major mistake of dumping all user data into their local storage.
- **Local Storage Concern:** If data is kept in local storage, it does not go off unless the user or developer manually removes it.
- **Log Security:** One company example showed encrypted credentials in the database, but they stored the **username and password in plain text in the log**.

### E. Specific Security Concerns

- **Cross-Site Scripting (XSS):** This is a huge concern and is highly important.
- **Phishing/CSRF:** Other concerns include phishing attacks (e.g., crazy offers like winning an iPhone voucher) and CSRF (Cross-Site Request Forgery) tokens. Phishing can result in malicious messages being sent to friends or other activity being initiated by the victim.
- **Role-Based Access (Authorization):** If a system has different roles (e.g., admin panels accessible publicly), developers must ensure things are not leaked to lower roles, such as giving indirect access to something critical when only analytics access was intended.

## III. Industry Experience and Interview Relevance

Security is not just about building systems; it is highly relevant for system design, professional audits, and technical interviews.

### A. Business Impact and Audits

- **Business Contracts:** Contracts, especially with banks, may require an entire scrutiny of the system to check security (including XSS and GDPR compliances) before the contract is secured.
- **Payment Gateways:** Payment Gateway integrations lead to security teams performing scrutiny, reviewing logs, client-side handling, key usage, and secure configurations. (Example cited: Flipkart).
- **User Injections:** Even if a user provides their own script (claiming "this is my website"), the security team (Example cited: Microsoft) will deny it, maintaining that the developer is responsible for security concerns.
- **Major Audits (IPO):** Going through an IPO (Example cited: Uber) involves huge audit trails and months of activity dedicated to checking system leaks. Audit firms may review code, create Jira tickets, and require developers to answer for their practices.
- **Storage Exploitation:** Developers must anticipate that users might exploit the system for unintended purposes, such as uploading movies of any size, format, or file type instead of just profile pictures, turning the site into an unexpected storage bucket and incurring huge storage costs.

### B. Interview Expectations and Preparation

- **Standard Topics:** Basic questions on **CORS** (Cross-Origin Resource Sharing) and its relation to security are popular in interviews.
- **Must-Knows:** Key questions asked include:
    - How to handle PII data (e.g., storing it, logging it).
    - Cross-site scripting concerns.
    - Considerations for input sanitization and validation.
    - Knowing **what to keep in which web storage**.
- **System Design:** In system design interviews, especially for senior engineers, candidates should dedicate **five minutes to discussing security** and how they would secure the system, even if the interviewer does not specifically ask.
- **First Mover Advantage:** Talking about security first builds trust and impresses the interviewer, suggesting the candidate thinks about security at a "Next Level".
- **Critical Systems:** If a system design is related to **finance** or **critical data**, security must be addressed by default.
- **Senior Expectation:** As a senior developer, one cannot claim ignorance regarding security when facing audits.
- **Collaboration:** Engineers must be in sync with backend teams and should voice their concerns about backend practices to ensure overall system security.

### C. Personal Interview Experience

One specific interview experience mentioned involved an interviewer from Zeta (a direct-eye company) who literally **grilled** the candidate on:

1. **Input Sanitization:** Specifically during a machine coding problem where the candidate was hitting APIs without validating input.
2. **Web Storage Usage:** Questions about when to use session storage, cookies, and local storage.

## IV. Conclusion and Call to Action

Security requires maintaining guidelines and checks. A great engineer’s responsibility is keeping things sane and intact regarding security.

- **Continuous Learning:** The speakers (Chirag and AE) are sharing insights from their experiences at Microsoft, Uber, Flipkart, Paytm, and other startups. They encourage viewers to continue watching upcoming videos where Chirag will dive deep into keywords like authorization, authentication, RBA (Role-Based Access), and cross-site scripting.
- **Preparation:** Viewers should take a notebook and a pen to take down notes, keep them handy, and learn the material.
- **Actionable Advice:** After watching the episodes, developers should check their own applications, find security bugs, and fix them.
- **Professional Growth:** Start asking security-related questions during interviews if you genuinely care about the business, users, and data.
- **Course Promotion:** The introductory video mentions the recently launched "Namaste frontend system design course," which covers system design from fundamental to advanced levels (LLD and HLD).
- **Engagement:** Don't forget to like, subscribe, and share the video.

--------------------------------------------------------------------------
--------------------------------------------------------------------------
## Supplementary Video Resources
Video_URL : https://youtu.be/80VviDER96I?si=MbnNKZGyI4fRIRZw (Introduction to Web Security)
Video_URL : https://youtu.be/-GfSbk_VqSk?si=HR_9QvWk9lU33ipz (Mastering Web Security)
