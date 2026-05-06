# TheIntroDB NPM Package

[![npm package][npm-img]][npm-url]
[![Build Status][build-img]][build-url]
[![Downloads][downloads-img]][downloads-url]
[![Code Coverage][codecov-img]][codecov-url]

Typed TypeScript client for the TheIntroDB API, generated around the published OpenAPI contract.

Detailed documentation can be found at: https://theintrodb.github.io/theintrodb-npm/

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

## How The API Returns Times

TheIntroDB returns media timestamps from `GET /media` in arrays of objects shaped like:

```json
{
  "start_ms": 30000,
  "end_ms": 90000
}
```

Important details:

- The API returns response times in milliseconds, using the raw fields `start_ms` and `end_ms`
- Each segment type such as `intro`, `recap`, `credits`, and `preview` is returned as an array
- `start_ms: null` means the segment starts at the beginning of the media
- `end_ms: null` means the segment continues to the end of the media
- A segment array may be omitted entirely if no submissions exist for that segment type

This package normalizes those raw values into:

- `startMs`
- `endMs`
- `durationMs`
- `startsAtBeginning`
- `endsAtMediaEnd`

Example raw API response fragment:

```json
{
  "intro": [
    {
      "start_ms": null,
      "end_ms": 90000
    }
  ],
  "credits": [
    {
      "start_ms": 1800000,
      "end_ms": null
    }
  ]
}
```

Normalized package result:

```ts
{
  intro: [
    {
      startMs: 0,
      endMs: 90000,
      durationMs: 90000,
      startsAtBeginning: true,
      endsAtMediaEnd: false,
    },
  ],
  credits: [
    {
      startMs: 1800000,
      endMs: null,
      durationMs: null,
      startsAtBeginning: false,
      endsAtMediaEnd: true,
    },
  ],
}
```

For submissions, the API accepts either:

- `startSec` and `endSec` in seconds with decimal precision
- `startMs` and `endMs` in integer milliseconds

The package converts second-based inputs into millisecond request values before sending them to TheIntroDB.

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

[build-img]:https://github.com/theintrodb/theintrodb/actions/workflows/release.yml/badge.svg
[build-url]:https://github.com/theintrodb/theintrodb/actions/workflows/release.yml
[downloads-img]:https://img.shields.io/npm/dt/theintrodb
[downloads-url]:https://www.npmtrends.com/theintrodb
[npm-img]:https://img.shields.io/npm/v/theintrodb
[npm-url]:https://www.npmjs.com/package/theintrodb
[codecov-img]:https://codecov.io/gh/theintrodb/theintrodb/branch/main/graph/badge.svg
[codecov-url]:https://codecov.io/gh/theintrodb/theintrodb
