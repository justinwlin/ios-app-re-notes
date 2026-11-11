# Working in this repository

## Read the committed knowledge first

Start with `knowledge/AGENT_HANDBOOK.md`. It contains the distilled workflow,
decision rules, and verification boundaries in text. Read
`knowledge/VIDEO_LESSONS.md` for the conclusions from the ten collected videos and
the relevant file under `knowledge/techniques/` for an implementation procedure.
These documents are self-contained; opening the source videos is optional.

When new evidence changes a conclusion, update the relevant knowledge text. Keep
source links as citations alongside the explanation, not as a substitute for it.

## Artifacts

Keep downloaded apps, assets, Hopper databases, and signing material in ignored
local folders (`artifacts/`, `local/`). Never commit `.ipa` files, Apple Account
credentials, certificates, provisioning profiles, or pairing files.

## Evidence

Preserve distinctions between static observations, emulation, and device tests.
Never convert an unconfirmed iPhone result into a success claim. Patch addresses
are tied to binary hashes, not app names. Keep the source IPA unchanged.
