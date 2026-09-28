# XKCD → EPUB Downloader

A small desktop app (Tkinter GUI) that downloads comics from [xkcd.com](https://xkcd.com) using its official public JSON API and packages them into a single `.epub` file you can read on any e-reader or e-reader app.

## What it does

- Asks you for a comic number range (or grabs everything up to the latest comic automatically).
- Downloads each comic's metadata (title, alt text, date) and image in parallel, using a thread pool, so it's much faster than downloading one at a time.
- Automatically retries failed requests (timeouts, rate limits, server errors).
- Skips numbers that don't exist (xkcd has a couple of gaps in its numbering).
- Builds a standard EPUB2 file from scratch (no third-party ebook library needed) with one chapter per comic — image, title, alt text, and date — plus a working table of contents.
- Shows live progress: a percentage bar and a scrolling log of each comic as it's fetched.
- Lets you cancel a run part-way through.

## Extra info

- I made this for fun so it is free.
- Feal free to do whatever you want with it.
- If you enjoyed it please star it.
- To veiw the EPUB use and EPUB vewing softwere but it is desined to be sent to a Ereader to be read.
- I have also attached a version that is compiled to windows (it's a exe) just download it and run it with no other installation needed. (It uses PyInstaller)
- Have fun.

## How it works

1. **Fetching data** — xkcd exposes a JSON endpoint for every comic at `https://xkcd.com/<num>/info.0.json`, and `https://xkcd.com/info.0.json` for the latest one. The app calls these with the `requests` library.
2. **Parallel downloads** — a `concurrent.futures.ThreadPoolExecutor` fires off several fetches at once (default 8, adjustable in the UI), since these are network-bound requests. A `requests.Session` with connection pooling and automatic retries (via `urllib3.Retry`) is reused across all of them.
3. **Progress reporting** — worker threads push status messages onto a thread-safe `queue.Queue`; the main GUI thread polls that queue every 80ms and updates the progress bar, percentage, status line, and log without blocking the interface.
4. **Building the EPUB** — an EPUB file is just a specially-structured ZIP archive. The script writes the required `mimetype`, `META-INF/container.xml`, an OPF manifest/spine, an NCX table of contents, and one XHTML chapter + image per comic, then zips it all up with Python's built-in `zipfile` module. No external ebook-writing library is required.

## Requirements

- **Python 3.8+**
- Tkinter — included with most standard Python installations. On some Linux distros you may need to install it separately, e.g.:
  ```bash
  sudo apt install python3-tk
  ```

### Pip libraries

Only two third-party packages are needed (everything else used — `tkinter`, `zipfile`, `queue`, `threading`, `concurrent.futures`, `os`, `time`, `io` — is part of the Python standard library):

```
requests
urllib3
```

`urllib3` is normally installed automatically as a dependency of `requests`, but it's listed explicitly since the script imports its `Retry` class directly.

## Setup instructions

1. **Install Python 3** if you don't already have it: https://www.python.org/downloads/

2. **Download `xkcd_to_epub.py`** to a folder on your computer.

3. **Install the dependencies:**
   ```bash
   pip install requests urllib3
   ```
   On some systems (particularly newer Debian/Ubuntu installs) you may need:
   ```bash
   pip install requests urllib3 --break-system-packages
   ```
   Or use a virtual environment instead:
   ```bash
   python3 -m venv venv
   source venv/bin/activate      # On Windows: venv\Scripts\activate
   pip install requests urllib3
   ```

4. **Run the script:**
   ```bash
   python xkcd_to_epub.py
   ```

## Using the app

1. Enter a **From #** and **To #** for the comic range you want, or click **Use latest** to auto-fill the current highest comic number.
2. Adjust **Parallel downloads** if you want more or fewer simultaneous requests (higher is faster but heavier on the network).
3. Choose where to save the output `.epub` file under **Output**.
4. Click **Download and Build EPUB**. Progress and a running log appear below; you can click **Cancel** at any time to stop.
5. Once finished, open the resulting `.epub` in any e-reader (Kindle via conversion, Apple Books, Calibre, etc.).

## Notes

- Comics belong to their creator, Randall Munroe, and are distributed here only in the form xkcd.com already publishes them — this tool is for personal archiving/reading convenience, not redistribution.
- Be considerate of xkcd's servers: very large ranges with a high parallel-download count will fire off a lot of requests quickly. The built-in retry logic handles occasional rate-limiting, but there's no need to push it too hard.
