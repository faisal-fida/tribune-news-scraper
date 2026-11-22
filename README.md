# tribune-news-scraper

An async scraper for The Express Tribune (tribune.com.pk) "latest" listing that stores articles in Supabase, with a small FastAPI wrapper.

## Status

Work in progress and not runnable as a whole:

- `utils/scraper.py` is a standalone script. It calls `asyncio.run(main())` at import time and imports `db_init` as a sibling module, so it only works when started from inside `utils/` (`cd utils && python scraper.py`).
- `app.py` imports `get_news_data` from a top-level `scraper` module that does not exist at that path, and calls it without the required `url` argument, so the `/news` endpoint does not work.
- `db_init.update_data` uses a `.where(...)` call the Supabase client does not have.
- Only the standalone scraper script reflects real working logic.

## Features (what the scraper does)

- Fetches listing pages `https://tribune.com.pk/latest?page=N` for pages 900 to 4999 and extracts story links.
- Fetches each story and parses title, author, date and article text with BeautifulSoup.
- Uses `httpx` with `asyncio` and a semaphore of 100 concurrent requests.
- Inserts batches of 1000 rows into the Supabase table `news_data`.

## Tech stack

Python, httpx, BeautifulSoup, Supabase client, FastAPI, Uvicorn.

## Project structure

```
app.py            FastAPI app (GET / and GET /news, see Status)
utils/scraper.py  async scraper
utils/db_init.py  Supabase client and insert/get/update helpers
requirements.txt
```

## Setup and run

```bash
pip install -r requirements.txt
```

Create `.env`:

| Variable | Purpose |
|---|---|
| `SUPABASE_URL` | your Supabase project URL |
| `SUPABASE_KEY` | Supabase API key |

A Supabase table `news_data` with columns `title`, `author`, `date`, `article_text`, `url` is expected.

```bash
cd utils && python scraper.py
```

Be considerate: the defaults send thousands of requests to a live news site. Lower the page range and semaphore first, and check the site's terms.

## Limitations

- No retry or error handling around requests, and a page with no stories breaks the flattening step.
- No tests. The scraped site's HTML selectors may have changed since early 2024.

## License

MIT, see `LICENSE`.
