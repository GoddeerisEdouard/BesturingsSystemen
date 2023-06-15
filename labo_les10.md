# Deel VI: I/O-Systeemaanroepen (p.37/51)

**1. Schrijf een C-programma dat een bestand van ongeveer 10MB aanmaakt met willekeurige lettertekens gelegen in het gesloten interval [a..z].**

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/types.h>
#include <stdlib.h>


int main(int argc, char **argv){
    int fd=open("file", O_WRONLY | O_CREAT);
    if (fd<0){
        perror(argv[0]);
        exit 1;
    }
    srand(getpid()); // getpid() als randomizer
    for (int i=0;i<10*1024*1024;i++){
        char ch=rand()%26+'a';
        int n = write(fd, &ch, 1);
        if (n!=1){
            perror(argv[0]);
            exit(1);
        }
    }
    close(fd);

    return 0;
}
```
```sh
gcc 1.c -o 1
./1
```
vervolgens kan men kijken als het bestand is aangemaakt via  
`ll -h`  
`-h` staat voor "human readable", zodat men 10M als size ziet  


**2. Tot nog toe werd er niets gezegd over de optimale buffergrootte. De bedoeling is een C-programma te ontwikkelen dat het hierboven aangemaakte bestand verschillende keren inleest en dit met buffergroottes van 1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048, 4096 en tot slot 8192 bytes. De uitvoer van het programma moet er ongeveer zo uitzien:...**  
overgeslagen  

**3. Doe nu hetzelfde maar voor de write-systeemaanroep. Maak voor iedere verschillende buffergrootte een bestand aan van ongeveer 10MB (cfr. vraag 1). Nadat je de tijd voor een gegeven buffergrootte hebt opgemeten moet je vanzelfsprekend het aangemaakte bestand terug verwijderen. Een bestand verwijderen kan je doen m.b.v. de unlink-systeemaanroep.**  
Deze oefening is eig een soort uitbreiding op oef 1.
```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/types.h>
#include <stdlib.h>
// om zaken te meten
#include <pthread.h>

void vul (unsigned char * buffer, int n){
    for (int i=0;i<n;i++){
        buffer[i]=rand()%26+'a';
    }
}


int main(int argc, char **argv){
    srand(getpid()); // getpid() als randomizer
    unsigned char buffer[BUFSIZ]; // buffer is meteen groot gng door deze statisch eenmalig te alloceren
    vul(buffer, BUFSIZ);

    // <<= is shift left is dus *=2
    for (int i=1;i<=BUFSIZ;i<<=1){
        double start=clock();
        int fd=open("file", O_WRONLY | O_CREAT);
        if (fd<0){
            perror(argv[0]);
            exit(1);
        }
        // bijhouden hvl bytes reeds weggeschreven zijn
        int total=0;
        // loopen tot wanneer men er net niet over geraakt
        for (int j=0;j<(10*1024*1024)-1;j+=i){
            total+=write(fd, buffer, i);
        }
        // dit zorgt ervoor dat de file exact 10MB groot zal zijn
        total += write(fd, buffer, (10*1024*1024)-total);
        close(fd);
        // verwijdert het bestand
        unlink("file");
        printf("BUFSIZ=%10d total=%10d time=%.2f\n", i, total, (clock()-start)/CLOCKS_PER_SEC)
    }

    return 0;
}
```
`unlink` de hardlink verwijderen, als dit toevallig de laatste is, zal het bestand dus ook worden vrijgegeven/verwijderd  
info hierover in `man 2 unlink`  

note: uiteraard eerst de file die reeds was aangemaakt bij oef 1 verwijderen
```sh
gcc 3.c -o 3
./3
```

**4. Herneem vraag 3 maar voeg aan de flags-parameter de O_SYNC vlag toe. Dit laatste zorgt ervoor dat er noch in USER-mode noch in KERNEL-mode zal worden gebufferd en dat de bytes rechtstreeks naar de schijf zullen worden geschreven.**
**Ter info, de vaste schijf beschikt om performantieredenen over een bepaalde hoeveelheid write-back-cache-geheugen waar gegevens van I/O- schrijfopdrachten tijdelijk worden bewaard. Dit betekent dat er bij een eventuele spanningsonderbreking wel degelijk gegevens kunnen verloren gaan. Bij programma’s, zoals gegevensbanken, waar gegevensverlies nefast is, wordt er aangeraden om “write-caching” uit te schakelen. Dit kan je doen door in een shell de opdracht `hdparm -W0 /dev/sda` uit te voeren.**  
dit is gwn kleine aanpassingen doen aan vorige oefening ( `| O_SYNC`)   
men moet wel nog `dnf install hdparm` doen om die opdracht te kunnen uitvoeren  
verder wordt er niet echt iets gevraagd  

**5. Bestudeer gronding de werking van de shell-opdracht cat en schrijf in C een eigen versie van cat. Bekijk bv. wat er gebeurt wanneer je de opdracht `cat /etc/passwd - /etc` opgeeft of wanneer cat geen argumenten meekrijgt. Wanneer je een directory opgeeft als argument, geeft cat een foutmelding. Dit gedrag hoef je niet na te bootsen.**  
`cat` zonder argumenten laat ons toe gewoon te typen en wnr men op enter drukt kan men op de volgende lijn typen  

`-` is synoniem voor lezen standaardinvoer & schrijven standaard uitvoer
```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/types.h>
#include <stdlib.h>
// om zaken te meten
#include <pthread.h>
// voor string vergelijken
#include <string.h>

