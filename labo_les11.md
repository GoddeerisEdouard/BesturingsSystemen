# Deel VII: Processen en POSIX-threads
(p40/51)  

intro tekst eens lezen  

**1. Gegeven onderstaande code:**
```c
int main(int argc, char **argv){
...
fork();
fork();
fork();
...
}
```
**Voorspel hoeveel kindprocessen er zullen worden aangemaakt zonder de code uit te voeren.**  
2^3 = 8 kindprocessen  

"bij `fork()` keren na einde proces parent & child terug naar waar cursor staat te pinken (na de fork dus)"  

**2. Schrijf een programma dat drie kindprocessen aanmaakt en zorg ervoor dat ieder kindproces zijn proces-ID naar het scherm schrijft en daarna stopt. Het proces-ID kan je m.b.v. de systeemaanroep getpid() opvragen en een proces kan je beëindigen met de functie `exit(int exitstatus)`. De exitstatus van een correct beëindigd proces is steeds 0 terwijl een waarde verschillend van 0 duidt op een fout.**
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
// voor waitpid
#include <sys/wait.h>
#include <errno.h>

int main(int argc, char ** argv){
    int i;
    // lijst met proces IDs bijhouden
    int pids[3];

    for (i=0;i<3;i++){
        pid[i] = fork();
        // error opvangen van onverwachte pid
        if (pids[i] < 0){
            perror(argv[0]);
            exit(1);
        // kindproces returnt altijd 0
        // parent proces returnt het pid (via getpid())
        } else if (pids[i] == 0){
            // newline niet vergeten voor lijnbuffering
            printf("Dit is kindproces %d met PID %d\n", i+1, getpid());
            // men is er hier dan zeker van dat het kindproces hier stopt
            exit(0);
        }
    }

    // wachten op elk kindproces om te eindigen
    for (int i=0;i<3;i++){
        // syntax: pid_t waitpid(pid_t pid, int *status_ptr, int options)
        waitpid(pids[i], NULL, 0);
    }

    return 0;
}
```
```sh
gcc 2.c -o 2
./2
```

**3. Schrijf een C-programma writestring.c dat het proces-ID naar het scherm schrijft gevolgd door de string die als enige parameter wordt meegegeven en vervolgens 10 seconden wacht vooraleer te eindigen.**
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char ** argv){
    if (argc!=2){
        perrror(argv[1]);
        exit(1);
    }

    printf("PID: %d\n string: %s\n", getpid(), argv[1]);
    sleep(10);

    return 0;
}
```
```sh
gcc writestring.c -o writestring
./writestring
```

**1. Herschrijf nu opdracht 2 waarbij het kindproces de systeemaanroep execv gebruikt om “writestring hello” uit te voeren. Opgelet: In het ouderproces moet je wachten tot wanneer het kindproces klaar is, waarna je nog een boodschap naar het scherm schrijft. Dit doe je door gebruik te maken van de systeemaanroep waitid of waitpid.**  
eerst kleine twist  
een oude examenvraag was om het commando `xxd -g1` na te bootsen, met meerdere argumenten
```c
#include <stio.h>
#include <sys/wait.h>
#include <unistd.h>

int main(int argc, char **argv){
    // ieder argument overlopen
    for (int i=1;i<argc;i++){
        // zeker char * args[] gebruiken!
        char *args[]={"xxd","-g1", argv[i], char(*)0};
        int pid=fork();
        if (pid==0){
            // per meegegeven argument xxd uitvoeren
            if (execvp("xxd", args)<0){
            // men zou ook execlp kunnen gebruiken
            // if(execpl("xxd", "xxd", "-g","1", argv[i], (char*)0)<0)
                perror(argv[0]);
                exit(1); // continue; // in geval van gebruik execlp
            }
            return 0;
        }
        // waitpid in de lus zodat er nooit parallel wordt uitgevoerd
        waitpid(pid, NULL, 0);
    }


    return 0;
}
```
voorbeeld gebruik
```sh
gcc xxd_oef.c
/a.out /etc/passwd /etc/group
```

oplossing
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
// voor waitpid
#include <sys/wait.h>
#include <errno.h>

int main(int argc, char ** argv){
    // casting toevoegen aan die 0 (zodat er een adres van gemaakt wordt)
    // zonder casting klopt dit enkel op een 64-bit systeem
    // maar men moet er eig zeker van zijn dat de pointer 8 byte groot is
    // daar zorgt de casting dus voor
    // syntax {<naam_programma, <argument>, pointer}
    char * args[]={"writestring","hallo",char(*)0};

    // execv betekent array meegeven
    // execl betekent oplijsting meegeven
    // hier kan nog op het einde bij beiden p aan toegevoegd worden obv padvariabele 
    // bij zelfgeschreven programma's hoeft dit niet
    // als men de p meegeeft zal hij dit zoeken op naam
    // bv execvp("xxd", ...)
    if (execv("writestring", args)<0){
        perror(argv[0]);
        exit(1);
    }
    return 0;
}
```
```sh
gcc 3.c
./a.out
```

**2. Idem als deel 1 maar maak nu gebruik van de systeemaanroep execl. De laatste C-string-parameter moet (char \*)0 zijn om het einde van de opsomming aan te geven. Bemerk dat gewoon 0 schrijven niet voldoende is en zelfs fout is. Wanneer de grootte van een int verschillend is van de grootte van een char \*, zal het aantal argumenten dat doorgegeven wordt aan execl verkeerd zijn.**  
overgeslagen  



**4. 5.**  
overgeslagen "gaan we niet doen"  

**6. Schrijf een programma dat een getal neemt op de commandolijn en evenveel processen genereert als dat getal aangeeft. Binnen ieder kindproces wordt een willekeurig getal genereert en opgestuurd naar de het ouderproces. Het ouderproces bepaalt het grootste van de gegeneerde getallen bepaalt en vervolgens brengt het ieder kindproces op de hoogte van wie de winnaar is, ttz. welk proces het grootste getal heeft gegenereerd. De uitvoer voor zes kindprocessen ziet er als volgt uit:**
```
Process 1819 is the winner
I'm the winner!
Process 1819 is the winner
Process 1819 is the winner
Process 1819 is the winner
Process 1819 is the winner
```

korte note hierover die pas volgende les wordt besproken:  
met `read` opdrachten blokkeren we deze processen & met `write` opdrachten deblokkeren we ze  

oplossing zonder winnaar uit te schrijven:
```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <unistd.h>


