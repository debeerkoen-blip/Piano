# Piano leerapp (prototype)

Doel: een webapp die piano leert, met gamification. De microfoon luistert naar het spel.
Beginners eerst. Latere doelen: liedjes uit Spotify als wensenlijst met voortgang per nummer.

## Huidige stand
- `index.html` is één bestand: HTML, CSS en JavaScript, zonder build-stap.
- Microfoon: twee onafhankelijke detectoren (autocorrelatie en YIN) die het eens moeten zijn voor een toon telt, plus een state machine (zoeken/bevestigen/vasthouden). Herkent één noot tegelijk.
- Live-indicator op de notenbalk: toont wat de microfoon echt hoort (teal = betrouwbaar, amber = onzeker), ook tijdens "Stem af op C". Bedoeld om de herkenning te debuggen.
- Notenbalk: SVG. C-noten zijn oranje. Vingerzetting onder elke noot als R1 t/m R5 (rechterhand). Maatstrepen en kwartrusten (`null` in `notes`) worden getekend.
- Speel af: een balk loopt over de noten met aftellen (4 tellen).
- Oefenmodus: alleen toonhoogte, zonder tijd.
- Niveau (Beginner/Medior/Advanced, `DIFFICULTIES`): bepaalt het tijdvenster, de drempel voor een perfecte timingbonus, en of alleen de exacte octaaf telt (advanced) of elke octaaf van dezelfde toon (beginner/medior).
- XP, reeksbonus, sterren per lied. Punten per noot zijn opgesplitst in toonpunten en timingpunten, zichtbaar in de feedbacktekst.
- Liederen staan in de array `SONGS`: `notes`, `fingers`, `bpm`, `draft`.
- Zoekkaart onder de notenbalk: opent een Google-zoekopdracht (`site:imslp.org <term>`) in een nieuw tabblad, exact wat IMSLP's eigen zoekvak doet (nagekeken door hun zoekformulier in te vullen). Geen eigen API-aanroep: Openopus en IMSLP sturen geen CORS-header mee, dus een fetch vanuit de browser wordt altijd geblokkeerd. IMSLP heeft ook geen `Special:Search`-pagina, die eerdere aanname bleek fout. Getest met een headless browser; de linknavigatie werkt, geen noten of metadata worden automatisch overgenomen.

## Let op
- Ode an de vreugde: noten en vingerzetting zijn uit het hoofd ingevuld, nog niet gecontroleerd.
- Gymnopédie nr. 1: concept (`draft: true`). Controleer tegen de partituur op IMSLP (publiek domein) voordat je die gebruikt.
- Nog niet getest met een echte piano. Test met microfoon in een rustige kamer.
- Zoekkaart: IMSLP laat soms een "bevestig dat je geen bot bent"-pagina zien in plaats van direct de zoekresultaten. Dat is IMSLP's eigen bot-bescherming, geen fout in de app.

## Mogelijke volgende stappen
1. Testen op telefoon en laptop. Timing-venster en detectiedrempels afstellen.
2. Voortgang opslaan in localStorage.
3. Rechterhand en linkerhand scheiden. Akkoorden later.
4. Spotify-koppeling: liked songs als wensenlijst, met een voortgangsbalk per nummer.
5. Losse modules splitsen (audio, notenbalk, spelmotor) zodra het groter wordt.

## Werkafspraken
- Schrijf teksten in de app in het Nederlands.
- Geen em dashes in teksten.
- Verander de nootdata alleen na controle tegen een bron.
