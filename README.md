# Audiera Skill

AI music creation skill for OpenClaw — generate lyrics and complete songs with vocals via Audiera platform.

## Install

```bash
openclaw skill install github:audiera/audiera-skills
```

## Configure

1. Get an API key from [Audiera Settings](https://ai.audiera.fi/settings/api-keys) with scopes:
   - `lyrics_generate` — for lyrics generation
   - `music_generate` — for music generation

2. Add to your OpenClaw config (`~/.openclaw/openclaw.json`):

```json
{
  "skills": {
    "entries": {
      "audiera": {
        "enabled": true,
        "env": {
          "AUDIERA_API_KEY": "sk_audiera_your_key_here"
        }
      }
    }
  }
}
```

Or set as environment variable:

```bash
export AUDIERA_API_KEY="sk_audiera_your_key_here"
```

## Usage

### Weekly Leaderboard Data
```
> Show me Audiera's weekly leaderboard
```

### Chain Stats Data
```
> Show me Audiera's latest chain stats
```

### Lyrics Generation
```
> Write lyrics about summer love and beach sunsets
```

### Song Generation
```
> Write lyrics about chasing dreams, then generate a pop song with artist Kira
> Create an electronic track with artist Ray using these lyrics: [Verse] ...
```

The music API accepts either full lyrics or just an inspiration/theme. The skill waits for the async music task to finish, polls the task status every 5 seconds for up to 5 minutes, then returns the final generated music data. Completed tasks may return more than one song in `data.musics`, so the skill should prefer `data.musics` and fall back to `data.music` only if needed. If the task does not finish in time, it should tell the user generation failed or timed out.

For song generation, `styles` is required. Supported values are `Pop`, `Rock`, `Hip-Hop`, `Country`, `Dance`, `Electronic`, `Disco`, `Blues`, `Jazz`, `Folk`, `Latin`, `Metal`, `Punk`, `R&B`, `Soul`, `Funk`, `Reggae`, `Indie`, `Afrobeat`, `Classical`, and `World-music`. Prefer 1 to 3 styles. If the user does not name a style and the prompt does not imply one clearly, default to `Pop`.

## API Reference

| Capability | Endpoint | Method |
|---|---|---|
| Generate Lyrics | `https://ai.audiera.fi/api/skills/lyrics` | POST |
| Get Weekly Leaderboard Data | `https://ai.audiera.fi/api/skills/weekly-leaderboard` | GET |
| Get Chain Stats Data | `https://ai.audiera.fi/api/skills/chain-stats` | GET |
| Create Song | `https://ai.audiera.fi/api/skills/music` | POST |
| Poll Song Status | `https://ai.audiera.fi/api/skills/music/{taskId}` | GET |

See [SKILL.md](./SKILL.md) for full API details.

## Support

- Website: https://ai.audiera.fi
- Issues: https://github.com/audiera/audiera-skills/issues

## License

MIT
