# Piano leerapp (prototype)

Doel: een webapp die piano leert, met gamification. De microfoon luistert naar het spel.
Beginners eerst. Latere doelen: liedjes uit Spotify als wensenlijst met voortgang per nummer.

## Huidige stand
- `index.html` is één bestand: HTML, CSS en JavaScript, zonder build-stap.
- Microfoon: twee onafhankelijke detectoren (autocorrelatie en YIN) die het eens moeten zijn voor een toon telt, plus een state machine (zoeken/bevestigen/vasthouden). Herkent één noot tegelijk.
- Live-indicator op de notenbalk: toont wat de microfoon echt hoort (teal = betrouwbaar, amber = onzeker), ook tijdens "Stem af op C". Bedoeld om de herkenning te debuggen.
- Notenbalk: SVG. C-noten zijn oranje. Vingerzetting onder elke noot als R1 t/m R5 (rechterhand). Maatstrepen en kwartrusten (`null` in `notes`) worden getekend.
- Notatieregels die de app volgt: maatsoort (teller/4) boven elke notenbalk, stokrichting wisselt
  bij de middenlijn (erop of erboven: stok omlaag; eronder: stok omhoog), hele noten zonder stok,
  en een kruis/mol/herstelteken wordt maar één keer per maat per toonhoogte getoond (niet bij elke
  herhaling). Nog niet geïmplementeerd: samengebalkte achtste noten (elke achtste krijgt nu een
  los vlaggetje) en een echte toonsoort/voortekening (elke verhoging/verlaging staat los bij de
  noot, er is geen mol- of kruisvoortekening aan het begin van de balk).
- Speel af: een balk loopt over de noten met aftellen (4 tellen).
- Geluid bij "Speel af": echte pianosamples (Salamander Grand Piano, CC BY 3.0, opnames van
  Alexander Holm, gehost via Tone.js op `tonejs.github.io/audio/salamander/`). 30 opnames (elke
  3 halve tonen), de tussenliggende toonhoogtes worden gepitcht met `playbackRate`. Een noot
  stopt na haar eigen tel-duur met een korte fade (demper), niet pas als de hele opname is
  uitgeklonken. Lukt het laden niet (geen internet, host geblokkeerd), dan valt de app terug op
  de oudere synthetische pianoklank (oscillators met boventonen, inharmoniciteit en een
  hameraanslag), zonder dat "Speel af" stopt. Bronvermelding staat onderaan de pagina. Naast
  "Speel af" staat een los icoontje dat altijd met pianogeluid afspeelt, ook als "Met geluid"
  in de instellingen uit staat.
- De notenbalk scrollt automatisch mee met de voortgang: tijdens "Speel af" volgt hij de
  speelstreep, in oefenmodus elke nieuwe noot. De huidige noot komt links in beeld met nog 1
  oude noot ervoor als context.
- De live-indicator blijft minstens een halve seconde zichtbaar na het laatste signaal, in
  plaats van meteen te verdwijnen zodra de toon wegvalt.
- Oefenmodus: alleen toonhoogte, zonder tijd.
- Niveau (Beginner/Medior/Advanced, `DIFFICULTIES`): bepaalt het tijdvenster, de drempel voor een perfecte timingbonus, en of alleen de exacte octaaf telt (advanced) of elke octaaf van dezelfde toon (beginner/medior).
- XP, reeksbonus, sterren per lied. Punten per noot zijn opgesplitst in toonpunten en timingpunten, zichtbaar in de feedbacktekst.
- Liederen staan in de array `SONGS`: `notes`, `fingers`, `bpm`, `durations` (optioneel), `draft`.
- Notenwaarden: `durations` (in tellen) laat achtste noten, gestippelde noten en halve noten toe.
  Zonder `durations` duurt elke noot één tel, zoals voorheen. Noten kunnen een mol (`Bb4`) of
  kruis (`F#4`) hebben.
