# Ask For An Adversarial Review

There are two things I like to do to get better AI-assisted review of the code
that me and my agents are writing.

First, I find it essential to do the review with fresh or mostly fresh context.
Maybe even a different model (e.g. GPT-6.1 Sol instead of Opus 5.5).
Specifically, I don't want an agent with all the context of how the feature was
implemented and the reasoning behind that implementation also doing the
reviewing. That implementer context effectively creates a positive bias toward
the implementation as it is.

Second, I like to provide the review agent with a thorough prompt outlining how
it should approach doing an _adversarial review_ of this feature. I don't want
simply, "Please review this branch". I don't even want, "Do an adversarial
review of this branch".

This is the prompt I want. Notice it is tailored to a specific feature I'm
working on. I love that it includes something like "Assume the author missed
something."

```
You are an adversarial code reviewer. Your job is to find real defects in this
branch, not to praise it or suggest style nits. Assume the author missed
something.

## Scope
Repo: Rails app (RSpec tests in `spec/`, `bundle exec rspec`; lint `bundle exec
standardrb`).
Review `git diff main...HEAD` on branch `claude/optimistic-curie-w67rb3`. The
commits:
- caf63be Add a TIL index and link TIL categories to tag pages
- 9fccfae Interleave TILs and blogmarks on tag pages
- 67980ce Style tag page content types as beats, not tags
- dab6534 Simplify TIL listing queries and tag page view

What the branch does: adds a public `/tils` index. Tag pages
(`/tags/:normalized_value`) now show blogmarks with that Tag plus published TILs
whose `category` matches, merged newest first (TILs by `first_committed_at`,
blogmarks by `created_at`). Each entry gets a "beat" label
(`app/views/shared/_beat.html.erb`). A new `TilPost.for_listing` scope selects a
subset of columns.

## Attack these specifically
1. **Data exposure:** can unpublished TILs (`published_at: nil`) or private data
   leak through `/tils`, tag pages, related posts, or counts? Is
   `allow_unauthenticated_access only: [:index, :show]` correct?
2. **Tag/category matching:** `Tag.normalized_value` vs `TilPost.category`. What
   happens with case differences, spaces, hyphens, unicode, or a category that
   normalizes differently from its Tag? Can one topic split across two pages, or
   produce a 404 when it shouldn't? Does `tag_path(til_post.category)` always yield
   a URL that resolves?
3. **`for_listing` partial selects:** does any code that renders these records
   (`public_param`, `til_path`, partials, helpers, callbacks) touch a column
   that wasn't selected, causing `ActiveModel::MissingAttributeError`? Check
   `public_param`'s implementation.
4. **Ordering:** check ties, nil timestamps, timezone or date-boundary issues in
   `strftime` / `to_date` / year grouping, and whether the sort is stable and
   deterministic.
5. **Routing:** do `/tils` vs `/tils/:id` (with the hex constraint) conflict
   with each other or with existing routes or formats (`.atom`, `.json`)?
6. **Performance:** look for N+1 queries (blogmark tags, related posts) and
   unbounded loads (all published TILs on `/tils`, all blogmarks for a big tag).
   Is there missing pagination that matters?
7. **XSS / HTML safety:** `markdown(entry.notes)`, titles, categories in URLs
   and attributes, `datetime` attributes.
8. **`script/cloud-setup.sh`:** does the `sed` on
   `/etc/apt/sources.list.d/ubuntu.sources` fail under `set -e` when the file is
   missing (older Ubuntu, non-cloud env)?
9. **Tests:** do the specs actually prove the behavior? Look for assertions that
   would pass even if the code were broken (e.g. `body.index` matching text
   elsewhere on the page, layout nav containing "TIL"). What important cases are
   untested?
10. **Styling:** will Tailwind's build include the new classes
   (`text-amber-700`, `text-sky-700`, `tracking-widest`)? Does the beat stay
   accessible (icon `aria-hidden`, text readable, contrast)?

## Rules
- Verify every finding by reading the code, and run a spec or `bin/rails runner`
repro where you can. Drop anything you can't substantiate.
- For each finding give: file:line, a concrete failure scenario (input/state →
wrong result), severity (high/med/low), and a minimal fix.
- Rank most severe first. No style or naming nits. If something is fine, don't
mention it.
- Don't modify code; report only.
```

I'm not writing an adversarial review prompt like this for each feature.
Instead, I ask the implementer agent (or even a fresh agent) to:

> Write an adversarial review prompt for this branch

I copy and paste the resulting prompt into a fresh agent session and see what it
comes up with.

This certainly isn't the only step to reviewing code with agents, but for many
features it is a great starting point.
