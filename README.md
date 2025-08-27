# FT.com Automated Analysis Pipeline

This project is an advanced, end-to-end data pipeline designed to overcome the robust security and paywall measures of the Financial Times (`ft.com`). It automates the entire process from data extraction to intelligence reporting, culminating in an AI-generated PowerPoint presentation that summarizes key themes from the day's articles.

This solution was developed as a proof-of-concept to demonstrate advanced web scraping techniques, resilience against anti-bot systems, and the integration of Large Language Models (LLMs) for automated analysis.

## Features

-   ✅ **Advanced Paywall & Anti-Bot Bypass:** Utilizes a sophisticated multi-layered approach, with **FlareSolverr** as the core engine to defeat Cloudflare's JavaScript challenges and other advanced bot detection measures.
-   ✅ **Resilient Content Extraction:** Implements a fallback strategy using **Archive.today** (`archive.ph`) to find and parse the latest available snapshot of an article if direct access fails.
-   ✅ **Automated Link Discovery:** Uses a headless browser (Playwright) to navigate the dynamic main page of `ft.com`, handle cookie banners, and gather a comprehensive list of the latest article URLs.
-   ✅ **AI-Powered Thematic Analysis:** Leverages the **Google Gemini API** to analyze the content of the scraped articles, identify dominant themes, and generate concise summaries and key findings.
-   ✅ **Automated Presentation Generation:** Automatically creates a professional, well-designed **PowerPoint (`.pptx`) presentation** from the AI-generated analysis, complete with a title slide, themed content slides, and references.
-   ✅ **Concurrent & Efficient:** Processes articles concurrently to speed up the scraping process.

## System Architecture

The pipeline operates in three distinct phases:

### Phase 1: Link Discovery (Playwright)
- A headless browser is launched to render the JavaScript-heavy `ft.com` homepage.
- It programmatically handles the cookie consent banner.
- It scrapes all unique article URLs from the main page and passes them to the next phase.

### Phase 2: Content Extraction (FlareSolverr & Archive.today)
- For each URL, the script sends a request to a local **FlareSolverr** instance.
- FlareSolverr acts as a proxy, using its own headless browser to solve any Cloudflare challenges and render the page.
- The script uses a robust fallback mechanism, querying **Archive.today** to find a clean, un-paywalled version of the article.
- An intelligent parser extracts the latest snapshot link from the archive's intermediate page and fetches the final content.

### Phase 3: AI Analysis & Reporting (Gemini & python-pptx)
- The successfully scraped article content is compiled and sent to the **Google Gemini API**.
- A detailed prompt instructs the LLM to perform thematic analysis and return a structured JSON object.
- The script parses this JSON and uses the `python-pptx` library to automatically build a professional PowerPoint presentation.

## Setup and Installation

Follow these steps to set up and run the project on your local machine.

### Prerequisites
-   [Python 3.9+](https://www.python.org/downloads/)
-   [Docker Desktop](https://www.docker.com/products/docker-desktop/)

### Step 1: Clone the Repository
Clone this project to your local machine:
```bash
git clone https://your-github-repository-url.git
cd your-project-directory
```

### Step 2: Set Up and Run FlareSolverr
This project depends on FlareSolverr to bypass Cloudflare. The easiest way to run it is with Docker.

1.  **Pull the FlareSolverr image:**
    ```bash
    docker pull ghcr.io/flaresolverr/flaresolverr:latest
    ```

2.  **Run the FlareSolverr container:**
    ```bash
    docker run -d --name flaresolverr -p 8191:8191 -e LOG_LEVEL=info ghcr.io/flaresolverr/flaresolverr:latest
    ```
    This command starts FlareSolverr in the background and makes it available at `http://localhost:8191`.

3.  **Verify it's running:** Open a web browser and navigate to `http://localhost:8191`. You should see a JSON response confirming FlareSolverr is running.

### Step 3: Install Python Dependencies
Create a `requirements.txt` file with the following content:

```text
requests
beautifulsoup4
playwright
google-generativeai
python-pptx
```

Then, install all the required libraries using pip:
```bash
pip install -r requirements.txt
```
After installation, you need to install the browser binaries for Playwright:
```bash
playwright install
```

### Step 4: Configure the Script
Open the `Scraper.py` file and edit the `CONFIG` dictionary at the top:

1.  **Set Your Gemini API Key:**
    Replace `"Put_your_key_here"` with your actual Google Gemini API key. You can get a free key from [Google AI Studio](https://aistudio.google.com/app/apikey).
    ```python
    "GEMINI_API_KEY": "AIzaSy...your...real...key...",
    ```

2.  **Set the Output Path (Optional):**
    By default, the presentation is saved to your user's Documents folder. You can change this to any path on your system.
    ```python
    "OUTPUT_FILE": "path/to/your/FT_Analysis_Presentation.pptx",
    ```

## How to Run
Once FlareSolverr is running in Docker and the script is configured, execute the script from your terminal:

```bash
python Scraper.py
```

The script will proceed through the three phases and, upon completion, you will find two output files:
-   `bypassed_ft_articles.json`: A JSON file containing all the successfully scraped raw data.
-   `FT_Analysis_Presentation.pptx`: The final, AI-generated PowerPoint presentation.

## Scheduling for Daily Ingestion
To meet the requirement for daily ingestion, this script can be scheduled using:
-   **Cron:** On Linux or macOS, a cron job can be set up to run the script at a specific time every day.
-   **GitHub Actions:** A workflow can be configured to run the script on a schedule, which is ideal for a cloud-native, automated solution.
