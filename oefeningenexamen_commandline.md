# Oefeningenexamen Besturingssystemen 2022 - commandline

### 1. [1 pt] Geef een overzicht van alle bestanden in de /etc map die beginnen met pass of die meer dan 100 bytes groot zijn. Eventuele fouten mogen niet op het scherm verschijnen.

`find /etc -type f \( -name "pass*" -o -size +100c \) 2>/dev/null`

Zoek (`find`) in de map /etc (`/etc`) naar bestanden (`-type f`) die beginnen met pass (`-name pass*`) of (`-o`) die meer dan 100 bytes groot zijn (`-size +100c`). Fouten verschijnen niet op het scherm (`2>/dev/null`)


### 2. [1 pt] Zelfde als 1 maar maak nu per gevonden bestand een symbolische link aan met dezelfde naam als het gevonden bestand en dat in de huidige map. Fouten mogen niet getoond worden. Dit kan met één commando!

`find /etc -type f \( -name "pass*" -o -size +100c \) -exec ln -s {} . \; 2>/dev/null`

Het ln-commando wordt uitgevoerd voor elk gevonden bestand (`-exec ln -s`) en `{}` wordt vervangen door het pad naar het gevonden bestand. `.` geeft aan dat de symbolische link in de huidige map wordt aangemaakt


### 3. [1 pt] Geneer 10 unieke getallen binnen het interval [100-200] en schrijf die naar het scherm.

`seq 100 200 | sort -R | head -n 10`

Genereer getallen in het interval [100-200] (`seq 100 200`), randomize hun volgorde (`sort -R`) en toon de eerste 10 (`head -n 10`)


### 4. [2 pt] Maak gebruik van een pipeline om 10 unieke getallen binnen het interval [100-200] samen te tellen en de som naar het scherm te schrijven.

`seq 100 200 | sort -R | head -n 10 | paste -sd+ - | bc`

De 10 getallen worden gecombineerd (`paste`) met het "+"-teken als scheidingsteken (`-sd+ -`) en de uiteindelijke expressie wordt geëvalueerd met het commando `bc`


### 5. [1 pt] Hoe kan je van een willekeurig bestand de inhoud naar het scherm schrijven zonder de laatste byte?

`head -c -1 <bestandsnaam>`


### 6. [1 pt] Hoe kan je enkel de laatste byte van een willekeurig bestand naar het scherm schrijven?

`tail -c 1 <bestandsnaam>`


### 7. [2 pt] Genereer 10 willekeurige 64 bit getallen die je in unsigned decimal vorm naar het scherm schrijft, 1 getal per lijn. Andere zaken zijn overbodig en mogen niet naar het scherm worden geschreven.

`od -An -tu8 -w8 \dev\urandom | head -n 10`

Het commando `od` leest gegevens uit `\dev\urandom` en formatteert ze naar de vorm unsigned decimal (`-tu8`) met breedte 8 bytes (`-w8`) (=64 bits) zonder adresinformatie (`-An`). Enkel de eerste 10 getallen worden getoond (`head -n 10`)


### 8. [1 pt] Hoe kan je een uniek bestand aanmaken met 6 karakters? De naam van het aangemaakte bestand wordt op het scherm getoond.

`mktemp XXXXXX`

`X` staat voor een willekeurige letter of cijfer, dus `XXXXXX` zijn 6 willekeurige letters of cijfers


### 9. [1 pt] Gegeven het bestand met als naam words.txt. Geef een overzicht van alle woorden die 4 of meer opeenvolgende klinkers bevatten. (bv. zomertoernooien)

`grep -E [aeiouy]{4} ./words.txt`


### 10. [1 pt] idem als 9 maar nu met 7 klinkers of meer die niet opeenvolgend moeten zijn. (bv. zweefvliegtuigjes)

`grep -E \(.*[aeiouy].*\){7} ./words.txt`


### 11. [3 pt] Ontbind met de opdracht factor één getal (één getal meegeven als parameter) in priemfactoren en bepaal hoeveel keer de factor 2 voorkomt. Dit aantal wordt naar het scherm geschreven.

`factor <getal> | grep -o "2" | wc -l`

`factor <getal>` ontbindt het getal in zijn priemfactoren. Daarvan worden enkel de tekens "2" overgehouden met `grep -o "2"` en het aantal regels (of dus het aantal keer dat "2" voorkomt) wordt geteld met `wc -l`
