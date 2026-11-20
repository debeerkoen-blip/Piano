# Piano leerapp (prototype)

Doel: een webapp die piano leert, met gamification. De microfoon luistert naar het spel.
Beginners eerst. Latere doelen: liedjes uit Spotify als wensenlijst met voortgang per nummer.

## Huidige stand
- `index.html` is één bestand: HTML, CSS en JavaScript, zonder build-stap.
- Microfoon: twee onafhankelijke detectoren (autocorrelatie en YIN) die het eens moeten zijn voor een toon telt, plus een state machine (zoeken/bevestigen/vasthouden). Herkent één noot tegelijk.
  Staat de microfoon nog uit, dan staat dat expliciet in de `playLabel` ("Zet eerst de
  microfoon aan...", in plaats van "Speel: ...") en krijgt de microfoonknop een zachte amber
  highlight (`.micOffHint`, dezelfde kleur als het conceptlabel, geen alarmrood: dit is een
  herinnering, geen foutmelding), zodat je niet denkt dat spelen al werkt terwijl er nog niet
  geluisterd wordt. Verdwijnt zodra de microfoon aan staat (`startMic()`/`render()`). Het
  systeemvenstertje van de browser zelf ("Toestaan dat ... je microfoon gebruikt?") kan een
  website niet stylen, dat is bewust zo (tegen misbruik).
- "Speel af": een op-tijd gespeelde noot bevestigen kost altijd even (`CONFIRM_MS` plus
  een paar meetrondes van de microfoon, in de praktijk 150-250ms). `MISS_GRACE_MS` geeft
  `tick()`'s "gemist"-sweep die tijd, anders streepte die een op-tijd gespeelde noot al
  weg vóórdat de herkenning 'm kon bevestigen (vooral merkbaar bij Advanced, waar het
  tijdvenster zelf al korter is dan die bevestigingstijd). De score zelf wordt er niet
  soepeler van: `onHeard()` rekent nog steeds met het moment waarop de toon voor het
  eerst gehoord werd, niet met het latere bevestigde moment.
- `passedIdx` hoort bij `idx`, maar loopt los daarvan mee op basis van alleen de rode
  speelstreep (geen tijdvenster, geen `MISS_GRACE_MS`). `idx` mag dankzij het niveau-venster
  (vooral ruim bij Beginner) en `MISS_GRACE_MS` een stuk achterblijven: dat is bewust, voor de
  score. Maar zonder `passedIdx` kleurden noten op de notenbalk pas lang nadat de streep ze al
  voorbij was, wat verwarrend oogde. `passedIdx` bepaalt nu de kleur (rood zodra de streep een
  noot voorbij is, zonder venster), `noteResults` wint daar altijd van: speel je 'm alsnog op
  tijd (binnen het venster), dan kleurt 'ie alsnog groen in plaats van rood. `idx` blijft de
  echte score-pointer en bepaalt (ongewijzigd) wat het pianotoetsenbord en "Speel: ..." tonen.
- Live-indicator op de notenbalk: toont wat de microfoon echt hoort (teal = betrouwbaar, amber = onzeker), ook tijdens "Stem af op C". Bedoeld om de herkenning te debuggen.
- Notenbalk: SVG. C-noten zijn oranje. Vingerzetting onder elke noot als R1 t/m R5 (rechterhand). Maatstrepen en kwartrusten (`null` in `notes`) worden getekend.
  De hele notatie (lijnafstand, notenkoppen, lettergroottes, afstand tussen noten) loopt via
  `STAFF_SCALE` (1,3) in `drawGrandStaff`, via de helper `S(v) = v * STAFF_SCALE`. Er zat te
  veel witruimte in de kaart t.o.v. hoe klein de notatie oogde; dit vergroot alles in verhouding
  (viewBox-breedte/hoogte schalen mee, dus geen scheve aspect ratio). `STAFF_X0`/`NOTE_SPACING`
  zijn de geschaalde basismaten die ook buiten `drawGrandStaff` gebruikt worden (`xOf`, `noteX`,
  `xAtTime`, `timeAtX`, `scrollStaffFollow`); die rekenen niet meer met de losse getallen 110/34.
- Notatieregels die de app volgt: maatsoort (teller/4) boven elke notenbalk, stokrichting wisselt
  bij de middenlijn (erop of erboven: stok omlaag; eronder: stok omhoog), hele noten zonder stok,
  en een kruis/mol/herstelteken wordt maar één keer per maat per toonhoogte getoond (niet bij elke
  herhaling). Nog niet geïmplementeerd: samengebalkte achtste noten (elke achtste krijgt nu een
  los vlaggetje) en een echte toonsoort/voortekening (elke verhoging/verlaging staat los bij de
  noot, er is geen mol- of kruisvoortekening aan het begin van de balk).
