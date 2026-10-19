---
name: youtube-textnotes
description: Fetch YouTube video metadata via yt-dlp and save it to textnotes. Use when user provides a YouTube URL and wants to save channel, duration, cover, release date, and URL as a note.
---

# YouTube → Textnotes

This skill wraps `textnotes` for YouTube URLs. Use `yt-dlp` to fetch metadata, then `textnotes create` with mapped fields.

## Workflow

1.  **Fetch metadata** with `yt-dlp` (installed):
    ```bash
    yt-dlp --dump-json --no-playlist "<youtube_url>" 2>/dev/null | python3 -c "
    import json,sys
    d=json.load(sys.stdin)
    print('title:',d.get('title'))
    print('channel:',d.get('channel') or d.get('uploader'))
    print('duration:',d.get('duration')) # seconds
    print('thumbnail:',d.get('thumbnail'))
    print('upload_date:',d.get('upload_date')) # YYYYMMDD
    print('url:',d.get('webpage_url'))
    print('description:',d.get('description'))
    "
    ```

2.  **Map fields**:
    - `title` → `--title`
    - `description` → `--content` (save as note content, markdown)
    - `channel` / `uploader` → `--fields-author`
    - `duration` (seconds) → minutes: `round(duration / 60, 2)` → `--fields-duration` (e.g. `64` → `1.07`)
    - `thumbnail` (or `thumbnails[-1].url` fallback, e.g. `maxresdefault.webp`) → `--fields-cover`
    - `upload_date` `YYYYMMDD` → `YYYY-MM-DD` (`20170521` → `2017-05-21`) → `--fields-release-date`
    - `webpage_url` → `--url` (dedicated flag, not `fields`)
    - always add `--fields-entry-klass "Video"` → (always set, not from yt-dlp)

3.  **Create note**:
    ```bash
    textnotes create --title "<title>" --url "<webpage_url>" --content "<description>" \
      --fields-entry-klass "Video" \
      --fields-author "<channel>" \
      --fields-duration "<minutes>" \
      --fields-cover "<thumbnail_url>" \
      --fields-release-date "<YYYY-MM-DD>"
    ```

    Example from `https://www.youtube.com/watch?v=iTHtXgnhqfM`:
    ```bash
    textnotes create --title "Dolby Atmos 5.1 Surround Sound Test" \
      --url "https://www.youtube.com/watch?v=iTHtXgnhqfM" \
      --content "Amaze - Dolby Atmos\nTry it for your home theater soundbars and subwoofers!" \
      --fields-author "JD | Media" \
      --fields-duration "1.07" \
      --fields-cover "https://i.ytimg.com/vi/iTHtXgnhqfM/maxresdefault.jpg" \
      --fields-release-date "2017-05-21"
    ```

4.  **Verify**:
    ```bash
    textnotes --format json read --title "<title>" | python3 -m json.tool
    textnotes --format json search "<keyword>" | python3 -m json.tool
    ```

## Tips

- Quote titles with special chars: `--title "Dolby Atmos 5.1 Surround Sound Test"`
- If `thumbnail` is null, use `thumbnails` array last element: `python3 -c "import json; d=json.load(open(...)); print(d['thumbnails'][-1]['url'])"`
- Always convert `duration` and `upload_date` before passing to `textnotes`; store duration as string minutes.
- Use `--format json` for verification to see `fields` object.
