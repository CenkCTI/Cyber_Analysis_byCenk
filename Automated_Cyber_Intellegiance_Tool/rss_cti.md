---
layout: default
title: Automated CTİ_Tool
nav_order: 1
parent: Automated CTI Tool
---



"""
RSS Feed Reader & SQLite Storage Pipeline
==========================================
Features:
  - Fetches multiple RSS feeds via feedparser
  - Extracts title, link, published date, and summary
  - Stores entries in SQLite, using link as a unique key (no duplicates)
  - Structured for future spaCy NLP / entity extraction
"""

import sqlite3
import logging
from datetime import datetime
from time import mktime, struct_time

import feedparser  # pip install feedparser

# ── Optional: uncomment when you're ready to add spaCy NER ────────────────────
# import spacy
# nlp = spacy.load("en_core_web_sm")  # python -m spacy download en_core_web_sm
# ─────────────────────────────────────────────────────────────────────────────

# ---------------------------------------------------------------------------
# Configuration
# ---------------------------------------------------------------------------

DB_PATH = "rss_feeds.db"

RSS_FEEDS = [
    "https://feeds.bbci.co.uk/news/world/rss.xml",          # BBC World News
    "https://rss.nytimes.com/services/xml/rss/nyt/World.xml",# NYT World
    "https://feeds.reuters.com/reuters/topNews",             # Reuters Top News
    "https://hnrss.org/frontpage",                           # Hacker News
]

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s  %(levelname)-8s  %(message)s",
    datefmt="%Y-%m-%d %H:%M:%S",
)
log = logging.getLogger(__name__)


# ---------------------------------------------------------------------------
# Database helpers
# ---------------------------------------------------------------------------

def get_connection(db_path: str = DB_PATH) -> sqlite3.Connection:
    """Open (or create) the SQLite database and return a connection."""
    conn = sqlite3.connect(db_path)
    conn.row_factory = sqlite3.Row   # lets you access columns by name
    return conn


def create_tables(conn: sqlite3.Connection) -> None:
    """
    Create the schema if it doesn't exist yet.

    Table: feed_entries
      id            – auto-increment primary key
      feed_url      – source feed URL (for traceability)
      title         – article headline
      link          – article URL  ← UNIQUE KEY (prevents duplicates)
      published_at  – ISO-8601 datetime string, nullable
      summary       – plain-text excerpt / description
      entities_json – reserved column for future spaCy entity JSON
      fetched_at    – timestamp when this row was inserted
    """
    conn.execute("""
        CREATE TABLE IF NOT EXISTS feed_entries (
            id            INTEGER PRIMARY KEY AUTOINCREMENT,
            feed_url      TEXT    NOT NULL,
            title         TEXT,
            link          TEXT    NOT NULL UNIQUE,   -- <-- dedup key
            published_at  TEXT,
            summary       TEXT,
            entities_json TEXT,                      -- future spaCy output
            fetched_at    TEXT    NOT NULL
        )
    """)
    # Index on published_at to speed up time-range queries later
    conn.execute("""
        CREATE INDEX IF NOT EXISTS idx_published_at
        ON feed_entries (published_at)
    """)
    conn.commit()
    log.info("Database schema ready.")


# ---------------------------------------------------------------------------
# Parsing helpers
# ---------------------------------------------------------------------------

def parse_date(entry: feedparser.FeedParserDict) -> str | None:
    """
    Return an ISO-8601 date string from a feedparser entry, or None.
    Tries 'published_parsed' then 'updated_parsed'.
    """
    for attr in ("published_parsed", "updated_parsed"):
        value: struct_time | None = getattr(entry, attr, None)
        if value:
            try:
                return datetime.fromtimestamp(mktime(value)).isoformat()
            except (OverflowError, OSError):
                pass
    return None


def clean_html(text: str | None) -> str:
    """
    Very lightweight HTML tag stripper for summaries.
    Replace with html.parser or bleach for production use.
    """
    if not text:
        return ""
    import re
    return re.sub(r"<[^>]+>", "", text).strip()


# ---------------------------------------------------------------------------
# Entity extraction stub  (activate once spaCy is installed)
# ---------------------------------------------------------------------------

