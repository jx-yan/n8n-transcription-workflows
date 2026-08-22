# n8n transcription workflows

Three importable [n8n](https://n8n.io) workflows that turn media into text automatically, using transcription Actors on [Apify](https://apify.com) via the official Apify n8n integration (`@apify/n8n-nodes-apify`). Import once, set two fields, activate — every new item arrives as a finished transcript.

| Workflow | Trigger | What you get |
|---|---|---|
| [`podcast-transcript-workflow.json`](./podcast-transcript-workflow.json) | New episode in a podcast RSS feed | Full transcript of every new episode (~$0.01/audio-minute, e.g. a 45-min episode ≈ $0.45) |
| [`youtube-channel-transcript-workflow.json`](./youtube-channel-transcript-workflow.json) | New video on a YouTube channel (via the channel's RSS feed) | Captions transcript of every new video ($0.005/video) |
| [`drive-folder-transcript-workflow.json`](./drive-folder-transcript-workflow.json) | New audio/video file in a Google Drive folder | Transcript of every recording dropped into the folder (~$0.01/audio-minute) |

## Setup (all three)

1. In n8n, make the Apify node available — n8n Cloud: add the verified Apify node from the nodes panel; self-hosted: **Settings → Community Nodes → Install** → `@apify/n8n-nodes-apify`.
2. **Workflows → Add workflow → Import from File** with the JSON.
3. Edit the two highlighted settings (source feed/folder + your Apify API token credential — free account at [apify.com](https://apify.com), token under Console → Settings → API & Integrations), then **Activate**.

Each workflow contains sticky notes explaining every step, the per-item cost, and the spend cap (`Maximum Cost per Run`) that guarantees no surprise bills — files over budget are skipped, never partially billed.

## Actors used

- [Audio Transcriber](https://apify.com/kaz_kakyo/audio-transcriber) — audio/video file URLs → transcript (speech-to-text, diarization, SRT, summaries; Google Drive/Dropbox/Apple Podcasts share links resolve automatically)
- [YouTube Transcript Scraper](https://apify.com/kaz_kakyo/youtube-transcripts) — YouTube captions (manual or auto-generated), timestamps, SRT

Output contract for both: one JSON row per item with `type: "transcript"` or `type: "error"`; error rows are never billed, and documented fields are additive-only so these workflows don't break on Actor updates.

## License

MIT — use, modify, and redistribute freely.
