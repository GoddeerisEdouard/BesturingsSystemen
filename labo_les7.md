# Deel V: Programmeren in Bash - vervolg

## opdrachten (p.28/51)

**85. 86. 87. 88.**  
overgeslagen  

---
**Herhalingsinstructies**  
**Output van een commando verwerken in een script**  
intro tekst samengevat  
uitvoer van commando opslaan in een variabele in een script kan op verschillende manieren:  
- `cut` met een delimiter
- volledige regel opslaan & stringoperatoren gebruiken
- `read` in combinatie met `IFS` om argumenten op te splitsen
opletten met pipen, gebeurt enkel door standaardinvoer
- dmv arrays alle delen opslaan in arrayelementen mbv `IFS`
`IFS` is standaard op whitespace (zijnde spatie, tag & regeleinde)
- && en of || operatoren kunnen vaak if testen vervangen

**89. Ontwikkel een shellscript dat als (enige) parameter een bestandsnaam heeft. Als output moet het script het gemiddelde aantal karakters per regel en het gemiddelde aantal woorden per regel uitschrijven. Gebruik de output van het commando wc en probeer met de diverse methodes die hierboven aangehaald werden (behalve de tweede) om deze output op te splitsen. Opgelet: de shell behandelt alle getallen als integers. Niettemin kun je er, met behulp van wat wiskunde, voor zorgen dat delingen correct worden afgerond. Hoe?**
```sh
#!/bin/bash

# als een parameter meegegeven is...
if (($#!=1));then
    echo "Fout aantal parameters" >&2
    exit 1
fi
# als parameter wel bestaand bestand is
# opgelet, gebruik spaties!
if [[ ! -f "$1" ]];then
    echo "$1 is geen regulier bestand" >&2
    exit 1
fi


# de 3 getallen (length words count) eruit halen
# die laatste <(...) heet proces restitutie
# want bij bv 
# commando1 | commando2
# verliest commando1 alle ingestelde variabelen van commando2 omdat die werd uitgevoerd in een subshell
# dit kan je dus "oplossen" door proces restitutie
# commando1 > >(commando2)
# dat was een vb van output redirection
# men zou ook output redirection kunnen toepassen:
# commando2 < <(commando1)
read l w c rest < <(wc "$1")
# kan ook via een tabel:
# array=( $(wc "$1") )
# l=${array[0]}
# w=${array[1]}
# c=${array[2]}

((gemw=(w+l/2)/l))
((gem_c=(c+l/2)/l))
# dit kan uiteraard ook
# gem_c=$((c+l/2)/l))

# "" zeker errond zodat "=" tekens  niet verkeerd geïnterpreteerd worden
echo "Gemiddeld aantal woorden per lijn => ${gem_w}"
echo "Gemiddeld aantal karakters per lijn => ${gem_c}"
```
**90. Wat moet je doen opdat de read-instructie lijnen niet alleen in woorden zou opsplitsen op basis van spaties, tabs en regeleinden, maar ook op basis van :-tekens?**
De IFS aanpassen
`IFS=${IFS}:`

**91. Ontwikkel een script om de gebruikersnaam (veld 1 van  etc/passwd) te bepalen aan de hand van een gebruikers-ID (veld 3 van /etc/passwd) dat je als (enige) parameter aan het script meegegeven hebt. Je kunt de output van het commando grep zowel met cut, read als met een array analyseren.**
/etc/passwd output formaat: `naam:x:uid:gid:...`
we gaan gebruik maken van grep

```sh
#!/bin/bash

# check aantal parameters
if (($#!=1));then
    echo "Usage: $0 <user_id>"
    exit 1
fi

#controleren als $1 een getal is
# ipv grep kunnen we ook gebruik maken van =~ vr reguliere expressies
# [1-9][0-9] dit vermijdt dat we 00 zouden kunnen ingeven
# opgelet, bij =~ de reguliere expressie niet tss quotes zetten (anders maakt bash er een string van)
# bij grep daarentegen moet je wel quotes errond zetten
if [[ ! "$1" =~ (^[1-9][0-9]+$)|(^0$) ]];then
    echo "$1 is geen getal" >&2
    exit 1
fi

user_id=$1
# .* wil zeggen 0 of meerdere willekeurige tekens
username=$(grep -E "^.*:x:$user_id:" /etc/passwd)
# dit is nu eens via IFS ook:
# IFS=: read username x uid rest < <(grep E "^.*:x:$1:" /etc/passwd)

# checks if username is empty
if [[ -z "$username" ]];then
    echo "User with uid $user_id not found" >&2
    exit 1
else
    echo "User with uid $user_id is $username"
fi

exit 0
```

---
**Lussen: while en until**

