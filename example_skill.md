---
name: youtube-transcript
description: Fetch transcripts from YouTube videos via your residential proxy API. Use when Claude needs to get captions/subtitles from YouTube videos for summarisation, analysis, note-taking, or any task requiring video transcript text. Accepts video IDs or full YouTube URLs.
---

# YouTube Transcript Skill

Fetch transcripts from YouTube videos via your residential proxy API.

## API Base URL
`https://your-domain.example.com`

## Authentication
All requests require the `X-Proxy-Token` header:
```
X-Proxy-Token: YOUR_TOKEN_HERE
```

## Endpoints

### Get Transcript with Timestamps
```
GET /transcript/{video}?lang=en
```
Returns full transcript with timing data.

**Response:**
```json
{
  "video_id": "dQw4w9WgXcQ",
  "language": "English",
  "language_code": "en",
  "is_generated": true,
  "segments": [
    {"text": "Never gonna give you up", "start": 0.0, "duration": 2.5},
    ...
  ]
}
```

### Get Plain Text Only
```
GET /transcript/{video}/text?lang=en
```
Returns transcript as plain text without timestamps.

**Response:**
```json
{
  "video_id": "dQw4w9WgXcQ",
  "language": "English",
  "language_code": "en",
  "text": "Never gonna give you up Never gonna let you down..."
}
```

### List Available Languages
```
GET /transcripts/{video}
```
Returns all available transcript languages for a video.

**Response:**
```json
{
  "video_id": "dQw4w9WgXcQ",
  "transcripts": [
    {"language": "English", "language_code": "en", "is_generated": true, "is_translatable": true},
    {"language": "Spanish", "language_code": "es", "is_generated": false, "is_translatable": true}
  ]
}
```

## Video Input Formats
The `{video}` parameter accepts:
- Video ID: `dQw4w9WgXcQ`
- Full URL: `https://www.youtube.com/watch?v=dQw4w9WgXcQ`
- Short URL: `https://youtu.be/dQw4w9WgXcQ`

## Example Usage (Python)

```python
import urllib.request
import json

VIDEO = "dQw4w9WgXcQ"  # or full YouTube URL
LANG = "en"

url = f"https://your-domain.example.com/transcript/{VIDEO}?lang={LANG}"
req = urllib.request.Request(url, headers={
    "X-Proxy-Token": "YOUR_TOKEN_HERE"
})

with urllib.request.urlopen(req, timeout=30) as resp:
    data = json.loads(resp.read().decode())
    
# Save to outputs
with open("/mnt/user-data/outputs/transcript.json", "w") as f:
    json.dump(data, f, indent=2)

# Or save as plain text
text = "\n".join(seg["text"] for seg in data["segments"])
with open("/mnt/user-data/outputs/transcript.txt", "w") as f:
    f.write(text)
```

## Common Use Cases

1. **Summarise a YouTube video** - Fetch transcript, then summarise
2. **Search for specific content** - Fetch transcript, search for keywords
3. **Create notes from lectures** - Fetch transcript, structure into notes
4. **Translate content** - Fetch transcript in original language, translate
5. **Extract quotes with timestamps** - Use full transcript with timing data

## Error Handling

| Status | Meaning |
|--------|---------|
| 401 | Invalid or missing auth token |
| 404 | Video not found or transcripts disabled |
| 500 | Server error (check video exists and has captions) |

## Notes
- Auto-generated captions are available for most videos
- Manual captions (when available) are typically higher quality
- Use `?lang=` to request specific languages (falls back to available)
- This API should run on a residential IP to bypass YouTube's cloud IP blocks