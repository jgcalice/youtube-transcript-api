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

This repo includes an `example_skill.skill` file for use with Claude's computer use / skills feature.

### Setup

1. Deploy this service on your residential IP (or any IP not blocked by YouTube)
2. Expose via Cloudflare Tunnel or similar
3. Add your domain to Claude's allowed network list in settings
4. Edit the skill file to add your domain and auth token
5. Import the skill into Claude

### Example Prompts

Once the skill is installed, you can ask Claude things like:

**Summarisation**
- "Summarise this YouTube video: https://www.youtube.com/watch?v=dQw4w9WgXcQ"
- "Give me the key points from this lecture: [URL]"
- "What are the main arguments in this video?"

**Note Taking**
- "Create study notes from this YouTube tutorial: [URL]"
- "Turn this video into bullet points I can review later"
- "Extract the step-by-step instructions from this how-to video"

**Search & Analysis**
- "Does this video mention anything about [topic]?"
- "Find all the timestamps where they discuss [subject]"
- "What questions does the speaker answer in this video?"

**Content Extraction**
- "Get me the transcript from this video"
- "What languages are available for this video's captions?"
- "Extract quotes from this interview about [topic]"

**Translation & Accessibility**
- "Get the Spanish transcript for this video"
- "Translate the key points from this video into French"

### Why This Exists

YouTube blocks requests from cloud IPs (like Claude's sandbox), so fetching transcripts directly fails. This service runs on your residential IP and proxies the requests, making YouTube transcripts accessible to Claude.

### Skill File Contents

The skill teaches Claude:
- Which API endpoints to call
- How to authenticate requests
- What response formats to expect
- How to handle errors

You'll need to edit `example_skill.skill` (it's just a zip file) to replace:
- `https://your-domain.example.com` → your actual domain
- `YOUR_TOKEN_HERE` → your `PROXY_AUTH_TOKEN` value

## License

MIT
