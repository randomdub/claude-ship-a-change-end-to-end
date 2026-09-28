# NOTES: PUT /users/:id

## The plan
I had Claude plan in plan mode before it wrote any code, and I approved the plan as written. The plan had two parts:

- A new `updateUser` function in `db/store.js`. Like `getUserById`, it returns nothing when the user doesn't exist, so the route decides what to send back.
- A new `PUT /users/:id` route in `routes/users.js`. PUT replaces the whole user, so both `name` and `email` are required. They have to be non-empty text, the email has to look like an email, and extra spaces are trimmed off. Bad input gets a 400 and an unknown user gets a 404.

The plan also pointed out something I wouldn't have noticed: the 404 test was already passing before I'd written anything. Express returns its own 404 for a route that doesn't exist, so the test was passing for the wrong reason. Partial updates (PATCH) and duplicate-email checks were left for later.

## Model
I used Opus 5.5. The code itself is small, but getting the edge cases right and reviewing the result carefully is where a stronger model helps. Sonnet would probably have been fine for writing the code alone.

## Commits
I split the work into four commits, one step each:

1. The store function
2. The route, which is the commit that made the endpoint tests pass
3. The bug fix from the review
4. This file

I kept the review fix as its own commit so the history shows what the review changed.

## What the review caught
All the tests passed, and the review still found a real bug. The route turned the id into a number with `Number()`, which also accepts ids like `0x1`, `1e0` and `1.0` as 1. So `PUT /users/0x1` quietly updated user 1. Now only plain digit ids are accepted, and anything else gets a 404.

The review also confirmed these cases already worked: blank or non-text fields, bad emails, an empty body, and an `id` sent in the body (it's ignored). It turned up two older issues that I left alone and listed in the PR: `GET /users/:id` has the same id problem, and badly formed JSON gets Express's HTML error page.
