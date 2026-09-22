# Audio transcription API examples and n8n workflows

Start with the [podcast audio-to-text guide](./podcast-transcription-guide.md): a short runnable sample, Audio versus Media actor choice, transparent costs, and recovery rules for RSS automation.

| Example | Use | Status |
|---|---|---|
| [Podcast guide](./podcast-transcription-guide.md) | One audio file or podcast feed to text and subtitles | Audio/media sample verified live, September 21, 2026 |
| [Podcast RSS workflow](./podcast-transcript-workflow.json) | Poll a feed and submit one capped transcription batch | Manual-trigger live Apify path verified; live RSS polling and restart recovery still unverified |
| [Drive workflow](./drive-folder-transcript-workflow.json) | Drive recordings to text | Actor locator corrected; complete workflow remains unverified |
| [YouTube workflow](./youtube-channel-transcript-workflow.json) | Channel uploads to captions | Actor locator corrected; complete workflow remains unverified |

The files are free; running the linked Apify Actors incurs their published usage charges. You need an Apify account and token, plus the official Apify n8n community integration for the workflows.

## Start safely

1. Run the short sample in the guide and inspect its transcript and subtitles.
2. Import the podcast workflow into your n8n instance and configure feed, credential, batch budget, output destination and error workflow.
3. Test new, duplicate, malformed and failed items before activation. Keep paid-node retries off and recover output from an existing Apify run before starting another. The integration can still retry HTTP429/5xx internally; limits apply separately to each Actor run.

**These templates do not establish exactly-once delivery.** Date-based RSS polling can miss late episodes and replay edited or manually retried items. A production pipeline needs a durable episode/run ledger and reconciliation. The podcast revision preserves error rows; the older Drive/YouTube templates still require a separate failure-handling and replay audit before unattended use. Read the guide's complete test limits.

## Actors

- [Audio Transcriber](https://apify.com/kaz_kakyo/audio-transcriber): direct audio/video files and supported share links to text, SRT and speaker labels.
- [Media URL Transcriber](https://apify.com/kaz_kakyo/media-url-transcriber): podcast RSS, Vimeo, Loom and HLS to text, SRT or VTT.
- [YouTube Transcript Scraper](https://apify.com/kaz_kakyo/youtube-transcripts): published video captions and timestamps.

## License

MIT — use, modify, and redistribute freely.
