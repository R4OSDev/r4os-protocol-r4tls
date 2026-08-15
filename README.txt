R4TLS.R4P
=========

R4TLS.R4P ist die wiederverwendbare TLS-Protokollbasis fuer R4OS.

Stand 0.62.14:

- Artefakt: `R4TLS.R4P`
- Zielpfad im Image: `C:\R4OS\PROTOCOLS\R4TLS.R4P`
- R4P-Rolle: `security.tls`
- Kategorie: `data`
- Erste Konsumentenlinie: RDPSVC fuer moderne Windows-mstsc-Kompatibilitaet

0.55.16 legt bewusst noch keinen falschen TLS-Erfolg an. Das Modul stellt
zunaechst den R4P-Besitz, den Build-/Image-Pfad und eine erste Dispatch-Kante
bereit. `op_capabilities` liefert die sichtbaren Rollen-/Stufendaten,
`op_classify_record` validiert einen TLS-Record-Header und
`op_selftest` prueft diese Basis intern.

Seit 0.55.17 besitzt R4TLS einen staerkeren Record-Layer und einen
ClientHello-Parser. `op_classify_record` meldet jetzt Record-Kind,
Fragmentlaenge und ob der uebergebene Buffer den angekuendigten Record
vollstaendig enthaelt. `op_parse_client_hello` verarbeitet einen vollstaendigen
TLS-Handshake-Record mit ClientHello und meldet Legacy-Version,
Handshake-Laenge, Cipher-Suite-Anzahl, Compression-Methoden und die fuer mstsc
wichtigen Extensions `server_name`, `supported_versions`, `supported_groups`,
`signature_algorithms`, `key_share` und `alpn`.

Seit 0.55.18 besitzt R4TLS die ServerHandshake-Basis fuer den modernen
mstsc-Pfad. `op_plan_server_handshake` wertet das ClientHello auf TLS 1.2,
`TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`, X25519 und RSA-PKCS1-SHA256 aus und
liefert einen sichtbaren Plan mit ServerHello, Certificate,
ServerKeyExchange, ServerHelloDone, Transcript-Grenze und den vorgesehenen
Systempfaden `C:\R4OS\CONFIG\TLS\R4TLSDEV.CRT` sowie
`C:\R4OS\CONFIG\TLS\R4TLSDEV.KEY`. `op_build_server_handshake_fixture`
erzeugt daraus deterministische TLS-Handshake-Bytes fuer Selftests und spaetere
RDPSVC-Anbindung. Der Vertrag meldet weiterhin `stream_ready=no`; es gibt also
keinen falschen TLS-Erfolg und noch keinen verschluesselten Record-Stream.

Seit 0.55.19 besitzt R4TLS den Stream-I/O-Vertrag fuer die naechste
RDPSVC-Anbindung. `op_stream_contract` beschreibt Read, Write, Flush, Close,
Alert, WouldBlock, Disconnect und Backpressure als gemeinsame Konsumentenform.
`op_stream_write_record`, `op_stream_read_record` und
`op_stream_alert_record` erzeugen beziehungsweise verarbeiten deterministische
TLS-Application-/Alert-Records fuer Harness und Wire-Vertrag. Diese Records
sind eine Testschutz-Fixture fuer Record-Grenzen, Buffer-Regeln und
Integritaetspfad; sie ersetzten damals noch nicht AES-GCM, Key Schedule oder
fertige TLS-Crypto und meldeten deshalb in 0.55.19 `cipher_ready=no`.

`op_stream_transport_status` nimmt einen `NetServiceTcpResult` von TCPSVC an
und mappt ihn auf einheitliche Stream-Zustaende. `pending_rx`, `tx_window`,
`service_status`, `lifecycle_cause`, `handle_valid` und `conn_valid` bleiben
Eigentum von R4NET/TCPSVC. R4TLS wertet diese Felder nur aus und verschiebt
keine Transport-Policy in das Protokollmodul.

Seit 0.55.23 besitzt R4TLS den produktiven TLS-1.2-Krypto-Kern fuer den
mstsc-Zielpfad. `op_productive_contract` beschreibt den wiederverwendbaren
Vertrag, `op_tls12_x25519` berechnet Server-Public-Key und Shared Secret,
`op_tls12_key_schedule` bildet TLS-1.2-PRF/HMAC-SHA256 auf Master Secret,
Client-/Server-Write-Keys und feste AES-GCM-IVs ab. `op_tls12_protect_record`
und `op_tls12_open_record` erzeugen beziehungsweise oeffnen echte
TLS-1.2-AES-128-GCM-Records mit `seq64+type+0303+plain_len` als AAD,
4-Byte-Fixed-IV plus 8-Byte-Explicit-Nonce und 16-Byte-Tag. Manipulierte
Tags liefern `stream_result_integrity_failed`.

