# TheIntroDB NPM Package

Typed TypeScript client for the TheIntroDB API, generated around the published OpenAPI contract.

Detailed GitHub Pages docs live in `docs/`.

## Install

```bash
npm install theintrodb
```

## Features

- Centralized request, parsing, and validation logic
- Runtime validation of request payloads and API responses
- `GET /media` works without auth and can optionally use a current user API key
- `POST /submit` requires the current user's API key so submissions go to that account
- Normalized timestamp handling keeps `null` end times as "end of media" and converts `null` starts to `0`
- Reusable client or standalone function usage
- Optional logging hooks for auth-sensitive request flows
- Jest-based unit coverage for transport, parsing, and edge cases

## Example Usage

```ts
import { createIntroDbClient } from 'theintrodb';

const client = createIntroDbClient({
  // Optional: pass a logger if you want TIDB request/auth messages.
  // `console` is fine for quick debugging; omit this entirely if you do not want logs.
  logger: console,
});

// Public movie lookup: no API key required.
const publicMedia = await client.getMedia({
  tmdbId: 12345,
});

// Public TV lookup: include `season` and `episode` for a specific episode.
const publicEpisodeMedia = await client.getMedia({
  tmdbId: 67890,
  season: 1,
  episode: 1,
});

// Optional current-user API key:
// this can include that user's still-pending submissions in the response.
const mediaWithMyPendingSubmissions = await client.getMedia(
  {
    tmdbId: 12345,
  },
  {
    apiKey: currentUserApiKey,
  }
);

// Submissions always require the current user's API key.
// The key should belong to the end user so the submission is attached to their account.
const submission = await client.submitMediaTimestamp(
  {
    tmdbId: 12345,
    type: 'movie',
    segment: 'intro',
    startSec: 0,
    endSec: 90,
  },
  {
    apiKey: currentUserApiKey,
  }
);
```

## Auth Model

- `getMedia()` does not require authentication
- `getMedia()` optionally accepts the current user's API key; when provided, the API may include that user's pending submissions in the weighted result
- `submitMediaTimestamp()` always requires the current user's API key
- Do not ship a shared application API key; this package is designed for end-user keys

## API Notes

### Get Media Timestamps

- Use `tmdbId` for movies
- Use `tmdbId`, `season`, and `episode` for TV episodes
- `imdbId` is supported but `tmdbId` remains the preferred canonical identifier
- Segment arrays are omitted by the API when no submissions exist for that segment type

### Submit Timestamps

- Submit one segment at a time to `submitMediaTimestamp()`
- Use either `startSec`/`endSec` or `startMs`/`endMs`, never both
- `intro` and `recap` allow `null` starts
- `credits` and `preview` allow `null` ends, which mean "to the end of media"

## Timestamp Semantics

- `intro` and `recap`: `null` or `0` start values are normalized to `0`
- `credits` and `preview`: `null` end values remain `null`
- Normalized responses include `startsAtBeginning` and `endsAtMediaEnd` flags

## Submission Example

```ts
await client.submitMediaTimestamp(
  {
    tmdbId: 12345,
    type: 'tv',
    season: 1,
    episode: 1,
    segment: 'credits',
    startSec: 5400,
    endSec: null,
  },
  {
    apiKey: currentUserApiKey,
  }
);
```

## Important

- User API keys are optional for reads and required for submissions
- When a user supplies their API key to `getMedia()`, they can see pending submissions tied to their own account
- This package validates payloads locally before making requests and validates JSON responses after receiving them

## Build

```bash
npm run build
npm test
```
