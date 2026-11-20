# Piano leerapp (prototype)

Doel: een webapp die piano leert, met gamification. De microfoon luistert naar het spel.
Beginners eerst. Latere doelen: liedjes uit Spotify als wensenlijst met voortgang per nummer.

## Huidige stand
- `index.html` is één bestand: HTML, CSS en JavaScript, zonder build-stap.
- Microfoon: autocorrelatie in de Web Audio API (`detectPitch`). Herkent één noot tegelijk.
- Notenbalk: SVG. C-noten zijn oranje. Vingerzetting onder elke noot als R1 t/m R5 (rechterhand).
- Speel af: een balk loopt over de noten met aftellen (4 tellen). Noten tellen binnen een venster van een halve tel.
- Oefenmodus: alleen toonhoogte, zonder tijd.
- XP, reeksbonus, sterren per lied (op basis van fouten en gemiste noten).
- Liederen staan in de array `SONGS`: `notes`, `fingers`, `bpm`, `durations` (optioneel), `draft`.
- Notenwaarden: `durations` (in tellen) laat achtste noten, gestippelde noten en halve noten toe.
  Zonder `durations` duurt elke noot één tel, zoals voorheen. Noten kunnen een mol (`Bb4`) of
  kruis (`F#4`) hebben.
- "Lied toevoegen": paneel onderaan waarmee je zelf een lied overtypt vanaf een foto of pdf van
  bladmuziek (alleen als kijkhulp ernaast, wordt niet bewaard) en opslaat. Zelf toegevoegde
  liederen staan in `localStorage` (`pianoCustomSongs`), niet in de `SONGS`-array, en krijgen
  automatisch `draft: true`.

## Let op
- Ode an de vreugde: noten en vingerzetting zijn uit het hoofd ingevuld, nog niet gecontroleerd.
- Gymnopédie nr. 1: concept (`draft: true`). Controleer tegen de partituur op IMSLP (publiek domein) voordat je die gebruikt.
- Sinterklaas (Daar wordt aan de deur geklopt): concept (`draft: true`), overgetypt van een foto
  van de bladmuziek. Alleen de rechterhand/melodie, geen baspartij of akkoorden. Vingerzetting is
  alleen zeker op de plekken waar de partituur een cijfer gaf; daartussen is dat cijfer
  aangehouden. Controleer tegen de partituur voordat je 'm serieus oefent.
- Nog niet getest met een echte piano. Test met microfoon in een rustige kamer.

## Mogelijke volgende stappen
1. Testen op telefoon en laptop. Timing-venster en detectiedrempels afstellen.
2. Voortgang opslaan in localStorage.
3. Rechterhand en linkerhand scheiden. Akkoorden later.
4. Spotify-koppeling: liked songs als wensenlijst, met een voortgangsbalk per nummer.
5. Losse modules splitsen (audio, notenbalk, spelmotor) zodra het groter wordt.
6. "Lied toevoegen": noten uit een pdf met meerdere bladzijdes, of een los baspartij-veld voor de
   linkerhand.

## Werkafspraken
- Schrijf teksten in de app in het Nederlands.
- Geen em dashes in teksten.
- Verander de nootdata alleen na controle tegen een bron.
