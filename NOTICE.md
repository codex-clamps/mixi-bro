# Notices and provenance

## TV Bro

mixi-bro is planned as a modified browser derived from or informed by TV Bro:

- upstream repository: <https://github.com/truefedex/tv-bro>
- inspected/import baseline: TV Bro 2.1.6, commit `10b8b1a9ff29fcbe5c55052eec61caf553d6964f`
- original copyright: Copyright (c) 2019, Fedir Tsapana
- upstream license copy: [`licenses/TV_BRO_LICENSE.md`](licenses/TV_BRO_LICENSE.md)

Any distributed modified binary must use a different name, icon, and application ID and must include an About view stating that it uses TV Bro sources with a link to the upstream repository. mixi-bro's proposed identity is deliberately separate.

## Mozilla GeckoView

mixi-bro is intended to embed Mozilla GeckoView. GeckoView, Firefox, and Mozilla names and marks belong to their respective owners. Use of GeckoView does not make mixi-bro Firefox and does not imply Mozilla endorsement. Exact GeckoView and transitive license notices must be generated from the resolved release dependency graph before distribution.

## AndroidX and other dependencies

AndroidX, Kotlin, Gradle, and other third-party dependency notices must be generated and reviewed as part of the release process. This bootstrap does not select a final license for new mixi-bro-authored code; maintainers must make that decision before accepting broad contributions or distributing binaries.
