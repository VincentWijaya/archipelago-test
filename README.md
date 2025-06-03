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
### Level 1: Title Case Function
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