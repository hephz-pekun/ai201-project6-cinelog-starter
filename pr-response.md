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