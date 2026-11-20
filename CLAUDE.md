# Piano leerapp (prototype)

Doel: een webapp die piano leert, met gamification. De microfoon luistert naar het spel.
Beginners eerst. Latere doelen: liedjes uit Spotify als wensenlijst met voortgang per nummer.

## Huidige stand
- `index.html` is één bestand: HTML, CSS en JavaScript, zonder build-stap.
- Microfoon: twee onafhankelijke detectoren (autocorrelatie en YIN) die het eens moeten zijn voor een toon telt, plus een state machine (zoeken/bevestigen/vasthouden). Herkent één noot tegelijk.
  Staat de microfoon nog uit, dan krijgt de microfoonknop een zachte amber highlight
  (`.micOffHint`, geen alarmrood: dit is een herinnering, geen foutmelding), zodat je niet
  denkt dat spelen al werkt terwijl er nog niet geluisterd wordt. Verdwijnt zodra de microfoon
  aan staat (`startMic()`/`render()`). Het systeemvenstertje van de browser zelf ("Toestaan
  dat ... je microfoon gebruikt?") kan een website niet stylen, dat is bewust zo (tegen
  misbruik). Er stond hier ook een losse tekstmelding ("Speel: E4 met R3" / "Zet eerst de
  microfoon aan..."), die is weg: het pianotoetsenbord (zie verderop) laat de te spelen toets
  al zien, een tweede tekstuele versie van diezelfde info was overbodig.
- Afgesproken gedrag, geen bug: "Speel af" mét geluid (eigen luidspreker) + microfoon tegelijk
  aan levert geen herkenning op, de noten worden dan niet groen. "Speel af" zónder geluid (de
  normale manier om te oefenen: je speelt zelf op een echte piano terwijl de notenbalk/het
  toetsenbord als gids meeloopt) werkt wel gewoon, de microfoon registreert dan je eigen
  toon en toont de animaties zoals bedoeld. Reden: dit is vermoedelijk geen bug in de
  herkenning zelf. Getest (digitaal, buiten de speaker/microfoon
  om, door het geschakelde geluid rechtstreeks af te tappen naar een analyser) dat
  `detectPitch`/`detectPitchYIN` de juiste toon gewoon herkennen voor het hele notenbereik dat
  de liedjes gebruiken, zelfs met een ruwe simulatie van een telefoonluidspreker (laagfrequenties
  weggefilterd, flink wat geluidsverlies): dat wijst níet op een drempel- of algoritmeprobleem.
  Waarschijnlijker: `getUserMedia` vraagt al `echoCancellation:false`, maar mobiele browsers
  (vooral iOS Safari) negeren dat vaak, juist omdat akoestische echo-onderdrukking specifiek
  bedoeld is om het eigen afgespeelde geluid uit de microfoon-opname te filteren (zoals bij
  een videobelgesprek, zodat je jezelf niet via de ander terughoort) — dat gebeurt dan op
  OS-niveau, niet iets wat vanuit de pagina is uit te zetten. Dat verklaart ook meteen waarom
  dit juist mét oordopjes (akoestisch dan helemaal gescheiden) nooit zou werken, en waarom
  een ANDER apparaat (niet dit toestel) het geluid laten afspelen dicht bij de microfoon, wel
  zou moeten werken: dat is geen "eigen-geluid", dus geen reden voor de telefoon om het weg te
  filteren. Twee automatische detectoren het eens laten zijn blijft verder ook gewoon bedoeld
  voor een echte akoestische piano (of stem) in de kamer, niet voor het eigen afspeel-geluid van
  het toestel zelf.
  Zijdelings gevonden tijdens dit onderzoek (geen van beide relevant voor de hierboven
  gebruikte noten): de herkenning heeft een vast bereik van ongeveer 80Hz tot 1000Hz
  (`maxLag = sr/80`, `minLag = sr/1000` in beide detectoren), dus zowel een erg lage noot
  (onder C3) als een erg hoge (boven B5) valt daarbuiten.
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
- Notenbalk: SVG. C-noten zijn oranje. Vingerzetting onder elke noot als L1 t/m L5 (linkerhand).
  De rechterhand (R1 t/m R5) kreeg vroeger ook een label onder de noot, maar dat staat nu
  al op het pianotoetsenbord (zie verderop), dus dat label op de notenbalk is weg: twee keer
  dezelfde vingerzetting tonen was overbodig en gaf drukte. Maatstrepen en kwartrusten (`null` in `notes`) worden getekend.
  De vaste canvashoogte in `drawGrandStaff` (`fullHeight`) reserveert standaard genoeg ruimte
  boven de g-sleutel voor een paar hulplijntjes, ook als dit specifieke lied daar nooit gebruik
  van maakt: dan oogde dat als onnodige witruimte boven de notenbalk. De viewBox snijdt dat stuk
  nu weg (`topCrop`, alleen het zichtbare kader verandert, alle tekencoördinaten blijven
  ongewijzigd) tot net genoeg voor de sleutel, of iets meer als dit lied toch hogere noten heeft
  (`usedRightSteps`, per lied opnieuw berekend over alle noten, niet alleen het zichtbare stuk,
  anders zou de marge tijdens het spelen zichtbaar verspringen).
  De hele notatie (lijnafstand, notenkoppen, lettergroottes, afstand tussen noten) loopt via
  `STAFF_SCALE` (1,3) in `drawGrandStaff`, via de helper `S(v) = v * STAFF_SCALE`. Er zat te
  veel witruimte in de kaart t.o.v. hoe klein de notatie oogde; dit vergroot alles in verhouding
  (viewBox-breedte/hoogte schalen mee, dus geen scheve aspect ratio). `STAFF_X0`/`NOTE_SPACING`
  zijn de geschaalde basismaten die ook buiten `drawGrandStaff` gebruikt worden (`xOf`, `noteX`,
  `xAtTime`, `timeAtX`, `scrollStaffFollow`); die rekenen niet meer met de losse getallen 110/34.
- De hele kaart (titel, statistieken, werkbalk, notenbalk, toetsenbord, feedbacktekst) past op
  één scherm, zonder dat je tijdens het spelen hoeft te scrollen (alleen de bronvermelding
  onderaan de pagina mag daarbuiten vallen, dat is geen interactief onderdeel). Twee plekken
  hadden daar apart werk voor nodig, in allebei de gevallen door de notenbalk zelf een kleinere,
  schermhoogte-afhankelijke hoogte te geven (`width:auto; min-width:0`, met horizontaal scrollen
  voor lange liederen zoals al gebeurde op een smalle telefoon) in plaats van op volle breedte
  te schalen:
  - Liggende telefoon (`@media (orientation:landscape) and (max-height:500px)`): naast de
    notenbalk zelf (`height:min(44vh,230px)`) zijn ook titel, statistieken en feedbacktekst
    compacter gemaakt (kleinere `min-height`/`padding`/`line-height`, minder marge ertussen):
    die hadden allemaal een eigen minimale knop-/touch-target-hoogte (44px) die in deze context
    meer ruimte opeist dan de inhoud zelf nodig heeft.
  - Rechtop (portretstand) met een lied met linkerhand (grote notenbalk, twee regels, van
    zichzelf hoger): `render()` zet een klasse `hasLeft` op `.staffWrap` zodra er een linkerhand
    is (`contextNotes.length > 0`), en alleen in portretstand (niet in de liggende/brede
    varianten hierboven, die hebben al hun eigen hoogte) krijgt de notenbalk dan
    `height:clamp(140px, calc(100vh - 400px), 420px)`: `calc(100vh - 400px)` is de ruimte die
    overblijft nadat de rest van de kaart (gemeten, vrij stabiel ongeacht schermbreedte) zijn deel
    al heeft gehad, `clamp()` voorkomt dat dat op een heel klein of heel groot scherm absurd
    uitpakt. Een lied zonder linkerhand (de normale, enige hand die je oefent) is van zichzelf al
    laag genoeg en blijft gewoon op volle breedte schalen, ongewijzigd.
- Notatieregels die de app volgt: maatsoort (teller/4) boven elke notenbalk, stokrichting wisselt
  bij de middenlijn (erop of erboven: stok omlaag; eronder: stok omhoog), hele noten zonder stok,
  en een kruis/mol/herstelteken wordt maar één keer per maat per toonhoogte getoond (niet bij elke
  herhaling). Achtste noten die binnen dezelfde tel op elkaar volgen krijgen een gezamenlijke balk
  (`computeBeamGroups`/`buildBeamLookup`) in plaats van elk een los vlaggetje; een achtste die
  alleen in zijn tel staat (bijvoorbeeld na een gestippelde noot, of onderbroken door een rust)
  krijgt nog gewoon een vlaggetje. Nog niet geïmplementeerd: een echte toonsoort/voortekening
  (elke verhoging/verlaging staat los bij de noot, er is geen mol- of kruisvoortekening aan het
  begin van de balk).
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
- Gemiste noten (door tick()'s "gemist"-sweep, niet door een verkeerd gespeelde toon) kleuren
  helemaal rood op de notenbalk (notenkop én rand, niet alleen een rand om een verder normale
  notenkop: dat was te subtiel om in één oogopslag te zien), een goed gespeelde noot helemaal
  groen. Bijgehouden in `noteResults` (per noot-index 'hit' of 'missed'), gereset bij elk nieuw
  lied en bij elke nieuwe poging. Geldt alleen voor de rechterhand: de linkerhand (ter context,
  niet beoordeeld omdat de pitch-detectie maar één toon tegelijk herkent) blijft altijd gewoon
  zwart, ook als die toon tegelijk "voorbij" is volgens de speelstreep, want er is geen oordeel
  om te tonen. De paarse "dit moet je nu spelen"-markering is geen oordeel en blijft daarom wel
  gewoon een rand om een normale notenkop, niet de hele noot. Geen aparte "Gemist: ..."-tekst
  in de feedback meer bij een gemiste noot (de rode notenkop en de stand van de speelstreep
  laten al zien wat er gemist is); bij een op tijd gespeelde noot staat er nog gewoon een
  positieve feedbacktekst.
- Een juist geraakte noot krijgt een korte "flits"-animatie (`animateNoteHit()`): de noot veert
  even op en er dijt een ring omheen uit die wegvaagt, zoals in ritmespellen als Guitar Hero.
  Gemiste of nog niet gespeelde noten blijven gewoon staan, geen animatie. Werkt los van
  render()/drawGrandStaff (die tekenen de hele notenbalk steeds opnieuw): de animatie wordt
  apart getriggerd, meteen na het herteken, op precies de noot die net geraakt is.
- XP, reeksbonus, sterren per lied. Punten per noot zijn opgesplitst in toonpunten en timingpunten, zichtbaar in de feedbacktekst.
- Geen voortgangsbalk en geen "Klaar! ★★★"-melding meer onder de notenbalk. In plaats daarvan
  een pianotoetsenbord (88 toetsen, A0 t/m C8, `renderKeyboard()`) dat precies laat zien welke
  toets en welke vinger nu aan de beurt is (de rechterhand): de huidige toets licht paars op
  met de vingerzetting erop, net als de "huidige noot"-markering op de notenbalk. De
  linkerhand-noot die er op dat moment bij hoort (`currentContextMidis()`, ook bij een
  akkoord) licht teal op, ter context, zonder vingerzetting: alleen de rechterhand wordt
  geoefend. Geen highlight meer zodra het lied klaar is (`idx >= notes.length`). XP/reeks/sterren
  tellen nog gewoon door, alleen de tekstmelding bij het afronden is weg.
  Het toetsenbord staat vast: alle 88 toetsen schalen naar de beschikbare breedte in plaats van
  op ware grootte te scrollen met een automatisch meebewegende (en dus steeds verspringende)
  scrollpositie. Altijd zichtbaar, ook rechtop op een smalle telefoon: 88 toetsen passen dan niet
  breed genoeg om goed leesbaar te zijn, maar dat mag best klein uitpakken (op verzoek), beter
  dan 'm helemaal te verbergen. Rechtop staat het toetsenbord gewoon in de normale doorloop van
  de kaart, onder de notenbalk.
  Op een telefoon in liggende stand of een breder scherm (media query op `orientation: landscape`
  of `min-width: 700px`, dezelfde drempel als voor een tablet/laptop) staat het toetsenbord
  bovendien met `position:fixed` vast onderaan het scherm (niet gewoon ergens in de
  meescrollende kaart): op een telefoon in liggende stand is de kaart vaak hoger dan het scherm
  (vooral bij een lied met linkerhand), dus zonder dit zou je tijdens het spelen zelf moeten
  scrollen om het toetsenbord te zien, en leek het net of het "bewoog" omdat zijn plek op het
  scherm niet stabiel was. `main` krijgt dan evenveel extra `padding-bottom` gereserveerd (108px,
  of 76px in de extra compacte telefoon-liggend-variant) zodat deze vaste balk niet over de
  feedbacktekst heen schuift.
  `renderKeyboard()` kleurt de huidige toets op dezelfde pointer als de notenbalk gebruikt
  (`playing ? passedIdx : idx`, zie hierboven), niet zomaar op `idx` zelf: tijdens Speel af liep
  de paarse toets anders niet synchroon met de paarse markering op de notenbalk, omdat `idx` dan
  bewust wat coulance heeft (venster/`MISS_GRACE_MS`) en `passedIdx` niet.
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
- Na elke update kan het zijn dat je op je telefoon (vooral mobiele Safari) nog een oude,
  gecachte versie van de pagina ziet, ook na een gewone herlaad: GitHub Pages stuurt een
  `cache-control: max-age=600`-header mee, en Safari hergebruikt soms een lokale kopie zonder
  die te verversen. Zie je iets dat niet klopt met wat hier in CLAUDE.md staat (bijvoorbeeld
  vingerzetting die er nog oud uitziet), controleer dan eerst of het om een verouderde cache
  gaat voordat je een "bug" gaat zoeken: open de pagina in een privénavigatievenster (dat
  gebruikt nooit een cache) of sluit het tabblad helemaal en open de link opnieuw. Er staan
  best-effort no-cache meta-tags in de `<head>`, maar browsers negeren die meestal voor de
  echte HTTP-cache.
- Ode an de vreugde: noten en vingerzetting zijn uit het hoofd ingevuld, nog niet gecontroleerd.
- Gymnopédie nr. 1: nog `draft: true`, maar de noten en duren komen nu uit een machineleesbare
  bron (Mutopia Project, zie de toelichting in `SONGS`), niet meer van een foto. Wat nog wel
  eigen werk is: vingerzetting (zelf bedacht, niet gecontroleerd) en een paar vereenvoudigingen
  waar de bron meer stemmen tegelijk heeft dan de app kan weergeven (zie de toelichting bij het
  lied zelf voor precies welke maten).
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
