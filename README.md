# 🍔 Serverless AI Agent Lab (Azure)

## 📌 Overview
This lab walks you through building and deploying a **serverless AI agent** on Azure. The solution uses a language model to understand user requests and dynamically interact with backend services through structured tools, demonstrating a real-world, production-style architecture.

---

## ⚙️ What You’ll Build
- A conversational AI agent powered by Azure OpenAI  
- A serverless backend using Azure Functions  
- A tool-based interaction layer using MCP (Model Context Protocol)  
- A data layer using Cosmos DB  
- A frontend hosted on Azure Static Web Apps  

---

## 🔄 How It Works
1. The user interacts with the web application.  
2. Requests are sent to the Agent API (Function App).  
3. The agent interprets intent and decides whether to respond or call a tool.  
4. Tools are exposed via the MCP server and invoke backend APIs.  
5. Data is retrieved or stored in Cosmos DB.  
6. The response flows back to the user in real time.  

---

## 🚀 Outcome
By completing this lab, you will understand how to design and deploy a **modular, scalable AI agent system** using Azure services, and how to extend it with new capabilities.

---