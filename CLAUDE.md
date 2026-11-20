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
- Liederen staan in de array `SONGS`: `notes`, `fingers`, `bpm`, `draft`.

## Let op
- Ode an de vreugde: noten en vingerzetting zijn uit het hoofd ingevuld, nog niet gecontroleerd.
- Gymnopédie nr. 1: concept (`draft: true`). Controleer tegen de partituur op IMSLP (publiek domein) voordat je die gebruikt.
- Nog niet getest met een echte piano. Test met microfoon in een rustige kamer.

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
