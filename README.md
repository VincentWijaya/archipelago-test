# Web Developer Screening Questions - Senior Fullstack Candidate

## Basic Questions

### Photo Moderation System Design

For a photo submission and moderation system, I'd opt for a robust backend managing the photo states.

**Logic Design:**
When a user uploads a photo, it would initially be set to a `pending` status. An API endpoint (e.g., `/api/photos/upload`) would handle the file upload and metadata. The photo file itself would be stored in object storage like AWS S3, with its URL saved in our database. Photos in `pending` status would populate a moderation dashboard. Moderators could 'Approve' (changing status to `published` for public visibility) or 'Reject' (changing status to `rejected`, potentially with a reason) the photos. Notifications could be triggered to inform the user of the moderation outcome.

**Technologies:**
* **Backend:** Node.js with Express for flexibility, or Golang for performance and concurrency if image processing were a future consideration.
* **Database:** PostgreSQL for its ACID compliance and structured nature, ideal for photo metadata and user info.
* **File Storage:** AWS S3 for scalable and reliable object storage.
* **Frontend:** Vue.js or React for an interactive user and moderator interface.

**Data Structure (PostgreSQL example for `photos` table):**
```sql
CREATE TABLE photos (
    id UUID PRIMARY KEY, -- or auto-incrementing integer
    user_id INT REFERENCES users(id),
    s3_url VARCHAR(255) NOT NULL,
    status VARCHAR(20) NOT NULL CHECK (status IN ('pending', 'published', 'rejected')),
    upload_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    moderation_timestamp TIMESTAMP,
    moderator_id INT REFERENCES employees(id), -- or moderators table
    rejection_reason TEXT,
    title VARCHAR(255),
    description TEXT,
    tags JSONB -- or a separate photo_tags join table
);
```

This design ensures a clear audit trail and efficient querying based on moderation status.

## Database Questions

### Level 1 (Novice)
To count customers from Germany:
```sql
SELECT COUNT(CustomerID)
FROM Customers
WHERE Country = 'Germany';
```
This query shows how many customers there are from Germany.

### Level 2 (Business Admin)
To list countries by customer count (most to least, min 5 customers):
```sql
SELECT Country, COUNT(CustomerID)
FROM Customers
GROUP BY Country
HAVING COUNT(CustomerID) >= 5
ORDER BY COUNT(CustomerID) DESC;
```

### Level 3 (Average Developer)
Reverse engineering the provided results, the query to get customer order counts, first order, and last order dates:
```sql
SELECT
    c.CustomerName,
    COUNT(o.OrderID) AS OrderCount,
    MIN(o.OrderDate) AS FirstOrder,
    MAX(o.OrderDate) AS LastOrder
FROM
    Customers c
    INNER JOIN Orders o ON c.CustomerID = o.CustomerID
GROUP BY
    c.CustomerName
HAVING
	COUNT(o.OrderID) >= 5
ORDER BY
    MAX(o.OrderDate) desc;
```

## JavaScript/TypeScript Questions
### Level 1.1: Title Case Function
```javascript
function titleCase(str) {
  if (!str) return "";

  const words = str.toLowerCase().split(' ');

  const capitalizedWords = words.map(word => {
    const firstLetter = word.charAt(0).toUpperCase();
    const restOfWord = word.slice(1);
    return firstLetter + restOfWord;
  });

  return capitalizedWords.join(' ');
}

// ✅ Test Cases
console.log(`"I'm a little tea pot" should return a string: ${typeof titleCase("I'm a little tea pot") === 'string'}`);
console.log(`"I'm a little tea pot" should return "I'm A Little Tea Pot": "${titleCase("I'm a little tea pot")}"`);
console.log(`"sHoRt AnD sToUt" should return "Short And Stout": "${titleCase("sHoRt AnD sToUt")}"`);
console.log(`"SHORT AND STOUT" should return "Short And Stout": "${titleCase("SHORT AND STOUT")}"`);
```

### Level 1.2: Word Frequency Counter

```javascript
function wordFrequencyCounter(text) {
  const wordCounts = {};

  const words = text.toLowerCase().split(/\s+/);

  for (const word of words) {
    if (!word) continue;

    if (wordCounts[word]) {
      wordCounts[word] += 1;
    } else {
      wordCounts[word] = 1;
    }
  }

  return wordCounts;
}

