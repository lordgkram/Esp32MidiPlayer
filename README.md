# Esp32MidiPlayer
Benutzte Bibliotheken:
* fortyseveneffects/MIDI Library@^5.0.2
* knolleary/PubSubClient@^2.8
* bblanchon/ArduinoJson@^6.17.3
* https://github.com/tzapu/WiFiManager.git#v2.0.11-beta

## Platformen
* ESP32
* ESP8266

## Setup
1. copy `src/bspmyauth.h` to `../myauth.h`
1. modify `../myauth.h` (the relaive paths are given to this Readme file)
1. copy `src/bspmysettings.h` to `src/mysettings.h` (optional)
1. modify `src/mysettings.h` (optional)
1. compile and flash
1. setup wifi

## Playmidi-Syntax

`[;[l][n]][-][bpm<BPM>] <Segment>[:<Segment>][:<Segment>]...`/`~<lied Preset>`

### Segment (optional)
`[!][-][<INSTRUMENT>] <NOTEN...>`

Segmente sind dazu da, um mehrere Spuren einfach gleichzeitig zu Spielen.
Alle Segmente werden von beginn des Stückes gespielt, außer vor dem Segment steht ein `!`.
Falls vor einem Segment ein `-` steht wird dieses Segment im entgegengesetztem Modus zu dem Lied Start Modus gespielt.
Segmente werden von 0 aufwertzt indiziert

**Achtung**: Es ist zu beachten, dass ein Segment, wenn dieses vom Start an gespielt wird, im ersten Midi-Kanal gespielt wird, und man somit zuerst den Kanal wechseln muss, wenn man mehrere Instrumente will.

### Flag Erweiterter Modus (optional)

`-...`

Ist das allererste Zeichen ein `-`, wird der erweiterte Modus aktiviert. Die Noten werden dann so lange gehalten, bis sie zum nächsten Mal erwähnt werden.


### BPM (optional)

`... bpm<BPM> ...`

Setzt das Tempo des Liedes in *beats per minute* (BPM). Standardmäßig werden die Lieder mit Tempo 240 wiedergegeben.


### Instrument (optional)
Setzt das genutzte Instrument.
Liste der verfügbaren Instrumente:
* `piano` (Klavier)
* `vibes` (Vibraphon)
* `organ` (Orgel)
* `guitar` (Gitarre)
* `brass` (Bläser)
* `drums` (Schlagzeug)
* oder Nummer des Midi-Instruments

#### Schlagzeug
Nach General MIDI ist das Schlagzeugt auf `k10` jedoch kann es seien,
dass einige Synthesizer das nicht beachten,
und ein Instrument anbieten, deswegen gibt es `idrums`.
Und es wird empholen nur `k10` für Schlagzeug zu verwenden und mit `idrums`.

### Noten

`<Note> [M][Note] [M][Note]...`

Es können beliebig viele Noten hintereinander auftauchen. Einzelne Noten werden mit Leerzeichen getrennt. Sollen mehrere Noten zugleich gespielt werden, so werden die Noten ohne Leerzeichen hintereinander geschrieben.

#### Notenbestandteile

Eine Note setzt sich zusammen aus der **Tonhöhe**, einem **Oktavzeichen** (optional), einem **Vorzeichen** (optional) und dem **Notenwert** (optional).

Beispiele für Noten sehen folgendermaßen aus: `C` oder `d''#4`
##### Tonhöhe
Die Tonhöhe ergibt sich aus den deutschen Notenbezeichnungen (`CDEFGAH`) und einem optionalen Vorzeichen, welches nach der Notenbezeichnung bzw. der Oktavenzeichen steht (`#`, `b`).
Die Oktave ergibt sich aus bis zu drei Oktavzeichen (`'`)
Die Reihenfolge der Oktaven von tief nach hoch sieht folgendermaßen aus:  
`C''' C'' C' C c c' c'' c'''`.

