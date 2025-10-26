# DNSpy

DNSpy is a small GUI tool for performing DNS queries and basic enumeration of common DNS record types.

[GitHub Repo](https://github.com/nicoleman0/DNSpy)

## Overview

DNSpy provides a simple Tkinter-based interface to run DNS queries (A, AAAA, CNAME, MX, NS, TXT, SOA) and view results in a scrolling output pane. It supports verbose mode, multi-threaded query execution, and error logging.

## Features

- Query common DNS record types (A, AAAA, CNAME, MX, NS, TXT, SOA)
- Toggle verbose output
- Multi-threaded enumeration to keep the UI responsive
- Error logging to dns_query_errors.log
- Small, single-file GUI entrypoint: `dnspy_gui.DNSpyGUI`

## Installation

- Ensure Python 3.x is installed.

- Get the files

```bash
git clone https://github.com/nicoleman0/DNSpy
cd DNSpy
```

- Install dependencies:

```bash
pip install dnspython
```

## Quick Start — GUI

Run the main GUI application:

```bash
python dnspy_gui.py
```

Usage flow:

1. Enter a target domain in the "Target Domain" field.
2. Check or uncheck the DNS record types to query.
3. Toggle "Verbose Output" to see per-query status.
4. Click "Run (F5)" or press F5 to start enumeration.
5. Results and errors appear in the output pane. See the implementation in `dnspy_gui.DNSpyGUI`.

## Quick Start — Programmatic API

You can call the DNS query function directly from your code:

```python
from query import query_dns

# Query A records
results = query_dns("example.com", "A")

# Query MX records using specific nameservers
results = query_dns("example.com", "MX", nameservers=["8.8.8.8", "8.8.4.4"])
```

Function implementation: see `query.query_dns` for behavior and error logging.

## Logging & Errors

DNS errors are logged to `dns_query_errors.log`. The GUI shows brief error messages in the output pane; detailed errors are written to the log file.

## Development

- GUI code: `dnspy_gui.py` — contains the Tkinter app and `DNSpyGUI` class.
- Query logic: `query.py` — contains `query_dns` and logging setup.

Contributions:

- Open a PR, include tests if applicable, and follow the MIT license in [LICENSE](LICENSE).

## Troubleshooting

- If queries time out or fail, try using explicit nameservers with `query_dns(..., nameservers=[...])`.
- Ensure network/DNS access is available from the host running the tool.
