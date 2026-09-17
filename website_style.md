# style.md — Berkeley Technology Review

Engineering principles for the BTR website. Distilled from Guillermo Rauch's *7 Principles of Rich Web Applications* (2014) and adapted for an editorial site, where the core job is getting an article in front of a reader.

**Prime directive:** minimize the time between wanting to read something and reading it. Every rule below follows from that. When two rules conflict, the one that gets words on screen sooner wins.

---

## 1. Ship readable HTML in the first response

Article text must never wait on JavaScript. Pre-render or statically generate every page that has content in it; the reader should be reading before our bundle has finished parsing.

- Body copy, headline, byline, and date arrive in the initial HTML.
- Aim to have something readable inside the first ~14KB — roughly one TCP round trip.
- Inline critical/structural CSS; defer theme, fonts, and anything below the fold.
- Comments, related articles, newsletter widgets, and analytics load after.
- Test with JS disabled. If the article disappears, the page is broken.

**Don't:** ship an empty `<body>` with a `<script>` tag and fetch the article client-side.

## 2. Act on input immediately

Once the page is interactive, JS exists to hide network latency, not add to it.

- Nav opens, filters apply, and search results begin rendering on keystroke — not after a round trip.
- Move to the next layout optimistically, before the data arrives. The reader sees structure instantly; content fills in.
- No spinner under 1 second. Below ~100ms nothing is needed at all; below ~1s the reader tolerates the wait without feedback. Delay any loading indicator past that threshold so fast responses never flash one.
- Exception: anything with real consequences (submitting to the tip line, account actions) confirms only after the server does.

## 3. Reactivity only where the content is actually live

Most of our content is frozen the moment it publishes; pretending otherwise is wasted complexity. Apply live updates narrowly:

- Live event coverage and election-night style posts.
- Comment threads and reactions on an open article.
- Editor previews — changes in the CMS appear without a manual refresh.

Everywhere else, cache hard and revalidate in the background. If a page never changes, don't open a socket for it.

## 4. Own the data exchange

When we do talk to the server, handle the failure cases the browser used to handle for us.

- Retry failed requests on the reader's behalf; surface an error only after retries fail.
- Treat an unexpected 403 as an expired session and prompt to sign in rather than dumping a raw error.
- Buffer unsent input (a half-written comment, a saved-article toggle) in `localStorage` and flush it when the connection returns.
- Warn before navigation that would discard unsaved editor or comment input.

## 5. Don't break history — enhance it

The back button is a feature of the platform, and readers have decades of expectations about it.

- Infinite scroll and "load more" must `pushState`. Back should return to the feed, not the top of the page.
- Restore scroll position, including after a return from an external site.
- Make back instant: cache the previous page and render it from memory, then reconcile any changes.
- Collapsed/expanded state that only makes sense within a history entry stays with that entry — a fresh navigation starts fresh.

## 6. Push code updates, not just content

A tab left open for a week should not be running last week's code.

- Send a version identifier with outgoing requests; when the server sees a stale one, tell the client.
- Prefer swapping modules over forcing a full refresh. If a refresh is unavoidable, do it only when the page is backgrounded and no input is in progress.
- Keep state out of the DOM so components can re-render on update without side effects.

## 7. Predict the next click

The fastest request is the one already in flight.

- Prefetch an article on link hover, and on viewport entry for touch devices.
- Prefetch the next article in an issue while the reader is still on the current one.
- Respect `prefers-reduced-data` and metered connections; prediction is an optimization, never an obligation.

---

## Before merging

- [ ] Page renders and is readable with JS off
- [ ] First meaningful paint within one round trip on a cold cache
- [ ] No spinner appears for work under 1 second
- [ ] Back button restores position and state
- [ ] Failed requests retry before they surface as errors
- [ ] Link prefetch is in place on article lists
