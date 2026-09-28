# NOTES: PUT /users/:id

## The plan I approved
Claude planned the change in plan mode before touching any code. The plan added an `updateUser(id, { name, email })` helper in `db/store.js`, following the `getUserById` pattern: it returns `undefined` when the user is missing, so the route picks the status code. It then added `PUT /users/:id` in `routes/users.js`. Because PUT is a full replacement, both fields are required. They must be non-empty strings, the email must look like an address, and values are trimmed. The route validates first and looks up second, returning 400 for bad input and a JSON 404 for an unknown id. The plan also pointed out that the 404 test was already passing by accident, since Express returns its default 404 for an unmatched route. So the handler needed to return a real 404 of its own. I approved the plan without edits. PATCH, duplicate-email checks, and tightening POST's validation were left out of scope.

## Model choice
Opus 5.5. The feature is small, but most of the work is in edge cases and in reviewing the result, and that's where the stronger model pays off. A cheaper model like Sonnet would probably have been fine for writing the code itself.

## How I split the commits
1. Store helper (`updateUser`): the data-access layer on its own.
2. The `PUT` route with validation and 404: the feature itself, which turns the endpoint tests green.
3. Fix from the review: reject non-integer ids.
4. This NOTES.md.

Each commit is one logical change and passes lint. The review fix is its own commit so that the review's effect shows up in the history.

## What the review caught
The tests were green, but the review still found a real bug. The route converts the id with `Number()`, which accepts `"0x1"`, `"1e0"` and `"1.0"`, so `PUT /users/0x1` silently updated user 1. The fix is to accept only plain digit ids and return 404 for anything else. The review also confirmed that blank or non-string fields, bad emails, an empty body, and an `id` sent in the body are all handled correctly. One pre-existing issue came up that I left alone: malformed JSON gets Express's default HTML 400 page on every route. The same `Number()` quirk also affects `GET /users/:id`. Both are noted in the PR as follow-ups.
