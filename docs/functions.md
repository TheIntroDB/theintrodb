# Functions

## createIntroDbClient(options?)

Creates a reusable TIDB client.

### Signature

```ts
createIntroDbClient(options?: TIDBClientOptions): TIDBClient
```

### Use This When

- You want shared `baseUrl`, `fetch`, `headers`, `logger`, or `apiKey`
- You prefer method-based access: `client.getMedia()` and `client.submitMediaTimestamp()`

## getMedia(params, transportOptions?)

Fetches timestamps for a movie or TV episode.

### Signature

```ts
getMedia(
  params: GetMediaParams,
  transportOptions?: TIDBTransportOptions
): Promise<MediaRecord>
```

### Auth

- No auth required
- Optional current-user API key supported
- When a current-user key is supplied, the API can include that user's pending submissions in the response

### Params

- `tmdbId`: Preferred canonical identifier
- `imdbId`: Optional fallback identifier
- `season`: Required with `episode` for TV episode lookups
- `episode`: Required with `season` for TV episode lookups

### Returns

A normalized `MediaRecord`.

## submitMediaTimestamp(input, transportOptions?)

Submits a single segment timestamp payload.

### Signature

```ts
submitMediaTimestamp(
  input: SubmitMediaTimestampInput,
  transportOptions?: TIDBTransportOptions
): Promise<SubmissionResponse>
```

### Auth

- Always requires the current user's API key
- The key should belong to the end user
- The package throws before making the request if the key is missing

### Input Rules

- Use either seconds or milliseconds, not both
- `intro` and `recap` allow `null` starts
- `credits` and `preview` allow `null` ends
- TV submissions require `season` and `episode`
- Movie submissions must omit `season` and `episode`

## buildMediaQuery(params)

Builds the query string used for `/media`.

### Signature

```ts
buildMediaQuery(params: GetMediaParams): URLSearchParams
```

### Use This When

- You want to inspect or reuse the generated query parameters
- You want package-level validation before making your own fetch call

## serializeSubmissionRequest(input)

Validates and converts a high-level submission input into the raw `/submit` request body.

### Signature

```ts
serializeSubmissionRequest(
  input: SubmitMediaTimestampInput
): SubmissionRequestPayload
```

### Normalization Rules

- `intro` and `recap` starts of `0` or `null` become `start_ms: null`
- `credits` and `preview` null ends stay `end_ms: null`
- seconds are rounded to milliseconds

## parseMediaResponse(body)

Validates raw JSON against the expected `/media` schema and returns a normalized `MediaRecord`.

### Signature

```ts
parseMediaResponse(body: unknown): MediaRecord
```

## parseSubmissionResponse(body)

Validates raw JSON against the expected `/submit` schema and returns a normalized `SubmissionResponse`.

### Signature

```ts
parseSubmissionResponse(body: unknown): SubmissionResponse
```

## normalizeSegmentTimestamp(timestamp)

Normalizes a raw segment timestamp into the package's runtime format.

### Signature

```ts
normalizeSegmentTimestamp(
  timestamp: SegmentTimestampRaw
): NormalizedSegmentTimestamp
```

### Output Behavior

- `start_ms: null` becomes `startMs: 0`
- `end_ms: null` stays `endMs: null`
- `durationMs` is `null` if the end is unknown
