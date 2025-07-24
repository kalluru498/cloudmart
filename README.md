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

---

## 3. Project Setup and Local Development

To get the CloudMart Customer Support Assistant up and running on your local machine, follow these steps:

### Prerequisites
Make sure you have the following installed:
* Node.js (LTS version recommended)
* npm (comes with Node.js)
* AWS CLI (configured with appropriate credentials for DynamoDB and Bedrock)

### Installation
1. Clone the repository:
```bash
git clone https://github.com/your-username/cloudmart-ai-assistant.git
cd cloudmart-ai-assistant
```

2. Install backend dependencies:
```bash
npm install
```
(Navigate to the frontend directory and install its dependencies if applicable)

### Configuration
Create a `.env` file in the root of your backend directory and populate it with your environment variables:
```
PORT=5000
OPENAI_API_KEY=your_openai_api_key_here
OPENAI_ASSISTANT_ID=your_openai_assistant_id_here
AWS_REGION=your_aws_region_here # e.g., us-east-1
AWS_ACCESS_KEY_ID=your_aws_access_key_id_here
AWS_SECRET_ACCESS_KEY=your_aws_secret_access_key_here
BEDROCK_AGENT_ID=your_bedrock_agent_id_here
BEDROCK_AGENT_ALIAS_ID=your_bedrock_agent_alias_id_here
```

* Make sure your OpenAI Assistant is set up with the appropriate tools (e.g., `cancel_order`, `delete_order`) as defined in `aiService.js`.
* Ensure your Bedrock Agent is configured with relevant actions, potentially including the product recommendations action group handled by `index.js`.

### Database Setup (DynamoDB)
Ensure you have DynamoDB tables named `cloudmart-orders` and `cloudmart-products` in your configured AWS region. You can create them via the AWS Console or AWS CLI.

Example `cloudmart-orders` table schema:
* Primary Key: `id` (String)
* Attributes: `userEmail`, `status`, `createdAt`, etc.

Example `cloudmart-products` table schema:
* Primary Key: `id` (String)
* Attributes: `name`, `description`, `price`, `image`, `createdAt`, etc.

You can populate sample product data using the `populateProductsTable()` function (if implemented in your `aiService.js`).

### Running the Application
1. Start the backend server:
```bash
npm start
```
The server will typically run on http://localhost:5000.

2. Start the frontend application:
(Instructions will vary depending on your frontend setup, e.g., `npm start` in the frontend directory).

## 4. API Endpoints

The backend exposes the following API endpoints:

### AI Endpoints (`/api/ai`)
* **POST `/api/ai/start`**: Starts a new OpenAI conversation thread.
  * Returns: `{ threadId: string }`
* **POST `/api/ai/message`**: Sends a message to the OpenAI Assistant and retrieves its response.
  * Request Body: `{ threadId: string, message: string }`
  * Returns: `{ response: string }`
* **POST `/api/ai/bedrock/start`**: Starts a new Bedrock conversation.
  * Returns: `{ conversationId: string }`
* **POST `/api/ai/bedrock/message`**: Sends a message to the Bedrock Agent and retrieves its response.
  * Request Body: `{ conversationId: string, message: string }`
  * Returns: `{ response: string }`

### Order Endpoints (`/api/orders`)
* **POST `/api/orders`**: Create a new order
* **GET `/api/orders`**: Get all orders
* **GET `/api/orders/user?email={email}`**: Get orders by user email
* **GET `/api/orders/:id`**: Get order by ID
* **PUT `/api/orders/:id`**: Update an existing order
* **DELETE `/api/orders/:id`**: Delete an order

### Product Endpoints (`/api/products`)
* **GET `/api/products`**: Get all products
* **GET `/api/products/:id`**: Get product by ID
* **POST `/api/products`**: Create a new product
* **PUT `/api/products/:id`**: Update an existing product
* **DELETE `/api/products/:id`**: Delete a product

## 5. Key Technologies Used
* **Backend**: Node.js, Express.js
* **AI Integration**: OpenAI Assistant API, AWS Bedrock Agent Runtime
* **Database**: AWS DynamoDB
* **AWS SDKs**: @aws-sdk/client-bedrock-agent-runtime, @aws-sdk/client-dynamodb, @aws-sdk/lib-dynamodb
* **Utilities**: dotenv, uuid
* **Frontend**: React (assuming a React frontend based on CustomerSupportPage.jsx)

## 6. Future Enhancements
* **Enhanced Order Management**: Implement more complex order status workflows (e.g., shipping, delivery)
* **Product Recommendation Engine**: Leverage AI to provide personalized product recommendations
* **User Authentication and Authorization**: Integrate a user login system
* **Customer Profiles**: Allow the AI to access and update customer-specific information
* **Multi-language Support**: Extend the AI to handle requests in various languages

## 7. Contributing
We welcome contributions! Please see our CONTRIBUTING.md (if applicable) for guidelines on how to submit pull requests, report issues, and more.

## 8. License
This project is licensed under the MIT License - see the LICENSE file for details.