Ein tiefes Cis schreibt man zum Beispiel so: `C''#`
##### Notenwert (optional)
Die Länge ist eine Ganzzahl und gibt den Kehrwert der Notenlänge an.
* 1 entspricht einer ganzen Note
* 2 entspricht einer halben Note
* 4 entspricht einer Viertelnote
* 8 entspricht einer Achtelnote
* 16 entspricht einer Sechzehntelnote

Folgt der Länge ein Punkt (`.`), wird die Note um den anderthalbfachen Wert verlängert (Punktierung).

#### Besonderheit: Akkorde
Mehrere direkt aufeinanderfolgende Töne werden zusammen gespielt.
Beispie: `CEGC GHdg FAcf`

#### Modus wechsel
Mit `m` kann der Modus zwischen dem "standard" und dem "erweiterten" gewechselt werden.

#### Noten im *erweiterten Modus*

`[Ton]...[s][Ton]...[Länge / k / i]`

##### Besondere Funktionen
###### Instrument ändern
`i<instrument>`  
Wechselt das aktuelle Instrument für die nachfolgenden Noten.  
`-organ ... ipiano ...`

###### Alle Noten stoppen
Mit `s` werden alle Noten gestoppt, die zur Zeit klingen.

###### Midi-Channel wechseln
`k<Midikanal>`
Mit `k` kann der aktuelle Midi-Kanal von 1-16 gewechselt werden, um z.B. mehrere Instrumente gleichzeitig spielen zu können.
`-piano A k2 iorgan H 4 s`

###### Anschlagsgeschwindigkeit ändern
`v<Neue Geschwindigkeit>`
Mit `v` wird die Anschlagsgeschwindigkeit geändert \(0-127\) \(Keine Garantie da nicht jeder Midi-Synthesizer diese Funktion unterstützt\);
`-piano A1l v64 A1l`

###### Lautstärke ändern
`z<Neue Lautstärke>`
Mit `z` wird die Lautstärke in dem aktuellen Midi-Kanal geändert \(0-127\) \(Keine Garantie da nicht jeder Midi-Synthesizer diese Funktion unterstützt\);
`-piano A1l z64 A1l`

###### Spuren synchronisieren
`r[<Sync-Kanal>][-[<Zahl>]]`
Mit `r` kann man Spuren synchronisieren. Wenn der `Sync-Kanal` oder die `Zahl` nicht angegeben ist, ist jener Parameter 0.
Die Spur wartet ab diesem Befehl auf ein Sync-Signal auf dem `Sync-Kanal`, sobald dieses empfangen wird wird die Spur weiter bearbeitet.
Wenn das `-` gegeben wird, wird ebenfals auf ein Sync-Signal gewartet, jedoch wenn mehr oder gleich viele Spuren als mit `Zahl` warten \(diese Spur eingeschlossen\), wird ein solches Sync-Signal gessendet.
`-ArlC:G4r-lF4`

###### zuletzt genannte Note Stoppen
`l`
Mit `l` wird die zuletzt genannte Note gestoppt oder gestartet.

###### BPM Änderung
`q<neue BPM>`

###### controllchange 0 MSB-Bank standard: 0
`x<neuer Wert>`

###### controllchange 32 LSB-Bank standard: 0
`y<neuer Wert>`

###### controllchange 72 release standard: 63
`j<neuer Wert>`

###### controllchange 73 attack standard: 63
`o<neuer Wert>`

###### Segment paralel abspielen
beim parallelen abspielen wird das Segment während dem Akktuellem Segment Abgespielt
`t<segment nummer>`

###### Segment abspielen
bei diesem abspielen wird das akktuelle Segment pausiert, bis das Andere fertig ist
`T<segment nummer>`

###### Ton
Wenn der erweiterte Modus aktiv ist, wird ein Ton bis zur nächsten Erwähnung im gleichen Kanal oder bis zum nächsten `s` gespielt.  
`a (a wird gespielt) 4 (a wird gespielt) a (a wird nicht mehr gespielt)
und a (a wird wieder gespielt) 4 (a wird gespielt) s (a wird nicht mehr gespielt)`

