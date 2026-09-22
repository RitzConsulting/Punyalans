# Punyalans

Tactical hub for the Punyalans 7-a-side football team — an interactive pitch with
two formations, positional role guides, rotation planning and a team feedback chat.

Live site: https://ritzconsulting.github.io/Punyalans/

Adapted from the Minnals tactical planner, reworked for the Punyalans squad.

## Squad

| Position | Starters | Backups |
| --- | --- | --- |
| Goalkeeper | Anumodh, Sijo | — |
| Defence | Arun, Sujith R, Ajeeb | Rinto, Sujith Joseph, Sijo |
| Midfield | Praveen, Rithesh | Rinto, Ajeeb |
| Forward | Ebin | Jojan |

Eleven players, seven on the pitch. Sijo covers the goal and the back line; Rinto
and Ajeeb cover both defence and midfield, which is what makes the second shape
below possible.

## Formations

**3-2-1** (default) — back three, midfield two, lone striker.

```
        Praveen        Ebin
Arun    Sujith R              (Anumodh)
        Rithesh        Ajeeb
```

**2-3-1** — the centre back pushes into a midfield three.

```
        Praveen
Arun    Rithesh    Ebin       (Anumodh)
Ajeeb   Sujith R
```

Each shape has three rotations (Primary XI, backups, mixed), cycled with the
**🔄 Rotate** button.

## Using the page

- **🛡️ Def / ⚔️ Att** — shift everyone between the defensive and attacking shape.
- **📍 Pass** — step through the tactical passing patterns for the current
  formation (up-back-through, switch play, overlap runs, pullback and reset).
- **Click any player** — role guide: must do, must not, tips, and who swaps in.
- **Team Chat** — post feedback; pinned messages carry the standing instructions.

## Running it

It is a single static `index.html` with no build step. Open the file in a browser,
or serve the folder:

```sh
python -m http.server 8000
```

## Publishing to GitHub Pages

Settings → Pages → Source: *Deploy from a branch* → Branch `main`, folder `/ (root)`.

## Team chat (optional)

Chat works out of the box but is **local to your browser session** — messages
disappear on reload. To make them persist for everyone, create your own Firebase
Realtime Database and paste the config into `index.html` (search for
`PASTE YOUR FIREBASE CONFIG`). The in-app **Setup Firebase** link walks through it.

No Firebase credentials are committed here, because this repository is public.
If you add a config, lock the database rules down first — the default "test mode"
rules let anyone on the internet read and write your chat.

## A note on the login screen

The trivia gate is obfuscation, not security. The page source and the answer hash
are public, so treat everything in this repo as readable by anyone. Don't put
anything private in the chat pins or the roster.

The current answer is the team name, in lowercase. To change it, replace
`ANSWER_HASH` in `index.html` with the SHA-256 of the new lowercase answer:

```sh
printf 'youranswer' | sha256sum
```

## AI features

The **🤖 Rate** and **🤖 Summarize** buttons call the Anthropic API directly from
the browser. That call has no API key and browsers block it by CORS, so both
buttons currently fall back to a friendly message. Wiring them up needs a small
server-side proxy that holds the key — the chat itself is unaffected.
