You are building a specialized web scraper for Toll Tax Data. You strictly adhere to the "State-Diff-Report" architecture. Your job is not just to scrape, but to **track changes** in pricing over time using strict state management and delta reporting.

## 1. Directory & Naming Convention (Strict)
*   **Root:** All work must occur in `scrapers/<country>_toll_<name>/`.
*   **Main Script:** `<country>_toll_<name>.py`.
*   **State File:** `<name>_state.json` (Stores hashes/timestamps).
*   **Output Dir:** `<name>_output/` (Stores `data.json`, `data.csv`).
*   **Debug:** `result.md` (Raw Markdown content).
*   **Report:** `delta_report.xlsx` (Excel file highlighting price changes).

## 2. Domain Data Logic (Pydantic)
*   **Financial Precision:** `price` fields MUST be `int`. Never `str`.
*   **Localization:** 
    *   `name_local`: KEEP stations, locations, and names exactly as they appear in local language.
    *   `name_en`: Translate to English: descriptions, notes and any additional toll information.
*   **Vehicle Classes:** Standardize classes (e.g., "Class 1 (Cars)", "Class 2 (Trucks)") in the Pydantic descriptions.

## 3. The "State-Diff-Report" Pattern
Your main execution flow must strictly follow this sequence:
1.  **Load State:** Check `<name>_state.json` for the previous run's data hash.
2.  **Crawl & Extract:** Use `crawl4ai` to get fresh data.
3.  **Validate:** Ensure data matches the Pydantic schema.
4.  **Diff:** Use `deepdiff` (or set comparison) to detect changes between `old_state` and `new_data`.
5.  **Save:** 
    *   Write `output/*.json` and `output/*.csv`.
    *   Update `*_state.json` with the new hash and `last_updated` (UTC ISO).
    *   Write `result.md` (raw crawl markdown) for debugging.
6.  **Report:** Generate `delta_report.xlsx` (using Pandas/OpenPyXL) showing *only* changed rows or a summary of stability.

## 4. LLM Instruction Specifics (Toll Domain)
*   **Anti-Hallucination:** System prompt must explicitly state: "If a price is not listed in the table, return `null`. Do not invent prices."
*   **Table Targeting:** Toll data is almost always in HTML tables. Use the `css_selector` in Crawl4AI to target specific `<table>` or `div.pricing` elements to reduce noise.
*   **List Handling:** The LLM will likely return a `list[TollLocation]`. Handle this explicitly in your Pydantic extraction.

## 5. Required Dependencies (Domain Specific)
*   `deepdiff`: For detecting changes in nested JSON structures.
*   `pandas` & `openpyxl`: For generating the Excel delta report.
*   `python-dotenv`: For loading API keys.
