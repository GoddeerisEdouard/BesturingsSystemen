# Deel VII: Processen en POSIX-threads
## POSIX-threads
(p44/51)  

bij Threads kan men makkelijk delen, niet dmv gebruik van pipes, maar globale variabelen  

**Opdrachten**  

**1. Schrijf een programma dat gebruikmaakt van vier threads die elk een verschillend cijfer naar het scherm schrijven. Wanneer een thread het bijhorend cijfer 100 keer naar het scherm heeft geschreven, stopt de thread.**  
overgeslagen  

**2. Genereer 1.000.000 willekeurige reële getallen die je bijhoudt in een tabel. Schijf nu twee functies die zoeken naar respectievelijk het kleinste getal en het grootste getal en deze getallen als return-waarde teruggeven. Schrijf nu een hoofdprogramma dat gelijktijdig zoekt naar het grootste en het kleinste getal in een tabel van 1.000.000 reële getallen. Schrijf beide getallen naar het scherm.**
```c
#include <pthread.h>
#include <dtdlib.h>
#include <stdio.h>

// gebruik van struct om alle data die men wil delen bij te houden
typedef struct {
    // pointer nr tabel
    int *tab;
    // aantal alementen die men wil bijhouden in de tabel
    int n;
    // functie die moet uitgevoerd worden 
    int (*f)(int*, int);
} data_t;

int geef_grootste(int *tab, int n){
    int grootste = tab[0];
    for (int i=1;i<n;i++){
        if (tab[i]>grootste){
            grootste = tab[i];
        }
    }
    return grootste;
}

int geef_kleinste(int *tab, int n){
    int kleinste = tab[0];
    for (int i=1;i<n;i++){
        if (tab[i]<kleinste){
            kleinste = tab[i];
        }
    }
    return kleinste;
}

// algemene worker functie, vandaar void *
void * worker(void *args){
    // casten
    data_t *data=(data_t*)args;
    // opnieuw casten om waarschuwing te mijden
    return (void*)data->f(data->tab, data->n);
}

int main(){
    // tabel met threads aanmaken
    int tab[1000];

    for (int i=0;i<1000000;i++){
        tab[i]=i;
    }

    pthread_t kl, gr;
    data_t data_kl, data_gr;
    data_kl.tab=tab;
    data_kl.n=1000000;
    data_kl.f=geef_kleinste;

    data_gr.tab=tab;
    data_gr.n=1000000;
    data_gr.f=geef_grootste;

    // functie uitgelegd in man pthread_create
    // 2e param is altijd NULL & data_kl casten nr void* om waarschuwing te vermijden
    pthread_create(&kl, NULL, worker, void(*)&data_kl);

    pthread_create(&gr, NULL, worker, void(*)&data_gr);
    
    int grootste, kleinste;
    // wachten op thread om te beeïndigen
    // void ** wegens dat het een pointer is nr iets van het type void* (zijnde de returnwaarde van de worker functie)
    pthread_join(kl, (void**)&kleinste);
    pthread_join(gr, (void**)&grootste);

    printf("%d is het grootste en %d is het kleinste\n", grootste, kleinste);
    return 0;
}
```
```sh
gcc 2.c -pthread
./a.out
```

**3. Multithreading kan ook leiden tot snelheidswinst. Een mooi voorbeeld hiervan is bv. een matrixvermenigvuldiging. Om de snelheidswinst op te merken maak je best gebruik van twee vierkante matrices met 1000 rijen en 1000 kolommen. Ook gebruik je best vier tot acht threads om de het resultaat te berekenen. Gebruik voor de dimensie en ook voor het aantal threads constanten.**  

**Wanneer er bijvoorbeeld acht threads worden gebruikt, kan je het resultaat als volgt berekenen. De eerste thread laat je de 0de, de 8ste, de 16de, ... rij van het resultaat bepalen. De tweede thread ontfermt zich over de 1ste, 9de, 17de, ... rij van het resultaat. Iedere thread berekent dus DIM/8 rijen van het eindresultaat waarbij de rijen op een afstand van het aantal threads van elkaar liggen.**  

