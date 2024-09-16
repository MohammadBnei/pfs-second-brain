
**Connectivity Context Documentation**

**Purpose:**
The Connectivity Context is an object that serves as a single source of truth for each request, ensuring consistency across different interactions in the sales process.

**Key Characteristics:**

* **Data Storage:** The Connectivity Context is stored in Redis, identified by a unique "code" (session ID).
* **Request-Based Updates:** Each request represents a step in the sales process (Quotation, Prebooking, Booking), and updates or modifies the Connectivity Context according to business rules or user's choices.
* **Session Expiration:** The Connectivity Context object's expiration time in Redis corresponds to the user's session expiration time. Upon session expiration, the Connectivity Context is removed from Redis, and an error message is displayed to the user.
* **Successful Booking Session:** At the end of a successful booking session, the Connectivity Context is written to an Amazon S3 bucket and stored in the backoffice system for client services to take over.

**Business Rules:**

* Prices are very fluctuating in the flight sector, so it's essential to remove outdated prices by expiring the Connectivity Context upon session expiration.
* The Connectivity Context must be updated or modified according to business rules when a request is made.

**Technical Details:**

* The Connectivity Context object is stored in Redis with a unique "code" (session ID).
* Each request updates or modifies the Connectivity Context according to business rules and saves it back to Redis.
* Upon session expiration, the Connectivity Context is removed from Redis, and an error message is displayed to the user.
* At the end of a successful booking session, the Connectivity Context is written to an Amazon S3 bucket and stored in the backoffice system.

**Benefits:**

* Ensures consistency across different interactions in the sales process
* Removes outdated prices by expiring the Connectivity Context upon session expiration
* Provides a clear audit trail for client services to take over at the end of a successful booking session
