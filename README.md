# YouTube Transcript API

A simple REST API service that fetches YouTube video transcripts. Designed to run on a residential IP to bypass YouTube's cloud IP blocks.

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/health` | Health check |
| GET | `/transcript/{video}` | Get transcript with timestamps |
| GET | `/transcript/{video}/text` | Get transcript as plain text |
| GET | `/transcripts/{video}` | List available transcripts |

## Quick Start

```bash
# Build
docker build -t youtube-transcript-api .

# Run locally
docker run -p 8000:8000 -e PROXY_AUTH_TOKEN=your-token youtube-transcript-api

# Test
curl -H "X-Proxy-Token: your-token" http://localhost:8000/health
```

## Usage Examples

### Get transcript with timestamps

```bash
curl -H "X-Proxy-Token: your-token" \
  "https://yt-transcript.yourdomain.com/transcript/dQw4w9WgXcQ"
```

Response:
```json
{
  "video_id": "dQw4w9WgXcQ",
  "language": "English",
  "language_code": "en",
  "is_generated": true,
  "segments": [
    {"text": "We're no strangers to love", "start": 18.0, "duration": 3.5},
    ...
  ]
}
```

### Get plain text transcript

```bash
curl -H "X-Proxy-Token: your-token" \
  "https://yt-transcript.yourdomain.com/transcript/dQw4w9WgXcQ/text"
```

Response:
```json
{
  "video_id": "dQw4w9WgXcQ",
  "language": "English", 
  "language_code": "en",
  "text": "We're no strangers to love You know the rules and so do I..."
}
```

### List available transcripts

```bash
curl -H "X-Proxy-Token: your-token" \
  "https://yt-transcript.yourdomain.com/transcripts/dQw4w9WgXcQ"
```

### Specify language

```bash
curl -H "X-Proxy-Token: your-token" \
  "https://yt-transcript.yourdomain.com/transcript/dQw4w9WgXcQ?lang=es"
```

### Use full YouTube URL

```bash
curl -H "X-Proxy-Token: your-token" \
  "https://yt-transcript.yourdomain.com/transcript/https://www.youtube.com/watch?v=dQw4w9WgXcQ"
```

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `PROXY_AUTH_TOKEN` | Yes | `changeme` | Auth token for API access |
| `UPSTREAM_PROXY` | No | - | Optional upstream proxy URL |

## Deploy with Cloudflare Tunnel

Add to your tunnel config:
```yaml
- hostname: yt-transcript.yourdomain.com
  service: http://youtube-transcript-api:8000
```

## Claude Skill Integration

Add the domain to Claude's allowed network list, then use from skills:

```python
import urllib.request
import json

def get_transcript(video_id, lang="en"):
    url = f"https://yt-transcript.yourdomain.com/transcript/{video_id}?lang={lang}"
    req = urllib.request.Request(url, headers={"X-Proxy-Token": "your-token"})
    with urllib.request.urlopen(req, timeout=30) as resp:
        return json.loads(resp.read().decode())
```

## License

MIT
