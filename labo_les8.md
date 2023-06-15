# Deel V: Programmeren in Bash - vervolg

## opdrachten (p.31/51)
**94**  
hernemen maar dan met array  
én bepaalde uitzonderingen zoals unknown host  
(wat ook bij de "foute hostnamen" lijst zou terecht moeten gekomen zijn)  
bv  
`grep -B2 host ping.out`  
geeft als output
```sh
C:\WINDOWS\system32>ping -n 1 AL1103

Unknown host AL1103.
--
# enz enz
```

dus dit zou dan een script ervoor zijn;
```sh
#!/bin/bash
# -a zal via IFS automatisch de lijn "splitsen" en toekennen aan de variabele (array) in dit geval
while read -a array ;do
    if [[ "${array[0]}" =~^C ]];then
        # als de hostname een niet-van-null verschillende string is
        if [[ -n "$hostname" ]];then
            echo "$hostname"
        fi

        hostname="${array[3]}"
    fi

    if [[ "${array[0]}" =~ ^Reply ]];then
        unset hostname
    fi
done < ping.out
```

**95. Gebruik (enkel) het bestand /etc/passwd om voor alle groepsnummers het aantal gebruikers met hetzelfde primaire groepsnummer te tellen. Realiseer dit op twee manieren:**
**• Gebruik een while-lus met een read-commando om het bestand te overlopen en arrays om de gegevens op te slaan.**  
output formaat van /etc/passwd:  
`user:x:uid:gid:...`  
we hebben dus het gid nodig  
voorbeeld output van script:
```sh
0:5
1:2
2:1
5:1
```

```sh
#!/bin/bash

# numeriek geïndexeerde tabel declareren (dit is standaard, dus hoeft niet per se hier):
declare -a array

# ter info: string " tabel declareren
#declare -A array

while IFS=: read username x uid gid rest ; do
    # -z staat voor als hetgene wat volgt geëxpandeerd wordt nr een nullstring, dan is [[]] true
    # aka: als de sleutel niet bestaat, maken we hem aan
    # het aanmaken is wat volgt bij &&
    # als hij wel al bestaat (dus ||), dan voeren we dat andere commando uit
    [[ -z "${array[$gid]}" ]] && array[$gid]=1 || ((array[$gid]++))
done < /etc/passwd

# sleutels overlopen, indien men over de waardes zou willen itereren,
# dan is het zonder het !
for i in ${!array[@]};do
    echo "$i:${array[$i]}"
done
```

**• Sla eerst de gegevens geordend op in een tijdelijk bestand, en verwerk vervolgens dit bestand.**  
overgeslagen  

**96. Gebruik de bestanden /etc/group en /etc/passwd om een overzicht te maken van alle groepen, gevolgd door de volledige lijst van gebruikers die deze groep als primaire groep hebben. Gebruik een while-lus met een read-commando om het bestand /etc/group te overlopen en grep om de gebruikers op te sporen.**  
overgeslagen  

**97. Ontwikkel een script met juist twee parameters. De eerste parameter is de naam van een directory tree, de tweede parameter stelt een aantal bytes voor. Het script genereert de naam van alle bestanden in de directory tree waarvan de grootte de waarde van de tweede parameter overschrijdt. Bovendien wordt het totale aantal bestanden dat aan deze voorwaarde voldoet en het totale aantal bytes in deze bestanden gerapporteerd. Tip: Gebruik het find-commando met passende opties om de individuele bestanden te vinden. Gebruik de optie -printf om de noodzakelijke informatie op te vragen tijdens het zoeken.**
```sh
#!/bin/bash

# controleer aantal parameters
(($#!=2)) && { echo Usage 97.sh dir bytes >&2 ; exit 1 ; }

# parameters elk afzonderlijk controleren

# als 1e param geen dir is
[[ ! -d "$1" ]] && {echo "$1 is geen directory!" >&2 ; exit 1 ; }

# als 2e param gen getal is
# opnieuw opgelet dat bij =~ je de reguliere expressie NIET tss "" zet, bij grep MOET dit
[[ ! "$2" =~ ^[0-9]+$ ]] && { echo "$2 stelt geen getal voor!" >&2 ; exit 1 ; }

aantal=0
totaal=0
while read size file; do
    ((totaal+=size))
    ((aantal++))
    echo "$size $file"
# bij de -size optie staat de + voor "meer dan" en de c voor bytes
done < <(find "$1" -type f -size +${2}c -printf "%s %p\n")

echo "Totaal aantal gevonden bytes => $totaal"
echo "Totaal aantal bestanden => $aantal"
```

