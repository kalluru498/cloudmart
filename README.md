# CloudMart Customer Support Assistant

This document explains the architecture and operational flow of the AI-powered customer support assistant within the CloudMart application. This assistant goes beyond traditional chatbots by integrating directly with backend services to perform actions based on user requests, leveraging OpenAI's Assistant API with Function Calling.

---

## 1. Customer Support Assistant AI: Core Concept

Our AI assistant is not just a chatbot that provides information; it's an **Extended AI** that can:
* **Understand User Intent for Action:** It discerns when a user wants something done, not just answered.
* **Trigger Backend Operations:** It can initiate processes or modify data in our system.
* **Automate Tasks:** It handles common customer service requests end-to-end, reducing the need for human intervention.

This "extension" transforms the AI into an intelligent agent capable of interacting with the application's core functionalities.

---

## 2. Operational Flow: Cancelling a Customer Order (Example)

Let's illustrate the full end-to-end flow with a common customer request: **cancelling an order.**

### Scenario: User wants to cancel Order ID `CM-7890`.

**Flow Diagram:**
```
+------------------------------------------------------------------+
|                  Customer Support Assistant: How It Works           |
|                     (Example: Cancelling an Order)                 |
+------------------------------------------------------------------+
|                                                                    |
|  +---------------------+                                           |
|  |     1. Frontend     |                                          |
|  | (CustomerSupportPage.jsx) |                                    |
|  |---------------------|                                          |
|  | - User types:       |                                          |
|  |   "Cancel order CM-7890" |                                     |
|  +----------+----------+                                          |
|             |                                                    |
|             |  HTTP POST: /api/ai/message                        |
|             |  (Body: {threadId: "...", message: "Cancel..."})   |
|             v                                                    |
|  +---------------------+                                         |
|  |     2. Backend      |                                         |
|  |   (Node.js Express) |                                         |
|  |---------------------|                                         |
|  | `server.js` (init)  |                                         |
|  | `aiRoutes.js` (route matching) |                              |
|  | `aiController.js` (endpoint handler) |                        |
|  +----------+----------+                                         |
|             |                                                    |
|             |  Calls aiService.sendOpenAIMessage()               |
|             v                                                    |
|  +---------------------+                                         |
|  |   3. AI Service     |                                         |
|  |     (`aiService.js`) |                                        |
|  |---------------------|                                         |
|  | - Adds user message to OpenAI Thread.                         |
|  | - Initiates an OpenAI `Run`, providing Tool Definitions       |
|  |   (`cancelOrderFunction`).                                    |
|  | - Starts Polling `run` status from OpenAI.                    |
|  +----------+----------+                                         |
|             |                                                    |
|             |  OpenAI API Call: POST /threads/{id}/messages      |
|             |  OpenAI API Call: POST /threads/{id}/runs          |
|             v                                                    |
|  +---------------------+                                         |
|  |    4. OpenAI API    |                                         |
|  | (Assistant, Threads, Runs) |                                  |
|  |---------------------|                                         |
|  | - Processes message.                                          |
|  | - **Identifies Intent:** "Cancel Order".                      |
|  | - **Selects Tool:** `cancel_order` with `orderId: 'CM-7890'`. |
|  | - Sets `Run` Status to `requires_action`.                     |
|  +----------+----------+                                         |
|             ^                                                    |
|             |  OpenAI API Call: GET /threads/{id}/runs/{run_id}  |
|             |  (Returns `runStatus: "requires_action"`, `tool_calls`) |
|             |  (from `aiService.js` polling)                     |
|  +----------+----------+                                         |
|  |   3. AI Service     |                                         |
|  |     (`aiService.js`) |                                        |
|  |---------------------|                                         |
|  | - Detects `requires_action`.                                  |
|  | - Parses `tool_call` (`cancel_order`, `CM-7890`).             |
|  +----------+----------+                                         |
|             |                                                    |
|             |  Calls `orderService.cancelOrder('CM-7890')`       |
|             v                                                    |
|  +---------------------+                                         |
|  |   5. Order Service  |                                         |
|  |    (`orderService.js`) |                                      |
|  |---------------------|                                         |
|  | - Performs validation (e.g., `getOrderById`).                 |
|  | - Executes business logic to update database.                 |
|  +----------+----------+                                         |
|             |                                                    |
|             |  Database Interaction (e.g., Update status in DynamoDB) |
|             v                                                    |
|  +---------------------+                                         |
|  |  6. Database (DynamoDB) |                                     |
|  |---------------------|                                         |
|  | - Order status for 'CM-7890' changed to 'canceled'.           |
|  +----------+----------+                                         |
|             ^                                                    |
|             |  Returns Action Result (e.g., "Order canceled successfully") |
|             |  (from `orderService.js`)                          |
|  +----------+----------+                                         |
|  |   3. AI Service     |                                         |
|  |     (`aiService.js`) |                                        |
|  |---------------------|                                         |
|  | - Submits Tool Output back to OpenAI.                         |
|  +----------+----------+                                         |
|             |                                                    |
|             |  OpenAI API Call: POST /threads/{id}/runs/{run_id}/submit_tool_outputs |
|             v                                                    |
|  +---------------------+                                         |
|  |    4. OpenAI API    |                                         |
|  | (Assistant, Threads, Runs) |                                  |
|  |---------------------|                                         |
|  | - Receives tool output.                                       |
|  | - Generates final natural language response based on outcome. |
|  +----------+----------+                                         |
|             ^                                                    |
|             |  OpenAI API Call: GET /threads/{id}/messages       |
|             |  (from `aiService.js` polling)                     |
|  +----------+----------+                                         |
|  |   3. AI Service     |                                         |
|  |     (`aiService.js`) |                                        |
|  |---------------------|                                         |
|  | - Retrieves final Assistant response.                         |
|  +----------+----------+                                         |
|             |                                                    |
|             |  Returns AI Response to Controller                 |
|             v                                                    |
|  +---------------------+                                         |
|  |     2. Backend      |                                         |
|  |   (Node.js Express) |                                         |
|  |---------------------|                                         |
|  | `aiController.js` (responds to frontend) |                    |
|  +----------+----------+                                         |
|             |                                                    |
|             |  HTTP 200 OK: {response: "Your order CM-7890..."}  |
|             v                                                    |
|  +---------------------+                                         |
|  |     1. Frontend     |                                         |
|  | (CustomerSupportPage.jsx) |                                   |
|  |---------------------|                                         |
|  | - Displays AI's confirmation message to user.                 |
|  +---------------------+                                         |
|                                                                  |
+------------------------------------------------------------------+
```

