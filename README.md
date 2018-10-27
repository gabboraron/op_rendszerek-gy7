# GY7

jelzés küldés `kill`
signalok: azonosító egész szám

`sigque`<- a signal mellett egyetlen adatot lehet átküldeni, vagy egész vagy pointer

## GY3/`pipe.c` mappa

### csővezeték
- olyan mint egy sor, egyik végén be másion ki
- csak szülő - gyerek kommuinikációra használható

teljes kód:
```` C 
       #include <stdio.h>
       #include <stdlib.h>
       #include <unistd.h> // for pipe()
       #include <string.h>
    //
    // unnamed pipe example
    //
       int main(int argc, char *argv[])
       {
           int pipefd[2]; // unnamed pipe file descriptor array
           pid_t pid;
           char sz[100];  // char array for reading from pipe

           if (pipe(pipefd) == -1)     // itt jön létre a csővezeték
       {
               perror("Hiba a pipe nyitaskor!");
               exit(EXIT_FAILURE);
           }
           pid = fork();    // creating parent-child processes
           if (pid == -1)
       {
               perror("Fork hiba");
               exit(EXIT_FAILURE);
           }

           if (pid == 0)
       {                // child process
           sleep(3);    // sleeping a few seconds, not necessary        

                        //addig vár amíg nem érkezik adat a csővezetékben

               close(pipefd[1]);  //Usually we close the unused write end
           printf("Gyerek elkezdi olvasni a csobol az adatokat!\n");
               read(pipefd[0],sz,sizeof(sz)); // reading max 100 chars
               printf("Gyerek olvasta uzenet: %s",sz);
           printf("\n");
               close(pipefd[0]); // finally we close the used read end
           }
           else
           {    // szulo process
               printf("Szulo indul!\n");
               close(pipefd[0]); //Usually we close unused read end    //lezárjuk az olvasó véget
               write(pipefd[1], "Hajra Fradi!",13);                                //ezt akarjuk eljküldeni a csővezetékben, karakterszám + végkarakter
               close(pipefd[1]); // Closing write descriptor                   //lezárjuk az íróvéget
                                                                                               //a nem használt végét a csővezetéknek zárjuk le
               printf("Szulo beirta az adatokat a csobe!\n");
               fflush(NULL);     // flushes all write buffers (not necessary)
               wait();        // waiting for child process (not necessary)
                // try it without wait()
           printf("Szulo befejezte!");    
       }
       exit(EXIT_SUCCESS);    // force exit, not necessary
       }
````

Futó folyamatok kilistázása:
`ps`

´ps aux`

#### jelzésszinkronizálással lehet kommunikálni
##### nevesített cső
> mindenki rá tud csatlakozni egy adott csővezetékre és azon tud kommunikálni. Egy közösen ismert fájlon keresztül kommunikál.
> Két csővezeték egy beszélgetésre: egyik beszél másik hallgat
