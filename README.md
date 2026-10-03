# betty-skills

Skill packs and app builds for Betty for Mac, a desktop pet.

Betty checks this repo once a day. `feed.json` lists every pack and app build with its version,
size, SHA-256, and Ed25519 signature. Betty installs only items whose signature matches the public
key built into the app. Only the owner can push to this repo.

The pack format is described in `PACKS.md`.
