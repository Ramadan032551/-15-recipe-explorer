# Recipe Explorer

A Next.js 15 App Router app that explores recipe categories from [TheMealDB](https://www.themealdb.com/api/json/v1/1/).

Explore categories, search recipes, view ingredients and instructions, save favourites, and discover a random recipe.

## Run locally

```bash
npm install
npm run dev
```

The app is available at [http://localhost:3000](http://localhost:3000).

## Fetch caching in Next.js 15

This app explicitly sets a revalidation time on every MealDB request:

- Categories are cached and revalidated every 86,400 seconds (one day).
- Search results, category meals, and meal details are cached and revalidated every 3,600 seconds (one hour).
- Random recipe lookups use `next.revalidate: 0` so each request asks TheMealDB for a fresh recipe.

Favourites are stored as a validated JSON array of up to 50 meal IDs in an HTTP-only, same-site cookie. They are kept in the browser rather than a user account, so the collection is specific to that browser.

The main fetch cache options are:

- `cache: "force-cache"` stores and reuses a response until it is revalidated or invalidated. It is the explicit opt-in for persistent caching.
- `next: { revalidate: seconds }` caches a response and checks for an updated one after the interval. Use `0` to opt out of caching; `false` means no time-based revalidation.
- `cache: "no-store"` always fetches fresh data and does not put the response in the Data Cache.

The default changed in Next.js 15: `fetch` requests are **not cached by default** (`auto no cache`), unlike Next.js 14, where `fetch` was cached by default. GET Route Handlers also became uncached by default. This does not disable the Full Route Cache for pages that Next.js can statically render; it changes the default for data requests and GET Route Handlers. To preserve cached data behavior, opt into `force-cache` or provide a `revalidate` interval. The explicit `next.revalidate` settings in this app opt into caching regardless of the changed default.

## Streaming

The recipe detail page renders its main recipe independently from the related-recipes section. The related section loads inside React `Suspense`; while its category request is pending, the fallback skeleton streams in with the rest of the page instead of making the whole detail page wait. This improves perceived performance because readers can start viewing the recipe immediately while secondary content loads. In development only, that section waits one second before fetching so the streaming state is easy to see; production has no artificial delay.

## Choosing a data interaction

- **Fetch directly in a Server Component** for page data that only needs to be rendered on the server. It avoids an extra HTTP hop and keeps credentials and data access out of the browser. The search results page follows this approach.
- **Use a Route Handler** when a request needs a public HTTP interface, such as browser autocomplete, third-party integrations, or explicit response status and cache headers. The search API returns a small client-facing result shape.
- **Use a Server Action** for mutations initiated by the app UI, such as submitting a form that creates or updates data. Actions are not a general-purpose read API; use a Route Handler for client autocomplete and a Server Component for server-rendered reads.

## Route rendering

The production build classifies routes as follows:

- `○ /` and `○ /_not-found`: statically rendered. The homepage data is revalidated daily.
- `● /category/[name]`: statically generated for category params and revalidated hourly.
- `ƒ /api/search`: dynamic GET handler. Its query is request-specific; successful responses include HTTP shared-cache headers.
- `ƒ /favourites`: dynamic because it reads the visitor's cookies.
- `ƒ /meal/[id]`: dynamic because IDs are served on demand and the page reads the visitor's favourites cookie.
- `ƒ /search`: dynamic because results depend on the promised `searchParams` value.

The random recipe Server Action also makes a fresh, uncached `random.php` request.

## Deploy to Vercel

Import this repository into [Vercel](https://vercel.com/new), set the project Root Directory to `15-recipe-explorer`, and deploy with the default Next.js build settings. No API key is required for the TheMealDB endpoints used by this app.

Alternatively, install and authenticate the Vercel CLI, then run:

```bash
npm install --global vercel
vercel login
npx vercel
npx vercel --prod
```
