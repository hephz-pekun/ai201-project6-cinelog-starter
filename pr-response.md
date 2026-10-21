## Comment 1 — Rename

**What I did:**
Renamed `save_to_watchlist()` to `add_to_watchlist()` in
`services/watchlist_service.py` and updated all references in
`routes/watchlist/watchlist.py` to match the project's verb_to_noun naming
convention used by functions such as `add_to_collection()`.

**How I verified:**
Used `git grep save_to_watchlist` to locate all references, updated each call
site, and confirmed the search no longer returned any matches. Ran the test
suite afterward to verify imports and route wiring still worked.

## Comment 2 — Deduplication

**What I did:**
Added a duplicate check to `add_to_watchlist()` following the same pattern used by
`add_to_collection()` in `services/collection_service.py`.

`add_to_collection()` performs a query using `user_id` and `film_id` to check
whether an entry already exists before creating a new one. I implemented the
same behavior for watchlist entries.

If a matching watchlist entry already exists, the service now raises
`AlreadyInWatchlistError` instead of creating a duplicate record.

**How I verified:**
Reviewed the implementation of `add_to_collection()` and confirmed the watchlist
service follows the same flow. Ran the test suite after the change to verify no
existing behavior was broken.

## Comment 3 — Missing test

**What I did:**

Created `tests/test_watchlist.py` and added a test for the case where
`add_to_watchlist()` receives a film ID that does not exist.

I used `test_add_to_collection_nonexistent_film_raises()` from
`tests/test_collection.py` as the model so the watchlist tests follow the same
fixture setup, error handling pattern, and assertion style already used by the
project.

**How I verified:**

Ran:

```bash
pytest tests/test_watchlist.py -v