# Receep: Smart Receipt Capture & Expense Logging Tool

Receep is a mobile-first application designed to simplify the process for companies to collect, process, and expense meal receipts, facilitating VAT deduction. It leverages AI (OCR and LLM capabilities) for data extraction and integrates with Google Sheets for organized record-keeping.

## Features

*   **Mobile-First Design:** Intuitive interface for capturing receipt photos on the go.
*   **AI-Powered Data Extraction:** Uses OCR (Optical Character Recognition) via OCR.space and a Large Language Model (LLM like Google Gemini) to automatically extract key information from receipts:
    *   Total Amount
    *   Total VAT
    *   VAT Rate
    *   Receipt Date
    *   Receipt Number (Reference No.)
    *   Company Name & Address
    *   Tax ID (VKN) & Tax Office
*   **Automated Google Sheets Integration:** Extracted data is automatically populated into a Google Sheet.
    *   New rows are added for each processed receipt.
    *   A new sheet (tab) is created for each month (e.g., "June 2025", "July 2025"), keeping data organized and ready for accounting.
*   **Workflow Automation:** Powered by n8n for backend processing logika and integrations.
*   **Telegram Bot Integration (Optional):** Users can submit receipts via a Telegram bot.

## Tech Stack

*   **Frontend (Conceptual - for direct photo capture):** Simple HTML, CSS, JavaScript (Vanilla JS).
*   **Backend Workflow Automation:** [n8n](https://n8n.io/)
*   **OCR Service:** [OCR.space API](https://ocr.space/)
*   **LLM for Data Structuring:** Google Gemini API (via Vertex AI) or other compatible LLM.
*   **Data Storage & Output:** [Google Sheets API](https://developers.google.com/sheets/api)
*   **Deployment (Self-Hosted n8n):** Docker, Docker Compose on a cloud VM (e.g., Oracle Cloud, Render, Railway).
*   **Reverse Proxy & SSL (for live deployment):** Caddy (recommended for ease of use and automatic HTTPS).

## Project Structure
receep/
├── docker-compose.yml # Docker Compose configuration for n8n
├── frontend/ # Simple HTML/JS frontend (if used for direct upload)
│ ├── index.html
│ ├── script.js
│ └── style.css
├── receep-n8n-workflow.json / # Exported n8n workflow JSON files
├── .gitignore
├── LICENSE
└── README.md


## Setup and Deployment (Self-Hosted n8n on a Cloud VM)

This guide assumes you are deploying n8n using Docker and Docker Compose on a Linux VM (e.g., Ubuntu).

**Prerequisites:**

1.  A Cloud VM (e.g., Oracle Cloud Free Tier, DigitalOcean, AWS EC2) with Docker and Docker Compose installed.
2.  A domain or subdomain pointed to your VM's public IP address (e.g., `n8n.yourdomain.com`).
3.  API keys for:
    *   OCR.space
    *   Google Cloud Platform (for Gemini API and Google Sheets API - ensure these APIs are enabled in your GCP project and you have a service account JSON key for Sheets).
4.  (Optional) Telegram Bot Token from BotFather if using Telegram integration.

**Steps:**

1.  **Clone the Repository:**
    ```bash
    git clone https://github.com/cemdx/receep.git
    cd receep
    ```

2.  **Configure `docker-compose.yml` (Environment Variables):**
    The provided `docker-compose.yml` is a template. Critical environment variables like API keys and `WEBHOOK_URL` should ideally be set directly on your deployment platform or via a `.env` file (ensure `.env` is in `.gitignore`). For n8n, the following are important:
    *   `GENERIC_TIMEZONE`: e.g., `Europe/Istanbul`
    *   `WEBHOOK_URL`: Your n8n's public HTTPS URL (e.g., `https://n8n.yourdomain.com/`) - This is crucial for webhooks like Telegram.
    *   `N8N_USER_FOLDER=/var/n8n_data`: Ensures data persistence.
    *   *(You will need to add your API keys as n8n credentials مديرor environment variables within n8n workflows/settings, not directly in `docker-compose.yml` for security.)*

3.  **Setup Reverse Proxy and SSL (Caddy Recommended):**
    *   Install Caddy on your VM.
    *   Create/Edit `/etc/caddy/Caddyfile`:
        ```caddyfile
        n8n.yourdomain.com {  // Replace with your actual domain
            reverse_proxy localhost:5678
        }
        ```
    *   Reload Caddy: `sudo systemctl reload caddy`. Caddy will automatically obtain an SSL certificate from Let's Encrypt.
    *   Ensure ports 80 and 443 are open in your VM's firewall (e.g., OCI Security List).

4.  **Start n8n:**
    In the `receep` directory on your VM:
    ```bash
    docker-compose up -d
    ```

5.  **Initial n8n Setup:**
    *   Access n8n via `https://n8n.yourdomain.com`.
    *   Set up your owner account.
    *   (Optional) Set `N8N_SECURE_COOKIE=false` via environment variable if you are accessing via HTTP initially (not recommended for production). This should not be needed with Caddy and HTTPS.

6.  **Import Workflows:**
    *   In the n8n UI, import the workflow JSON files located in the `n8n_workflows/` directory of this repository.

7.  **Configure Credentials in n8n:**
    *   In the n8n UI (Settings > Credentials), add new credentials for:
        *   OCR.space (API Key)
        *   Google Sheets API (using the Service Account JSON key you obtained from GCP)
        *   Google Vertex AI / Gemini (API Key or Service Account, depending on your setup)
        *   Telegram Bot (API Token, if using)
    *   Update your imported workflows to use these newly created credentials.

8.  **Setup Google Sheet:**
    *   Create a new Google Sheet.
    *   Share this Google Sheet with the email address of the Google Service Account you are using for the Sheets API, giving it "Editor" permissions.
    *   Note the Spreadsheet ID (from its URL). You'll need this in your n8n Google Sheets node.
    *   The n8n workflow is designed to create new monthly sheets automatically (e.g., "June 2025").

## Workflow Overview

*(Briefly describe how your main n8n workflow(s) function. For example:)*

1.  **Input:** Receives an image webhook (from the frontend or Telegram bot).
2.  **Image Preprocessing (Optional):** Resizes or optimizes the image if necessary (e.g., to meet OCR.space limits).
3.  **OCR:** Sends the image to OCR.space API to extract raw text.
4.  **LLM Processing:** Sends the raw text to an LLM (e.g., Gemini) with a specific prompt to:
    *   Correct OCR errors.
    *   Extract and structure data into a predefined JSON format (receipt number, date, total, VAT, etc.).
    *   Perform consistency checks (e.g., VAT amount vs. rate and total).
5.  **Data Output:** Writes the structured JSON data as a new row to the appropriate monthly sheet in Google Sheets.

## Contributing

Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License

[MIT](https://choosealicense.com/licenses/mit/)