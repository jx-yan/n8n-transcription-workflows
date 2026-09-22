# Podcast audio to text: API quickstart and n8n RSS starter

Turn a permitted podcast recording into searchable text, subtitles or speaker-labelled notes. Start with one short file, check the output, then connect your feed. The examples use paid Apify Actors; the guide and workflow files are free.

## Choose the right Actor

| Your input | Actor | Managed transcription price |
|---|---|---|
| An episode's audio enclosure URL or an accessible audio/video file | [Audio Transcriber](https://apify.com/kaz_kakyo/audio-transcriber) | $0.01 per rounded minute |
| A whole podcast RSS feed, Vimeo/Loom URL or HLS VOD playlist | [Media URL Transcriber](https://apify.com/kaz_kakyo/media-url-transcriber) | $0.012 per rounded STT minute; published Vimeo captions $0.005 per item |

Both add a $0.00005 base start event at up to 1 GB memory. Larger allocations multiply that event. BYOK costs $0.004/min for Audio or $0.005/min for Media **plus your provider's separate bill**. Prices checked September 21, 2026; use each Actor's Pricing tab for current rates.

## First useful output

In [Audio Transcriber](https://apify.com/kaz_kakyo/audio-transcriber/input), paste:

```json
{"audioUrls":["https://dpgr.am/spacewalk.wav"],"includeSrt":true}
```

Run at 512 MB with a $0.05 maximum charge. The public sample is about 26 seconds: one transcript, rounded to one minute, **$0.01005**. Read `transcript` for text and `srt` for subtitles. It contains one speaker; use a permitted interview recording to demonstrate multiple speaker labels with `diarize: true`.

For an API client, install `apify-client`, set `APIFY_TOKEN` securely in your environment, and run:

```js
import { ApifyClient } from 'apify-client';
const client = new ApifyClient({ token: process.env.APIFY_TOKEN });
const run = await client.actor('kaz_kakyo/audio-transcriber').call(
  { audioUrls: ['https://dpgr.am/spacewalk.wav'], includeSrt: true },
  { memory: 512, maxTotalChargeUsd: 0.05 },
);
console.log({ runId: run.id, status: run.status, datasetId: run.defaultDatasetId });
const { items } = await client.dataset(run.defaultDatasetId).listItems();
console.log({
  transcripts: items.filter(x => x.type === 'transcript'),
  errors: items.filter(x => x.type !== 'transcript'),
});
```

An Apify token is required; a separate speech-to-text account is optional. Never paste tokens into a public workflow file or repository. A `SUCCEEDED` run can contain error rows, so check the dataset as well as status. For large batches, paginate the dataset API.

## What a podcast feed does

An RSS episode normally includes an `enclosure.url` pointing to its audio file. Pass that URL to Audio Transcriber. The webpage in `link` is often HTML and is not a substitute for an enclosure.

If you have only the feed URL, Media URL Transcriber accepts:

```json
{"mediaUrls":["YOUR_PODCAST_RSS_URL"],"episodesPerFeed":1,"includeSrt":true}
```

Replace the placeholder with a feed you are allowed to process. **Each new run selects the latest episode again.** This is an on-demand snapshot, not a new-episodes-only subscription.

## n8n: supervised RSS starter

Download [podcast-transcript-workflow.json](./podcast-transcript-workflow.json). It is inactive by default. Install the official Apify community package `@apify/n8n-nodes-apify`, import the JSON, and configure:

1. Feed URL and poll interval in **New episode published**.
2. Apify credential in **Transcribe episode**.
3. Budget for the **entire poll batch**, default $1 at 512 MB. That fits 99 whole managed minutes after the start event, not 100.
4. A persistent destination replacing **Results and errors**, saving both transcript and error rows.
5. An n8n error workflow to make node failures visible. Leave retries off on the paid node.

The preparation step rejects missing/non-HTTP(S) enclosures before paid work and collapses exact GUID or URL repeats within the poll. All accepted URLs enter **one** Actor run, avoiding a separate paid call for every input item. All returned rows are retained, including over-budget and failed files. Duplicate GUIDs keep the first occurrence; signed-URL changes without stable GUIDs can evade URL deduplication.

This starter has **no durable cross-execution deduplication**. It is suitable for a supervised pilot, not an unattended guarantee. A production integration needs a persistent episode ledger, concurrency control, saved Apify run IDs, and explicit reconciliation of uncertain starts and destination writes.

## Replay, restart and failure checks

| Situation | Behavior / recovery |
|---|---|
| Same GUID or exact enclosure repeated in one poll | One submitted URL |
| Missing enclosure or more than 500 unique files | Stops before paid work; recover from saved trigger input |
| Run returns only error/skip rows | Rows remain visible; not silently discarded |
| Empty dataset | Stops with instruction to inspect the existing Apify run |
| n8n request times out after starting a run | Run may still exist and charge; inspect Apify Console before replaying |
| Destination write fails | Recover from the existing dataset; do not start a new transcription |
| Manual rerun, edited feed publication date or lost trigger state | May submit and charge again |
| Backdated episode at/before the RSS cursor | May be missed by date-based polling; reconcile feed against your ledger |
| n8n restart | Persisted trigger state is not a delivery ledger; no exactly-once promise |

Checked September 22, 2026 with **n8n 2.40.5**: the unchanged workflow imported through CLI directory mode, and the real n8n JavaScript runner passed offline cases for batch deduplication, visible transcript/error output, rejection of missing enclosures before the paid step, and a visible empty-output failure. A manual replay submitted the work again, confirming the documented limitation. The offline cases use a manual trigger and local Actor stub; they make no paid calls.

**Still unverified:** a live authenticated call through the Apify community node, live RSS polling, and durable cross-execution/restart recovery. These checks are not end-to-end production certification. Keep the workflow inactive until you test your installed integration and configure both result storage and error handling.

For CLI import on n8n 2.40.5, put only the workflow JSON in a directory and use `n8n import:workflow --separate --input=/path/to/directory`. Directory mode generates a local workflow ID. Single-file CLI import of this ID-less template failed with `NOT NULL constraint failed: workflow_entity.id`; do not add a shared hard-coded ID just to silence it. Editor import was not tested by this check.

## Cost and retry rules

A 45-minute Audio episode costs $0.45005 managed at 512 MB. Four such episodes in four runs cost $1.8002. Minutes round up separately for each file. Optional chapters and your own provider/LLM usage are additional. `maxTotalChargeUsd` limits one run, not all polls, retries or monthly spend.

Charges precede saving output. If a run fails after charging, inspect its log and dataset before retrying. Keep the original run ID and seek recovery through the Actor's issue page where necessary. Never assume an HTTP timeout means no work happened.

## References

- [n8n RSS Feed Trigger](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.rssfeedreadtrigger/)
- [Apify n8n integration source](https://github.com/apify/n8n-nodes-apify)
- [Apify JavaScript client](https://docs.apify.com/api/client/js)
