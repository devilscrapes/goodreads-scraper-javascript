<p align="center"><a href="https://apify.com/DevilScrapes/goodreads-book-reviews-scraper"><img src="assets/banner.svg" alt="Goodreads Book &amp; Reviews Scraper — JavaScript example" width="100%"></a></p>

# Goodreads API alternative for JavaScript and Node.js: books, ratings and reviews as JSON

<p align="center">
<a href="https://apify.com/DevilScrapes/goodreads-book-reviews-scraper"><img alt="Run on Apify" src="https://img.shields.io/badge/Run%20on-Apify-DC2626?logo=apify&logoColor=white"></a>
<img alt="JavaScript" src="https://img.shields.io/badge/Node.js-20%2B-339933?logo=nodedotjs&logoColor=white">
<a href="LICENSE"><img alt="MIT license" src="https://img.shields.io/badge/license-MIT-0A0606"></a>
<a href="https://github.com/devilscrapes/goodreads-scraper-javascript/actions/workflows/ci.yml"><img alt="CI" src="https://github.com/devilscrapes/goodreads-scraper-javascript/actions/workflows/ci.yml/badge.svg"></a>
</p>

If your reading app, bookshop backend or Discord bot needs Goodreads data, there is no supported API to call any more. This repository fills the gap with one ES module that runs the **[Goodreads Book & Reviews Scraper](https://apify.com/DevilScrapes/goodreads-book-reviews-scraper)** through the official `apify-client` package. You pass book links, author or Listopia URLs, or ISBN strings; you get back plain objects with the canonical book URL, cover image, series, genres, ISBN-13, publication dates, average rating and the count of ratings in every star bucket. Reader reviews can ride along nested inside each book, each one tagged with its star rating and whether it contains spoilers. Apify does the page fetching; your Node service only starts a run and reads the objects. No soul required, just an Apify token.

> **Actor:** [Goodreads Book & Reviews Scraper](https://apify.com/DevilScrapes/goodreads-book-reviews-scraper) · **Publisher:** [Devil Scrapes](https://apify.com/DevilScrapes) · **Also in Python:** [goodreads-scraper-python](https://github.com/devilscrapes/goodreads-scraper-python)

## 🔥 What you get

- **Four ways in**: a single `/book/show/` link, an author's page, an author's full bibliography under `/author/list/`, or a Listopia list, and any mix of them in one run.
- **Lookup by ISBN or title**: each entry in `searchQueries` goes through the site's own autocomplete, with its regular search page as a fallback, and the top hits are expanded into full book rows.
- **Edition and work kept apart**: page count, format, publisher, ISBN-10, ISBN-13 and ASIN describe the edition you pointed at, while the rating average, rating count and star histogram belong to the work across all editions.
- **A five-bucket star histogram** in `ratings_count_distribution`, ordered from one star to five, so you can measure how divided readers are and not just the average.
- **Reviews as a nested list or as their own rows**: `outputMode` decides whether each book carries its reviews inside it or every review becomes a dataset row with the book's ID, title and URL attached.
- **Each book once per run**: a title reached from a list, an author page and a search query is fetched a single time, and `maxBooks` caps the total across all sources.
- **Failures stay local**: a page that does not load after several spaced-out attempts is logged and skipped, and the run only fails if no book at all could be read.

## 🎯 Use cases

### Show book cards in a reading app

Resolve the ISBN a user scans, cache the returned object by `book_id` and render title, cover from `image_url`, series badge and average rating. Because `isbn13` and `asin` are on the same object, linking out to a bookshop or an e-reader store takes no second lookup.

### Power a book-club bot

When someone in your Discord or Slack drops a Goodreads link, a serverless handler can start a one-book run, then reply with the genres, page count and a short excerpt from a review whose `has_spoiler` is false. Members get context without anyone spoiling the ending.

### Keep a catalogue in sync

A nightly job can re-run your stored book URLs and upsert each object by `book_id`. Changes in `average_rating`, `ratings_count` or `text_reviews_count` land in your database, and a webhook can notify editors when a title on their list moves.

## ⚡ Quick start

You need an Apify account and an API token from [Console → Settings → Integrations](https://console.apify.com/settings/integrations). Apify's free plan includes monthly usage credit; this Actor's event prices are listed under Pricing below.

```bash
git clone https://github.com/devilscrapes/goodreads-scraper-javascript.git
cd goodreads-scraper-javascript
npm install
export APIFY_TOKEN=your_token_here
node index.js                       # uses input.json
node index.js my-input.json --out data --max-charge 2
```

The script starts the [Goodreads Book & Reviews Scraper](https://apify.com/DevilScrapes/goodreads-book-reviews-scraper), waits for it to finish, prints a link to the run in Apify Console and saves every result to `results.json` and `results.csv`. It exits with code 1 and the run's status message if the run fails.

## 🧩 The core call

Here is the integration with everything else removed. `call()` resolves once the Actor run has ended, and `listItems()` returns the rows stored in that run's dataset.

```javascript
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: 'YOUR_APIFY_TOKEN' });
const run = await client.actor('DevilScrapes/goodreads-book-reviews-scraper').call({"startUrls": ["https://www.goodreads.com/book/show/2767052-the-hunger-games"], "maxBooks": 1, "includeReviews": true, "maxReviewsPerBook": 2, "outputMode": "books", "proxyConfiguration": {"useApifyProxy": true}});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
console.log(items);
```

## 📥 Input

`index.js` sends `input.json` to the Actor untouched. The sample asks for a single famous novel plus two of its reviews, which is enough to see the object shape for well under a dollar. Replace the link with your own, or put ISBNs and titles in `searchQueries` and leave `startUrls` empty.

```json
{
  "startUrls": [
    "https://www.goodreads.com/book/show/2767052-the-hunger-games"
  ],
  "maxBooks": 1,
  "includeReviews": true,
  "maxReviewsPerBook": 2,
  "outputMode": "books",
  "proxyConfiguration": {
    "useApifyProxy": true
  }
}
```

| Field | Type | Required | Default | Allowed values |
|---|---|---|---|---|
| `startUrls` | array | no | `[]` | — |
| `searchQueries` | array | no | `[]` | — |
| `maxBooks` | integer | no | `20` | — |
| `maxSearchResultsPerQuery` | integer | no | `3` | — |
| `includeReviews` | boolean | no | `false` | — |
| `maxReviewsPerBook` | integer | no | `10` | — |
| `outputMode` | string | no | `"books"` | `books`, `reviews` |
| `proxyConfiguration` | object | no | `{"useApifyProxy": true}` | — |

## 📤 Output

With the default settings every object is a book and its reviews sit in a `reviews` array. Flip `outputMode` to `reviews` and every object becomes one review instead, with `book_id`, `book_title` and `book_url` copied in so you can still group them. Missing values are `null`, never absent keys, which keeps TypeScript types honest.

| Field | Type | Always present |
|---|---|---|
| `book_id` | string | yes |
| `work_id` | string or null | no |
| `title` | string | yes |
| `title_complete` | string | yes |
| `url` | string | yes |
| `author_name` | string or null | no |
| `author_url` | string or null | no |
| `series_name` | string or null | no |
| `series_position` | string or null | no |
| `genres` | array | yes |
| `format` | string or null | no |
| `num_pages` | integer or null | no |
| `publisher` | string or null | no |
| `isbn` | string or null | no |
| `isbn13` | string or null | no |
| `asin` | string or null | no |
| `language` | string or null | no |
| `publish_date` | string or null | no |
| `original_publish_date` | string or null | no |
| `description` | string or null | no |
| `image_url` | string or null | no |
| `average_rating` | number or null | no |
| `ratings_count` | integer or null | no |
| `text_reviews_count` | integer or null | no |
| `ratings_count_distribution` | array | yes |
| `awards` | array | yes |
| `reviews` | array | yes |
| `reviews_returned_count` | integer | yes |
| `source_query` | string or null | no |
| `scraped_at` | string | yes |

## 💰 Pricing

| Event | Price (USD) |
|---|---|
| Actor start | $0.2 |
| Result row | $0.002 |

Prices as configured on 2026-08-25. The [Actor page](https://apify.com/DevilScrapes/goodreads-book-reviews-scraper) always shows the current price. Runs are billed to the Apify account that owns the token. Use `--max-charge` to cap what a single run can cost.

## 🛠️ Working with the data in JavaScript

The snippet below turns the saved rows into compact book cards for a frontend: it computes the five-star percentage from the histogram, keeps the first three genres and picks a spoiler-free review excerpt. Only Node.js built-ins are used.

```javascript
import { readFile, writeFile } from 'node:fs/promises';

const books = JSON.parse(await readFile('results.json', 'utf8'));

const cards = books.map((book) => {
    const histogram = book.ratings_count_distribution ?? [];
    const total = histogram.reduce((sum, n) => sum + n, 0);
    const clean = (book.reviews ?? []).find((r) => !r.has_spoiler && r.review_text);
    return {
        id: book.book_id,
        title: book.title_complete,
        author: book.author_name,
        isbn13: book.isbn13,
        rating: book.average_rating,
        fiveStarPct: total ? Math.round((histogram[4] / total) * 100) : null,
        genres: (book.genres ?? []).slice(0, 3).join(', '),
        excerpt: clean ? clean.review_text.slice(0, 280) : null,
    };
});

await writeFile('book-cards.json', JSON.stringify(cards, null, 2));
console.table(cards.map(({ excerpt, ...card }) => card));
```

## ❓ FAQ

### Can I call this from a serverless function?

Yes, as long as the platform lets the call wait for the run to end. For a long Listopia list, kick the run off asynchronously and have an Apify webhook notify your endpoint when the books are stored.

### Does the Actor log in to Goodreads?

No. It reads what a signed-out visitor sees, which is why reviews are limited to those rendered on the book page. Shelving books, posting reviews or reading private profiles are out of scope.

### What happens if one of my URLs is wrong?

Input validation rejects anything that is not a book, author, author-books or Listopia link on www.goodreads.com before the run does any work. A valid link whose page later fails to load is logged and skipped while the remaining books continue.

### How is a run billed?

There is a start fee per run and a per-row fee for every item written to the dataset. In `books` mode a row is a book; in `reviews` mode a row is a review. Pass `--max-charge` while you experiment so a large list cannot surprise you.

## 🧯 Troubleshooting

- **`Set APIFY_TOKEN first`**: export the token in the same shell, or put it in a local `.env` file your shell loads. Never commit it.
- **Run finished with `FAILED` or `ABORTED`**: open the printed Console link; the run log says why. Input validation errors name the field.
- **Fewer rows than expected**: a `--max-charge` cap also limits how many results can be charged. Raise the cap or lower the limits in your input.
- **Something looks wrong in the data**: open an issue on the [Actor page](https://apify.com/DevilScrapes/goodreads-book-reviews-scraper) so the maintainers see it.

## 🔗 More from Devil Scrapes

- [Goodreads Book & Reviews Scraper on Apify Store](https://apify.com/DevilScrapes/goodreads-book-reviews-scraper)
- [Python version of this example](https://github.com/devilscrapes/goodreads-scraper-python)
- [All Devil Scrapes Actors](https://apify.com/DevilScrapes)
- [Apify web scraping cookbook](https://github.com/devilscrapes/apify-web-scraping-cookbook)

Related Actors:

- [Crossref Works Scraper — DOIs, Authors & Citations](https://apify.com/DevilScrapes/crossref-works-scraper) — DOI and publisher metadata for academic titles that Goodreads covers thinly.
- [Wikiquote Scraper](https://apify.com/DevilScrapes/wikiquote-quotes-scraper) — sourced quotations from the authors on your list.
- [Trustpilot Reviews Scraper](https://apify.com/DevilScrapes/trustpilot-reviews-scraper) — review text and star ratings for companies, for the same sentiment tooling.

---

MIT licensed example code. The Actor itself runs on [Apify](https://apify.com/DevilScrapes/goodreads-book-reviews-scraper) and is maintained by [Devil Scrapes](https://apify.com/DevilScrapes). Not affiliated with Goodreads or Amazon.
