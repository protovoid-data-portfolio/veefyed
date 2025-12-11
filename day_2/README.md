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


## Issues Encountered
1. *Duplicate Products*

    **Issue**: Some products could appear on multiple pages, risking duplicates.

    **Solution**: Maintained a thread-safe processed set to track products already scraped.

2. *Multi-threading & Thread Safety*

    **Issue**: Using ThreadPoolExecutor for parallel scraping could cause race conditions when updating the processed set or results list.

    **Solution**: Added locks (processed_lock, results_lock, print_lock) to ensure thread-safe operations.

3. *Interrupt Handling*

    **Issue**: Long scraping runs could be interrupted by the user (Ctrl+C), causing partial data loss.

    **Solution**: Implemented signal handling with a stop flag to gracefully exit and save progress to JSON.

4. *Product Details Extraction*

    **Issue 1**: Some data fields (Brand, Category, SKU, EAN) were inconsistently formatted across products.

    **Issue 2**: Product capacity and ingredients were buried inside HTML `<p>` tags or text sections.

    **Solution**:

    Created custom logic to parse multiple `<span>` elements inside product_meta for brand/category.

    Extracted capacity and ingredients from `<p>` tags between Product contains: and Product effects:

5. *Non-ASCII Characters*

    **Issue**: Product names, Brand and ingredients contain characters like â€“, â€™, â‰ˆ, which appear due to mis-decoding of UTF-8 as Windows-1252.

    **Solution**: Added a sanitize_text function to normalize these characters and replace them with proper Unicode equivalents.

6. *Periodic Saving*

    **Issue**: Scraper could fail mid-run due to network issues or site changes.

    **Solution**: Implemented periodic saving of the processed set every SAVE_INTERVAL products.

7. *API Usage*

    **Details**: Google's custom Search API was used along with dynamic query, built using already scrapped information
    The enrichment module has the capability to build multiple search queries BUT a single one was used due to free tier limit (100 requests per day)
    - "{product_name} {brand} official ingredients"

8. *Data Validation Logic*
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

10. *How reliability is determined*

    Google search results are not blindly accepted.
    
    Each result is ranked by a Relevance Score that determines trustworthiness.
    
    Below Criteria was used:
    - Similarity between product name and result title
    - Similarity between product name and snippet
    - Brand match inside displayLink
    - search result containing links from `ingredient analysis websites` (incidecoder, cosdna, skincarisma, paulaschoice) are considered very reliable
    - Looping through Ranked Results to pick/populate from highest-ranked once first


<br/>
<br/>
<br/>

---
# QudoBeauty Scraper (modular)

## 1. Setup

   - Create virtualenv and install requirements:
       
       python -m venv venv
       
       source venv/bin/activate
       
       pip3 install -r requirements.txt

   - create .env and fill in:
       
       GOOGLE_API_KEY, GOOGLE_CX (optional; if not provided enrichment is skipped)

   - Edit config.json if you need to change settings.

## 2. Run

   - python3 scraper.py

   The script:
     
    - resumes from processed.json if found,
     
    - saves images into images/,
     
    - writes CSV to config.json's CSV_OUTPUT (utf-8-sig).

