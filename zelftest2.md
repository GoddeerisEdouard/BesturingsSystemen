# Zelftest 2
zelf gemaakt, met uitgebreide uitleg  
don't take my word for it  
obv de kennis uit de les, dus geen chatGPT  

---
**1. Zoek naar alle bestanden of symlinks met een grootte van 1M of die de voorbije 24u werden gewijzigd.**  
oplossing:  
`find /* -type f,l -size 1M -o -mtime 0`  

uitleg:  
bestanden of symlinks zoeken doen we via `find` en bepaalde opties  
in de `man` page eens rondkijken  
Nice => in de EXAMPLES van `man find` staat een soortgelijke vraag:
"... have been modified in the last twenty-four hours"
> find $HOME -mtime 0  

       Search for files in your home directory which have been modified in the  
       last  twenty-four  hours.  This command works this way because the time  
       since each file was last modified is divided by 24 hours  and  any  re‐  
       mainder  is  discarded.  That means that to match -mtime 0, a file will  
       have to have a modification in the past which is  less  than  24  hours  
       ago.  

alle bestanden:
`/*`  

filetypes die we nodig hebben:  
`f` regular file  
`l` symlink  
types moeten met `,` samengevoegd worden.
dan hebben we dit dus al  
`-type f,l`  

een grootte van 1M:  
`-size 1M`  
ter info kan men bv `+1M` of `-1M` doen voor respectievelijk minder of meer dan een bepaalde grootte te doen.  

de voorbije 24u gewijzigd:  
`-mtime 0`  
`0` staat voor vandaag, `10` zou bv staan voor de afgelopen 10 dagen
kennen we dankzij het EXAMPLE  

het enige dat ons nu nog rest is die `of`
dit kan bij find via de optie `-o`  

**2. Geef hieronder de opdracht of pipeline van opdrachten om de bovenstaande uitvoer te ordenen op basis van het inodenummer (grootste eerst, kleinste laatst).**  
oplossing:  
`find /* -type f,l -size 1M -o -mtime 0 | ls -i | uniq | sort -r -k 1,1 | cut -d' ' -f1`  
uitleg:  

eerst en vooral dus de inodenummers bekijken:  
`-ls i`  

output uniek maken zodat we kunnen sorteren:  
`uniq`  

omgekeerd (groot -> klein) sorteren op basis van het eerste veld (de inode nummers)  
`sort -r -k 1,1`  

optioneel kan men nog op het einde de inode nummer wegwerken om exact hetzelfde formaat als vorige oefening te krijgen  
hierbij gaan we van output  
```sh
921435 mrro.txt
921413 oeuf
920066 meow.txt
```
naar output
```sh
mrro.txt
oeuf
meow.txt
```
we verwijderen de eerste kolom  
`cut -d' ' -f1`  
`-d' '`: delimiter spatie  
`-f1` eerste veld  

**3. Volgens de manpagina van find kan je met de optie -o op twee dingen testen en zal de naam van het object uitgeprint worden wanneer een van de twee voorwaarden voldaan is. Ook kan je haakjes gebruiken om dit te doen**  
**(let wel, in BASH moet je die wel escapen met een \ om aan te geven dat die haken voor find bedoeld zijn).**  
**Geef een overzicht van alle bestanden in de map /etc die een grootte hebben kleiner dan 10K of groter dan 100K.**  
oplossing:  
`find /etc \( -size -10K -o -size +100K \)`

