# 🍔 AI Restaurant Assistant: Enterprise RAG Automation

AI Customer Support & Knowledge Base RAG Bot for Restaurants
A production-ready AI automation system utilizing RAG (Retrieval Augmented Generation) to provide multilingual customer support via WhatsApp, featuring high-availability model redundancy and automated knowledge synchronization from Google Drive.


## 🌟 Project Overview
This is not a basic chatbot. It is a **Retrieval-Augmented Generation (RAG)** system. Instead of relying on the AI's general knowledge, the bot queries a specialized vector database containing the restaurant's actual menu, pricing, and policies, ensuring **zero hallucinations** and 100% factual accuracy.

### Key Business Value:
- **24/7 Automation:** Handles FAQs, menu queries, and location details without human intervention.
- **Zero-Code Management:** The restaurant owner simply edits a Google Doc to update the bot's knowledge.
- **Multilingual Support:** Native support for English, Bangla, and Banglish.
- **High Availability:** Built-in redundancy to ensure the bot never goes offline.

---

## 🏗️ Technical Architecture

The system is composed of 5 interconnected n8n workflows:

### 1. Knowledge Ingestion (The Bridge)
- **Trigger:** Google Drive (Polls for file updates).
- **Process:** Extracts text $\rightarrow$ Generates embeddings via **Hugging Face (`all-MiniLM-L6-v2`)** $\rightarrow$ Stores in **Supabase Vector Store**.
- **Outcome:** Ensures the AI always has the latest menu and pricing.

### 2. Main AI Bot (The Brain)
- **Trigger:** WhatsApp Cloud API (Webhook).
- **Logic:** 
  - **Filtering:** Discards "message read" receipts to save API costs.
  - **Redundancy:** Uses **Groq (Llama 3.3)** as primary and **Google Gemini** as a fallback.
  - **Memory:** Uses a Window Buffer to remember the customer's name and context.
  - **RAG Tool:** Performs a similarity search in Supabase to find the exact answer.
- **Outcome:** A natural, conversational experience on WhatsApp.

### 3. Error Handling (The Safety Net)
- **Trigger:** Any node failure in the main workflow.
- **Action:** Sends an immediate alert to the Admin via **Telegram**.
- **Outcome:** Zero downtime; the developer is notified of issues before the client notices.

### 4. Daily Summary (The Manager)
- **Trigger:** Scheduled (Every midnight).
- **Action:** Aggregates total chats, fallback errors, and system events from Supabase.
- **Outcome:** Provides the business owner with a professional daily performance report on Telegram.

### 5. Drive Reconciliation (The Janitor)
- **Trigger:** Scheduled (Hourly).
- **Action:** Compares Google Drive files with Supabase records.
- **Outcome:** Automatically "soft-deletes" data from the AI's memory if the corresponding file is removed from Drive.

---

## 🛠️ Tech Stack

| Component | Technology | Purpose |
| :--- | :--- | :--- |
| **Orchestration** | n8n | Workflow automation and API glue |
| **Channel** | WhatsApp Cloud API | Customer-facing interface |
| **LLMs** | Groq (Llama 3.3) / Gemini | Natural language understanding & generation |
| **Embeddings** | Hugging Face | Turning text into mathematical vectors |
| **Vector DB** | Supabase (pgvector) | High-speed semantic search |
| **CMS** | Google Drive | Easy knowledge management for the client |
| **Admin Alerts** | Telegram Bot API | Real-time system monitoring |

---

## 🚀 Implementation Highlights (Engineering Challenges Solved)

- **Dimensionality Alignment:** Solved complex vector mismatches by aligning Hugging Face (384-dim) with Supabase `vector(384)` and the search function.
- **Asynchronous Response:** Implemented a `Respond to Webhook` strategy to prevent Meta API timeouts.
- **JSON Formatting:** Solved the "Bad Request" issues by sanitizing AI newlines using Regex replacements.
- **Strict Prompting:** Developed a "Prohibited" prompt structure to prevent the AI from "data-dumping" the entire menu during simple greetings.
- **Race Condition Prevention:** Built a sequential fallback chain (Groq $\rightarrow$ Gemini) to ensure only one response is sent per message.

---

## 📈 Results
- **Response Time:** < 2 seconds.
- **Accuracy:** 100% grounded in provided knowledge.
- **Maintenance:** 0 manual updates required to the bot's logic after deployment.