// Test Case
const testString = "Four One two two three Three three four four four";
const result = wordFrequencyCounter(testString);
console.log(result);

/*
Expected Output:
{
  one: 1,
  two: 2,
  three: 3,
  four: 4
}
*/
```
### Level 2: Fix delay with Promises

```javascript
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Example usage:
delay(3000).then(() => console.log('runs after 3 seconds'));
```
### Level 2.5: Async/await
```javascript
// Convert fetchData to return a Promise
function fetchDataPromise(url) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (!url) {
        reject("URL is required");
      } else {
        resolve(`Data from ${url}`);
      }
    }, 1000);
  });
}

// Convert processData to return a Promise
function processDataPromise(data) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (!data) {
        reject("Data is required");
      } else {
        resolve(data.toUpperCase());
      }
    }, 1000);
  });
}

// Rewrite using Async/Await
async function processWorkflow(url) {
  try {
    const data = await fetchDataPromise(url);
    const processedData = await processDataPromise(data);
    console.log("Processed Data:", processedData);
  } catch (error) {
    console.error("Process Error:", error);
  }
}

// Example usage:
processWorkflow("[https://example.com](https://example.com)");
processWorkflow(null); // To test the error case
```

### Level 3-4 (See livechat folder)

## Vue.js

#### Explain Vue.js reactivity and common issues when tracking changes.

Vue's reactivity system efficiently updates the DOM by converting data properties into reactive getters and setters. When a reactive property is used in a component, Vue tracks it as a dependency. Upon a change, the setter triggers an update, leading to a targeted re-render of affected components using a virtual DOM.

Common Issues:

- **Adding new properties to reactive objects (Vue 2)**: Directly adding this.myObject.newProp = 'value' wasn't reactive. Solved by Vue.set() or creating a new object. Vue 3 (using Proxies) largely resolves this.
- **Directly modifying array indices or length (Vue 2)**: this.myArray[index] = newValue or this.myArray.length = 0 were not reactive. Required array methods (splice, push). Vue 3's Proxies handle these better.
- **Using non-reactive data**: Data not properly declared as reactive won't trigger updates.
- **Asynchronous updates**: Vue batches DOM updates. nextTick() is needed to access DOM after a reactive change.
- **Nested object/array changes**: While Vue 3 is improved, deeply nested changes might still require re-assigning entire objects/arrays for optimal reactivity.

#### Describe data flow between components in a Vue.js app

Vue's data flow is primarily unidirectional, ensuring predictability and maintainability.

1. **Props Down, Events Up**:
1.1 **Props Down**: Parent components pass data to children via props. Children should treat props as immutable.
1.2 **Events Up**: Children communicate back to parents by emitting custom events. Parents listen for these events and update their own data, which then flows down.
2. **Slots**: Allow parent components to inject content into specific areas of a child's template, providing flexible component composition.
3. **Provide/Inject**: Used for passing data deeply to nested components without "prop-drilling." A parent provides data, and any descendant can inject it. Used cautiously, as it can obscure data flow.
4. **State Management Libraries (Vuex/Pinia)**: For larger applications, a centralized store (like Pinia or Vuex) acts as a single source of truth. Components commit mutations or dispatch actions to update the store, and read state directly from it.

#### List the most common cause of memory leaks in Vue.js apps and how they can be solved.

Memory leaks in Vue apps occur when references to objects persist unexpectedly, preventing garbage collection.

1. **Event Listeners Not Cleaned Up**: Adding ```window.addEventListener``` or custom event listeners in ```mounted()``` but failing to ```removeEventListener``` in ```beforeUnmount()``` (or ```destroyed()``` in Vue 2).
2. **Timers Not Cleared**: ```setInterval``` or ```setTimeout``` calls not being ```clearInterval``` or ```clearTimeout``` in ```beforeUnmount()```.
3. **Unsubscribed Subscriptions**: Failing to unsubscribe from observables (e.g., RxJS) or close WebSocket connections when the component is unmounted.
4. **Global Objects/Caches Holding References**: Components registering with a global object or third-party library without deregistering upon unmount.
5. **Improper ```v-if``` vs ```v-show``` use**: ```v-show``` only toggles visibility, keeping components mounted, which can accumulate leaks if not properly managed, whereas ```v-if``` truly mounts/unmounts.
6. The key solution is to clean up what you set up: if you create or open something in ```mounted()``` (or similar lifecycle hooks), ensure it's destroyed or closed in ```beforeUnmount()```.

#### What have you used for state management

My primary choice for Vue.js state management has transitioned to Pinia. Before Pinia, I extensively used Vuex, especially for Vue 2 projects.

I prefer Pinia due to its:

- **Simpler API**: Feels more aligned with the Composition API (ref/reactive).
- **Native TypeScript Support**: Excellent out-of-the-box type safety.
- **No Mutations Concept**: Simplifies state modification directly within actions.
- **Modularity**: Easy to define and use modular stores without complex configurations.
  
For smaller applications, I sometimes rely on the Composition API's ref/reactive combined with provide/inject to avoid over-engineering.

#### What’s the difference between pre-rendering and server side rendering?

Pre-rendering (Static Site Generation - SSG) vs. Server-Side Rendering (SSR)

Both are techniques to improve initial load time and SEO for JavaScript applications, but they differ in when and how HTML is generated.

##### Pre-rendering (Static Site Generation - SSG)

* **When:** HTML is generated at **build time**.
* **How:** A tool (e.g., Nuxt.js, Next.js) crawls routes and creates static HTML files for each.
* **Delivery:** Static HTML served directly from a CDN. JavaScript then "hydrates" it into an interactive SPA.
* **Pros:**
    * Extremely fast initial load (low TTFB)
    * Excellent for SEO
    * Simple hosting
* **Cons:**
    * Requires rebuild on content change
    * Best for mostly static content

##### Server-Side Rendering (SSR)

* **When:** HTML is generated **on the fly for every request** on a server.
* **How:** The server runs your application code, renders initial HTML for the specific route, and sends it to the browser.
* **Delivery:** Browser receives and displays HTML, then JavaScript hydrates it.
* **Pros:**
    * Good for SEO
    * Faster initial load than pure client-side rendering
    * Suitable for dynamic content
* **Cons:**
    * Requires a server (higher infrastructure cost/complexity)
    * Can have higher TTFB than SSG due to server processing

Essentially, SSG is like baking a cake in advance, while SSR is baking a fresh cake per order.

## Website Security Best Practises

Security is paramount. My top priorities for web security include:

* **Input Validation & Sanitization:**
    * Never trust user input.
    * Validate data types, lengths, formats, and ranges.
    * Sanitize input to neutralize malicious characters (e.g., prevent SQL Injection, XSS).
    * Always validate on both client and server-side, with server-side being the ultimate authority.

* **HTTPS Everywhere:**
    * Enforce HTTPS for all traffic using robust TLS certificates.
    * This encrypts communication, protecting data integrity and confidentiality.
    * Implement HSTS (HTTP Strict Transport Security).

* **Strong Authentication & Authorization:**
    * **Authentication:**
        * Enforce strong password policies.
        * Use MFA (Multi-Factor Authentication).
        * Securely hash and salt passwords (e.g., bcrypt).
    * **Authorization:**
        * Implement robust RBAC (Role-Based Access Control) or ABAC (Attribute-Based Access Control) systems.
        * Verify permissions on the server-side for every sensitive operation.

* **Protection Against OWASP Top 10:**
    * Actively mitigate common vulnerabilities like:
        * XSS (Cross-Site Scripting) (output encoding, CSP - Content Security Policy).
        * CSRF (Cross-Site Request Forgery) (anti-CSRF tokens).
        * Broken Access Control (proper authorization checks).

* **Dependency Management & Patching:**
    * Regularly audit and update all third-party libraries/frameworks to patch known vulnerabilities.

* **Secure Error Handling & Logging:**
    * Provide generic error messages to users.
    * Log detailed errors securely on the server.
    * Monitor logs for suspicious activity.

* **Least Privilege Principle:**
    * Grant only the minimum necessary permissions to users, applications, and services.

## Website Performance Best Practises

* **Minimize HTTP Requests & Optimize Asset Delivery:**
    * Concatenate and minify CSS/JS.
    * Use bundlers (Webpack, Vite).
    * Optimize images for lower requests (e.g., CSS sprites, though less critical with HTTP/2).

* **Image Optimization:**
    * Compress images without quality loss.
    * Implement responsive images (`srcset`, `sizes` attributes).
    * Lazy load images (`loading="lazy"`).
    * Utilize modern formats (WebP, AVIF).

* **Leverage Caching:**
    * **Browser Caching:** Configure HTTP caching headers for static assets (long cache durations with cache busting).
    * **CDN:** Distribute static assets closer to users.
    * **Server-side Caching:** Cache frequently accessed data or dynamic content.

* **Optimize Critical Rendering Path & Defer Non-Critical Assets:**
    * Use `defer` or `async` for `<script>` tags to prevent blocking.
    * Prioritize above-the-fold content.

* **Efficient JavaScript Execution:**
    * Apply tree shaking and code splitting.
    * Minimize DOM manipulation.
    * Consider Web Workers for heavy computations.

* **Enable Gzip/Brotli Compression:**
    * Compress text-based resources transmitted over the network.

* **Avoid Render-Blocking Resources:**
    * Regularly audit with tools like Google Lighthouse to identify and resolve bottlenecks.

## Golang
```go
package main

