# Deel V: Programmeren in Bash - vervolg

## opdrachten (p.34/51)

**102. ...**  
overgeslagen  

**103. Enerzijds kun je met behulp van het commando ps -e informatie opvragen over alle processen die actief zijn. De vier kolommen in de output tonen respectievelijk het proces-ID (PID), de TTY device file van de (pseudo-)terminal, de CPU time, en het commando dat het proces opgestart heeft. Anderzijds kun je met behulp van het commando kill -KILL pid een proces met willekeurig proces-ID afbreken. Ontwikkel een script dat alle processen afbreekt waarvan het commando één van de strings bevat die als parameters bij het oproepen van het script meegegeven wordt. Indien geen enkele parameter meegegeven wordt, moet het script een gesorteerde lijst weergeven van alle unieke commandonamen van actieve processen. Behalve de interne instructies (if, for, case, let, while, read enz.) mag je ook het externe commando sort gebruiken. Om problemen te vermijden, schrijf je bij het testen de kill-opdracht uit naar standaarduitvoer i.p.v. deze daadwerkelijk uit te voeren.**  
Deze vraag^ is ooit eens gesteld geweest op een examen  

ter info  
zo zou men alle processen opgelijst krijgen
```sh
# de -s (squeeze) optie bij tr vervangt meerdere opeenvolgende spaties door 1 spatie
`ps -e | tr -s " " | cut -d " " -f 5 | sort`
```
dan kan men nog `uniq -u` of `-d` toevoegen voor de unieke of duplicate processen  
PS we wisten dat er meerdere spaties op elkaar volgden door eens de spaties te vervangen door . en dan de output te bekijken `ps -e | tr " " "."`  


ps -e heeft als output (met wat spaties tss iedere kolom)  
`PID TTY TIME CMD`

```sh
#!/bin/bash

# controleer aantal args
if (($#>0));then
    # read squeezed automatisch
    while read pid term time cmd;do
        #eerste lijn negeren wegens dat dit de legende is
        ((eerste==0)) && eerste=1 || {
            for i in "$@";do
                # strings vergelijken dus "" en spaties
                [[ "$i" == "$cmd" ]] && echo "kill $pid"
            done
        }
        echo $cmd
    done < <(ps -e)
else
    # unieke commandolijst dmv string tabel
    # de waarde per key zal er niet toe doen, we zullen enkel de keys overlopen
    declare -A array
    eerste=0
    while read pid term time cmd;do
        ((eerste==0)) && eerste=1 || array["$cmd"]=1
    done < <(ps -e)

    for i in "${!array[@]}";do
        echo $i
    done | sort

fi
```

**104. Ontwikkel een script dat (zonder getopt te gebruiken) alle opties die aan het script worden meegegeven naar standaarduitvoer wegschrijft, één per regel. Je moet dus alle karakters die voorkomen in parameters die beginnen met een minteken verzamelen, en deze één voor één verwerken. Bekommer je niet om opties die meerdere keren zouden voorkomen. Voor de argumentenlijst -Ec -rq /etc/passwd -a moet het script dus als uitvoer E, c, r, q en a produceren.**  
durft hij regelmatig te vragen op examen (is in verleden al gebeurd)  
voorbeeld output bij  
`./104.sh -f file -Enq -Ep`
```sh
f
E
n
q
E
p
```
waarbij mocht bijvoorbeeld geen file zijn meegegeven, men een foutboodschap uitschrijft (dus het is niet zo simpel als gwn alle opties te overlopen)

```sh
#!/bin/bash

function printerr(){
    echo "Usage 104.sh [-f file] [-...]" >&2

}

# zolang $@ een non-zero value heeft
while [[ -n "@" ]];do
    case "$1" in
            -f)     [[ -n "$2" && -f "$2" ]] || { 
                printerr
                exit 1
              }
                    echo "f"
                    shift
                    ;;
            # -n optie leest per n karakters
            -?*)    while read -n1 letter;do
                    # als letter non-zero is en niet "-"
                        [[ -n "$letter" && $letter != "-" ]]  && echo $letter
                    done

                    ;;
            *)      printerr
                    exit 1
                    ;;
    # einde case
    esac


    shift
done
```

**105. 106.**  
overgeslagen  

**107.**  
"eens interessant om te maken"  

**108. 109. 110.**  
overgeslagen  

**111.**  
te ingewikkeld, zal niet meer gevraagd worden  

**112. Het bestand pagefile.out bevat de uitvoer van een Windows batch file: (te downloaden op amerigo)**
```sh
dir \\AL005951\c$\pagefile.sys
dir \\AL005952\c$\pagefile.sys
...
```
**Elk van de 2.901 dir-opdrachten werd beantwoord met regels van de vorm:**
```sh
C:\WINDOWS\system32>dir \\AL005951\c$\pagefile.sys
Volume in drive \\AL005951\c$ is Windows
Volume Serial Number is D0A0-4386
Directory of \\AL005951\c$ # in deze lijn zijn we geïnteresseerd
02/08/20 02:12 146.800.640 pagefile.sys
1 File(s) 146.800.640 bytes
577.850.880 bytes free # in deze lijn zijn we geïnteresseerd
```
**De regel met de woorden bytes free vermeldt de beschikbare ruimte op het volume C:.**
**Maak een script dat een tekstbestand genereert met de namen van alle toestellen die minder dan 80 MB vrij hebben op de C: schijf, één per regel.**  

lijnen waarin we geïnteresseerd zijn:  
- beginnend met "Directory of"
(in ons geval weten we gwn dat v1 het woord Directory zal meoten bevatten...)

- eindigend met free  
hier kunnnen we de . vervangen door een lege string om 1 getal uit te komen (kunnen we doen dmv substrings, waarvan de pdf wordt gegeven op het examen)  
`${variabele//pattern/string}` -> vervangt ALLE occurrences  
dus  
`${variabele//./}`

```sh
#!/bin/bash

# -r optie staat voor het vermijden dat \ als escape character wordt gezien
# we zitten namelijk met windows output, waar ^ het escape character is
while read -r v1 v2 v3;do
    [[ "$v1" == "Directory" ]] && {
        # # is vooraan % is achteraan ( en 2 achter elkaar is greedy)
        # vooraan knipt men 2 backslashes weg (vervangen met niks)
        hulp={$v3##\\\\}
        # achteraan wegknippen (vervangen) tot \c$
        comp=${hulp%\\c$}
        echo $comp
    }
    [[ "$v3" == "free" ]] && {
        # vervangen van . met niets
        size=${v1//./}
        ((size<80*1024*1024)) && echo $comp
    }


done < pagefile.out
```