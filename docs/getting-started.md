# Getting Started

## Install

```bash
npm install theintrodb
```

## Import

```ts
import {
  createIntroDbClient,
  getMedia,
  submitMediaTimestamp,
  type TIDBClient,
  type TIDBClientOptions,
} from 'theintrodb';
```

## Create A Client

```ts
const client = createIntroDbClient({
  logger: console,
});
```

### Client Options

- `baseUrl`: Override the API base URL. Defaults to `https://api.theintrodb.org/v2`.
- `apiKey`: Optional current-user API key. Useful if you want a client instance that always acts on behalf of one user.
- `headers`: Additional headers merged into each request.
- `fetch`: Custom fetch implementation for tests, Node runtimes, or other environments.
- `logger`: Optional TIDB logger. Pass `console` for simple logging, or omit it to disable logs.

## Read Media Data

### Movie Example

```ts
const movie = await client.getMedia({
  tmdbId: 12345,
});
```

### TV Episode Example

```ts
const episode = await client.getMedia({
  tmdbId: 67890,
  season: 1,
  episode: 1,
});
```

### Optional Current-User Key

`getMedia()` is public. If you provide the current user's API key, the API can include that user's pending submissions in the weighted result.

```ts
const media = await client.getMedia(
  {
    tmdbId: 12345,
  },
  {
    apiKey: currentUserApiKey,
  }
);
```

## Submit Timestamps

`submitMediaTimestamp()` always requires the current user's API key.

```ts
const result = await client.submitMediaTimestamp(
  {
    tmdbId: 12345,
    type: 'movie',
    segment: 'intro',
    startSec: 30.5,
    endSec: 90.2,
  },
  {
    apiKey: currentUserApiKey,
  }
);
```

## Time Format Rules

Use exactly one format:

- `startSec` and `endSec`
- `startMs` and `endMs`

Do not mix seconds and milliseconds in the same submission.

## Timestamp Semantics

- `intro` and `recap`: `null` or `0` starts are normalized to `0`
- `credits` and `preview`: `null` ends stay `null`
- Normalized results include:
  - `startsAtBeginning`
  - `endsAtMediaEnd`
  - `durationMs`

## Logging

Logging is optional.

You can omit the logger entirely:

```ts
const quietClient = createIntroDbClient();
```

Or pass `console`:

```ts
const debugClient = createIntroDbClient({
  logger: console,
});
```

The package logs when:

- `getMedia()` uses a current-user API key
- `submitMediaTimestamp()` sends a submission with a current-user API key
- `submitMediaTimestamp()` is called without a required user API key
