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
- **📍 Pass** — walk through the tactical passing patterns for the current
  formation (up-back-through, switch play, overlap runs, pullback and reset).
  Each pattern runs slowly, and the pass lines stay on the pitch as the move
  builds, so you can see where everyone went. To change the pace, edit
  `PASS_SPEED` in `index.html` — higher is slower, `1` is the original speed.
- **Click any player** — role guide: must do, must not, tips, and who swaps in.
- **Team Chat** — shared across the whole team. Put your name in the box once
  (it is remembered, and required) and post; everyone else sees it live, on any
  device.

## Running it

It is a single static `index.html` with no build step. Open the file in a browser,
or serve the folder:

```sh
python -m http.server 8000
```

## Publishing to GitHub Pages

Settings → Pages → Source: *Deploy from a branch* → Branch `main`, folder `/ (root)`.

## Team chat

Messages are shared. They live in a Firebase Realtime Database
(`punyalans-chat`), so what one person types appears for everyone, on any device,
and survives a reload. Nobody needs an account or a login — type a name and post.

GitHub Pages serves static files and has no server of its own, so a shared chat
has to live somewhere that accepts writes. That is all the database is for.

### Why the keys are in the source

The values in `fbCfg` are not secrets. Every Firebase web app ships them in
public JavaScript; they identify the project, they do not grant access. Access is
decided by `firebase-rules.json`, which is deployed on the database itself:

- anyone may **read** the chat, and **append** a message;
- nobody may **edit or delete** one, not even the person who wrote it;
- a message must have an author, text, a time and a timestamp, and nothing else;
- author is capped at 40 characters, text at 500;
- everything outside `punyalans/chat/messages` is unreachable — no reading the
  database root, no writing anywhere else.

Since anyone who finds the page can post, treat the chat as public. It is for
talking about the shape, not for anything private.

### GitHub will flag the API key — that is expected

GitHub secret scanning raises a "Google API Key" alert on `apiKey` in `index.html`.
It is not a leak. Firebase web apps are built to ship that value publicly: it names
the project, it does not grant access to anything. Rotating it would publish a new
one on the next deploy and change nothing.

What was checked on this specific key, rather than assumed:

| Tried with the key | Result |
| --- | --- |
| Delete or edit chat messages | denied — the database rules decide, not the key |
| Create an account (Firebase Auth) | `CONFIGURATION_NOT_FOUND` — Auth is not enabled |
| Read Firestore | 404 — no Firestore database exists |
| Read the storage bucket | 404 — no bucket provisioned |
| Call an unrelated Google API (Maps) | refused — the key is scoped to Firebase services |
| Run up a bill | no billing account is attached to the project |

The key is additionally restricted to referrers `ritzconsulting.github.io` and
localhost, so it will not work from someone else's page.

Close the alert as **"Won't fix"** rather than "Revoked" — the key is real and
still in use, it is just not secret.

### Editing the rules

`firebase-rules.json` is the source of truth. To change it:

```sh
curl -X PUT "https://punyalans-chat-default-rtdb.firebaseio.com/.settings/rules.json" \
  -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  --data-binary @firebase-rules.json
```

Deleting a message needs owner rights, which the rules deliberately do not grant
to the page — do it from the Firebase console, or with an access token.

If the database is ever unreachable the page says so and keeps your messages on
your own device for that session.

## A note on the login screen

The gate is a riddle with a one-word answer. It is obfuscation, not security:
the page source and the answer hash are public, so treat everything in this repo
as readable by anyone. Don't put anything private in the chat or the roster.

The answer is deliberately not written down here, and the team name is kept off
the login screen — naming the team there would have given it away.

To change the riddle, edit the `login-question` text and replace `ANSWER_HASH` in
`index.html` with the SHA-256 of the new lowercase answer:

```sh
printf 'youranswer' | sha256sum
```