uitleg:  
haakjes met `\` zodat deze worden geïnterpreteerd bij de `-o` aka `-or`  
men kan altijd nagaan als de output wel klopt dmv `-ls`  

**4. Geef van alle bestanden die zich bevinden in de /etc directory bevinden en die een grootte hebben van minder dan 10K het aantal lijnen. Doe dit op twee manieren:**  

**a. Met de optie -exec van find**  
`find /etc -type f -size -10k -exec wc -l {} \;`  
uitleg:  
syntax:  
`find [path] [options] -exec [command] {} \;`  

**b. Door alle namen via een pipe door te geven naar het commando xargs die dan op iedere bestandsnaam het commando wc uitvoert.**   
`find /etc -type f -size -10k | xargs wc -l`

**5. Bij het bovenstaande heb je wellicht geen rekening gehouden met bestandsnamen die spaties bevatten. Bij find kan je vragen om de bestandsnamen af te sluiten met een null-karakter i.p.v. een newline (\n) door middel van de optie -print0.**  
**Gebruik nu opnieuw xargs met bijhorende parameters om op iedere null-terminated bestandsnaam “wc -l” toe te passen.**  
`find /etc -type f -size -10k print0 |xargs -0 wc -l`  
het totaal blijft hetzelfde, niet zeker als dit wel correct is dan...  

**6. Gebruik het commando shuf om 16 getallen te genereren tussen 10 en 50 en gebruik xargs om die uit te schrijven in een raster van 4 bij 4.**
`shuf -i 10-50 -n16 | xargs -n4`

**7. Zelfde vraag als hierboven maar doe dit nu met printf (2 karakter per getal) en command substitution. Je zal zien dat dit een betere uitlijning geeft wanneer je “%2d” gebruikt voor ieder getal.**  
`printf "%2d %2d %2d %2d\n" $(shuf -i 10-50 -n 16)`  
dankzij commmand substitution wordt de uitvoer meteen meegegeven met printf

**8. Hoe worden lijneindes aangegeven in Windows en hoe gebeurt dit in Unix/Linux?**  
**Waarom is het belangrijk om te werken met tekstbestanden/scripts/... die regeleindes hebben overeenkomstig het besturingssysteem? Geef enkele voorbeelden waar het fout kan gaan.**  
in Windows: `\r\n`  
in Unix / Linux: `\n`  
wellicht bij het opsplitsen in nieuwe lijnen waarbij men bv telkens een `\r` karakter zou onnodig zoeken bij een Linux tekstbestand.  

**9. Geef de reguliere expressie om te testen of een lijn een getal bevat.**  
`[0-9]?`
**Geef nu de reguliere expressie om te testen of een lijn uitsluitend een getal bevat. Maak een testbestand aan waar je enkele lijnen naartoe wegschrijft die voldoen en die niet voldoen.**  
`^[0-9]+$`

```sh
55
voldoet niet
45115561
vo453ldoet niet
1
meow
6 6 3
```

**Test vervolgens met het commando grep uit of de reguliere expressie die je hebt uitgedacht, voldoet.**  
`grep -E "^[0-9]+$" meow.txt`


**10.Een geldig e-mailadres voldoet aan de volgende regel “naam@domeinnaam” waarbij zowel de naam als de domeinnaam kan bestaan uit verschillende delen gescheiden door punten.**  
**De (domein)naam bestaat uitsluitend uit letters en/of cijfers. Test opnieuw uit of een regel van een tekstbestand uitsluitend een geldig e-mailadres bevat (dus tussen het begin en het einde van de regel bevindt zich een e-mailadres).**  
(domein)naam:  
`[0-9a-zA-Z]+`

en nu zwieren we er nog punten tussen  
`\.`  

gecombineerd:  
`^[0-9a-zA-Z]+[0-9a-zA-Z\.]*[0-9a-zA-Z]+@[0-9a-zA-Z]+[0-9a-zA-Z\.]*[0-9a-zA-Z]+$`  

tamelijk omslachtig, ik weet het, maar het voldoet aan de beschrijving van de opgave.  
de afwisseling van `+` en `*` na de`[]` zorgt ervoor dat we niet beginnen of eindigen met een `.`.  
een deftige email validatie ziet er als volgt uit:  
`^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$`
waarbij `\w` voor om het even welk alfanumeriek karakter staat (en underscore)  

**11.Waarvoor dient in een script de shebang-lijn? (i.e. #!/bin/bash)?**
Hiermee geeft men de interpreter mee bij een uitvoerbaar bestand  
bij python zou dit dan bv `#!usr/bin/python` zijn  