- Zoekkaart onder de notenbalk: opent een Google-zoekopdracht (`site:imslp.org <term>`) in een nieuw tabblad, exact wat IMSLP's eigen zoekvak doet (nagekeken door hun zoekformulier in te vullen). Geen eigen API-aanroep: Openopus en IMSLP sturen geen CORS-header mee, dus een fetch vanuit de browser wordt altijd geblokkeerd. IMSLP heeft ook geen `Special:Search`-pagina, die eerdere aanname bleek fout. Getest met een headless browser; de linknavigatie werkt, geen noten of metadata worden automatisch overgenomen.
- "Lied toevoegen": paneel onderaan waarmee je zelf een lied overtypt vanaf een foto of pdf van
  bladmuziek (als kijkhulp ernaast, naast het live voorbeeldbalkje, wordt zelf niet bewaard) en
  opslaat. Zelf toegevoegde liederen staan in `localStorage` (`pianoCustomSongs`), niet in de
  `SONGS`-array, en krijgen automatisch `draft: true`.
- Linkerhand/baspartij: een lied kan optioneel `left: { notes, fingers, durations }` hebben
  (zelfde vorm als rechterhand). Dan tekent de notenbalk een grote notenbalk (twee regels, met
  bassleutel) en staan beide handen op de juiste tel onder elkaar. Geoefend (microfoon,
  scoring) wordt nog altijd alleen de rechterhand: de pitch-detectie herkent maar één toon
  tegelijk, dus allebei de handen tegelijk beoordelen kan nog niet betrouwbaar. De linkerhand
  staat er gedimd bij als voorbeeld en klinkt wel mee bij "Speel af" met geluid aan. "Lied
  toevoegen" heeft een los rijenblok voor de linkerhand (optioneel).
- Automatisch noten uit een foto/pdf lezen (optical music recognition) doet de app zelf niet: dat
  vraagt een externe beeldherkenningsdienst en dus een server, wat niet past bij één bestand
  zonder build-stap, en de uitkomst op een foto is sowieso niet gegarandeerd correct. In plaats
  daarvan: geef de foto aan Claude in de chat, die leest 'm nauwkeurig af (zo nodig met
  pixelmetingen op de bladmuziek) en vult het lied in.

## Let op
- Ode an de vreugde: noten en vingerzetting zijn uit het hoofd ingevuld, nog niet gecontroleerd.
- Gymnopédie nr. 1: concept (`draft: true`). Controleer tegen de partituur op IMSLP (publiek domein) voordat je die gebruikt.
- Sinterklaas (Daar wordt aan de deur geklopt): concept (`draft: true`), overgetypt van een foto
  van de bladmuziek, inclusief linkerhand. Vingerzetting is alleen zeker op de plekken waar de
  partituur een cijfer gaf; daartussen is dat cijfer aangehouden. Het laatste akkoord van de
  linkerhand (einde maat 15 en heel maat 16) is in de bron een akkoord van twee noten; de app kent
  nog geen akkoorden, dus dat is teruggebracht tot één noot (C naar F, als V-I slotcadens).
  Controleer tegen de partituur voordat je 'm serieus oefent.
- Nog niet getest met een echte piano. Test met microfoon in een rustige kamer.
- De pianosamples komen van `tonejs.github.io`. In een gesandboxde devomgeving (zoals deze
  cloud-sessie) moet dat host expliciet aan de netwerktoegang zijn toegevoegd, anders faalt de
  fetch stil en hoor je de oudere synthetische klank.
- Zoekkaart: IMSLP laat soms een "bevestig dat je geen bot bent"-pagina zien in plaats van direct de zoekresultaten. Dat is IMSLP's eigen bot-bescherming, geen fout in de app.

## Mogelijke volgende stappen
1. Testen op telefoon en laptop. Timing-venster en detectiedrempels afstellen.
2. Voortgang opslaan in localStorage.
3. Linkerhand ook kunnen oefenen (niet alleen tonen): bijvoorbeeld een handkeuze naast Niveau,
   die bepaalt welke hand de microfoon beoordeelt. Akkoorden (meerdere noten tegelijk) later.
4. Spotify-koppeling: liked songs als wensenlijst, met een voortgangsbalk per nummer.
5. Losse modules splitsen (audio, notenbalk, spelmotor) zodra het groter wordt.
6. "Lied toevoegen": noten uit een pdf met meerdere bladzijdes.

## Werkafspraken
- Schrijf teksten in de app in het Nederlands.
- Geen em dashes in teksten.
- Verander de nootdata alleen na controle tegen een bron.
