# Kopenhagen Golf – Projekt-Leitfaden für Claude

Deutschsprachige Golf-Wertungs-App für die Spielrunde „Kopenhagen". Reine Single-File-Web-App (HTML + CSS + JS in einer Datei), **kein Build-Schritt**, kein Backend. Läuft als GitHub Pages und als Home-Screen-App auf dem iPhone.

## Dateien
- **`index.html`** = Produktion (live).
- **`test.html`** = Sandbox zum gefahrlosen Ausprobieren. Inhaltlich identisch zu `index.html`, **außer**: andere localStorage-Schlüssel (TEST) und im Titel/H1 als „TEST" markiert.
- **Wichtig: Code-Änderungen immer in BEIDE Dateien übernehmen** (gleiche Stelle, gleiche Logik). Zuerst gern in `test.html` testen, dann nach `index.html` spiegeln.

Speicher-Schlüssel (localStorage):
- Produktion (`index.html`): `kopenhagen-golf-rechner-v5`, `kopenhagen-saison-v1`, `kopenhagen-weightfactor-v1`
- Sandbox (`test.html`): `kopenhagen-golf-rechner-TEST-eingabe`, `kopenhagen-saison-TEST-v1`, `kopenhagen-weightfactor-TEST-v1`

## Arbeitsweise / Workflow
- **Claude editiert nur die Dateien.** Commit + Push macht der User (Olaf) selbst in GitHub Desktop. **Claude committet/pusht nicht.** Remote: `olaf-git-hub/kopenhagen_app`, Branch `main`.
- Vor dem Übergeben: JS-Syntax prüfen (z. B. Script-Block extrahieren und mit `new Function(...)` testen) und – wenn möglich – im Browser/Preview verifizieren.
- **Datenschutz:** Spielergebnisse/Saison-Daten sind privat und liegen **außerhalb** dieses (öffentlichen) Repos. **Niemals Spieldaten-JSON (z. B. `kopenhagen-saison.json`) ins Repo committen.**
- UI-Texte sind deutsch; neuen Code im Stil des umgebenden Codes schreiben (Namen, Kommentare, Formatierung).

## Domänen-Regeln (Wertung)
- **Kopenhagen-Punkte** je Loch nach Netto: `pointVector(3) = [2,1,0]`, `pointVector(4) = [3,2,1,0]`. Bei echtem Gleichstand (gleiche Punkte UND gleiche Schläge) werden die Punkte gemittelt.
- **Vorgaben (Netto):** automatische Schläge nach HCP des Lochs aus der Gesamtvorgabe, pro Loch überschreibbar (18-Loch-Matrix). Anzeige als „V<n>".
- **2-Spieler-Modus = Matchplay** (deutscher Fachbegriff: Lochwettspiel), **KEINE Kopenhagen-Punkte.** Pro Loch gewinnt der niedrigere Netto-Score; Anzeige mit US-Begriffen (`2 up`, `All Square`, `dormie`, `wins 3 & 2`). Replay/Punkteprüfung/Bild-Teilen/Rundenrückblick sind im 2er ausgeblendet. **UI-Label heißt überall „Matchplay"** – der intern gespeicherte Modus-Schlüssel bleibt aber `r.mode === "match"`.
- **Matchplay-Anzeige (Lochtabelle/Karten):** Vorgaben werden nur als **Netto-Differenz** gezeigt (`V<n>` nur beim Spieler mit echtem Schlagvorteil; haben beide gleich viele Schläge ⇒ kein V). Loch-Ausgang als kleines **Symbol neben dem Namen/Score**: 🟢 = Loch gewonnen, ⚪ = geteilt (bei beiden Spielern), Verlierer ohne Symbol. Kein „gewonnen/verloren/geteilt"-Text mehr.
- **Order of Merit:** Ranking nach Ø Meritpunkten pro **Kopenhagen**-Runde (offizielle Wertung ab ⌈Kopenhagen-Runden/3⌉). **Matchplay-Runden** zählen NICHT ins Punkte-Ranking, aber in den **Ø Brutto** (nur volle 18-Loch-Runden, Brutto – nicht Netto). Rundenzahl-Hinweis: „N (+M)" (M = zusätzliche Matchplay-Runden).

## Code-Orientierung (wichtige Funktionen)
- Wertung: `calculateRound`, `calculateHole`, `calculateMatch`, `formatMatchStatus`, `holeOutcomeForPlayer`, `matchNetStroke` (Netto-Vorgabe-Differenz), `matchHoleSymbol` (🟢/⚪).
- Anzeige: `renderSummary` / `renderMatchSummary`, `buildTable`, `renderMobileCards`, `renderPlayerDetail`, `buildConfirmButton`.
- Saison/OoM: `buildRoundSnapshot`, `computeOrderOfMerit`, `openSeasonTable`, `renderRoundStandingsHtml`, `openPlayerProfile`. Spieler werden über Runden hinweg **tolerant per Name** zusammengefasst: `seasonNameKey` (`trim()` + Kleinschreibung) und `sameSeasonName` – Anzeige bleibt wie eingegeben, nur die Zuordnung ist tolerant. Gespeicherte Saison-Daten werden dabei nicht verändert.
- Spracheingabe (Scores): `processVoiceScore` (wertet alle Erkennungs-Alternativen aus, nimmt die mit den meisten Treffern), `parsePlayerScores`, `findNumberAfterPlayer`, `extractScoreNumbers`; Wörterbücher `speechNumberWords`, `speechPlayerAliases`.
- Springen Stand/Tabelle: `toggleSummaryJump`, `syncJumpButtonToScroll` (Scroll-Listener hält Label/Modus an der echten Position), `scrollToSummary`, `scrollBackToTable`.
- Teilen/Links: `createShareLink` (`#s=`), `createSeasonShareLink` (`#oom=`), `shareRound`, `shareSeason`.
- Eingabe: `stepScore`, `commitTypedScore`, `confirmHole`, `editHole`/`cancelHoleEdit`, `resetHole`.

Spieleranzahl: 2 (Matchplay), 3 oder 4. `state.playerCount === 2` ⇒ `isMatchPlay()`.
