---
name: audiera
description: AI music creation — generate lyrics and complete songs with vocals via Audiera platform
user-invocable: true
metadata: {"openclaw":{"emoji":"🎵","homepage":"https://ai.audiera.fi","primaryEnv":"AUDIERA_API_KEY","requires":{"env":["AUDIERA_API_KEY"]}}}
---

# Audiera AI Music Creation

You have access to Audiera's AI music creation platform. You can generate lyrics and create complete songs with vocals.

## Authentication

All requests require the `AUDIERA_API_KEY` environment variable, passed as a Bearer token:

```
Authorization: Bearer $AUDIERA_API_KEY
```

## Capabilities

### 1. Generate Lyrics

When the user asks to write, generate, or create lyrics, call the lyrics API.

**Endpoint:** `POST https://ai.audiera.fi/api/skills/lyrics`

**Request:**
```json
{
  "inspiration": "<user's theme, idea, or inspiration>"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "lyrics": "<generated lyrics>"
  }
}
```

**Example flow:**
- User: "Write lyrics about summer love"
- Extract the theme: "summer love"
- Call the API with `{"inspiration": "summer love"}`
- Return the generated lyrics to the user

### 2. Get Weekly Leaderboard Data

When the user asks for the current Audiera weekly leaderboard, trending songs, top played songs, community favorite, or rising track, call the weekly leaderboard API.

**Endpoint:** `GET https://ai.audiera.fi/api/skills/weekly-leaderboard`

**Response:**
```json
{
  "success": true,
  "data": {
    "topPlayed": [
      {
        "id": 555,
        "title": "City's Pulse",
        "plays": 9789,
        "likes": 411,
        "playsGrowth": 9789,
        "url": "https://ai.audiera.fi/music/555"
      }
    ],
    "communityFavorite": {
      "id": 3383,
      "title": "In His Eyes",
      "plays": 70,
      "likes": 4,
      "url": "https://ai.audiera.fi/music/3383"
    },
    "risingTrack": {
      "id": 555,
      "title": "City's Pulse",
      "plays": 9789,
      "likes": 411,
      "playsGrowth": 9789,
      "url": "https://ai.audiera.fi/music/555"
    }
  }
}
```

Use this endpoint to return the structured leaderboard data to the user. Do not generate or post a tweet.

### 3. Get Chain Stats Data

When the user asks for Audiera platform growth, chain stats, user totals, NFT totals, or BEAT burn data, call the chain stats API.

**Endpoint:** `GET https://ai.audiera.fi/api/skills/chain-stats`

**Response:**
```json
{
  "success": true,
  "data": {
    "totalUsers": 5415609,
    "lastWeekActiveUsers": 314394,
    "nftTotalSupply": 1625083,
    "lastWeekBurnedBeat": 397900,
    "totalBurnedBeat": 4344729
  }
}
```

Use this endpoint to return the structured chain stats data to the user. Do not generate or post a tweet.

## Available Artists

Use the following fixed artist mapping when the user names a singer/voice. If the user provides a name, map it to the corresponding `artistId`. If the user provides an `artistId` directly, use it as-is.

| Artist | artistId |
|---|---|
| Jason Miller | `i137z0bj0cwsbzrzd8m0c` |
| Dylan Cross | `xa6h1wjowcyvo1r87x1np` |
| Trevor Knox | `woujl3ws7l9z6df4p0y03` |
| Rhea Monroe | `yinjs025l733tttxgy2w5` |
| Kira | `osipvytdvxuzci9pn2nz1` |
| Ray | `jyjcnj6t3arzzb5dnzk4p` |
| Kayden West | `lwyap77qryxy4xz8vo11m` |
| Talia Brooks | `hcqa005jz02ikis7xt2q4` |
| Sienna Blake | `yk5reyqu4ko46k5knx4w9` |
| Briana Rose | `tzuww7dbsh4enwifaptfl` |
| Leo Martin | `udst8rsngyccqh3e2y80a` |
| Amelia Clarke | `9j282m4kbg1dukqabas3h` |

### 4. Generate Song

When the user asks to generate, create, or make a song/music, call the music API. You can send full lyrics directly, or send only `inspiration` and let the API generate lyrics internally before creating the song. Music generation itself is a two-step process: create a task, then poll for completion.

