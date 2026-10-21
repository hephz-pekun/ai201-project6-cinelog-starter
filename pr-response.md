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

## Comment 6 — Rebase and Conflict Resolution

**What conflicted:**

During the rebase onto the updated main branch, `models.py` conflicted because
the watchlist feature branch was built using integer film IDs while the main
branch had already migrated film identifiers to UUIDs.

**How I resolved it:**

I updated `WatchlistEntry.film_id` to use `db.String(36)` to match the UUID
format used throughout the application. While resolving the conflict, I
preserved the watchlist-specific `index=True` setting from the feature branch.

**How I verified it:**

After resolving the conflict, I completed the rebase, reran the full test
suite, and confirmed that all tests passed successfully.

## Stretch Features

### Stretch Feature 1: remove_from_watchlist()

**What I implemented:**

I added a `remove_from_watchlist()` service function to allow users to remove
films from their watchlist.

The implementation follows the same design pattern used by
`remove_from_collection()`. The function searches for a matching
`WatchlistEntry` using `user_id` and `film_id`, raises a
`NotInWatchlistError` if no matching entry exists, and deletes the entry if
found.

**How I verified:**

I manually tested the removal workflow and confirmed that watchlist entries are
successfully deleted from the database and no longer appear in subsequent
watchlist queries. If no matching watchlist entry exists, a NotInWatchlistError is raised.
I added a test to verify successful removal behavior.

---

### Stretch Feature 2: Additional Watchlist Test

**What I implemented:**

I added a new automated test,
`test_add_to_watchlist_duplicate_raises()`, to validate watchlist
deduplication behavior.

The test attempts to add the same film to a user's watchlist twice and verifies
that an `AlreadyInWatchlistError` is raised instead of creating duplicate
records.

**Why I chose this test:**

I chose this case because duplicate prevention was introduced as part of the code review feedback and
represents a likely regression point. Adding an automated test helps ensure that
future changes do not accidentally remove the duplicate-checking behavior.

**How I verified:**

I executed the full pytest suite and confirmed that the new test passes while
all previously existing tests continue to pass. The test verifies duplicate entries cannot be created and that
AlreadyInWatchlistError is raised. uplicate

---

### Stretch Feature 3: Visibility Toggle

**What I implemented:**

I extended `add_to_watchlist()` to accept a `public` parameter, allowing
callers to explicitly control the visibility of watchlist entries.

The watchlist route was updated to accept an optional `public` value from the
request payload and pass it to the service layer. Existing behavior remains
unchanged because the default value remains `True`.
```
{
  "film_id": "...",
  "public": false
}
```

**Why this improves the design:**

During the review discussion, I identified privacy as an important tradeoff of a
public-by-default watchlist. Allowing callers to specify visibility at creation
time gives users more control over their data while preserving the existing
social-sharing experience.

**How I verified:**

I added and tested watchlist entries using both public and private visibility
settings and confirmed that the `public` field is stored correctly in the
database.

## AI Usage

I used Microsoft Copilot during this project to assist with several development
and review tasks.

During the code review phase, I used AI assistance to analyze reviewer comments,
identify the affected files, and compare the implementation of
`add_to_watchlist()` against the existing `add_to_collection()` pattern before
implementing deduplication support.

During the rebase phase, I used AI assistance to reason through merge conflicts
caused by the film ID migration from integer identifiers to UUIDs. The AI
helped identify which portions of the main branch should be preserved and which
watchlist-specific changes still needed to be retained.

I also used AI assistance to review commit history, verify conventional commit
formatting, check my work against the assignment rubric, and improve the clarity
of written responses in `pr-response.md`.

All code changes, testing, debugging, conflict resolution, commit creation, and
final verification were performed manually in my local development environment.

## PR Description

### Summary

This PR completes and improves the watchlist feature for CineLog. Users can add
films to a personal watchlist, retrieve their saved watchlist, and manage
entries through the watchlist service layer and API endpoint. In addition to the
original implementation, I addressed reviewer feedback, added missing tests,
resolved a rebase conflict after a main-branch refactor, and completed several
stretch features.

### Design Decisions

**Default Visibility (`public=True`)**

I chose to keep watchlist entries public by default. CineLog is designed as a
social film-tracking platform, and a public watchlist supports discovery and
sharing of viewing interests without requiring additional user configuration.

The tradeoff is that a private default would better prevent accidental sharing.
While I recognize that concern, I believe a public default better aligns with
the platform's social goals. To provide additional flexibility, I implemented a
visibility toggle that allows callers to explicitly create private watchlist
entries.

**Watchlist Sort Order**

I would prefer sorting watchlist entries by `date_added` rather than
alphabetically. A watchlist represents future viewing intent, and recently added
films are more likely to reflect a user's current interests. Showing newer
entries first helps surface films the user is most likely considering watching
next.

I understand the argument for alphabetical ordering because it improves
discoverability in larger lists, but I believe chronological ordering better
supports the primary purpose of a watchlist. If CineLog later supports multiple
sorting options, I would support offering both.

### Code Review Feedback Addressed

- Renamed `save_to_watchlist()` to `add_to_watchlist()` to match project naming
  conventions.
- Added duplicate-entry protection and introduced
  `AlreadyInWatchlistError`.
- Added a missing test for nonexistent `film_id` values.
- Rebased onto the updated main branch and resolved the UUID migration conflict
  by updating `WatchlistEntry.film_id` to use UUID-compatible identifiers.
- Preserved watchlist-specific behavior such as indexing while resolving the
  conflict.

### Stretch Features Implemented

- Added `remove_from_watchlist()`.
- Added an additional watchlist deduplication test.
- Added a visibility toggle allowing callers to specify `public=True` or
  `public=False` when creating watchlist entries.

### Manual Testing

1. Start the application.
2. Create a test user and film record.
3. Add a film to the watchlist using the watchlist endpoint.
4. Verify the film appears in the user's watchlist.
5. Attempt to add the same film again and confirm that a duplicate-entry error
   is raised.
6. Attempt to add a nonexistent `film_id` and confirm that
   `FilmNotFoundError` is raised.
7. Create a watchlist entry with `"public": false` and verify that the stored
   entry reflects the private visibility setting.
8. Remove a film from the watchlist and confirm that it no longer appears in
   retrieval results.
9. Run the automated test suite:

   ```bash
   pytest tests/ -v
   ```

   ### Git Log
   ![alt text](<Git Log - Oneline.png>)