### Wiederholung
Achtung: Der aktuelle Modus wird nicht beachtet, woduch eine Wiederholung im selben Modus aufhören sollte, wie sie angefangen hat.

#### w
Aktiviert/Deaktiviert die Wiederholung
`...w<0>w...`→`...<0><0>...`

#### n
Noten die nach dem ersten Spielen gespielt werden
`...w<0>n<1>w...`→`...<0><1><0>...`

#### u
`u` wird nach `n` genutzt um weitere Wiederholungen anzuhängen mit optionalen Noten. Wird nach einem `u` direkt ein `-<zahl>` geschrieben, so wird das `u` als zahl\*`u` gewärtet.
* `...w<0>n<1>u[2]w...`→`...<0><1><0>[2]<0>...`
* `...w<0>n<1>u[2]u[3]w...`→`...<0><1><0>[2]<0>[3]<0>...`
* `...w<0>n<1>u-1[2]w...`→`...<0><1><0>[2]<0>...`
* `...w<0>n<1>u-2[2]w...`→`...<0><1><0><0>[2]<0>...`
* `...w<0>n<1>u-3[2]w...`→`...<0><1><0><0><0>[2]<0>...`
* ...

#### Beispiele
* `cw d ew g` → `c d e d e g`
* `cw d en fw g` → `c d e f d e g`
* `cw d en fu dw g` → `c d e f d e d d e g`
* `cw dnu-3w e` → `c d d d d d e`

### Puffer (optional)
`;...` falls keine unten genannten Pufferaktionen folgen werden folgende Noten in den Puffer geschrieben.

#### Puffer löschen
`;l...` löscht den Puffer.

#### neuen Puffer erstellen
`;[l]n...` erstellt einen neuen Puffer, folgende Noten werden eingefügt.

### Lied Preset
* `0`- `118 c4 C'4 mi119 c'2`
* `1`- `brass F2 G#2 F4 F8 A#4 F4 D# F2 c2 F4 F8 c# c4 G# F4 c4 f4 F4 D# D#8 C G# F1` - `Klavier` Beispielmelodie
* `2`- `-126 c1 1 1 1 1 1` - Applaus
* `3`- ... - Gravity Falls
* `4`- ... - Vivaldi
* `5`- ... - cowntdown\(/Vorhersage/Umfrage Melodie\)
* `6`- ... - irgendwas was `d7gr` gemacht hat
* `7`- ... - trinken
* `8`- ... - Ducktales
* `9`- ... - Wenn ich einmal reich wär - Abomelodie
* `10`- ... - Für Elise
* `11`- ... - Sandmann
* `12`- ... - Airwolf

## Mqtt

Der Midi-Player reagiert auf den MQTT-topic `playmidi`. Dort wird ein einfacher String mit JSON-Daten oder rohen Lieddaten empfangen.
Wenn eine Nachricht vom MQTT-topic `killmidi` emfangen wird, wird das aktuelle lied unterbrochen und die play queue wird geleert.

## JSON

Es handelt sich um ein JSON-Objekt mit folgenden Tags:
* `length` / `laenge` - maximale Liedlänge
* `midi` - Daten des Liedes

optional:
* `allowBuffer` / `aktiviereBuffer` - falls vorhanden und der Wert `true` ist, wird die Bufferfunktion aktiviert.

Bei aktiviertem Buffer werden folgende Tags benötigt:
* `user` / `nutzer` - Name des Users
* `maxBufferSize` / `maximaleBufferGroesse` - die maximale Größe des Puffers
folgende sind optional:
* `refundTopic` mit `refundID`: sendet auf `refundTopic` die Nachricht `refundID`, wenn die Aktion refundet werden soll (Puffer schreiben, Warteschlange voll). Die Felder müssen Strings seien