**12.Wat is het type van iedere variabele in BASH?**
string  

**13.Wanneer is het handig of zelfs noodzakelijk om bij het uitschrijven van een variabele de variabelenaam tussen accolades te plaatsen?**
wanneer de variabele naam meer dan 1 karakter als lengte heeft  
bv  
`${meow}`  
zou dan niet per ongeluk kunnen worden gelezen als `${m}eow`  
=> verwarring vermijden  

**14.Schrijf een BASH-script dat een getal inleest (geen controle nodig) en de derdemacht van dat getal naar het scherm schrijft (via arithmetic expansion).**
Een intermezzo....de magie van process substition...  
eens lezen in de zelftest zelf  
heb niet echt gebruik gemaatk van het intermezzo, maar bon
```sh
#!/bin/bash

read getal;

echo $(( getal ** 3))
```
```sh
chmod a+x macht.sh
./macht.sh
```

**15.Het commando read splitst op basis van de inhoud van de IFS-variabele. Zijn er meer variabelen dan dat er elementen zijn, dan zullen de overtollige variabele leeg zijn.**  
**Zijn er minder variabelen dan dat er elementen zijn dan zal de laatste variabele het restant van de lijn bevatten. IFS-wijzigen om bv. read te laten splitsen op basis van iets anders dan whitespace kan gewoon door de volgende constructie: ...**  

**Schrijf een script dat een IPv4-adres inleest en dat daarna dat IPv4-adres opsplitst in vier variabelen. Daarna check je of ieder van deze variabelen bestaat en of dat het een getal voorstelt en of dat dat getal binnen de grenzen [0-255] ligt. Indien deze voorwaarde niet voldaan is, dan is het geen geldig IPv4-adres! Het script eindigt met een passende foutboodschap wanneer er geen geldig IPv4 adres werd ingelezen. (Met een reguliere expressie testen op de geldigheid van een IPv4-adres is niet eenvoudig maar de constructie hierboven maakt het makkelijker)**  
```sh
#!/bin/bash

read ipadres
# -r voor een escape reeks te voorkomen
# -a voor het als een array te kunnen opslaan
IFS=. read -r -a vars <<< "${ipadres}"

(( ${#vars[@]} != 4 )) && { echo "Geen geldige IPwaarde!"; exit 1; }

for waarde in "${vars[@]}"
do
    [[ -n "${waarde}" ]] && (( ${waarde} >= 0 && ${waarde} <= 255 )) || { echo "Geen geldige IPwaarde!"; exit 1; }
done

echo  "Geldige IP waarde"
```

**16.Schrijf nu hetzelfde script als hierboven maar geef het IP-adres als een parameter mee met het script. Als de parameter niet wordt meegegeven, dan wordt verondersteld dat het IP-adres dat je moet checken 192.168.16.8 is.**

```sh
#!/bin/bash
given=${@}

# := wil dus zeggen als given empty zou zijn, wordt het vervangen door ...
check=${given:=192.168.16.8}
# ofwel ipv al het bovenstaande
# (( ${#}) == 0 )) && check=192.168.16.8 || check=${@}

IFS=. read -r -a vars <<< "${check}"

(( ${#} != 4 )) && { echo "Geen geldige IPwaarde!"; exit 1; }

for waarde in "${vars[@]}"
do
    [[ -n "${waarde}" ]] && (( ${waarde} >= 0 && ${waarde} <= 255 )) || { echo "Geen geldige IPwaarde!"; exit 1; }
done

echo  "Geldige IP waarde"
```

**17.Gebruik stringoperatoren om alle klinkers uit een woord te vervangen door een punt. (probeer dit ook eens uit via de externe opdracht tr)**  
`${woord//[aeiouy]/.}`  

