# TV Guide prototype

A spoiler-safe AI guide for what you're watching: explains the scene, catches you up, and answers questions about cast and crew. This prototype is the **answer engine plus a test page**. It does not run on Fire TV yet.

## Run it

```bash
cd tv-guide
npm start            # http://localhost:3000
npm test             # spoiler-clamp tests
```

No dependencies, Node 20+. With no keys it runs on a fictional mock catalogue ("The Lighthouse Keepers", "Paper Moons") and a stub answerer, so you can see exactly what the model would be given.

To go live, copy `.env.example` to `.env` and add:

- `TMDB_READ_TOKEN` (or `TMDB_API_KEY`) for real titles, cast, crew and episode synopses
- `ANTHROPIC_API_KEY` for real answers

## How the spoiler clamp works

The model never sees anything the viewer hasn't reached. `src/context.js` drops later seasons and episodes before building the prompt, and `src/data.js` doesn't even fetch seasons beyond the current one. The current episode's synopsis is included but labelled as covering the whole episode, together with how far through the viewer is. `test/context.test.js` passes in the full show and asserts that future twists don't appear.

## Layout

```
server.js          HTTP server + JSON API
src/context.js     spoiler clamp, system prompt, message builder
src/data.js        loads title + only the seasons that may be reached
src/tmdb.js        TMDB client (mock fallback)
src/llm.js         Anthropic Messages API call (mock fallback)
src/mock-data.js   fictional catalogue
public/index.html  test page: pick title, episode, position, then ask
```

## Known limitations

- **No video access.** Answers come from synopses and metadata, not the screen. Scene-level answers are inferences, and the model is told to say so.
- **Within-episode precision is coarse.** TMDB has episode-level synopses only, so "what's happening now" relies on the position percentage. Timed subtitles (where you have the rights) would be the next big accuracy gain.
- **Cast lists are series-wide.** Character names can hint at future arcs. The context labels this and the prompt forbids volunteering unseen characters, but it is a prompt-level guard, not a hard filter.
- **Playback detection isn't built.** On Fire TV the position would come from the active media session; here you set it by hand.
- **Keep API keys on the server.** Never ship them inside the Fire TV app.
