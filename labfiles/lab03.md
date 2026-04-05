# Lab 03: Explore MCP Tools and Integrate with GitHub Copilot

### Estimated Duration: 1 Hour

## Exercise Overview

So far you have interacted with the Contoso Burgers agent exclusively through the web chat interface. But one of the most powerful aspects of the Model Context Protocol is that the same MCP server can be consumed by **any MCP-compatible client** — not just the web app.

In this lab, you will connect directly to the live Burger MCP Server using the **MCP Inspector**, a browser-based tool that lets you call MCP tools manually without any AI in the middle. You will then integrate the same MCP server into **GitHub Copilot in VS Code**, enabling you to place burger orders and query the menu directly from your IDE using natural language in the Copilot chat sidebar.

## Exercise Objectives

- **Task 1:** Explore and test all 9 MCP tools using the MCP Inspector
- **Task 2:** Connect the Burger MCP Server to GitHub Copilot in VS Code and interact with it via Copilot Agent Mode

---

## Task 1: Explore MCP Tools Using the MCP Inspector

The **MCP Inspector** is an official developer tool from the MCP project that gives you a direct window into any MCP server. You connect it to your server's endpoint, it fetches the list of available tools, and you can invoke each one with custom inputs and see the raw response — no LLM, no chat interface, just pure MCP.

This is valuable for two reasons: it lets you verify your MCP server is working correctly independent of the AI layer, and it helps you understand exactly what data each tool returns before the agent formats it into a response.

> **The 9 tools exposed by the Burger MCP Server:**
>
> | Tool | What it does |
> |---|---|
> | `get_burgers` | Returns the full burger menu |
> | `get_burger_by_id` | Returns details for a specific burger |
> | `get_toppings` | Returns all available toppings |
> | `get_topping_by_id` | Returns details for a specific topping |
> | `get_topping_categories` | Returns topping category groupings |
> | `get_orders` | Returns all orders for the current user |
> | `get_order_by_id` | Returns details for a specific order |
> | `place_order` | Places a new burger order |
> | `delete_order_by_id` | Cancels a pending order |

### Steps

1. Open **Azure Cloud Shell** in the Azure Portal if it is not already open.

2. From the project root directory, start the MCP Inspector using `npx`:

   ```bash
   npx -y @modelcontextprotocol/inspector
   ```

   The Inspector will start and print a URL to the terminal — something like:

   ```
   MCP Inspector running at http://127.0.0.1:6274
   ```

   ![MCP Inspector start](../Screenshots/Exercise-03/ex03-task01-step02.png)

   > **Cloud Shell note:** Since Cloud Shell runs in a browser, you cannot open `localhost` URLs directly. Click the **Open in new tab** icon that Cloud Shell shows next to the URL, or use the **Web Preview** feature (port 6274) to open the Inspector in your browser.

3. The MCP Inspector opens in your browser. You will see a connection panel on the left. Configure the connection as follows:

   | Field | Value |
   |---|---|
   | **Transport Type** | Streamable HTTP |
   | **URL** | `https://<your-burger-mcp-url>.azurewebsites.net/mcp` |

   Then click **Connect**.

   ![MCP Inspector connection](../Screenshots/Exercise-03/ex03-task01-step03.png)

   > **Your Burger MCP URL** is in your `.env` file as `BURGER_MCP_URL`. You can also retrieve it quickly with:
   > ```bash
   > azd env get-values | grep BURGER_MCP_URL
   > ```

4. Once connected, click the **Tools** tab. You should see all 9 tools listed with their names and descriptions.

   ![Tools list](../Screenshots/Exercise-03/ex03-task01-step04.png)

   > This list is what the LangChain.js agent receives at runtime when it initializes. The agent loads these tool definitions and uses them to decide which tool to call based on your message.

5. Click on **`get_burgers`** and then click **Run Tool**. You will see the raw JSON response containing all the burgers in your menu — the same data the chat agent uses when you ask "what's on the menu?".

   ![get_burgers result](../Screenshots/Exercise-03/ex03-task01-step05.png)

6. From the `get_burgers` response, **copy the `id` field** of any burger. You will need it for the next step.

7. Click on **`get_burger_by_id`**. In the input field, paste the burger ID you copied and click **Run Tool**. You should get the full details for that specific burger.

   ![get_burger_by_id result](../Screenshots/Exercise-03/ex03-task01-step06.png)

8. Now test **`place_order`**. This tool requires two inputs:

   | Input | Value |
   |---|---|
   | `userId` | Any string identifier, e.g. `test-user-01` |
   | `burgers` | An array with the burger ID from step 6, e.g. `[{"burgerId": "<id>", "quantity": 1}]` |

   Fill in the inputs and click **Run Tool**.

   ![place_order input](../Screenshots/Exercise-03/ex03-task01-step07.png)

9. You should receive a response confirming the order was placed, including an `orderId`. Switch to your **Burger Web App** tab — the order should appear on the live dashboard.

   ![Order on dashboard](../Screenshots/Exercise-03/ex03-task01-step08.png)

10. Copy the `orderId` from the `place_order` response. Click on **`get_orders`**, run it with the same `userId` you used, and confirm your order appears in the list.

    ![get_orders result](../Screenshots/Exercise-03/ex03-task01-step09.png)

11. Finally, test **`delete_order_by_id`** using the `orderId` and `userId` from above. Run the tool and confirm the order is cancelled.

    ![delete_order_by_id result](../Screenshots/Exercise-03/ex03-task01-step10.png)

    > **What you have just validated:** The Burger MCP Server is fully functional and independent of the AI layer. Every tool works correctly when called directly. This is exactly the foundation any MCP-compatible client needs to consume your server — including GitHub Copilot, which you will set up next.

