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
```

## Comment 4 — Default visibility

**My position:**
I chose to keep `public=True` as the default value for watchlist entries.

**Reasoning:**
CineLog is built around film discovery and social activity. A public-by-default
watchlist supports that goal by making saved films visible without requiring
additional configuration from the user. This reduces friction and encourages
sharing viewing interests with other users.

Because the watchlist is intended to represent films a user plans to watch
rather than personal account information, I believe a public default aligns
with the expected behavior of a social film-tracking platform.

**Tradeoff acknowledged:**
A reviewer could reasonably argue that private-by-default is safer because it
prevents accidental sharing. Privacy mistakes are harder to undo than
discoverability mistakes, and a private default would better protect users who
do not review application settings carefully.

I agree that this is a legitimate concern. However, I still favor `public=True`
because CineLog is positioned as a social film-tracking platform where sharing
viewing interests is a core behavior. If the platform expands privacy features
in the future, I would recommend exposing visibility as an explicit user choice
at creation time rather than relying solely on a default.

## Comment 5 — Sort order

**My position:**
I would change the watchlist ordering to use `date_added` rather than
alphabetical title order.

**Reasoning:**
A watchlist primarily represents viewing intent. The most recently saved films
typically reflect a user's current interests, so showing newer additions first
surfaces the films a user is most likely considering watching next.

This is also consistent with how the collection service already emphasizes
chronological activity by sorting collection entries by date added.

**Engagement with reviewer's point:**
I agree that alphabetical ordering improves discoverability and gives users a
consistent location for a film within a large watchlist. For very large lists,
this may reduce the effort required to locate a specific title.

However, I believe a watchlist primarily represents future viewing intent rather
than archival storage. Recently added entries usually reflect current interest,
so date-added ordering better surfaces the films users are most likely to act
on next. While alphabetical sorting is valuable, I think chronological ordering
better supports the primary purpose of a watchlist. If CineLog introduces
user-selectable sorting in the future, I would support providing both options.