- Speel af: een balk loopt over de noten met aftellen (4 tellen). Tijdens het aftellen zelf
  staat er een zachte, halfdoorzichtige lay-over (`#countInOverlay`) met een groot cijfer
  boven op de notenbalk, in plaats van alleen tekst in de feedback.
- Geluid bij "Speel af": echte pianosamples (Salamander Grand Piano, CC BY 3.0, opnames van
  Alexander Holm, gehost via Tone.js op `tonejs.github.io/audio/salamander/`). 30 opnames (elke
  3 halve tonen), de tussenliggende toonhoogtes worden gepitcht met `playbackRate`. Een noot
  stopt na haar eigen tel-duur met een korte fade (demper), niet pas als de hele opname is
  uitgeklonken. `ensureAudioCtx()` maakt de audioCtx + laadt de samples (idempotent: een tweede
  aanroep hergebruikt de bestaande, in plaats van 'm te vervangen); zowel de microfoon als het
  aantikken van een losse noot gebruiken die ene functie. Geluid wordt gepland via `audioNow()`
  in plaats van rechtstreeks `audioCtx.currentTime`: die laatste is het moment waarop een
  geluid de audio-pijplijn ingaat, niet waarop het uit de speaker komt (`outputLatency`/
  `baseLatency`, soms tientallen ms). Zonder die correctie klonk elke noot steeds een beetje
  ná de rode speelstreep en de notenanimatie, die wel exact in real time lopen. Lukt het laden niet (geen internet, host geblokkeerd), dan valt de app terug op
  de oudere synthetische pianoklank (oscillators met boventonen, inharmoniciteit en een
  hameraanslag), zonder dat "Speel af" stopt. Bronvermelding staat onderaan de pagina. Naast
  "Speel af" staat een los icoontje dat altijd met pianogeluid afspeelt, ook als "Met geluid"
  in de instellingen uit staat.
- De notenbalk scrollt automatisch mee met de voortgang. Tijdens "Speel af" staat de rode
  speelstreep vast op 1/3 van het zichtbare stuk (`PLAYHEAD_FRACTION`, in `scrollStaffFollowPlayhead()`):
  de notenbalk schuift daar continu onderdoor, als een pianorol. Scrol je zelf tijdens het
  spelen, dan bevriest `tick()` meteen (geen noten die stiekem als gemist tellen) en stopt
  het geluid van vóór het slepen direct, in plaats van onhoorbaar/niet-gesynchroniseerd door
  te spelen; 300ms na het loslaten springt de speelpositie naar wat er dan onder die
  1/3-lijn staat (`seekTo()`, zonder de score te resetten) en speelt het geluid vanaf daar
  verder. In oefenmodus (geen speelstreep) komt de huidige noot nog links in beeld met 1
  oude noot ervoor als context (`scrollStaffFollow()`, de oorspronkelijke functie).
- De live-indicator blijft minstens een halve seconde zichtbaar na het laatste signaal, in
  plaats van meteen te verdwijnen zodra de toon wegvalt.
- Oefenmodus: alleen toonhoogte, zonder tijd.
- Op een noot tikken laat 'm horen (de echte pianoklank, via `ensureAudioCtx()` +
  `scheduleNoteSound()`), handig om te weten hoe de toon klinkt voordat je 'm zelf speelt.
  Alleen op de hand die je oefent (rechts), en alleen als er niet wordt afgespeeld (anders
  loopt het door elkaar met de speelstreep en het handmatig scrollen). Elke noot heeft een
  onzichtbare, ruimere tikzone (`r:18`, vóór schaling) boven op de echte notenkop, voor
  makkelijker raken met je vinger.
- Niveau (Beginner/Medior/Advanced, `DIFFICULTIES`): bepaalt het tijdvenster, de drempel voor een perfecte timingbonus, en of alleen de exacte octaaf telt (advanced) of elke octaaf van dezelfde toon (beginner/medior).
  Beginner heeft een ruim tijdvenster (2 tellen), zodat de juiste toon al telt ook al ben je
  lang niet precies op de tel; dat kost alleen de timingbonus, niet de toonpunten. Beginner is
  het standaardniveau.
