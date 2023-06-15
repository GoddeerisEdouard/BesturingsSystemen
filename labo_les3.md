# Deel V: Programmeren in Bash - vervolg
## opdrachten (p.14/51)

**Maak voor onderstaande opdrachten de volgende lege bestanden aan: a, b, c, d, e, ab.c, a.b, b.a, b.c, c.d en d.e.**  
`touch a b c d e ab.c a.b b.a b.c c.d d.e`

**12. Voer de volgende opdrachten uit:...**  
essentie:  
`*` staat voor geen of meerdere willekeurige karakters  
dus `dir a*.*` zoekt voor om het even welke dir met een a, willekeurige karakters, en punt en mogelijks nog karakters erna  
`\` het escape teken: zorgt voor daadwerkelijke naam  
dus `dir a\*` zoekt voor dir met name `a*`  

via `man bash` en zoekt op Pathname expansion kan men de betekenissen van alle "special characters" opzoeken  

**13. Voorspel en controleer de uitvoer van de opdrachten:**  
`printf "%s\n" [abcd]`: geef me een willekeurig bestand van 1 karakter (%s) dat 1 vd karakters in []  

`printf "%s\n" [!abcd]`: zelfde maar dan alles __behalve__ a b c of d  

`printf "%s\n" [^abcd]` zelfde als `!`  

`printf "%s\n" [a-d]` alle karakters van a-d in [], dus koppelteken kan gebruik worden bij een logische opeenvolging  

`printf "%s\n" [abcd]*[abcd]` bestand startend en eindigend met een a, b, c of d, de karakters ertss doen er niet toe  

**14. Voer de onderstaande commando’s uit.**  
zelfde als erboven met wat andere combinaties tss []  
`[a-e]`: van a tot e  
`[a/-e]`: exacte match, dus `[a/-e]`  
`[a\-e]`: escape dus a en e  
`[!\!]*` alles behalve `!` gevolgd door om het even welke karakters   

**Wat is de bedoeling van het \-teken in deze opdrachten? Wat gebeurt er indien je het verkeerde /-teken gebruikt?**  
`\`: escape karakter om dus exacte matches te zoeken en "speciale karakters" niet in te lezen als speciaal  
`/`: no idea yet (misschien ook exacte match?)  

**15. Hoe kan je een lijst met bestandsnamen bekomen die precies uit één enkel karakter bestaan?**  
`printf "%s\n" ?` of `ls ?` of `echo ?`  
=> `?`: om het even welk ene karakter (verplicht 1)  

**16. Vraag een lijst met bestandsnamen die uit precies twee karakters bestaan. Vergelijk de uitvoer met die van de vorige opgave.**  
`find . -name '??'` 
(not sure)

**17. Voer volgende opdrachten uit:...**  
essentie: `*` geeft alle gevonden files terug  
dus bv `ls *`, `dir *`  

dit tss `''` zetten neemt dit dan letterlijk  
bv `ls '*'`, gaat op zoek nr files met name *  
bij echo en printf wordt dan * geprint ipv een error bij ls als hij deze niet vindt  

**18. Zorg dat er geen bestanden in de werkdirectory staan waarvan de naam met abc begint. Verklaar dan het verschil in uitvoer tussen volgende opdrachten:**  
essentie: printf zal bij geen matches toch nog eens hetgene waarnaar men zoekt afprinten  


**19. Voer de opdracht rm –f ?? uit. Verklaar daarna het verschil in uitvoer tussen volgende opdrachten:**  
verwijdert alle bestanden met exact 2 karakters als bestandsnaam verschillen:  
`???`: exact 3 karakters  
`??e` 2 karakters gevolgd door e  
`??` 2 karakters (omdat deze bestanden reeds gewist zijn zal `printf "%s\n" ??` gewoon ?? printen)  

**20. 21. 22.**  
essentie van deze oefeningen:  
brace expansion  
`{1..5}` 1 2 3 4 5  
stappen maken  
`{0..6..2}` 0 2 4 6  
prefix en postfix  
`a{0..3}c` a0c a1c a2c a3c  
combinaties  
`{0..3}{0..2}` 00 01 02 10 11 12 30 31 32  
andere combinatie (nested)  
`{{0..3},{a..c}}` 0 1 2 3 a b c  

eval zorgt dat een expansie een 2e keer wordt overlopen  
kan bij de ksh (kornshell) handig zijn bij tilde expressies  
`echo ~{mail, root}` zal bv werken bij een gewone shell  
bij ksh moet men echter  
`eval echo ~{mail, root}` doen om eerst de lijst van mail en root te maken en vervolgens de tildes om te zetten  
met eval kunnen we ook variabelen gebruiken in de braces  
```sh
a=2
eval echo {0..8..$a}
unset a # to remove the variable
```

**Redirection, piping en filtering, process substitution en het commando find**  
verschillen tss   
0 standaardinvoer  
1 standaarduitvoer  
2 standaardfout  

essentie: bij het printen van output wordt niet altijd meteen "geflushed" men kan dit forceren met een newline character `\n`  

**23. Verwijder de werkbestanden tmp\*.txt, indien die reeds zouden bestaan. Voer hiertoe het commando rm -f tmp\*.txt uit. Voer daarna achtereenvolgens deze opdrachten uit:...**  
essentie:  
`>` output uitschreven nr bestand  
bv `echo "hello" > file.txt`  

`>>` bestand appenden  
bv `echo "meow" >> animalsounds.txt`  

`1>` standaard output wegschrijven (wat default is)  
bv `echo "meow" 1> animalstuff.txt`  

volgorde van redirection doet er niet toe  
zo is `du /etc > tmp.txt` hetzelfde als `> tmp.txt du /etc`  

verder doet  
`du /var > tmp.txt > tmp2.txt`  
de output van `du /var` wegschrijven nr tmp2.txt, tmp.txt is leeg  

**24. Verwijder eerst het werkbestand tmp.txt. Voer daarna volgende opdrachten uit:...**  
`set -o noclobber`  
wanneer dit aan staat zal bij gebruik van `>` een foutmelding krijgen  
dan is het niet meer mogelijk om bestanden te gaan overschrijven  
dit kan nog steeds via het `>|` teken  

dus best  gewoon noclobber uitzetten:  
`set +o noclobber`  

**25. ...**  
essentie:  
output uitschrijven nr ene bestand en foutmeldingen nr ander bestand via  
`du /etc > tmp1.txt 2>tmp2.txt`  
**26. Je kunt ook omleiden naar de vuilnisbak (/dev/null). Verklaar de uitvoer bij uitvoering van volgende opdrachten:**  
essentie: wegschrijven nr niks (mocht het echt moeten) `/dev/null`  

**27. Ga na wat het effect is van cat /dev/null > test.txt**  
maakt bestand leeg  
op die manier kan je een bestand leegmaken dat al bestaat  

**28. Veronderstel dat een of ander programma continu informatie wegschrijft in een logbestand (dat hierdoor onbeperkt en continu in grootte toeneemt), en dat niemand geïnteresseerd is in dit logbestand. Hoe kun je vermijden dat je periodiek het bestand moet legen of verwijderen?**  
men kan een softlink van het bestand waar normaal gzn naar geschreven wordt linken met `/dev/null`  
dus bv `yes y > t` zou snel een bestand t volschrijven ( men kan de schijfruimtes nagaan via `df` )  
maar wanneer men van t een softlink naar `/dev/null` maakt wordt dit dus eigenlijk meteen ook verwijderd  
`ln -s /dev/null t`

**29. Bekijk verdere mogelijkheden met de opdrachten:**  
introductie om alles te begrijpen:  
`2>&1`: "schrijf 2 naar kanaal 1" -> zonder die `&` zou men de fouten `2` wegschrijven naar een bestand met naam 1  
`&` wordt dus gevolgd door een kanaal  
dus commando `du /etc 2>&1 > /dev/null` zal kanaal 1 (stdout) uitschrijven nr `/dev/null` & stderr uitschrijven nr kanaal 1  (printen dus). `> /dev/null` = `1>/dev/null`  
opgelet, de volgorde van de pipes doet er toe  
zo zal `du /etc > /dev/null > 2>&1` ervoor zorgen dat zowel stderr als stdout in de prullenbak belanden. Dit is enkel nuttig om de exitstatus te achterhalen, waarbij de uitvoer er niet toe doet (if testen)  

aantal fouten tellen:  
`du /etc 2>&1 1>/dev/null | wc -l`  
dit kan dankzij de fouten een gebufferde uitvoer te maken zodat je ze door de pipeline kan trekken. Dit komt meer voor omdat men wil verder werken op de fouten  

! enkel gebufferde uitvoer (schermuitvoer) gaat door een pipe, dus je kan de stderr bufferen door `2>&1`  

`du /etc >tmp.txt 2>&1`:  
`du /etc 2>&1 >tmp.txt`:  
`u /etc &>tmp.txt`:  
`du /etc 2>tmp.txt >tmp.txt`:  
`exec 3>tmp.txt ; du /etc >&3 2>&1 ; exec 3>&-`:  

**30. Verwijder eerst het werkbestand tmp.txt, indien dit reeds zou bestaan. Vergelijk daarna telkens de uitvoer en de inhoud van tmp.txt bij uitvoering van volgende commandoregels. Vergeet de spaties niet!**  
`du /etc ; du /var`: voert op eenzelfde commandolijn verschillende commando's na elkaar uit  
`du /etc ; du /var > tmp.txt`: zelfde maar bij 2e commando redirection nr tmp.txt  
`( du /etc ; du /var ; ) > tmp.txt`: groeperen, opgelet ! spatiegevoelig, maakt gebruik van een subshell (zie onderstaand)  
`{ du /etc ; du /var ; } > tmp.txt`: zelfde, maar de accolades geven aan dat de commando's worden uitgevoerd in een shell die dezelfde shell als als u het gewoon zou intypen (het BASHPID is anders)  
zo zal `(echo $BASHPID)` een andere uitvoer hebben dan `{ echo $BASHPID ; }`, bij de ronde haakjes gebruik je een kopie van de huidige shell  
=> ronde haakjes `()` gebruiken de fork() system call  