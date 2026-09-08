# R4TLS.R4P

`R4TLS.R4P` is an independent R4OS protocol module implemented in Zig.

## Package

- Version: `0.2.6`
- Image target: `/R4OS/PROTOCOLS/R4TLS.R4P`
- Image scope: `slim`
- Canonical project manifest: `module.R4MF`

The manifest is the single source of truth for the artifact, imports, image
target, and package metadata.

## Build

On Windows:

    Build.bat

On Linux or macOS:

    ./Build.sh

The build starters resolve the current local R4OS dependency checkouts through
`Settings.R4S`. The URL and hash entries in `build.zig.zon` record the
last verified standalone dependency identities; workspace builds use the
mapped local checkouts.

## Documentation

Detailed German technical notes from the migration are preserved in
`DOCUMENTATION.de.txt`. Source-transfer provenance is recorded in
`PROVENANCE.txt`.

## License

Original R4OS material is licensed under Apache License 2.0. See `LICENSE`
and `NOTICE`. Any repository-specific external material is documented in
`THIRD_PARTY_NOTICES.md`.


Sitzungszufall ab 0.78.61
-----------------------
X25519-Server-Secret und server_random werden fuer jeden Handshake getrennt
mit SDK secure_random.fill erzeugt. Keine Uhren-/Kennungsableitung und kein
konstanter Produktwert. Fehlende Entropie liefert -11 bei Ausgabelaenge 0.
R4LB enthaelt nun den privaten R4L2-Zustand: Magic4, Transcriptlaenge4,
client_random32, server_random32, server_secret32, Transcript (max.8192).
R4LS wird abgewiesen. Der Aufrufer behaelt den Zustand bis LiveFinish und
uebertraegt ausschliesslich die getrennte Server-Flight an den Peer. Finish
leitet die Schluessel aus genau diesem Secret ab. Lokale Secret-Kopien werden
nach Verwendung geloescht; keine vollstaendige Speicherbereinigung behauptet.
Konstante ECDH-Werte bleiben ausschliesslich in isolierten Prueffaellen.
TLS-Zertifikat und RSA-Schluessel sind weiterhin gesondertes Systemmaterial.
