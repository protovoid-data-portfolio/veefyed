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
    - Image downloading - [Saves images to local images/ folder with sanitized filenames]
    - Resource monitoring (optional) - [Can log CPU and memory usage periodically] 
    - Result storage - [Saves final results as qudo_moisturizers_full.csv]

## Dependencies:

- Python 3.8+
- requests
- beautifulsoup4
- pandas
- tqdm (optional for progress tracking)
- psutil (optional for monitoring)
- concurrent.futures (built-in)
- threading, signal, os, json, re, urllib.parse

## Usage Example:
> python3 scraper.py

- Resumes automatically if processed.json exists.
- Save interval: every 5 products (configurable via SAVE_INTERVAL).
- Images saved to images/ folder.
- CSV saved as qudo_products.csv.


## Folder Structure:
<pre>
qudo_scraper/
├── flow_basic.txt         # basic flow
├── flow_states.txt        # execution flow 
├── flow-diagram.png       # The execution flow of the program
├── scraper.py             # main scraper script
├── processed-30_products.json         # saved processed 30 products
├── processed-boarderline.json         # saved processed products that needed special attention
├── qudo_products-30.csv               # final CSV for 30 products
├── qudo_products-boarderline.csv      # final CSV for 5 products that needed special attention
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

    Extracted capacity and ingredients from `<p>` tags between `Product contains:` and `Product effects:`

5. *Non-ASCII Characters*

    **Issue**: Product names, Brand and ingredients contain characters like â€“, â€™, â‰ˆ, which appear due to mis-decoding of UTF-8 as Windows-1252.

    **Solution**: Added a sanitize_text function to normalize these characters and replace them with proper Unicode equivalents.

6. *Periodic Saving*

    **Issue**: Scraper could fail mid-run due to network issues or site changes.

    **Solution**: Implemented periodic saving of the processed set every SAVE_INTERVAL products.