Seit 0.55.24 besitzt R4TLS den produktiven Zertifikat-/Key-Materialpfad fuer
TLS 1.2. `R4TLSDEV.CRT` enthaelt ein selbstsigniertes X.509-DER-Dev-Zertifikat
plus RSA-Public-Key-Felder, `R4TLSDEV.KEY` den passenden RSA-2048-Private-Key
im R4OS-Entwicklungsformat. R4TLS parst beide Dateien beim Initialisieren,
validiert DER, Modulus/Exponent und Key-Match und meldet klare Statuswerte wie
`bad-pem`, `missing-field`, `bad-hex`, `field-too-large`, `bad-type`,
`key-mismatch` oder `sign-failed`.

`op_tls12_rsa_key_contract` (`op18`) beschreibt den geladenen Materialzustand.
`op_tls12_sign_server_key_exchange` (`op19`) signiert Eingaben im Format
`R4SG + Payload` als RSA-PKCS1-SHA256 und liefert `R4SR`, SignatureScheme
`0x0401`, Signaturlaenge und Signaturbytes zurueck. Die ServerHandshake-
Fixture nutzt seit 0.55.24 die geladenen DER-Zertifikatsbytes und schreibt in
ServerKeyExchange keine leere Platzhaltersignatur mehr, sondern eine echte
Signatur ueber `client_random + server_random + ecdh_params`.

Seit 0.55.26 nutzt ServerKeyExchange auch fuer X25519 kein synthetisches
Public-Key-Platzhalterfeld mehr. R4TLS haelt ein eigenes Server-Secret fuer den
aktuellen Entwicklungsserverpfad und schreibt den daraus ableitbaren
X25519-Public-Key in die `ecdh_params`. Der Selftest extrahiert diesen Public
Key aus der erzeugten ServerHandshake-Fixture, vergleicht ihn mit
`X25519.recoverPublicKey(server_x25519_secret)` und verwirft den alten
`0x40..0x5F`-Platzhalter explizit. Das ist noch kein vollstaendiger
TLS-Sessionzustand, beseitigt aber einen echten spaeteren mstsc-
KeyAgreement-Blocker.

Seit 0.55.27 besitzt R4TLS einen eigenstaendigen TLS-1.2-Session-Harness.
`op_tls12_session_contract` (`op20`) beschreibt den Besitz von
ClientKeyExchange, ChangeCipherSpec, Finished-Verifikation, Server-Finished
und Sequenzen. `op_tls12_session_harness` (`op21`) verbindet die vorhandenen
ClientHello-, ServerHandshake-, X25519-, PRF- und AES-GCM-Kanten zu einer
deterministischen Session-Sequenz. Der Harness parst ClientKeyExchange,
berechnet das Shared Secret aus dem R4TLS-Server-Secret und dem Client-Public-
Key, leitet Master Secret und Write-Keys ab, verschluesselt den Client-
Finished-Record, oeffnet und verifiziert ihn, erzeugt danach den Server-
Finished-Record und prueft negative Faelle fuer kaputten ClientKeyExchange,
Bad-Finished-Tag und falschen Transcript-Hash.

Seit 0.55.30 besitzt R4TLS zusaetzlich die Live-Session-Schnittstelle fuer
echte Client-Flights. `op_tls12_live_begin` (`op22`) nimmt einen
TLS-ClientHello-Record entgegen, plant und baut den ServerHandshake und gibt
`R4LB` mit serialisiertem `R4LS`-Sessionzustand sowie der ServerHandshake-
Flight zurueck. `R4LS` enthaelt Client-/Server-Random und das bisherige
Handshake-Transcript, damit der naechste Aufruf ohne RDPSVC-private
TLS-Zustandslogik fortsetzen kann.

`op_tls12_live_finish` (`op23`) nimmt `R4LS` plus ClientKeyExchange,
ChangeCipherSpec und verschluesselten Client-Finished-Record entgegen,
berechnet Shared Secret, Master Secret und AES-GCM-Keys, oeffnet und
verifiziert Client Finished und liefert danach `R4LF`. Darin liegen ein
`R4LK`-Streamzustand und die Server-Flight aus ChangeCipherSpec und
verschluesseltem Server Finished. `R4LK` speichert Client-/Server-Sequenzen,
AES-GCM-Write-Keys, Fixed-IVs, Master Secret und den TLS-Public-Key-Hash fuer
die spaetere CredSSP-PubKeyAuth-Bindung. Der R4TLS-Selftest faehrt diese
Begin-/Finish-Kette mit einer deterministischen Fixture durch.

Falls Zertifikat und Private Key beim fruehen Protocol-Init noch nicht lesbar
waren, laedt R4TLS das Material beim ersten TLS-Dispatch lazy ueber die
gespeicherte Protocol-API nach; RDPSVC muss diesen Dateipfad nicht kennen.

