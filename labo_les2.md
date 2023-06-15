# Deel II: Compileren in de Shell

`htop`: dynamisch procesgebruik zien  
dit commando kan bv voorkomen dat bepaalde processen gebruikt worden  
er kan ook een prioriteit voor processen worden ingesteld (hoe minder 'nice', hoe hoger de prioriteit)


# Deel III: Profiler programma’s
enkele informatieve commando's  
zoals `strace` (systemcall trace) (om na te gaan als er bv een kindproces wordt aangemaakt ~fork(), ...)  
en dergelijke
 
# Deel IV: Een eerste programmeeropdracht
opdracht is te moeilijk dus mag overgeslagen worden

# Deel V: Programmeren in Bash

uitvoeren van commando's gebeurt eigenlijk altijd in 2 stappen  
1. fork(): maakt een kindproces aan (die een kopie van het ouderproces is, meerbepaald de shell)  
2. exec(): via exec systemcall wordt process overschreven met het uitvoerbare dat je opgeeft op de commandolijn

## opdrachten (p.13/51)
**1. Met welk van de commando’s cp, dd, ln, mktemp, touch en cat kan je vlug een aantal (lege) bestanden aanmaken waarvan de namen als parameters van het commando worden opgegeven?**  
als men `man <commando>` uitvoert kan men onder NAME of DESCRIPTION een korte beschrijving vinden wat elk commando doet.  

Uit de description van `touch` halen we  
> Update  the  access  and modification times of each FILE to the current time.  
A FILE argument that does not exist is created empty, ...  

voorbeeld van gebruik: `touch a b c d` maakt bestanden aan van a b c en d aan (ofwel via een interval `touch {a..d}`)  

**2. Wat verschijnt er op het scherm indien je de opdracht `head /etc/passwd` uitvoert?**  
de eerste 10 lijnen  
**Zoek nu de verwante opdracht voor het tonen van de laatste lijnen van een bestand. Hoe kan je steeds de laatste lijnen van een bestand op het scherm laten verschijnen wanneer er een ander proces achteraan het bestand lijnen toevoegt?**  
laatste lijnen: `tail /etc/passwd`  
via `-n` optie het aantal laatste lijnen kiezen  
via `-c` de laatste bytes (=karakter, want 1 byte = 1 karakter)  
voorbeelden:  
- laatste karakter van een bestand uitschrijven:  
    `tail -c 1 <bestand>`  
    bij /etc/passwd is het laatste karakter een newline, dus zal je niks als output zien  

ander ongerelateerde vraag (kwam wel voor op een vorig examen) die opgelost werd:  
**"Geef een willekeurig 64bit getal"**  
We weten dankzij `strace mktemp -u` dat /dev/urandom wordt gebruikt in mktemp om random getallen te zoeken, dus dit gaan we dan ook gebruiken:  
`head -c 8 /dev/urandom`  
=> dit print telkens 8 verschillende random bytes  
Dit zijn momenteel bytes, we moeten dit nog presenteren als een getal, dit kan gebeuren via een convert commando  
`od` die bytes kan converteren naar decimalen of andere formaten.  
converteren naar decimaal  (dus zowel positief als negatief) `od -An -td8`  
converteren nr unsigned (dus enkel positief) `od -An -tu8`  
De `-An` optie moet er standaard bij, ander krijg je adresprefixen erbij  
`-t` wil zeggen converteren naar... en wat volgt is het formaat (unsigned / decimaal / character (c) )  

Dus `head -c 8 /dev/urandom | od -An -td8`  
als men nu ipv 1 getal 8 getallen zou willen tss 0 & 255:  
`head -c 8 /dev/urandom | od -An -tu1`  


**3. Waarvoor dient de optie -rf bij de opdracht rm? Maak met een editor een bestand aan met als naam “-rf”. Hoe kan je het bestand “-rf“ verwijderen?**  
> -r, -R, --recursive  
              remove directories and their contents recursively  
=> dus inhoud van mappen zal ook verwijderd worden  

> -f, --force  
              ignore nonexistent files and arguments, never prompt  