def extract_entities(text: str) -> str:
    """
    Placeholder for spaCy Named Entity Recognition.

    When ready:
      1. pip install spacy
      2. python -m spacy download en_core_web_sm
      3. Uncomment the import block at the top of this file.
      4. Replace the body below with the real implementation.

    Returns a JSON string like:
      [{"text": "OpenAI", "label": "ORG"}, {"text": "San Francisco", "label": "GPE"}]
    """
    # ── Real implementation (uncomment when spaCy is available) ──────────────
    # import json
    # doc = nlp(text[:100_000])   # guard against huge texts
    # entities = [{"text": ent.text, "label": ent.label_} for ent in doc.ents]
    # return json.dumps(entities, ensure_ascii=False)
    # ─────────────────────────────────────────────────────────────────────────

    return ""   # empty string = "not yet extracted"


# ---------------------------------------------------------------------------
# Core pipeline
# ---------------------------------------------------------------------------

def fetch_feed(feed_url: str) -> list[feedparser.FeedParserDict]:
    """Download and parse a single RSS/Atom feed. Returns a list of entries."""
    log.info("Fetching: %s", feed_url)
    parsed = feedparser.parse(feed_url)

    if parsed.bozo:
        # bozo=True means feedparser encountered a malformed feed
        log.warning("  Malformed feed (%s): %s", feed_url, parsed.bozo_exception)

    entries = parsed.entries
    log.info("  Found %d entries.", len(entries))
    return entries


def store_entries(
    conn: sqlite3.Connection,
    feed_url: str,
    entries: list[feedparser.FeedParserDict],
) -> tuple[int, int]:
    """
    Insert new entries into the database.

    Uses INSERT OR IGNORE so duplicate links are silently skipped.
    Returns (inserted_count, skipped_count).
    """
    now = datetime.utcnow().isoformat()
    inserted = skipped = 0

    for entry in entries:
        link: str = getattr(entry, "link", "").strip()
        if not link:
            log.debug("  Skipping entry with no link.")
            skipped += 1
            continue

        title    = getattr(entry, "title",   "") or ""
        summary  = clean_html(getattr(entry, "summary", "") or "")
        pub_date = parse_date(entry)

        # ── Future hook: extract entities from title + summary ────────────
        # entities_json = extract_entities(f"{title}. {summary}")
        entities_json = ""   # leave blank until spaCy is wired up
        # ─────────────────────────────────────────────────────────────────

        cursor = conn.execute(
            """
            INSERT OR IGNORE INTO feed_entries
                (feed_url, title, link, published_at, summary, entities_json, fetched_at)
            VALUES
                (?, ?, ?, ?, ?, ?, ?)
            """,
            (feed_url, title, link, pub_date, summary, entities_json, now),
        )

        if cursor.rowcount:
            inserted += 1
            log.debug("  + Inserted: %s", title[:80])
        else:
            skipped += 1
            log.debug("  ~ Duplicate skipped: %s", link)

    conn.commit()
    return inserted, skipped


def run_pipeline(feed_urls: list[str], db_path: str = DB_PATH) -> None:
    """
    Main entry point: iterate over all feed URLs, fetch, and store.
    """
    conn = get_connection(db_path)
    create_tables(conn)

    total_inserted = total_skipped = 0

    for url in feed_urls:
        try:
            entries = fetch_feed(url)
            ins, skp = store_entries(conn, url, entries)
            log.info("  → Inserted: %d  |  Skipped (duplicates): %d", ins, skp)
            total_inserted += ins
            total_skipped  += skp
        except Exception as exc:                      # noqa: BLE001
            log.error("  ✗ Failed to process %s: %s", url, exc)

    conn.close()
    log.info(
        "Pipeline complete. Total inserted: %d  |  Total skipped: %d",
        total_inserted,
        total_skipped,
    )


# ---------------------------------------------------------------------------
# Quick query helper (optional / for debugging)
# ---------------------------------------------------------------------------

def print_latest(db_path: str = DB_PATH, limit: int = 10) -> None:
    """Print the most recently fetched entries to stdout."""
    conn = get_connection(db_path)
    rows = conn.execute(
        """
        SELECT title, link, published_at, feed_url
        FROM   feed_entries
        ORDER  BY fetched_at DESC
        LIMIT  ?
        """,
        (limit,),
    ).fetchall()
    conn.close()

    print(f"\n{'─'*80}")
    print(f"  Latest {limit} entries in the database")
    print(f"{'─'*80}")
    for row in rows:
        print(f"  [{row['published_at'] or 'no date':>19}]  {row['title'][:55]}")
        print(f"   Source : {row['feed_url']}")
        print(f"   URL    : {row['link']}")
        print()


# ---------------------------------------------------------------------------
# Entry point
# ---------------------------------------------------------------------------

if __name__ == "__main__":
    run_pipeline(RSS_FEEDS)
    print_latest(limit=5)