<validation step="validate-mcp-inspector" />

> **Congratulations** on completing Task 1! You have tested all 9 MCP tools directly via the Inspector. In the next task, you will wire this same server into GitHub Copilot.

---

## Task 2: Integrate the Burger MCP Server with GitHub Copilot in VS Code

GitHub Copilot supports MCP servers natively in **Agent Mode**. By adding a simple configuration file to your project, Copilot will discover the Burger MCP Server's tools and make them available in the Copilot chat sidebar — meaning you can order burgers, check the menu, and view orders directly from VS Code using natural language, without ever opening the web app.

> **How does Copilot use MCP?** When you activate a connected MCP server in Copilot Agent Mode, Copilot fetches the tool list from the server at runtime — exactly the same way the LangChain.js agent does. The same 9 tools, the same server, the same responses. The only difference is that Copilot's own reasoning model decides when and how to call them.

### Steps

1. Open **VS Code** on your lab virtual machine. Open the `mcp-agent-langchainjs` project folder — use **File → Open Folder** and navigate to where you cloned the repository.

   ![Open project in VS Code](../Screenshots/Exercise-03/ex03-task02-step01.png)

2. In the VS Code **Explorer** panel, check if a `.vscode` folder already exists at the project root. If it does not, create it:

   - Right-click on the project root in the Explorer panel
   - Select **New Folder**
   - Name it `.vscode`

   ![Create .vscode folder](../Screenshots/Exercise-03/ex03-task02-step02.png)

3. Inside the `.vscode` folder, create a new file named `mcp.json`:

   - Right-click on the `.vscode` folder
   - Select **New File**
   - Name it `mcp.json`

   ![Create mcp.json](../Screenshots/Exercise-03/ex03-task02-step03.png)

4. Paste the following configuration into `mcp.json`, replacing the URL with your live Burger MCP Server endpoint:

   ```jsonc
   {
     "servers": {
       "burger-mcp": {
         "type": "http",
         "url": "https://<your-burger-mcp-url>.azurewebsites.net/mcp"
       }
     }
   }
   ```

   Save the file with **Ctrl+S**.

   ![mcp.json configuration](../Screenshots/Exercise-03/ex03-task02-step04.png)

   > **Your Burger MCP URL** is available in your `.env` file as `BURGER_MCP_URL`, or retrieve it from Cloud Shell:
   > ```bash
   > azd env get-values | grep BURGER_MCP_URL
   > ```

5. Once saved, VS Code will detect the MCP configuration. A **Start** button or a notification will appear at the top of the `mcp.json` file prompting you to activate the server. Click **Start**.

   ![Start MCP server in VS Code](../Screenshots/Exercise-03/ex03-task02-step05.png)

   > If you do not see the Start button, open the VS Code **Command Palette** (`Ctrl+Shift+P`) and run **MCP: List Servers** to trigger detection.

6. Open the **GitHub Copilot Chat** panel from the VS Code sidebar (the chat bubble icon). At the top of the chat input, switch to **Agent Mode** by clicking the mode selector and choosing **Agent**.

   ![Switch to Agent Mode](../Screenshots/Exercise-03/ex03-task02-step06.png)

7. In the chat input, click the **Tools** icon (plug/tool icon) to see the list of available tools. You should see **burger-mcp** listed with all 9 tools underneath it. Ensure it is enabled.

   ![Verify tools in Copilot](../Screenshots/Exercise-03/ex03-task02-step07.png)

8. Now test the integration. Type the following in the Copilot chat and press **Enter**:

   *"What burgers do you have on the menu?"*

   Copilot will call the `get_burgers` tool and return the menu directly in your IDE.

   ![Copilot menu response](../Screenshots/Exercise-03/ex03-task02-step08.png)

9. Place an order through Copilot:

   *"Order one burger for me — surprise me with any burger you think is good"*

   Watch Copilot decide which burger to pick, call `get_burgers` to choose one, then call `place_order` with your user context.

   ![Copilot place order](../Screenshots/Exercise-03/ex03-task02-step09.png)

   > **Tip:** If Copilot does not call the burger-mcp tools automatically, add `#burger-mcp` to your prompt to explicitly reference the server: *"Using #burger-mcp, what burgers do you have?"*

10. Switch back to your **Burger Web App** browser tab. The order placed through Copilot should appear on the live orders dashboard — confirming that the same MCP server powers both the web agent and your IDE assistant.

    ![Order from Copilot on dashboard](../Screenshots/Exercise-03/ex03-task02-step10.png)

11. Check your order history through Copilot:

    *"Show me my recent orders"*

    ![Copilot order history](../Screenshots/Exercise-03/ex03-task02-step11.png)

<validation step="validate-copilot-mcp-integration" />

> **Congratulations** on completing Exercise 3! You have tested MCP tools directly via the Inspector and integrated the same server into GitHub Copilot — demonstrating that one MCP server can power multiple AI clients simultaneously, with no changes to the server itself.

---

## Summary

In this exercise, you:

- Connected the **MCP Inspector** to your live Burger MCP Server and manually invoked all 9 tools — verifying the server works independently of any AI client
- Placed a test order and cancelled it directly through the Inspector, observing the raw MCP request and response format
- Created a `.vscode/mcp.json` configuration file to connect the Burger MCP Server to **GitHub Copilot in VS Code**
- Interacted with the agent from inside VS Code using Copilot Agent Mode — browsing the menu, placing an order, and confirming it appeared on the live Burger Web App dashboard

Click **Next** to proceed to Exercise 4, where you will extend the Burger MCP Server by building and deploying a brand new custom MCP tool.

![Next page](../Screenshots/Exercise-03/ex03-next.png)