import (
    "fmt"
    "strings"
)

func wordFrequencyCounter(text string) map[string]int {
    wordCounts := make(map[string]int)
    words := strings.Fields(strings.ToLower(text))

    for _, word := range words {
        if word == "" {
            continue
        }
        wordCounts[word]++
    }

    return wordCounts
}

func main() {
    testString := "Four One two two three Three three four four four"
    result := wordFrequencyCounter(testString)
    fmt.Println(result)
    // Expected Output:
    // map[one:1 two:2 three:3 four:4]
}
```

## Tools (Rate yourself 1 to 5)
* **Git:** 5/5
    * Highly proficient with Git, including complex branching, merging, rebasing, and conflict resolution in daily workflows.
* **Redis:** 4/5
    * Extensive experience with Redis for caching, session management, and messaging. Confident in setup and interaction, though I haven't delved into every advanced cluster setup.
* **VSCode / JetBrains:** 5/5
    * VSCode is my primary IDE; highly proficient with its features, extensions, and debugging.
* **Linux:** 4/5
    * Very comfortable with Linux environments, command line operations, server management, scripting, and understanding core concepts for web development and operations.
* **AWS:** 4/5
    * Hands-on experience deploying and managing applications on AWS, proficient with core services like EC2, S3, RDS, Lambda, CloudWatch, Route 53, and CloudFront.
  * **EC2:** 4/5
    * Comfortable with launching, configuring, and managing EC2 instances.
  * **Lambda:** 3/5
    * Experienced in using AWS Lambda for serverless functions, triggered by various events like API Gateway or S3.
  * **RDS:** 4/5
    * Solid experience with RDS for managed relational databases (PostgreSQL, MySQL).
  * **Cloudwatch:** 3/5
    * Regularly use CloudWatch for monitoring logs, setting alarms, and tracking basic metrics for AWS resources.
  * **S3:** 4/5
    * Proficient with S3 for static hosting, content storage, backups, and serving assets via CloudFront.
* **Unit testing:** 4/5
    * Strong believer in unit testing, applying mocks and assertions for good test coverage.
* **Kanban boards:** 5/5
    * Very familiar and experienced with Kanban boards and agile methodologies for task tracking, workflow management, and prioritization.