Important behavior:
- Do not stop after returning `taskId`.
- Stay in the same skill run and keep polling until the song is completed.
- Only return the final song URL to the user after polling succeeds.
- If the song is still not completed after 5 minutes, tell the user generation failed or timed out and ask them to try again later.

Style rules:
- `styles` is required for song generation
- Supported styles: `Pop`, `Rock`, `Hip-Hop`, `Country`, `Dance`, `Electronic`, `Disco`, `Blues`, `Jazz`, `Folk`, `Latin`, `Metal`, `Punk`, `R&B`, `Soul`, `Funk`, `Reggae`, `Indie`, `Afrobeat`, `Classical`, `World-music`
- Prefer 1 to 3 styles
- If the user clearly names one or more styles, use those
- If the user does not specify a style but the prompt strongly implies one, infer a reasonable style from the request
- If no style is given and no strong inference is possible, default to `Pop`

**Step 1 — Create task**

**Endpoint:** `POST https://ai.audiera.fi/api/skills/music`

**Request:**
```json
{
  "styles": ["<style1>", "<style2>"],
  "artistId": "<artist_id>",
  "lyrics": "<optional full lyrics text>",
  "inspiration": "<optional theme or short summary>"
}
```

At least one of `lyrics` or `inspiration` must be provided.

Supported styles include: `Pop`, `Rock`, `Hip-Hop`, `Country`, `Dance`, `Electronic`, `Disco`, `Blues`, `Jazz`, `Folk`, `Latin`, `Metal`, `Punk`, `R&B`, `Soul`, `Funk`, `Reggae`, `Indie`, `Afrobeat`, `Classical`, `World-music`.

**Response:**
```json
{
  "success": true,
  "data": {
    "taskId": "<task_id>",
    "pollUrl": "/api/skills/music/<task_id>"
  }
}
```

**Step 2 — Poll for result**

**Endpoint:** `GET https://ai.audiera.fi/api/skills/music/<task_id>`

Poll every 5 seconds until status is `completed` or timeout after 5 minutes.

Polling rules:
- Maximum attempts: 60
- Interval: 5 seconds
- Success condition: `data.status === "completed"` and either `data.musics.length > 0` or `data.music.url` exists
- Failure condition: the API returns `success: false`, or 5 minutes pass without completion
- Do not return raw `taskId` to the user unless they explicitly ask for it
- On timeout, tell the user the song is still processing or timed out and they should try again later
- Prefer `data.musics` when present
- Fall back to `data.music` only if `data.musics` is missing
- A completed task may return more than one generated song

**Response (completed):**
```json
{
  "success": true,
  "data": {
    "status": "completed",
    "music": {
      "id": 12345,
      "title": "Song Title",
      "url": "https://ai.audiera.fi/music/12345",
      "fileUrl": "https://cdn.audiera.fi/...",
      "duration": 180
    },
    "musics": [
      {
        "id": 12345,
        "title": "Song Title",
        "url": "https://ai.audiera.fi/music/12345",
        "fileUrl": "https://cdn.audiera.fi/...",
        "duration": 180
      },
      {
        "id": 12346,
        "title": "Song Title",
        "url": "https://ai.audiera.fi/music/12346",
        "fileUrl": "https://cdn.audiera.fi/...",
        "duration": 176
      }
    ]
  }
}
```

**Example flow:**
- User: "Generate a pop song about dreams"
- If lyrics are missing, either call the lyrics API first or send `inspiration` only and let the music API generate lyrics internally
- Ask which artist/voice they want if not provided
- Map the artist name to the fixed `artistId` list above
- Determine `styles` from the user's request, or default to `Pop` if no style is specified
- Call the music create endpoint with `styles`, `artistId`, and at least one of `lyrics` or `inspiration`
- Call the create endpoint and capture `taskId`
- Poll until completed or timeout
- If `musics` contains multiple songs, return all of them to the user in a clear list
- If only `music` is present, return that single song title, link, and duration

## Error Handling

If the API returns `success: false`, show the `message` field to the user. Common errors:
- 401: Invalid API key — ask the user to check their AUDIERA_API_KEY
- 403: Insufficient permissions — the API key needs `lyrics_generate` or `music_generate` scope (get your key at https://ai.audiera.fi/settings/api-keys)
- 400: Missing required parameters
- Timeout: if polling does not reach `completed` within 5 minutes, tell the user generation failed or timed out and they should try again later