---
**De for-lus**  
syntax wordt later toegepast  

**99. Bepaal voor een groep, waarvan het groepsnummer als enige parameter wordt meegegeven, de volledige lijst van gebruikersaccounts die behoren tot deze groep (ook als niet-primaire groep). Construeer twee oplossingen:**
**• Schrijf eerst alle gebruikersnamen weg naar een tijdelijk bestand; dubbels zijn voorlopig toegestaan. Filter vervolgens de dubbels hieruit en schrijf de resulterende gebruikerslijst uit.**  
**• Gebruik een associatieve array, met de gebruikersnamen als sleutels.**  
overgeslagen  

**100. Ontwikkel een script dat alle parameters uitschrijft die meer dan één keer voorkomen in de argumentenlijst van het script. De volgorde waarin de minstens dubbel voorkomende parameters worden uitgeschreven heeft geen belang (sorteren mag), maar je moet er wel voor zorgen dat parameters die meer dan twee keer voorkomen toch slechts eenmaal weggeschreven worden. Laat als eerste parameter ook eventuele opties -i of -I (van ignore case) toe, die desgewenst aangeven dat er geen onderscheid mag gemaakt worden tussen hoofdletters en kleine letters.**  
**Tip: denk terug aan instructies uit voorgaande oefeningen!**
```sh
#!/bin/bash

if [[ -n "$1" ]];then
    # de shift schuift alle positionele parameters 1 positie op nr links schuiven & valt de meest linkse weg
    # dit is hier nodig zodat men bv niet ./100.sh -i -a -i als zou zien als  dubbel gebruik van -i
    # de eerste -i geeft namelijk de case insensitivity aan
    [[ "$1" == "-i" || "$1" == "-I" ]] && { ignore=1; shift ; }

# reminder dat met -A er dus een string indexering zal zijn
declare -A array

for i in "$@";do
    # -z betekent true if length of string is zero
    # ook $i met "" rond omdat het een string is
    [[ -z ${array["$i"]} ]] && array["$i"]=1 || ((array["$i"]++))

# alle keys overlopen
for i in ${!array[@]};do
    # ${i,,}  => lowercase alles
    ((ignore==1)) && i=${i,,}
    ((array["$i"]>1)) && printf "%s\n" "$i"
done
```
vb van commando gebruik  
`./100.sh -z -a -a -e -e -f -f -eee -eee -z`  
of met -i gebruik  
`./100.sh -a -A -b -c -d -e -E -f -f`  

**101. Ontwikkel een script dat een beperkte versie van het commando wc simuleert. Het script moet het aantal regels en de bestandsnaam afdrukken van elk bestand dat als parameter meegegeven wordt. Het script mag enkel interne Bash-instructies (if, for, case, let, while, read, echo enz.) gebruiken en geen externe commando's; het gebruik van awk, sed, perl en wc in het bijzonder is niet toegelaten. Je zult bijgevolg elk bestand regel voor regel moeten inlezen en deze tellen. Het script moet bovendien een samenvattende regel weergeven met het totale aantal regels. Indien geen enkele parameter meegegeven wordt, neem je alle bestanden in de huidige werkdirectory in beschouwing. Los dit zo beknopt mogelijk op met de speciale notaties voor shell- variabelen**  

```sh
#/bin/bash

tot_lijnen=0
tot_woorden=0
tot_kars=0
for i in "$@";do
    [[ ! -f "$i" ]] && echo "$i is not a file" >&2
    lijnen=0
    woorden=0
    kars=0
    #foutboodschhaop vn read ook weggooien nr /dev/null
    while read lijn 2>/dev/null;do
        ((lijnen++))
        for j in $lijn;do
            ((woorden++))
        done
        # ofwel
        # array=( $lijn )
        #((woorden+=${#array[@]}))

        # +1 wegens newline die eigenlijk ook moet meegeteld worden
        ((kars=kars+${#lijn}+1))
    done < "$i"

    printf "%4d %4d %4d %+s\n" $lijnen $woorden $kars $i

    ((tot_lijnen+=lijnen))
    ((tot_woorden+=woorden))
    ((tot_kars+=kars))
done

(($#>1)) && printf "%4d %4d %4d total\n" $tot_lijnen $tot_woorden $tot_kars

```