**Om het opvullen van een matrix vlot te laten verlopen, geef je het element op ide rij en op de jde kolom de waarde i+j. Dit kan eenvoudig worden geprogrammeerd a.d.h.v. een dubbele for-lus. Doe dit voor beide matrices en merk op dat je dus identieke matrices met elkaar vermenigvuldigt. Schrijf ook een programma dat geen Pthreads gebruikt om duidelijk het verschil in snelheid te zien.**  
kleine intro van hoe matrix te maken  
(statisch vs dynamisch)
```c
int tab[2][2] = {{1,2}, {3,4}};
// in het  geheugen ziet dit er als [1,2,3,4] uit

/* tabel aanmaken met malloc
tabel aanmaken met aantal rijen */
int **tab=malloc(sizeof(int)*2);

// kolommen aanmaken
for (int i=0;i<2;i++){
    tab[i]=malloc(sizeof(int*2));
}

/* ziet er nu uit als [-]> [1|2]
                      [-]> [3|4]
p is dus een pointer nr int**  (int **p) */
```
beginoplossing
```c
#include <stdio.h>
#define DIM 2

// hier maken we gebruik van statisch allocatie
void matrix_mul(int (*matrixA)[DIM],int (*matrixB)[DIM], int (*res)[DIM]){
    for (int i=0;i<DIM; i++){
        for (int j=0;j<DIM;j++){
            int som = 0;
            for (int k=0;k<DIM;k++){
                som += matrixA[i][k]*matrixB[k][j];
            }
            res[i][j] = som;
        }
    }
}

void schrijf(int(*matrix)[DIM]){
    for (int i=0;i<DIM;i++){
        for (int j=0;j<DIM;j++){
            printf("%d ", matrix[i][j]);
        }
        printf("\n");
    }
}

int main(){
    int a[2][2]= {{1,2}, {3,4}};
    int b[2][2]= {{1,2}, {3,4}};
    int c[2][2];
    matrix_mul=(a,b,c);
    schrijf(c);

    return 0;
}

```
```sh
gcc 3.c
./a.out
```

hierop verder werkend (door threads toe te voegen)

```c
#include <stdio.h>
#include <pthread.h>

//aantal threads
#define N 4
// men kan de dimensies makkelijk aanpassen om te vergelijken met de niet-threaded versie
#define DIM 2

typedef struct{
    int aantalThreads;
    int startrij;
    int (*a)[DIM];
    int (*b)[DIM];
    int (*res)[DIM];
    // dit stelt de matrix_mul functie voor
    void(*f)(int (*f)(int (*)[DIM], int(*)[DIM], int (*)[DIM]), int, int);
} data_t;

// hier maken we gebruik van statisch allocatie
void matrix_mul(int (*matrixA)[DIM],int (*matrixB)[DIM], int (*res)[DIM], int thread_count, int startrij){
    // uitvoeren beginnend bij startrij
    for (int i=startrij;i<DIM; i+=thread_count){
        for (int j=0;j<DIM;j++){
            int som = 0;
            // ipv voor alle, maar voor een fractie
            for (int k=0;k<DIM;k++){
                som += matrixA[i][k]*matrixB[k][j];
            }
            res[i][j] = som;
        }
    }
}

void schrijf(int(*matrix)[DIM]){
    for (int i=0;i<DIM;i++){
        for (int j=0;j<DIM;j++){
            printf("%d ", matrix[i][j]);
        }
        printf("\n");
    }
}

void * worker(void *p){
    data_t *d = (data_t*)p;
    d->f(d->a,d->b,d->res,d->aantalThreads, d->startrij);
    return NULL;
}

void vul_matrix(int (*t)[DIM]){
    for (int i=0;i<DIM;i++){
        for (int j=0;j<DIM;j++){
            t[i][j]=i+j;
        }
    }
}

int main(){
    int a[DIM][DIM];
    vul_matrix(a);
    int b[DIM][DIM];
    vul_matrix(b);
    int c[DIM][DIM];
    pthread_t threads[N];
    data_t = data[N];
    for (int i=0;i<N;i++){
        data[i].a=a;
        data[i].b=b;
        data[i].res=c;
        data[i].f=matrix_mul;
        data[i].aantalThreads=N;
        data[i].startrij = i;
        pthread_create(&threads[i], NULL, worker, (void*)&data[i]);

    }
    for (int i=0; i<N;i++){
        // geen returnwaarde, dus geen 3e arg
        pthread_join(threads[i], NULL);
    }

    schrijf(c);

    return 0;
}

```
```sh
gcc 3.c -pthread
./a.out
```

**4.**  
niet doen wegens dat het recursie bevat & we het overige niet gezien hebben.  

**latere oefeningen ook niet te kennen**  

les eindigt met examenvragen te genereren via chatGPT  
bv een log file Analyzer  
waarbij men dus bij een gegeven log file
```
2023-05-0 15:15:20 ERROR: some error occurred
2023-05-0 15:18:30 INFO: booted
2023-05-0 15:20:00 WARNING: disk space running low
2023-05-0 15:25:30 INFO: I like cookies
...
```
de output zou kunnen parsen obv log level of dergelijke  
tip: om tijden te vergelijken: zet gwn om nr 1 groot getal