- Gemiste noten (door tick()'s "gemist"-sweep, niet door een verkeerd gespeelde toon) krijgen
  een rode rand op de notenbalk in plaats van groen, zodat je in één oogopslag ziet welke noten
  je niet op tijd te pakken kreeg. Bijgehouden in `noteResults` (per noot-index 'hit' of
  'missed'), gereset bij elk nieuw lied en bij elke nieuwe poging.
- Een juist geraakte noot krijgt een korte "flits"-animatie (`animateNoteHit()`): de noot veert
  even op en er dijt een ring omheen uit die wegvaagt, zoals in ritmespellen als Guitar Hero.
  Gemiste of nog niet gespeelde noten blijven gewoon staan, geen animatie. Werkt los van
  render()/drawGrandStaff (die tekenen de hele notenbalk steeds opnieuw): de animatie wordt
  apart getriggerd, meteen na het herteken, op precies de noot die net geraakt is.
- XP, reeksbonus, sterren per lied. Punten per noot zijn opgesplitst in toonpunten en timingpunten, zichtbaar in de feedbacktekst.
- Geen voortgangsbalk en geen "Klaar! ★★★"-melding meer onder de notenbalk. In plaats daarvan
  een pianotoetsenbord (88 toetsen, A0 t/m C8, `renderKeyboard()`) dat precies laat zien welke
  toets en welke vinger nu aan de beurt is (de rechterhand, zoals ook op de notenbalk): de
  huidige toets licht paars op met de vingerzetting erop, net als de "huidige noot"-markering
  op de notenbalk. Scrollt automatisch mee zodat de huidige toets gecentreerd in beeld blijft.
  Geen highlight meer zodra het lied klaar is (`idx >= notes.length`). XP/reeks/sterren tellen
  nog gewoon door, alleen de tekstmelding bij het afronden is weg.
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
- Akkoorden: alleen bij de linkerhand (om dezelfde reden als hierboven: de microfoon herkent
  maar één toon tegelijk, dus de rechterhand blijft monofoon). Op een tel staat dan een lijst
  notenamen (`['C3','E3','G3']`) in plaats van één naam. De notenbalk tekent dat als gestapelde
  notenkoppen met één gedeelde stok (stokrichting op de toonhoogte die het verst van de
  middenlijn afstaat) en een voorteken per toonhoogte; bij "Speel af" klinken alle tonen van het
  akkoord gelijktijdig. "Lied toevoegen" heeft een "+ akkoord"-knop per rij in het
  linkerhandblok, die een extra toonhoogte-kiezer toevoegt (alleen daar, niet bij de
  rechterhand).
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
- iPhone-liggend (landscape, lage viewport): de notenbalk is vergroot op verzoek
  (`min(48vh, 250px)`, was `min(42vh, 220px)`), maar de pagina was daar al wat hoger dan het
  scherm vóór die vergroting; nu moet je iets meer scrollen om de hele kaart te zien. Geen
  nieuw probleem, wel iets groter geworden. Zie "Mogelijke volgende stappen" punt 1.
- De pianosamples komen van `tonejs.github.io`. In een gesandboxde devomgeving (zoals deze
  cloud-sessie) moet dat host expliciet aan de netwerktoegang zijn toegevoegd, anders faalt de
  fetch stil en hoor je de oudere synthetische klank.
- Zoekkaart: IMSLP laat soms een "bevestig dat je geen bot bent"-pagina zien in plaats van direct de zoekresultaten. Dat is IMSLP's eigen bot-bescherming, geen fout in de app.

## Mogelijke volgende stappen
1. Testen op telefoon en laptop. Timing-venster en detectiedrempels afstellen.
2. Voortgang opslaan in localStorage.
3. Linkerhand ook kunnen oefenen (niet alleen tonen): bijvoorbeeld een handkeuze naast Niveau,
   die bepaalt welke hand de microfoon beoordeelt. Akkoorden bij de rechterhand (en ze laten
   meebeoordelen door de microfoon) staat hier ook nog: dat vraagt polyfone toonhoogteherkenning,
   iets heel anders dan de huidige twee monofone detectoren.
4. Spotify-koppeling: liked songs als wensenlijst, met een voortgangsbalk per nummer.
5. Losse modules splitsen (audio, notenbalk, spelmotor) zodra het groter wordt.
6. "Lied toevoegen": noten uit een pdf met meerdere bladzijdes.

## Werkafspraken
- Schrijf teksten in de app in het Nederlands.
- Geen em dashes in teksten.
- Verander de nootdata alleen na controle tegen een bron.