int main(int argc, char **argv){
    unsigned char buffer[BUFSIZ];
     // bij geen gegeven argumenten
    if (argc == 1){
        // korte herhaling eerste argument bij deze functies
        // 0: stdin, 1:stdout, 2:stderr
        int n=read(0,buffer,BUFSIZ);
        while (n!=0){
            write(1,buffer,n);
            n=read(0,buffer,BUFSIZ);
        }
        if (n<0){
            perror(argv[0]);
            exit(1);
        }

    } else {
        // alle argumenten overlopen
        for (int i=1;i<argc;i++){
            // als het een "-" is
            if (strcomp(argv[i], "-")==0){
                int n=read(0,buffer,BUFSIZ);
                while (n!=0){
                    write(1,buffer,n);
                    n=read(0,buffer,BUFSIZ);
                }
                if (n<0){
                    perror(argv[0]);
                    // read geeft regelmatig fouten, vandaar continue deze keer
                    continue;
                }
            // als het een bestand zou zijn
            }else  {
                int fd=open(argv[i], O_RDONLY);
                //controleren als bestand wel bestaat
                if (fd<0){
                    perror(argv[i]);
                    continue;
                }
                // lezen BUFSIZ aantal karakters
                int n=read(fd,buffer,BUFSIZ);
                // soms kan de buffer nog steeds vol zitten en kunnen we het bestand niet in 1x inlezen
                // vandaar deze while loop
                while (n>0){
                    write(1,buffer,n);
                    n=read(fd,buffer,BUFSIZ);
                }
                if (n<0){
                    perror(argv[i]);
                    continue;
                }
                close(fd);
            }
        }
    }



    return 0;
}

```
```sh
gcc 5.c
./a.out /etc/pa # voorbeeld van file die niet bestaat

./a.out /etc/passwd # voorbeeld met 1 file

./a.out /etc/passwd - - /etc/group # voorbeeld met vrije input na passwd outpuut
# NOTE: via ctrl+D kunnen we een EOF signaal geven
```

**6. Schrijf een eigen versie van de opdracht cp waar twee argumenten op de opdrachtlijn worden verwacht. Het eerste argument is het bronbestand en het tweede het doelbestand. Wanneer de eerste parameter een directory is, wordt een foutboodschap naar het scherm geschreven en stopt het programma met exit-status 1. Wanneer het tweede argument een directory is, wordt opnieuw gestopt met een passende foutboodschap en met exit-status 1.**  
**Om na te gaan of een argument een directory is, kan je gebruikmaken van de systeemaanroep stat. Om een programma te stoppen met exit-status 1 maak je best gebruik van de bibliotheekfunctie exit die dan achter de schermen de systeemaanroep _exit aanroept.**
```c
#include <unistd.h>
#include <fcntl.h>
#include <stdlib.h>

int main(int argc, char **argv){
    if (argc!=3){

        fprintf(stderr,"Usage cp <src> <dest>\n");
        exit(1);
    }

    //...

    return 0;
}
```

**7. Schrijf een C-programma met als naam watchfile.c dat één argument, een bestand, op de opdrachtlijn verwacht. Het programma loopt in een oneindige lus en schrijft telkens een boodschap naar het scherm wanneer het bestand dat op de opdrachtlijn werd meegegeven werd gewijzigd. Wanneer er geen argument werd opgegeven of het argument is geen gewoon bestand wordt een foutboodschap getoond en wordt het programma afgesloten met exit-status 1.**  
dmv de systemcall `stat` kan men nagaan als een bestand gewijzigd zou zijn.  
We gaan dit uiteraard doen door te kijken nr de tijd en niet de grootte.  
opnieuw info opvragen kan via `man 2 stat` om te zien hoe die functies in c precies werken  
gevonden syntax: 
```c
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>
int stat(const char *pathname, struct stat *statbuf)
// mocht men reeds over een file descriptor beschikken
int fstat(int fd, struct stat *statbuf)
// als men verder nr onder scrollt kan men zien wat de stat struct allemaal inhoudt
// daarin staat dan welke info van die struct precies de "time of last modification" is
//zijnde struct timespec st_mtim
```

oplossing:
```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/types.h>
#include <stdlib.h>
// om zaken te meten
#include <pthread.h>
// stat import
#include <sys/stat.h>

int main(int argc, char ** argv){
    // controle vn argumenten
    if (argc!=2){
        fprintf(stderr,"Usage watchfile <file>\n")
        exit(1)
    }

    struct stat s;
    // bij stat moet de bestandsnaam en de struct s ingevuld worden, vandaar het adres meegeven
    if (stat(argv[1], &s)<0){
        perror(argv[1]);
        exit(1);
    }
    // onderaan de `man 2 stat` stond dat we doorverwezen werden naar `man 7 inode`
    // in die man page staat hoe men kan checken als het wel een regulier bestand is ( S_ISREG(m) ), we willen namelijk geen directory wijzigingen nagaan
    if (!S_ISREG(s.st_mod)){
        fprintf(sderr, "Usage watchfile <file>\n")
        exit(1);
    }
    // we halen de starttijd van de struct
    // opnieuw gevonden in man 2 stat
    int start=s.st_mtim.tv_sec;
    while(1){
        // stat info opnieuw opvragen
        stat(argv[1], &s);
        // als men verschil vindt
        if(start!=s.st_mtim.tv_sec){
            printf("File has been changed\n")
            start=s.st_mtim.tv_sec;
        }
        // even process laten rusten
        sleep(10);
    }

    return 0;
}
```

```sh
gcc 7.c -o 7
./7 # zou error moeten tonen met Usage bericht
./7 <bestandsnaam>
# touch bestandsnaam zou dan een bericht moeten triggeren tijdens uitvoer v bovenstaande commando 
```

