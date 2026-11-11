# MBROLA NG - Resources

Data that the [MBROLA NG](https://github.com/Mbrola-NG/Mbrola-NG) apps read
**over the network at run time**, so it can be corrected without a new
release. It is also a submodule (`Resources/`) of the main repository, which
is where the builds take their fallback copy of the catalog from.

    catalog/catalog.json     the voice catalog
    assets/samples/<id>.wav  the short recording "Try" plays for a voice that is not installed yet

## The catalog

`catalog/catalog.json` says which languages and voices exist: where each voice
file is downloaded from, its size and SHA-256, its license, and its settings
(`base_pitch`, `phoneme_map`). The NVDA add-on, the Windows installer,
`mbrola-ng-voices` (Linux) and the Android and Apple apps all read it.

    https://raw.githubusercontent.com/Mbrola-NG/Resources/main/catalog/catalog.json

**What is on `main` is live.** Every installed copy fetches it when it starts
(raw.githubusercontent.com caches for about 5 minutes). Because of that:

- Keep `"schema": 1`. A catalog with another schema, one that does not parse,
  or one with no language the app can speak is ignored and the app keeps its
  last good copy, so a mistake does not break installed apps, but it does
  stop your change from reaching them. Check that the file is valid JSON.
- Every file listed for a voice needs its real `size` and `sha256`. The apps
  verify both and reject the download otherwise.
- A language only appears in an app that was built with its compiled `.dat`.
  Adding a catalog entry for a new language does not make it appear in apps
  that were built before the language existed.
- Each voice has its own license. `license.file` and `license.summary` are shown
  to the user before anything is downloaded; fill them in for a new voice.

## The samples

Each voice has a `"sample"` entry (next to `"files"`, not inside it, so a
sample never counts towards the download size of a voice):

    "sample": { "name": "cr1.wav",
                "urls": [ "<raw.githubusercontent.com ... /main/assets/samples/cr1.wav>",
                          "<cdn.jsdelivr.net ... @r-1/assets/samples/cr1.wav>" ],
                "size": 208790, "sha256": "..." }

The apps try the addresses in order and verify size and SHA-256. The second
address is pinned to a **tag**: jsDelivr keeps a branch cached for hours, so a
replaced file would fail its checksum there.

To replace or add a recording:

1. Generate it with `tools/make_voice_samples.py` of the main repository, then
   print the catalog blocks with `--catalog-entries`.
2. Commit the `.wav` files to `main` and push.
3. Create the next tag (`r-2`, `r-3`, ...) and push it:
   `git tag r-2 && git push origin r-2`. Keep the old tags: installed apps
   that have not refreshed the catalog still use them.
4. Only then put the new `size`, `sha256` and `@r-2` into the catalog and push
   it. The catalog goes last.

A change to the catalog alone (a new voice, a corrected license) needs no tag.

## License

`LICENSE` (GNU General Public License, version 2 or later, like MBROLA NG)
covers the catalog.

The recordings in `assets/samples/` are speech synthesized with the MBROLA
voices listed in the catalog and are published only for the "Try" button of the
apps. Each voice has its own license (usually "only with MBROLA, not for
commercial use"), which applies to its recording as well. The voice databases
themselves are never part of this repository.
