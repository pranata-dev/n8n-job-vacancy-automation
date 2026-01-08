# Automated Job Vacancy Tracker

An intelligent automation pipeline designed to streamline the job application tracking process using n8n, AI agents, and Telegram.

## The Story

As a final-year Physics student actively applying for internships, I realized that managing applications is a repetitive and time-consuming task. Manually copying data from job portals to a spreadsheet takes time away from what matters: preparing for interviews and refining skills.

To solve this, I engineered a "Human-in-the-Loop" automation system. Instead of manually typing data, I simply forward a job URL to my private Telegram bot. The system handles the rest—scraping the website, analyzing the content using LLMs, checking for duplicates, and structuring the data into my database.

This project demonstrates my ability to build full-cycle automation, handle API integrations, manage databases, and deploy containerized applications using Docker.

## System Architecture

The system is built on an event-driven architecture. It acts as a middleware between raw web data and a structured Google Sheets database.

### Logic Flow
1.  **Input**: User sends a text message to the Telegram Bot.
2.  **Classification**: The system analyzes if the message is a conversational greeting or a job URL.
3.  **Validation (Duplicate Check)**: Before processing, the system queries the Google Sheet to ensure this specific URL has not been processed before. This prevents data redundancy.
4.  **Data Acquisition**:
    * The system uses Jina AI (Reader API) to scrape the raw text content from the URL.
5.  **Smart Decision Making (Fallback Logic)**:
    * **Scenario A (Clean Data)**: If the text is readable, an AI Agent (LLM) parses the unstructured text into a strict JSON schema containing the Company Name, Position, Deadline, and Requirements.
    * **Scenario B (CAPTCHA Detected)**: If the scraper encounters a CAPTCHA or blocking mechanism, the system triggers a fallback routine, instructing the user to input the data manually via a structured format.
6.  **Storage**: Validated and parsed data is appended to Google Sheets.
7.  **Feedback**: The user receives a confirmation message with a direct link to the updated database.

```mermaid
graph TD
    A[Start: Telegram Trigger] --> B{Is Greeting?}
    B -- Yes --> C[Reply: Usage Instruction]
    B -- No --> D[Check Google Sheets for Duplicate]
    
    D --> E{Link Exists?}
    E -- Yes --> F[Stop: Send Duplicate Warning]
    E -- No --> G[HTTP Request: Jina AI Scraper]
    
    G --> H{Contains CAPTCHA?}
    H -- Yes --> I[Fallback: Request Manual Input]
    H -- No --> J[AI Agent Extraction]
    
    J --> K[Parse to JSON]
    K --> L[Append to Google Sheets]
    L --> M[Reply: Success & Sheet URL]