`echo $woord | tr [aeiuoy] '.'`  

**18.Hoe kan je van een getal dat de bestandsgrootte in bytes voorstelt en waarbij digit grouping gebruikt wordt, bv. 131.273.678, één getal maken en dat getal in kilobytes naar het scherm schrijven.**  
de `.` vervangen door niks
en vervolgens delen door duizend...

---
**Hieronder volgen enkele oefeningen op C en systeemaanroepen:**  

**19.Schrijf in C een eigen versie van het commando du. De bestandsgrootte kan je opvragen met de syscall stat.**  
inleiding:  
het commando `du` geeft de grootte van een bestand mee, gevolgd door de naam  
we gaan ervan uit dat men telkens een argument moet meegeven  
vbinvoer & uitvoer
```sh
./du_c a_file.txt
26 a_file.txt
```

omdat de opdracht niet specifieert als meerdere bestanden zijn toegelaten, heb ik gwn het `du -c <bestand>` commando herschreven:
oplossing:  
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
// voor O_RDONLY
#include <fcntl.h>

int main(int argc, char ** argv){
	// aantal  argumenten controle
	if (argc != 2){
		fprintf(stderr, "Gelieve slechts 1 argument op te geven\n");
		exit(1);
	}
	
    // hoe stat werkt staat reeds uitgelegd in ./labo_les10.md oef 7
	struct stat s;
	// als het bestand niet bestaat
	if (stat(argv[1], &s) < 0){
		fprintf(stderr, "%s bestaat niet\n", argv[1]);
		exit(1);
	} 
	
	printf("%d%10s\n", (int)s.st_size, argv[1]);

	return 0;
}
```

**20.Schrijf in C een eigen versie van het commando “wc -l” dat van de laatste 100 bytes van de bestanden die meegegeven worden op de commandolijn het aantal lijnen bepaalt. In BASH zou je dit als volgt schrijven .... `tail -c 100 bestand1 bestand2 | wc -l`. Doe dit zonder pipe en zonder execve syscalls maar dus louter met open/read/write/close/...!**
werkwijze:  
de laatste 100 bytes van een bestand = de laatste 100 karakters, want 1 byte = 1 karakter  
oplossing:  
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
int main(int argc, char ** argv){

	int totalLines = 0;
	
	// ieder argument overlopen (behalve de file naam zelf)
	for (int i=1;i<argc;i++){
		// bestand proberen uitlezen
		int fd = open(argv[i], O_RDONLY, 0);
 		if ( fd < 0){
			printf("%s is niet leesbaar\n", argv[i]);
			continue;
		}
		// informatie over bestand opslaan
		struct stat info;
		fstat(fd, &info);
		

		// grootte in bytes vh ingelezen bestand opslaan
		off_t size= info.st_size;
		/* springen nr laatste 100 bytes vh bestand
		  in "man 2 read" ondereen was te vinden dat deze functie bestond
		 vervolgens gewoon in "man lseek" de werking en het gebruik van de functie lezen
		dit deed ons ook begrijpen dat we SEEK_CUR nodig hebben (cursor wordt geplaatst op begin file 
 offset)*/
		lseek(fd, size -100, SEEK_CUR);
		// laatste 100 karakters uitlezen
		char buf[100];
		read(fd, &buf, 100*sizeof(char));
		close(fd);
		int lines=0;
		for (int character=0;character < 100; character++){
			if (buf[character] == '\n'){
				lines++;
			}
		}
		printf("%d%15s\n", lines, argv[i]);
		totalLines += lines;
	}
	printf("%d%10s\n", totalLines, "total");

	return 0;
}

```

**21.Doe nu hetzelfde als vorige vraag waar je wel gebruikmaakt van kindprocessen en een pipe als IPC. Gebruik ook execve of gelijkaardige syscalls om de programma’s aan te roepen.**  
TODO

**22.**  
TODO