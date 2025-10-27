# VTScan --- VirusTotal URL Scanner

*A command-line tool to scan URLs via the VirusTotal API and generate reports.*

[GitHub Repo](github.com/nicoleman0/VTScan)

## Overview

**VTScan** is a Python-built CLI tool designed with the express intent to simplify bulk URL scanning. It uses the VirusTotal API (v3), and after scanning it produces both a spreadsheet (Excel) and an HTML report.

Key features include:

- Scans one or more URLs in a single run.
- Accepts a VirusTotal API key either via command-line option or interactive prompt.
- Outputs a results workbook (`results.xlsx`) and a human-readable HTML report (`report.html`) using Jinja2.
- Displays progress updates, handles errors gracefully, and supports bulk workflows.

This tool is suited for analysts, SOC engineers or GRC practitioners who need to screen lists of URLs for malicious or suspicious verdicts in a repeatable way. It speeds up the process and allows for time to be spent on more productive matters.

## Prerequisites

In order to use VTScan as intended, you will need:

- Python 3.7 or higher (recommended)
- A valid VirusTotal API v3 key (free or paid tier)
  - Note: If using a free (public) key, there are rate limits.
- Install the required Python packages:
  
  ```bash
  pip install typer typing_extensions vt pandas jinja2
  ```

(Alternatively install via `requirements.txt`.) ([GitHub](https://github.com/nicoleman0/VTScan "GitHub - nicoleman0/VTScan: VirusTotal API URL Scanner"))

## Installation

1.  Clone or download the VTScan repository:

    ```bash
    git clone https://github.com/nicoleman0/VTScan.git
    cd VTScan
    ```

2.  (Optional) Create a virtual environment and activate it:

    ```bash
    python3 -m venv .venv
    source .venv/bin/activate
    ```

3.  Install dependencies via pip:

    ```bash
    pip install -r requirements.txt
    ```
    or install individually as shown above.

## Usage

### Basic invocation

```bash
python3 main.py <url1> <url2> ... [options]
```

For example:

```bash
python3 main.py https://example.com http://malicious-site.test
```

This will prompt for your API key (if not given via option) and then scan the URLs, producing `results.xlsx` and `report.html`. ([GitHub](https://github.com/nicoleman0/VTScan "GitHub - nicoleman0/VTScan: VirusTotal API URL Scanner"))

### View help / options

```bash
python3 main.py --help
```

This will list available flags/options, input formats, and output customizations.

### Options

Below are commonly supported options (based on the code structure; adjust if you add/modify):

-   `-k`, `--apikey` : Provide your VirusTotal API key via command line.

-   `-o`, `--output` : Specify the output directory or base filename (if supported).

-   `--urls-file` : (If implemented) Provide a file which contains a list of URLs to scan, one per line.

-   `--template` : (If configurable) Specify a custom HTML template for the report.

-   Rate-limit / delay options: If the tool supports automatically pausing between requests to avoid hitting API quota.

-   Error handling flags: e.g., skip on error, retry logic, etc.

*Note*: Because the repo is relatively small, you may want to review `main.py`, `scan.py`, `generate.py` for exact available options.

## Internals / Workflow

Here's a high-level overview of how VTScan works internally:

1.  **URL ingestion** -- The tool parses command-line URLs (and/or a file list) and filters/normalises them.

2.  **VirusTotal API interaction** -- Using the `vt` Python client library, it sends each URL to VirusTotal's URL analysis endpoint, retrieves the report.

    -   It must handle API key authentication, request rate limiting, and error states (e.g., "queued" status).

    -   Refer to official docs for API v3 usage. ([VirusTotal](https://docs.virustotal.com/docs/api-scripts-and-client-libraries?utm_source=chatgpt.com "API Scripts and client libraries"))

3.  **Data collection** -- For each URL the tool extracts key fields/allows summarised verdicts (such as malicious_count, harmless_count, last_analysis_date, etc.).

4.  **Workbook generation** -- Leveraging `pandas`, the results are consolidated into an Excel (.xlsx) workbook (`results.xlsx`).

5.  **HTML report generation** -- Via `jinja2` templating, the tool uses `template.html` to render a human-readable HTML report summarising the results (e.g., number of malicious URLs, top verdicts, links to full reports).

6.  **Output** -- The generated files are placed in the working directory or specified output directory; progress is printed to console.

7.  **Post-processing / workbook utilities** -- The `workbook.py` may include helper functions to inspect or filter the workbook after generation (e.g., pivoting results, adding summary sheets).

### File description

-   `main.py` -- The entry point CLI logic (argument parsing, overall workflow)

-   `scan.py` -- Contains logic to perform individual URL scans (API calls, error handling)

-   `generate.py` -- Logic to generate outputs: Excel workbook, HTML report

-   `workbook.py` -- Helper module for workbook manipulation/summary generation

-   `template.html` -- The HTML template for the HTML report

-   `requirements.txt` -- Python dependencies list

-   `LICENSE.txt` -- MIT license text

-   `.gitignore` -- Typical Python/IDE ignore file

-   `README.md` -- Basic overview (your current README)

## Examples

### Example 1: Single URL

```bash
python3 main.py https://safe-site.example.com
```

Output:

-   `results.xlsx` -- Excel file listing the URL and verdict (e.g., harmless)

-   `report.html` -- HTML summarising 1-URL scan.

### Example 2: Multiple URLs

```bash
python3 main.py https://site1.example.com http://suspect.test https://another.good.site
```

Output:

-   Workbook with three rows (one per URL)

-   HTML report with totals (e.g., 3 scanned, 1 flagged malicious)

### Example 3: With API key via option

```bash
python3 main.py https://example.com -k YOUR_API_KEY_HERE
```

This avoids the interactive prompt for your key and is useful for automation.

### Example 4: Batch via URL list (if supported)

If you add support (or if the tool supports) a `--urls-file urls.txt`, you might run:

```bash
python3 main.py --urls-file urls.txt -k YOUR_API_KEY_HERE
```

where `urls.txt` contains one URL per line.

## Best Practices & Notes

-   Use a dedicated VirusTotal API key and monitor your quota usage. Free keys have rate limits and automated bulk scans may exhaust them quickly. ([VirusTotal](https://docs.virustotal.com/docs/api-scripts-and-client-libraries "API Scripts and client libraries"))

-   Respect privacy and sensitive data. Please do *not* scan URLs that expose confidential information unless you have rights/consent, because once sent to VirusTotal they become part of the public dataset, viewable to anyone.

-   In an SOC/GRC context: Use VTScan as part of a workflow: e.g., export indicators from your SOC, feed them into VTScan, generate reports, filter for high-risk URLs and then escalate.

-   Consider adding logging/CSV output if you wish to integrate into SIEM or downstream workflows.

## Troubleshooting

| Issue | Possible cause & resolution |
| --- | --- |
| "Invalid API key" or "authentication failed" | Confirm you supplied the key correctly (via `-k` or prompt), check for leading/trailing spaces or newline characters. |
| "Rate limit exceeded" or very slow processing | You're hitting the free-key quota or making too many requests per minute. Add delays or upgrade your plan. |
| No output file generated | Check working directory permissions, ensure Python can write files, check if program aborted early due to error. |
| `report.html` is empty or broken template | Ensure `template.html` exists, confirm `jinja2` is installed, verify the HTML template path if you modified directory structure. |
| Workbook missing expected columns | The API may have returned fewer fields; check versions of `pandas` and `vt` library. Also inspect `workbook.py` logic for field extraction. |

## Contributing

Contributions are welcome! Here are some suggestions:

-   Add support for reading URLs from input files (`.txt`, `.csv`), or via stdin.

-   Add additional output formats: e.g., JSON, CSV, or a Markdown summary for GitHub Actions.

-   Improve the HTML template: add charts (e.g., bar chart of verdict counts) using libraries like `plotly` or `matplotlib`.

-   Add multi‐threading or asynchronous calls for faster scanning of large lists (while still respecting rate limits).

-   Add integration with a SIEM or Slack notification when malicious URLs are found.

-   Write additional tests for `scan.py`, `generate.py` and improve error-handling coverage.

If you submit a pull request:

-   Fork the repository, create a feature branch, write unit tests when applicable.

-   Ensure compatibility with Python 3.7+ and include any new dependencies in `requirements.txt`.

## Versioning & Releases

This repository uses semantic versioning (e.g., v1.0.2 is one of the releases).

For major updates (breaking changes), increment the major version; for backwards‐compatible enhancement, increment minor version; for bug fixes, increment patch version.

## License

VTScan is licensed under the MIT License. See the `LICENSE.txt` file for full text. ([GitHub](https://github.com/nicoleman0/VTScan "GitHub - nicoleman0/VTScan: VirusTotal API URL Scanner"))

## Acknowledgments

-   The `vt` Python client library (used to interact with VirusTotal's API)

-   The developers of the VirusTotal API and community contributions documented in their Developer Hub. ([VirusTotal](https://docs.virustotal.com/docs/api-scripts-and-client-libraries "API Scripts and client libraries"))

-   Myself. For creating the tool.

* * * * *