**Met een lus kan je een bepaald deel van het script herhalen zolang de exit status van een commando waar is. Je kunt hierbij het test-commando gebruiken, maar ook een read-commando, of om het even welk ander commando, of zelfs gegroepeerde commando's (tussen ronde haakjes of accolades, of een pipe).**  
vb
```sh
while commando
do
    # iets
done
```
lijen uitlezen (een cat command eigenlijk):
```sh
while read lijn
do 
    echo $lijn
done < /etc/passwd
```
via input redirection & proces restitutie:
```sh
while read lijn
do 
    echo $lijn
done < <(ps -ef)
```
teller
```sh
teller=0
while ((teller<10));do
    echo $teller
    ((teller++))
done
```
oneindige lus
```sh
while :;do
    :
done
```

**Als de exit status gelijk is aan 0 (waar), worden de opdrachten tot de afsluitende done uitgevoerd, waarna de voorwaarde-opdracht weer aan de beurt is. Wanneer de lus niet doorlopen wordt, is de exit status van de lus 0. Wanneer de lus wel doorlopen wordt, is de exit-status van de lus gelijk aan de exit status van de laatste opdracht die binnen de while-lus uitgevoerd werd. De until-lus is functioneel gelijk aan de while-lus, maar de exit status van de voorwaarde wordt geïnverteerd**  
TL;DR: bij while loop wanneer deze wordt overgeslagen is exit status 0, indien deze doorlopen wordt is exit status = exit status laatste opdracht  

**92. Ontwikkel een script dat het commando tail n simuleert. Als eerste argument moet een bestandsnaam opgegeven worden en als tweede argument mag het aantal regels opgegeven worden. Ontbreekt het tweede argument, dan worden de 10 laatste regels weergegeven. Realiseer dit op twee manieren:**

**• Gebruik een while-lus met een read-commando om het bestand te overlopen en een array om de gegevens cyclisch op te slaan.**
```sh
#cyclische tabel
array=()
teller=0

while read lijn;do
    array[$((teller%${2-10}))]=$lijn
    ((teller++))
done < "$1"

if ((teller<${2-10}));then
    teller2=0
    while ((teller2<teller));do
        echo ${array[$teller2]}
        (teller2++)
    done
else
    i=0
    while ((i<${2-10}));do
        echo ${array[$(((teeller+i)%{2-10}))]}
        ((i++))
    done
fi
```

**• Gebruik geen array, maar bepaal vooraf het aantal lijnen van het bestand.**
```sh
#!/bin/bash

# check aantal argumenten
if (($#!=1 && $#!=2));then
    echo "Fout aantal parameters!" >&2
    exit 1
fi

if [[ ! -f "$1" ]];then
    echo "$1 is geen geldig bestand!" >&2
    exit 1
fi

# enkel en alleen getal, dus niet de normale uitvoer ( wc -l "$1" ) waarbij men ook nog eens het bestand krijgt
tot_lijnen=$(wc -l < "$1")
# kan ook via read (waarbij men wel de bestandsnaam zou opvangen in een rest variabele)
# read tot_lijnen rest < <(wc -l "$1")

teller=0

while read lijn;do
    # ${2-10} wel zeggen als er geen getal is meegegeven, vervang door 10
    # bij := zou hij nog eens kijken als de variabele wel bestaat (ipv enkel te checken als deze leeg is)
    # dus :- zou eig ook mogen...
    if ((teller>=$((tot_lijnen-${2-10}))));then
        echo $lijn
    fi
    ((teller++))
# zolang men getallen in kan lezen van $1 
# opgelet, zeker "" errond bij input redirection om spaties te includeren
done < "$1"
```

**93. Hoe kun je met behulp van de while- of until-lus een aantal commando's oneindig lang laten uitvoeren? Onderbreek de uitvoering met Ctrl+C.**
```sh
while :;do
    <hier commando>
    :
done
```

**94. Het bestand ping.out (http://amerigo.ugent.be/besturingssystemen/2022-2023/ping.out) bevat de output van een Windows batch file:...**  
kan via wget gedownload worden  

eerst zoeken nr lijnen die beginnen met "Pinging"  
& lijnen die beginnen met "Reply"  
dmv de vlag zoeken we eerst nr pinging en dan nr Reply & opnieuw & opnieuw, ...  

! dit is nog niet volledig correct door "Unkown host" ipv Reply output soms  
```sh
#!/bin/bash

vlag=0
while read lijn;do
    if [[ $lijn =~ ^Pinging ]];then
        if ((vlag==1));then
            echo $hostname
        fi
        read pinging hostname brol <<< "$lijn"
        vlag=1
    fi
    if [[ $lijn =~ ^Reply ]];then
        vlag=0
    fi
done < ping.out
```
Men kan dit dan bv met grep -A4 \<een hostname> ping.out  
nagaan als de output wel degelijk klopt  
`-A4` is een optie die ook de 4 lijnen na de gevonden grep zal tonen, `-B4` zou dan de 4 vorige ook geven  