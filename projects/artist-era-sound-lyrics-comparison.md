# Artist/era sound & lyrics comparison tool

**Status:** exploring
**Started:** 2026-09-15
**Tags:** #python #ml #audio #nlp #streamlit #music

## What is it?
A tool that analyzes actual audio (ripped from CDs) to extract real audio features (tempo, key, energy, etc.) per song, and pulls lyrics (via Genius) for NLP-based comparison (sentiment, topic/theme modeling, embedding-based similarity). Computes both per album ("era"), and lets you compare artists/eras on sound, lyrics, and a combined score — e.g. testing whether early Katy Perry's sound and lyrical style are genuinely closer to early Pink's than to her own later work. Browsable via a Streamlit dashboard.

## Why
Sparked by a conversation about whether early Katy Perry and early Pink sound alike. Spotify's audio-features API (tempo/energy/danceability/valence etc.) was deprecated in November 2024 with no official replacement, so getting real sonic data means analyzing actual audio directly rather than relying on an API. Wanted a genuine ML/audio-analysis project, not just an API integration.

## Brainstorming
- **Why not the Spotify API directly:** the audio-features endpoint (the exact data this project needs) is deprecated for new apps, no path back. Confirmed no official alternative exists as of the research done for this note. Third-party scrapers/APIs claiming to restore it exist but sit in a legal/ToS gray area and aren't a durable foundation.
- **Chosen approach:** analyze real audio directly using Essentia (open-source C++/Python audio analysis library) — computes tempo, key, loudness, energy, and more from actual audio signals, no API dependency. Essentia also ships pretrained deep-learning models for higher-level descriptors (genre, mood, danceability), which is a natural on-ramp into actually using ML models, not just signal-processing math.
- **Audio source:** ripping CDs (sister has a large collection) — need a USB CD/DVD drive. Personal, non-distributed analysis of physically-owned media.
- **Era definition:** era = album. Compute average feature values across all tracks on an album, compare those album-level averages between artists/albums.
- **Dashboard:** Streamlit — chosen deliberately as a new tool to learn, well-suited to data/comparison dashboards, pure Python so it pairs naturally with the Essentia analysis code.
- **Licensing note:** Essentia is AGPLv3 — relevant if this were ever hosted as a shared service (must share source), not a concern for personal local use.
- **Lyrics source:** Genius's official API provides song metadata and a link to the lyrics page, not the lyrics text itself — actual lyrics require scraping the Genius webpage. Common practice (established libraries like `lyricsgenius` do this), but technically outside their official API terms. Low-stakes for a personal project, worth knowing going in.
- **Lyrics comparison approach:** all three angles wanted — sentiment analysis (emotional tone), topic/theme modeling (what songs are about), and embedding-based similarity (overall semantic closeness) — computed per track, then averaged/aggregated per album (era), same pattern as the audio features
- **Combining audio + lyrics:** both separate and combined views wanted — show sonic similarity and lyrical similarity as distinct scores/breakdowns, plus one overall combined similarity score blending both

## Specs / requirements
- **Language:** Python
- **Audio analysis:** Essentia (Python bindings) — extract tempo, key, energy, loudness, and other descriptors per track; consider Essentia's pretrained models for genre/mood/danceability as a stretch addition
- **Audio source:** ripped CD tracks (via USB CD/DVD drive), stored locally
- **Aggregation:** compute per-album (era) averages from per-track features
- **Comparison:** some similarity measure between two albums'/artists' era-average feature vectors (e.g. simple distance metric to start, could grow into a proper ML similarity/clustering model later)
- **Dashboard:** Streamlit app to browse artists/albums and view comparisons
- **Lyrics:** pulled via Genius (metadata via official API, lyrics text via scraping the lyrics page, e.g. using `lyricsgenius`)
- **NLP analysis:** sentiment analysis, topic/theme modeling, and embedding-based similarity per track, aggregated per album
- **Combined score:** a blended similarity score combining audio-feature similarity and lyrics similarity, shown alongside each individually

## Key decisions (resolved)

| Question | Decision |
|---|---|
| Feature/data storage | SQLite — better structural fit than CSV given the relational shape (artist → album → track, plus lyrics scores and combined scores), portable as a single file, works well with pandas/Streamlit |
| Ripped audio files | Kept as a personal backup (OneDrive), separate from the project itself — project works off a temporary local copy during processing, not the backup |

## Open questions
- Similarity metric: simple numeric distance (e.g. Euclidean/cosine on the averaged feature vector) vs. a more proper ML approach (clustering, learned similarity) — could start simple and grow into the ML angle
- How much of Essentia's deep-learning model set (genre/mood/danceability classifiers) to incorporate vs. sticking to core signal-processing descriptors for v1
- File organization/naming convention for ripped tracks so artist/album/era grouping is easy to automate
- Which sentiment/topic-modeling/embedding libraries to use for the lyrics side (e.g. a pretrained sentence-transformers model for embeddings, a topic modeling approach like LDA or a simpler keyword/theme extraction)
- How to weight audio vs. lyrics in the combined score — equal weighting, or user-adjustable?
- Handling instrumental tracks (no lyrics) — exclude from lyrics comparison, or note as a gap in that album's lyrics profile?

## Links
- Essentia: https://essentia.upf.edu/
- Streamlit: https://streamlit.io/
- Genius API: https://docs.genius.com/
