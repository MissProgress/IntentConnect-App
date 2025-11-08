# System Architecture: IntentConnect

The IntentConnect application requires high reliability for reminders and low latency for
database lookups (for notes and contact groups). A lightweight,  mobile-first severless 
architecture is proposed for speed and cost-efficient.

## 1. Core Technology Stack

| Component | Technology | Rationale |
| :--- | :--- | :--- |
| **Frontend (Mobile App)** | React Native / Expo | Allows a singlecodebase to deploy efficiently to both iOS and iOS and Android,  accelerating development. |
| **Backend / API** | Node.js (Serverless Functions like AWS Lambda/Google Cloud Function) | Cost-effective and highly scalable for handling api requests and processing scheduled reminders without managing dedicated servers. |
| **Database** | NoSQL (e.g., MongoDB / Google Cloud Firestore) | Optimized for the flexible, document-based data structure required for storing user contacts, call notes, and activity logs. Offers high read/write speed for mobile access. |
| **Scheduling Engine** | Dedicated Cron Service (e.g., AWS EventBridge Scheduler) | Handles the high-volume, precision-timing required for delivering daily/weekly/monthly reminders to the correct users globally. |

---

## 2. Component Communication & Data Flow

### A. Data Structure (Firestore Example)

The core data is stored in simple collections:
* **`users`:** Stores profile and subscription info.
* **`contacts`:** Stores each user's contact list, relationship group (`Family`, `Friends`), and **`next_checkin_timestamp`**.
* **`notes`:** Stores quick notes linked to a specific contact and timestamp (`call_date`, `note_content`).

### B. The Reminder Flow

1. **Setup:** When a user sets a reminder (e.g., "Call Mom every Tuesday"), the Frontend sends a request to the Serverless API.
2. **Scheduling:** The API updates the contact's `next_checkin_timestamp` in the database and registers a recurring event with the **Scheduling Engine**.
3. **Execution (Daily):** The Scheduling Engine runs daily, querying the database for all reminders whose `next_checkin_timestamp` falls within the next 24 hours.
4. **Notification:** The Engine triggers a third-party **Push Notification Service** (e.g., Firebase Cloud Messaging) to send a notification to the user's mobile device.
5. **User Interaction:** When the user taps the reminder, the mobile app retrieves the latest **In-Call Notes** for that contact from the Database *before* presenting the "Dial" button.

---

## 3. Technical Feasibility & Rationale

* **Feasibility:** The use of **React Native** drastically cuts down development time for cross-platform deployment. The **Serverless/NoSQL** combination is extremely scalable and cost-efficient for an application whose usage will fluctuate based on reminder times.
* **Intentionality:** The design strictly avoids automatically dialing or sending messages. The app only provides the prompt and the necessary context (In-Call Notes), keeping the final action (**tapping 'Dial'**) intentional and private, in line with the product's core philosophy.
* **Insights Feature:** The **Weekly/Monthly Insights** are calculated by running simple aggregate queries against the `notes` and `contacts` collections to measure consistency and identify neglected contacts. 
