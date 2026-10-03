# Betty skill packs

A pack adds jokes, lines, animations, and small scripts to Betty. This page covers the pack format, how Betty checks a pack, and how to make one.

## Where packs live

- `feed.json` lists every pack and app build.
- `packs/<id>/` holds the signed source folder of each pack.
- `releases/<id>-<version>.zip` holds the zip that Betty downloads.
- App builds are release assets on this repo. The feed links to them.

Only the owner can push to this repo. The main branch is protected.

## Pack folder

```
hello/
  pack.json
  script.js            optional
  sheets/walk.avif     sprite sheets named in pack.json
  signature.json       written when the owner signs the pack
```

## pack.json

Required fields:

- `id`: 2 to 40 characters of a-z, 0-9 and `-`. It cannot start with `-`. It is also the install folder name.
- `name`: shown in Settings.
- `version`: `MAJOR.MINOR.PATCH`, for example `1.2.0`.
- `minAppVersion`: same format. Older apps skip the pack.
- `description`: one sentence.
- `clips`: an object of clip name to clip. It can be empty.

A clip has these fields:

- `sheet`: path to a sprite sheet inside the pack.
- `frames`, `cols`, `fps`: positive numbers. `cols` cannot be more than `frames`.
- `loop`: true or false. The default is false.
- `lift`: optional. One number per frame.

Optional fields:

- `outfits`: an object of outfit name to `{ "items": [...], "clips": [...] }`.
- `jokes`: a list of `{ id, setup, punchline, tags, holiday, weather }`. `holiday` and `weather` can be null. Joke ids must be unique.
- `lines`: an object of situation to a list of strings.
- `instructions`: extra text for Betty's AI brain.
- `triggers`: a list of triggers.
- `script`: the string `"script.js"`. No other name is allowed.

Unknown top-level fields are ignored. An unknown trigger kind is an error.

## Triggers

Every trigger has a `kind` and a `run`.

`run` is `activity:<name>` or `script:<function>`. A script function is a global function in `script.js`. A `script:` trigger needs the `script` field.

- `time`: `from` and `to` as `HH:mm`. The window can cross midnight.
- `weather`: `weather`, a weather kind such as `rain`.
- `holiday`: `holiday`, a holiday name.
- `calendar`: `titleContains`, and an optional `minutesBefore`.
- `phrase`: `phrase`, text the user says or types.
- `idle`: `seconds`, more than 0.

Example:

```json
{ "kind": "phrase", "phrase": "hello betty", "run": "script:onHello" }
```

## Scripts

`script.js` runs in JavaScriptCore. It has no access to files, the network, or modules. Each pack has its own context.

The script defines global functions. Betty calls them when a trigger fires. Top-level code runs once when the pack loads.

The global `betty` object has these functions and no others:

- `betty.say(text)`: Betty shows the text.
- `betty.play(clip)`: Betty plays a clip.
- `betty.walk("left" | "right" | x)`: Betty walks one way or to a screen position.
- `betty.remind(text, minutesFromNow)`: Betty reminds the user later.
- `betty.ask(prompt)`: returns a Promise of the AI brain's answer.
- `betty.now()`: returns a `Date`.
- `betty.weather()`: returns the weather kind as a string.
- `betty.holiday()`: returns the holiday name, or null.
- `betty.nextEvent()`: returns `{ title, startMs, minutesUntil }`, or null.

`setTimeout`, `clearTimeout`, and `console.log` also work. Each call into the script gets 250 ms. A script that runs longer is stopped. A script that throws is logged and never crashes Betty.

## How Betty checks a pack

Betty checks once a day. At each step, a failure stops the install and leaves the installed packs as they were.

1. Betty downloads `feed.json` and checks every entry's signature. One bad entry rejects the whole feed.
2. Betty downloads the pack zip. It checks the size, the SHA-256, and the signature against the feed entry.
3. Betty reads the zip's file list. It refuses absolute paths, `..` paths, links, and special files.
4. Betty unpacks the zip into a private folder. It checks every file against the pack's `signature.json`.
5. Betty checks that the pack's `id` and `version` equal the feed entry.
6. Betty refuses a version older than the one installed.
7. Betty moves the pack into place in one step. It keeps the previous version for rollback.

A pack must be signed with the owner's Ed25519 key. The public key is built into Betty. A pack signed by any other key is refused.

## How a pack is signed

The pack signature is Ed25519 over `betty-pack-v1\n<digest>`. The digest is a SHA-256 over this text for every file except the top-level `signature.json`, sorted by path:

```
<relative path>\n<sha256 hex of the file>\n
```

The zip signature is Ed25519 over `betty-file-v1\n<sha256 hex of the zip>`. The two prefixes differ, so a pack signature never works as a zip signature.

## How to make a pack

1. Make a folder named for the pack id.
2. Write `pack.json` with the required fields.
3. Add sprite sheets, a `script.js`, or both if the pack needs them. Every file `pack.json` names must exist.
4. Delete any `.DS_Store` files.
5. Send the folder to the repo owner. The owner signs it, zips it, and adds it to `feed.json`.
6. For a new version, raise `version`. A published version is never replaced.

A working example is in `packs/hello/`.
