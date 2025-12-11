# Veefyed

# Project Name: QudoBeauty Moisturizer Scraper

## Purpose:

Scrapes moisturizer products from qudobeauty.com, including product metadata, ingredients, capacity, and images. Handles pagination, multi-threading, interruptions, and can resume scraping from where it left off.

## Features:

- <b>Multi-threaded scraping</b>: Uses ThreadPoolExecutor for concurrent product page scraping.

- <b>Thread-safe processed tracking</b>: Maintains a processed.json to avoid duplicate processing.

- <b>Graceful interrupt handling</b>: User can stop execution (CTRL+C) and resume later.

- <b>Pagination support</b>: Automatically collects product URLs across pages.

- <b>Product details extracted:</b>
    
    - Product Name
    - Brand
    - Category
    - SKU
    - EAN
    - Capacity / Size / Quantity
    - Ingredients
    - Product Image URL
    - Product Page URL

- Enriched Data
    - Official Product Page
    - Brand Website
    - External SKU/EAN
    - Country of Origin
    - Extra Description
    - External Ingredients

## Dependencies:

- Python 3.8+
- requests
- beautifulsoup4
- pandas
- tqdm (optional for progress tracking)
- psutil (optional for monitoring)
- concurrent.futures (built-in)
- threading, signal, os, json, re, urllib.parse
- SequenceMatcher

## Usage Example:

> python3 scraper.py

- Resumes automatically if processed.json exists
- Save interval: every 5 products (configurable via SAVE_INTERVAL)
- Images saved to images/ folder
- Enriches data using Google custom search (CSE)
- CSV saved as qudo_products.csv


## Folder Structure:
<pre>
qudo_scraper/
├── scraper.py             # main scraper script
├── requirements.txt       # required packages for this scraper
├── README.md              # The readme file, containing the documentation
└── images/                # downloaded product images
</pre>


## Q & A

1. *API Usage*

    **Google Custom Search API (CSE):**
    
    Google's custom Search API was used along with dynamic query, built using already scrapped information
    The enrichment module has the capability to build multiple search queries BUT a single one was used due to free tier limit (100 requests per day)
    - query: `"{product_name} {brand} official ingredients"`

    Each query is sent using:

    <pre>params = {
        "key": GOOGLE_API_KEY,
        "cx": GOOGLE_CX,
        "q": query,
        "num": GOOGLE_NUM_RESULTS
    }</pre>

    > response = requests.get(GOOGLE_SEARCH_URL, params=params)

    <br/>

    **How responses were used**

    For each API response:

    - Collect all search results (items)
    - Compute a “relevance score” for every result
    - Sort results by score (highest first)
    - Loop through results and enrich only the missing fields


2. *Data Validation Logic*
    - Input Validation (during scraping)

        - HTML elements checked before use.
        Example: if `<div id="tab-description">` does not exist, skip gracefully
        - Capacity extraction validated by regex. 
        Only values matching patterns like 125ml, 40g, 2 pcs, etc., are accepted
        - Ingredients captured only from the correct segment
        Between `<p>Product contains:</p>` and `<p>Product effects:</p>`

    - Enrichment Data Validation

        During Google enrichment:

        - Values are saved only if the field is still missing
        - SKU/EAN accepted only if numeric or explicitly mentioned in snippet
        - Ingredient links chosen only from recognized authority domains:
            - incidecoder
            - cosdna
            - skincarisma
            - paulaschoice

3. *How reliability is determined*

    Google search results are not blindly accepted.
    
    Each result is ranked by a Relevance Score that determines trustworthiness.
    
    Below Criteria was used:
    - Similarity between product name and result title
    - Similarity between product name and snippet
    - Brand match inside displayLink
    - search result containing links from `ingredient analysis websites` (incidecoder, cosdna, skincarisma, paulaschoice) are considered very reliable
    - Looping through Ranked Results to pick/populate from highest-ranked once first

