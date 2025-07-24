This diagram shows the flow of a single user request that leads to an AI-driven action.

+------------------------------------------------------------------+
|                  Customer Support Assistant: How It Works        |
|                     (Example: Cancelling an Order)               |
+------------------------------------------------------------------+
|                                                                  |
|  +---------------------+                                         |
|  |     1. Frontend     |                                         |
|  | (CustomerSupportPage.jsx) |                                       |
|  |---------------------|                                         |
|  | - User types:       |                                         |
|  |   "Cancel order CM-7890" |                                         |
|  +----------+----------+                                         |
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
|  |     (`aiService.js`) |                                         |
|  |---------------------|                                         |
|  | - Adds user message to OpenAI Thread.                         |
|  | - Initiates an OpenAI `Run`, providing Tool Definitions      |
|  |   (`cancelOrderFunction`).                                   |
|  | - Starts Polling `run` status from OpenAI.                   |
|  +----------+----------+                                         |
|             |                                                    |
|             |  OpenAI API Call: POST /threads/{id}/messages      |
|             |  OpenAI API Call: POST /threads/{id}/runs          |
|             v                                                    |
|  +---------------------+                                         |
|  |    4. OpenAI API    |                                         |
|  | (Assistant, Threads, Runs) |                                   |
|  |---------------------|                                         |
|  | - Processes message.                                          |
|  | - **Identifies Intent:** "Cancel Order".                      |
|  | - **Selects Tool:** `cancel_order` with `orderId: 'CM-7890'`.|
|  | - Sets `Run` Status to `requires_action`.                    |
|  +----------+----------+                                         |
|             ^                                                    |
|             |  OpenAI API Call: GET /threads/{id}/runs/{run_id}  |
|             |  (Returns `runStatus: "requires_action"`, `tool_calls`) |
|             |  (from `aiService.js` polling)                     |
|  +----------+----------+                                         |
|  |   3. AI Service     |                                         |
|  |     (`aiService.js`) |                                         |
|  |---------------------|                                         |
|  | - Detects `requires_action`.                                 |
|  | - Parses `tool_call` (`cancel_order`, `CM-7890`).             |
|  +----------+----------+                                         |
|             |                                                    |
|             |  Calls `orderService.cancelOrder('CM-7890')`      |
|             v                                                    |
|  +---------------------+                                         |
|  |   5. Order Service  |                                         |
|  |    (`orderService.js`) |                                        |
|  |---------------------|                                         |
|  | - Performs validation (e.g., `getOrderById`).                |
|  | - Executes business logic to update database.                 |
|  +----------+----------+                                         |
|             |                                                    |
|             |  Database Interaction (e.g., Update status in DynamoDB) |
|             v                                                    |
|  +---------------------+                                         |
|  |  6. Database (DynamoDB) |                                       |
|  |---------------------|                                         |
|  | - Order status for 'CM-7890' changed to 'canceled'.         |
|  +----------+----------+                                         |
|             ^                                                    |
|             |  Returns Action Result (e.g., "Order canceled successfully") |
|             |  (from `orderService.js`)                          |
|  +----------+----------+                                         |
|  |   3. AI Service     |                                         |
|  |     (`aiService.js`) |                                         |
|  |---------------------|                                         |
|  | - Submits Tool Output back to OpenAI.                         |
|  +----------+----------+                                         |
|             |                                                    |
|             |  OpenAI API Call: POST /threads/{id}/runs/{run_id}/submit_tool_outputs |
|             v                                                    |
|  +---------------------+                                         |
|  |    4. OpenAI API    |                                         |
|  | (Assistant, Threads, Runs) |                                   |
|  |---------------------|                                         |
|  | - Receives tool output.                                       |
|  | - Generates final natural language response based on outcome. |
|  +----------+----------+                                         |
|             ^                                                    |
|             |  OpenAI API Call: GET /threads/{id}/messages       |
|             |  (from `aiService.js` polling)                     |
|  +----------+----------+                                         |
|  |   3. AI Service     |                                         |
|  |     (`aiService.js`) |                                         |
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
|  | `aiController.js` (responds to frontend) |                     |
|  +----------+----------+                                         |
|             |                                                    |
|             |  HTTP 200 OK: {response: "Your order CM-7890..."}  |
|             v                                                    |
|  +---------------------+                                         |
|  |     1. Frontend     |                                         |
|  | (CustomerSupportPage.jsx) |                                       |
|  |---------------------|                                         |
|  | - Displays AI's confirmation message to user.                 |
|  +---------------------+                                         |
|                                                                  |
+------------------------------------------------------------------+