=> niet telkens bevestigen met "yes"  

open vim via `vi` typ `:w -rf` sluit vi met `:q`  
om dit bestand nu te verwijderen:  
`rm -- -rf`  
`--` geeft aan dat men op het einde van de opties zit  
**4. Welke opties moet je toevoegen aan het commando wc om enkel de grootte van een bestand te tonen zonder extra informatie?**  
de optie `-c` of `--bytes`  
opgelet! dit is enkel nuttig bij tekstbestanden en bv niet binaire bestanden  

**5. In Bash zijn er ook Bash-builtin opdrachten zoals cd, set, pwd, exec, printf en : waarvoor er geen aparte man-pagina’s beschikbaar zijn. Een overzicht kan je bekomen door man builtin of door de man-pagina van Bash op te vragen. Zoek informatie op over het gebruik van de opdrachten cd, set, pwd, exec, printf en :.**  
Bij built-in commando's best gebruik maken van `help` ipv `man` wegens dat men anders te veel info zou krijgen.  
dus bv  
`help cd | less`  

`cd`: change current directory  
`set`: geeft lijst van functies en variabelen die door het systeem ~bash gekend zijn  
`pwd`: print working directory -> schrijft volledige absolute pad uit  
`exec`: voert gegeven systemcall uit  
    dus bv `exec ls` uitvoeren in terminal zal het venster sluiten en overschrijven met wat ls gaf  
`printf`: formatteert end print argumenten volgens een gegeven formaat  
`:`: null commando, doet niks, exit status slaagt altijd  
**6. Wat doet het commando sync?**  
`man sync`  
> DESCRIPTION  
       Synchronize cached writes to persistent storage  
       -  
       If one or more files are specified, sync only them, or their containing
       file systems.  

=> alles wat met schrijfopdrachten in de disk te maken heeft worden niet meteen uitgevoerd, deze worden opgespaard tot het systeem eens een gunstig moment vindt.  
(bv bij een USB veilig te ejecteren is men er zeker van dat gecachte schrijfopdrachten doorgevoerd worden)  

alle buffers flushen &  
alle die gecachte opdrachten doorvoeren  
**7. Hoe kan je met het commando dd een afbeelding maken van een USB-pen? Welke device-file heb je hiervoor nodig? Bekijk de uitvoer van de opdracht “fdisk -l”.**  
Device files kunnen bekeken worden via `lsblk`  
vervolgens kan via `dd if=<MOUNTPOINT_FROM_LSBLK> of=disk.img` een afbeelding ervan krijgen  
bv `dd if=/dev/sdb1 of=disk.img`  
kan ook via  
`cat /dev/sbd1 > bestand`  
PS. dit blijft altijd even hangen omdat hij ermee bezig is.  

`fdisk -l` geeft:  
> List  the  partition  tables  for the specified devices and then exit.  If no devices are given, those mentioned in /proc/partitions (if that file exists) are used

**8. Hoe kan je met dd een kopie maken van de eerste 512 bytes van de vaste schijf? Bekijk met het commando strings welke tekststrings in die 512 bytes verscholen zitten.**  
zonder dd: `head /dev/sda -c 512`  
met dd: `dd if=/dev/sda of=mbr count=1 bs=512`  
`/dev/sda`: vaste schijf mount  
`bs=`: aantal bytes  
`count=`: aantal blokken van gegeven bytes  

`strings <file>` schrijft alle gevonden tekst uit nr het scherm  
in ons geval hadden we `strings mbr` nodig  
**9. Gebruik het commando find om een lijst van bestanden te krijgen die de afgelopen 24u nog werden aangepast.**  
werd overgeslagen  

**10. Met het commando wodim kan je van op de opdrachtlijn een CD/DVD-branden. Met het commando genisoimage kan je een ISO-bestand aanmaken. Hoe kan je van de inhoud van de /root directory een ISO-bestand maken?**  
wordt overgeslagen  

**11. Bij vraag 10 zal je merken dat de namen van de bestanden/directories werden gewijzigd. Je kunt dit vermijden door de image in Joliet-formaat weg te schrijven.**  
werd overgeslagen