int main(int argc, char ** argv){
    // we slaan even de controle op argumenten over

    // atoi staat vr ascii to integer
    int n=atoi(argv[1]);
    int pids[n];
    // filedescriptors van child nr parent
    int fds_CP[n][2];
    int numbers[n];

    // processen aanmaken
    for int(i=0;i<n;i++){
        // syntax: pipe([read, write])
        // pipe voor uitvoer vn kind mee te geven aan parent & dit later uit te printen
        // pipe vóór fork doen!
        pipe(fds_CP[i]);
        pids[i]=fork();
        // als het een kindproces is
        if (pids[i]==0){
            // read sluiten van huidige fds
            close(fd_CP[i][0]);

            // pid als randomizer (zeker per kindproces)
            srand(getpid());

            int number=rand();
            // output wegschrijven
            write(fds_CP[i][1], &number, sizeof(int));
            
            // inlezen van anderen
            read([i])

            //vermijdt dat kindproces ook processen begint te maken
            return 0;
        }
        // write sluiten van huidige fds (dit gebeurt in de parent)
        close(fds_CP[i][1]);
    }
    // weggeschreven output van kindproces outlezen & wegschrijven in de numbers variabele
    for (int i=0;i<n;i++){
        read(fds_CP[i][0], &numbers[i], sizeof(int));
        // vergeet \n niet voor buffer!
        printf("Child %d generated %d!\n", pids[i], numbers[i]);
    }

    // wachten op afloop van ieder kindproces
    for (int i=0;i<n;i++){
        waitpid(pids[i], NULL, 0);
    }
    return 0;

}
```

```sh
gcc 6.c -o 6
./6 <een getal>
```

nu, ipv constant dat n argument herhaaldelijk te gaan gebruiken en al die aparte variabelen aan te maken,  
kunnen we ook een struct maken die alles samenvoegt (niet noodzakelijk, gwn wat georganiseerder):  
hier wordt ook de winnaar uitgeschreven
```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/wait.h>
#include <unistd.h>


// type naam data_t zodat we nog steeds de reserved variabele "data" kunnen gebruiken (is iets typisch voor c)
typedef struct {
    int pid;
    int fd_CP[2];
    int fd_PC[2];
    int number;
} data_t;


int main(int argc, char ** argv){
    // we slaan even de controle op argumenten over

    // atoi staat vr ascii to integer
    int n=atoi(argv[1]);
    //gebruik van struct
    data_t data[n];

    // processen aanmaken
    for int(i=0;i<n;i++){
        // syntax: pipe([read, write])
        // pipe voor uitvoer vn kind mee te geven aan parent & dit later uit te printen
        // pipe vóór fork doen!
        pipe(data[i].fd_CP);

        //tweede pipe die winnaar bekend zal maken
        pipe(data[i].fd_PC);

        data[i].pid=fork();
        // als het een kindproces is
        if (data[i].pid==0){
            // pipes sluiten
            // read sluiten van huidige fds
            close(data[i].fd_CP[0]);

            close(data[i].fd_PC[1]);
            // pid als randomizer (zeker per kindproces zodat we niet telkens hetzelfde nummer krijgen)
            srand(getpid());

            int number=rand();
            int winnaar;

            // output wegschrijven
            write(data[i].fd_CP[1], &number, sizeof(int));
            
            //winnaar inlezen & variabele daarmee opvullen
            read(data[i].fd_PC[0], &winnaar, sizeof(int));

            // controleren indien winnaar
            if (winnaar == getpid()){
                printf("I'm the winner!\n");
            } else {
                printf("Process %d is the winner\n", winnaar);
            }

            //vermijdt dat kindproces ook processen begint te maken
            return 0;
        }
        // lees fds sluiten
        close(data[i].fd_PC[0]);

        // write sluiten van huidige fds (dit gebeurt in de parent)
        close(data[i].fd_CP[1]);

    }
    // weggeschreven output van kindproces outlezen & wegschrijven in de numbers variabele
    for (int i=0;i<n;i++){
        read(data[i].fd_CP[0], &data[i].number, sizeof(int));
        // vergeet \n niet voor buffer!
        printf("Child %d generated %d!\n", data[i].pid, data[i].number);
    }
    // index zal index nr grootste getal bevatten in onze data tabel
    int index=0;
    // we slaan eerste over omdat index dit als startwaarde heeft (we beginnen met er van uit te gaan dat de eerste het grootste is)
    for(int i=1;i<n;i++){
        if (data[i].number>data[index].number){
            index=i;
        }
    }

    // kindprocessen deblokkeren & doorgeven welk kindproces gewonnen is
    for (int i=0;i<n;i++){
        write(data[i].fd_PC[1], &data[index].pid, sizeof(int));
    }


    // wachten op afloop van ieder kindproces
    for (int i=0;i<n;i++){
        waitpid(data[i].pid, NULL, 0);
    }
    return 0;

}
```