### Explanation of the Blocks:

* **1. Frontend (`CustomerSupportPage.jsx`):**
    This is the user interface. It captures user input (messages) and sends them to the backend. It also displays the AI's responses back to the user.

* **2. Backend (Node.js Express - `server.js`, `aiRoutes.js`, `aiController.js`):**
    Your application's server, built with the Express.js framework.
    * `server.js`: The main Express application setup, receiving incoming HTTP requests.
    * `aiRoutes.js`: Defines the API endpoints (like `/api/ai/message`) and maps them to specific controllers.
    * `aiController.js`: Handles the incoming API requests, extracts data from the request body (e.g., `threadId`, `message`), and orchestrates calls to the underlying service layers.

* **3. AI Service (`aiService.js`):**
    This is the core business logic layer responsible for interacting with external AI providers (OpenAI in this case).
    * It manages the conversation flow by creating and updating threads.
    * It's responsible for defining and providing "tools" (functions) that the AI can "call."
    * It actively polls the AI provider for updates on the conversation run.

* **4. OpenAI API (Assistant, Threads, Runs):**
    The external AI service provided by OpenAI.
    * It hosts your configured AI Assistant.
    * It manages conversation `Threads` to maintain context.
    * It processes `Runs` of the Assistant against a thread, deciding whether to generate a text response or invoke a `tool_call`.

* **5. Order Service (`orderService.js`):**
    This is a service layer specific to your application's order management. It contains the actual functions (`cancelOrder`, `getOrderById`) that implement your business rules and interact with the database.

* **6. Database (DynamoDB):**
    Your persistent data storage where all application data, such as order details, are stored and managed.

---

This diagram visually represents the complete cycle of an AI-driven action, from user intent to backend execution and final AI confirmation. It highlights how the AI extends beyond mere conversation to leverage your application's backend services through sophisticated function-calling mechanisms.