Seit 0.55.32 besitzt R4TLS den produktiven Application-Record-Adapter ueber
dem `R4LK`-Streamzustand. `op_tls12_app_write` (`op24`) nimmt
`R4AW + R4LK + Plaintext` entgegen, schuetzt die Nutzdaten als TLS-1.2-
Application-Data mit Server-Sequenz, Server-Key und Server-IV und liefert
`R4WX + aktualisiertes R4LK + TLS-Record`. `op_tls12_app_read` (`op25`) nimmt
`R4AR + R4LK + genau einen Client-Application-Record` entgegen, oeffnet ihn
mit Client-Sequenz, Client-Key und Client-IV und liefert `R4RX +
aktualisiertes R4LK + Plaintext`. WouldBlock, Backpressure, Flush, Close und
Socket-Lifecycle bleiben weiter TCPSVC-/Consumer-Policy; R4TLS besitzt nur
Record-Schutz, Record-Oeffnung und Sequenzfortschreibung.

Der R4TLS-Selftest faehrt seit 0.55.32 die Live-Begin-/Finish-Kette und
anschliessend beide Application-Record-Richtungen. Der Selftest meldet
`live=ok`, `app_write=ok` und `app_read=ok`, damit RDPSVC und Host-Harness
diese Schicht pruefen koennen, ohne Windows-`mstsc`-Erfolg vorzutaueschen.

Seit 0.62.14 besitzt R4TLS auch die spiegelbildliche TLS-1.2-Clientseite.
Die Operationen 26 bis 32 stellen Clientvertrag, ClientHello,
Server-Handshake-Pruefung, Key Schedule, Finished-Verifikation,
Application-Record-Read/Write und einen deterministischen Gesamtharness
bereit. Die Clientseite akzeptiert nur
`TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256` mit X25519 und
RSA-PKCS1-SHA256. Hostname, Gueltigkeitszeit, Ausstellerkette und
Zertifikatssignatur werden vor dem ersten Application-Record geprueft.

Vertrauensanker sind keine Codekonstanten. R4TLS liest den versionierten
Katalog `C:\R4OS\CERTS\CATALOG.R4S`; dessen `ROOT`-Eintrag verweist auf
eine externe Zertifikatdatei unter `C:\R4OS\CERTS\ROOTS\`. Fehlt der
Katalog, bleibt der Entwicklungsanker als expliziter Dateipfad-Fallback
erhalten. Weder DER-Zertifikat noch Root-Datei liegen im Kernel, in R4DRAW,
in Klickifax oder als Bytefolge im R4TLS-Modul.

Die alten `op_stream_write_record`-/`op_stream_read_record`-Fixture-Records
bleiben nur fuer Diagnose und Record-Grenztests sichtbar. Sie gelten seit
0.55.23 nicht mehr als Erfolgsmodell fuer den produktiven TLS-Stream.

Zertifikat und Private Key bleiben R4TLS-Eigentum und liegen vertraglich unter
`C:\R4OS\CONFIG\TLS\R4TLSDEV.CRT` und
`C:\R4OS\CONFIG\TLS\R4TLSDEV.KEY`. Seit 0.55.23 besitzt die R4P-Protocol-API
eine read-only `file_read`-Kante, ueber die R4TLS diese Systemdateien beim
Initialisieren selbst prueft und den Ladezustand im Produktivvertrag meldet.
RDPSVC darf daraus keine private Zertifikat-/Key- oder RSA-Signatur-Logik
ableiten.

Buffer-/Ownership-Regeln:

- Der Input-Buffer gehoert immer dem Aufrufer und bleibt nach dem Dispatch
  gueltig oder freigebbar.
- Der Output-Buffer gehoert ebenfalls dem Aufrufer; R4TLS schreibt nur bis zur
  angegebenen Capacity und setzt `len`.
- Socket-Handle, Retry-Strategie, Wait-Schleifen, Flush-Warten und Close-Aufruf
  bleiben beim konsumierenden Dienst oder bei der spaeteren SDK-Schicht.
- `WouldBlock` kommt aus `net_service_status_would_block` oder dem passenden
  Lifecycle-Code; `Backpressure` ist sichtbar, wenn `tx_window=0`; Disconnect
  entsteht aus ungueltigem Handle/Conn oder terminalem Lifecycle.

Die naechsten Ausbaustufen gehoeren hierher:

- CredSSP-/R4AUTH-Live-Zustandsmaschine auf Basis des produktiven R4TLS-
  Streams.
- Weitere Cipher-Suites und TLS 1.3 nach dem ersten HTTP/1.1-Browserpfad.

RDPSVC darf diese Logik nicht privat duplizieren. Der Dienst bleibt RDP-
Konsument und nutzt spaeter den Protocol-Dispatch beziehungsweise eine daraus
abgeleitete SDK-Komfortschicht.
