## Sistema operativo

#### Que es un proceso

Un proceso es una imagen o programa en ejecución.\
Tiene varias secciones
- Texto
- Program counter
- Sección de datos
- Stack del proceso
- El heap

El codigo esta en la sección de texto, luego el compilador hace asignaciones en la parte de data, para variables locales o globales, la memoria temporal que usa el codigo es el stack, que entran datos apilados y se va moviendo así la memoria, es dinámica, las variables automaticas (return, funciones, etc...) se guardan al stack. El puntero por ejemplo, va en el heap generalmente, también es memoria dinámica.

##### Seccuencia de ejecucion:
- Fetch a la memoria
- Decode
- Ejecucíon, con uso de registros
- Escritura de resultados en la memoria y en en los registros
- Repetir

##### Proces control Block

Cada proceso esta representado por un PCB, este contiene varios datos que describen el estado del proceso 

Cuando se dice que un procesador esta corriendo varios procesos a la vez es que el watchdog timer esta setiado cada muy bajos milisegundos y cambia de proceso, entonces da la ilusion que son varios procesos a la vez

Cola FIFO first in first out cola LIFO last in first out

----
resumen control
---
Un proceso es un programa en ejecución.\
CPU en ejecución, primero hace el fetch de la instrucción en la memória, decodifica, ejecuta usando los registros, escribe los resultados en los registros y repite.\
Estados de un proceso: 
- New: esta siendo creado.
- Ready: el proceso esta esperando la CPU.
- Running: se esta ejecutando las instrucciones.
- Waiting: Esperando que ocurra un evento.
- Terminated: termino la ejecución del proceso.

![Procesos](estados.png)

Proces control Block (PCB)

el pcb tiene varios datos que describen el estado del proceso.
- Estado del proceso.
- Program counter.
- Registers.
- Planificación.
- Administración de memoria.
- Contabilidad.

Cuando se habla que varios procesos estan corriendo a la misma vez es que pasa muy rapido y se ve como un efecto de que son varios al mismo tiempo.\
Los cambios se llaman cambio de contexto. Los PCB guardan la información de estado de los procesos, estos cambios se pueden gatillar por una interrupción, por ejemplo un timer.

El planificador de prcesos es aquel que designa los distintos programas o tareas al CPU para que las ejecute.\
Los procesos estan listos para ser ejecutados entran a una cola de procesos (ready queue).

Un proceso puede crear distintos procesos, aquellos que crean procesos se llaman *procesos padres* y el nuevo procesos se llama *proceso hijo*.\ hay dos opciones, el proceso padre sigue ejecutando con los hijos o el padre espera a que los hijos terminen.\
 
Threads son los procesos livianos, un proceso pesado es con un solo thread.\
Usar threads mejora el rendimiento del sistema con el usuario.

---
### Material estudio tarea 1

resumen OSTEP clase 2
----
El sistema operativo se encarga de que los procesos se ejecuten de manera eficiente, esto lo hace mediante un metodo que se llama virtualización, lo cual toma las partes físicas de un SO y las tranforma en algo más general para usar la forma virtual de si mismo, nos referimos a las maquinas virtuales.\
El SO tambien tiene APIs que podemos usar para llamar funciones, estas son la system calls.

Si tenemos el siguente codigo:

````c
#include <stdio.h>
#include <stdlib.h>
#include <sys/time.h>
#include <assert.h>
#include "common.h"

int main (int argc, char *argv[]){
    if (argc != 2){
        fprintf(stderr, "usage: cpu <string>\n");
        exit(1)
    }

    char *str = argv[1];
    while (1){
        Spin (1);
        printf("%s\n",str);
    }
    return 0;
}
````
#### CPU
Lo que podemos ver es que es un loop infinito que solo printea lo que le damos como argumento al principio, cada print se va a hacer despues de un segundo por la funcion Spin.\
Lo importante que tenemos que ver que pasa si le digo que corra el codigo de 4 maneras distntas,
![CPU](Ej_CPU.png)
Lo que se puede ver es que corre 4 al mismo tiempo, lo que hace esta ilusion posible es la maquina virtual al virtualizar la cpu tiene por decirlo infinitas CPU, transforma una grande en varias chicas y por eso podemos ver esto.

#### Memoria

````c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include "common.h"

int
main(int argc, char *argv[])
{
int *p = malloc(sizeof(int)); // a1
assert(p != NULL);
printf("(%d) address pointed to by p: %p\n",
getpid(), p); // a2
*p = 0; // a3
while (1) {
Spin(1);
*p = *p + 1;
printf("(%d) p: %d\n", getpid(), *p); // a4
}
return 0;
}
````
La memoria es un arreglo con direcciones que guardan datos y para escribir en la memoria se tiene que actualizar y escribir encima de estas direcciones que tiene.\ lo importante aca es que la MV virtualiza la memoria y cada prceso tiene su propio espacio de memoria privado, no comparten memoria.

Para crear threads la linea de codigo es Pthread_create(), podemos ver un thread como un funcion que corre en los mismos espacios de memoria que las otras funciones, cuando le damos instrucciones muy largas a los threads podemos ver que estas se confunden y pueden no tirar el valor deseado esta pasa cuando tenemos varias instrucciones separadas, como ir a buscar a la memoria, luego un proceso y luego guardarlo en la memoria, esto hace que se cree un problema de concurrencia al no ejecutarse atomicamente (todo de a una).

#### Procesos

Los procesos son aquellas instrucciones que ejecuta el sistema operativo. Las intrucciones que lee un proceso esta en la memoria, al igual de donde estan los datos, osea los procesos estan muy ligados a la memoria en varios aspectos.\
API:
- Create: Cualquier SO tiene que tener una forma de crear nuevos procesos, podemos ver un ejemplo cuando ecribimos un comando en la terminal si o si se esta creando un proceso nuevo en el PC.
- Destroy: Al igual que podemos crear nuevos procesos tambien hay que poder destruirlos de manera forzosa.
- Wait: Hay veces que es bueno esperar que un proceso termine de correr.
- Miscellanius Control: Hay veces que esperar o matar un proceso no es lo deseado, por eso hay otrocontroles que son posibles, por ejemplo suspender un proceso.
- Status: Es ver la información de estado de un proceso 

##### Create
Lo primero que tenemos que hacer para que el sistema operativo corra un programa es cargarlo y un dato estatico en la memoria, luego la memoria tiene que asignar espacio para el run-time stack o solamente stack, en C este espacio se ocupa para las variables, los parametros de una función y returns. También tiene que asignar memoria del heap, este espacio se ocupa para las cosas dinamicas como los malloc() y el free() 

##### State
- Running: significa que un proceso esta corriendo en el procesador
- Ready: Significa que esta listo para correr pero el SO ha decidido que todavia no es momento para que lo haga.
- Blocked: Significa que el proceso hizo alguna operacion que no va a correr hasta que pase algun evento que haga que restaure su running state denuevo.
![state](State.png)

##### Fork() System Call
Se usa para crear nuevos procesos, es un llamado muy extraño.

````c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main(int argc, char *argv[]) {
printf("hello world (pid:%d)\n", (int) getpid());
int rc = fork();
if (rc < 0) { // fork failed; exit
fprintf(stderr, "fork failed\n");
exit(1);
} else if (rc == 0) { // child (new process)
printf("hello, I am child (pid:%d)\n", (int) getpid());
} else { // parent goes down this path (main)
int rc_wait = wait(NULL);
printf("hello, I am parent of %d (rc_wait:%d) (pid:%d)\n",
rc, rc_wait, (int) getpid());
}
return 0;
}
````
![fork](Fork.png)

Lo que podemos ver en primera instacia es primero que imprime un Hello World con el numero de PID que es el process idenifier, es como el rut del proceso. Luego llama a la funcion fork() que es una manera de hacer un proceso, lo que hace es llamar una copia casi exacta de la llamada que vendria siendo un proceso hijo, lo que se ve es que no corre dentro del main(). Es raro, el proceso hijo ahora tiene otro PI, osea que tiene su propio espacio en la memoria privado y es distinto que el proceso padre.

##### Wait() System Call
Hasta ahora pudimos crear un proceso hijo que hace un print y se va, es util a veces esperar a que un proceso hijo termine antes de que el padre empiece a correr, para eso usamos este System Call Wait().
````c
} else if (rc == 0) { // child (new process)
printf("hello, I am child (pid:%d)\n", (int) getpid());
} else { // parent goes down this path (main)
int rc_wait = wait(NULL); // aca esta el Wait()
printf("hello, I am parent of %d (rc_wait:%d) (pid:%d)\n",
rc, rc_wait, (int) getpid());
````
Como podemos ver en el fragmento modificado del codigo anterior aca hay un wait() hasta que el proceso hijo termine.

![w](wait.png)

##### Exec() System Call
Se usa cuando quiero correr un programa que es diferente que el llamado, por ejemplo el fork()

````c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

int main(int argc, char *argv[]) {
printf("hello world (pid:%d)\n", (int) getpid());
int rc = fork();
if (rc < 0) { // fork failed; exit
fprintf(stderr, "fork failed\n");
exit(1);
} else if (rc == 0) { // child (new process)
printf("hello, I am child (pid:%d)\n", (int) getpid());
char *myargs[3];
myargs[0] = strdup("wc"); // program: "wc" (word count)
myargs[1] = strdup("p3.c"); // argument: file to count
myargs[2] = NULL; // marks end of array
execvp(myargs[0], myargs); // runs word count
printf("this shouldn’t print out");
} else { // parent goes down this path (main)
int rc_wait = wait(NULL);
printf("hello, I am parent of %d (rc_wait:%d) (pid:%d)\n",
rc, rc_wait, (int) getpid());
}
return 0;
}
````
Como se puede ver en el codigo en la parte donde sale myargv[*] y luego execvp(myargs[0], myargs), executa otro programa que cuenta la cantidad de palabras que hay en el archivo del cual se esta haciendo el System Call.\
Lo que hace es que se le da un nombre de un ejecutable ("wc") y unos argumentos ("p3.c"), luego carga el codigo y los datos estaticos del ejecutable y sobre escribe su propio segmento de codigo y se reinicia el heap con los datos dinamicos, no crea otro proceso solo que transforma el que ya se esta ejecutando en otro programa.

Resumen ppt 2
----

##### Multiplexación del CPU

Con la multiprogramación el SO mantiene varios procesos ejecutando a la vez en la memoria principal, pero solo un proceso esta corriendo en cada CPU. El cambio de contexto es cuando hay cambio de CPU entre procesos, estos cambio de contexto se gatillan por un evento, por ejemlo una interrupción.

##### Planificación de procesos

Uno de los objetivos de la multiprogramación es mantener ocupada la CPU eficientemente, el tiempo de la CPU es gastado por vários procesos a la vez.

###### Colas De Planificación
Cuando los procesos son creados entran en una cola esperando a ser ejecutados por la CPU, esta cola esta en la memoria y se llama *"ready queue"*. Cuando el proceso es elejido para ser ejecutado se despacha.\
El proceso migra de colas durante su tiempo, el SO debe elejir con ciertos criterios cual es el proceso que se va a ejecutar, la naturaleza de los planificadores esta en función del tiempo.
- Planificados de corto plazo (despachador): Este elije con mayor frecuencia los trabajos que se van a ejecutar, es muy rapido.
- Planificador de mediano plazo: Saca los procesos de la memoria y son movidos al disco, cuando el proceso es ejecutado nuevamente lo saca del disco y lo mueve nuevamente a la memoria (swapping).
- Planificador de largo plazo (job sheduler): Solo tiene sentido en procesamiento de batches, los procesos se mantienen en un pool en almacenamiento masivo, elije el proximo trabajo a cargar en la memoria.

##### Creación de procesos
El creador de procesos se llama proceso padre y el creado se llama proceso hijo, cada proceso hijo puede crear más procesos y aasí crear un árbol de procesos. Cada proceso tiene su identificador PID.\
Hay dos posibilidades cuando se crea un proceso, que el padre siga corriendo con los procesos hijos o que este los espere hasta que termine.\
Costo de crear un proceso:
- Es barato construir el PCB
- Es caro crear las tablas y direcciones del proceso
- El proceso fork() que crea una copia de los datos del proceso padre es costoso.

En resumen podemos decir que crear un proceso nuevo es costos.

##### Procesos livianos

Nos vamos a referir a *thread* al flujo de ejecución que vive dentro de un proceso. Thread = proceso liviano, podemos decir que un proceso con un solo thread es un proceso pesado, por lo que inferimos que un proceso puede tener más de un thread. 

![thread](thread.png)

En la imagen se puede ver más o menos como el thread afecta en el proceso.\
Los thread encapsulan concurrencia, son el componente activo del un proceso, los espacios de direcciones encapsulan protección de memoria, previenen que los sistemas defectuosos boten el sistema, son el componente pasivo de un proceso.\
Las ventajas de usar multiples threads:
- Mejor respuesta del sistema al usuario.
- Facilidad de recursos compartidos, ya que los threads comparten la memoria y recursos del proceso al que pertenecen.
- Es más  economico, los threads son más rapidos y menos costosos de crear que un proceso nuevo.



###### Modelos 

El soporte para threads puede implementarse en espacio de usuarios y de SO (kernel). Los *User threads* son ejecutados por bibliotecas que ejecutan el espacio de usuario. Siempre ambos tipos de threads van a trener algún tipo de relacion.

- Muchos a uno: Implementa múltiples *user threads* asociados a un *kernel thread*, la admisitración esta hecha por la biblioteca del espacio de usuario, se bloquea si un thread hace una llamada bloqueante al sistema, un thread accede al kernel a la vez

![muchos_a_unos](muchos_a_uno.png)

- Uno a uno: Cada user thread tiene su kernel thread, permite mayor concurrencia que el modelo muchos a uno, permite que threads se ejecuten paralelamente en multiprocesadores, crear un kernel por cada user es más costoso.

![uno_uno](uno%20a%20uno.png)

- Muchos a muchos: Multiplexea varios user thread con kernel threads, el programador puede crear tantos threads como quiera y los kernel pueden correr en paralelo en sistemas multiprocesador.

###### Estados de un thread

Los threads comparten el contenido de memoria y el estado de entrada y salida (sistema de archivos, conexiones a la red...). Existe información de estado privado para cada thread, el thread tiene el mismo estado de vida que un proceso, new --> ready --> running --> waiting --> terminated.\

Creación y terminación de threads: Un thread puede crear hijos y puede esparar a la terminación de ellos, la operación join se utiliza tanto en los procesos como en el thread y permite esperar que un proceso padre espere al proceso hijo. Un thread puede renunciar voluntariamente al CPU mediante a la función yield.\
Los threads pueden ser creados de *"dos maneras"*, en el espacio de usuario o en el kernel, lo cual si se crea en el kernel tenemos que entender que las estructuras de datos se mantienen en el espacio del kernel. Las bilbiotecas que se ocupan hoy en dia son POSIX, WIN32, Java.\
POSIX ocupa Pthreads que es una especificación.\
Instrucciones:
- pthread_t: identificador de threads
- pthread_attr_t: set de atributos 
- pthread_create: creador de threads
- pthread_join: hace que el padre espere al hijo
- pthread_exit: para que termine si ejecución


Resumen ppt 3 
----

### Algoritmos de planificación

Ciclos de rafaga: generalmente los procesos involucran un ciclo compuesto por operaciones de CPU y E/S. Decimos que un proceso se ejecuta en ráfagas de CPU y E/S.

![rafagas](rafaga.png)

La medición de rafaga depende de cada computador, tienden a una distribución exponencial, gran cantidad de rafagas pequeñas y pocas de rafagas grandes.


##### Planificador de CPU

Razones de un planificado de CPU
- running a waiting: en una operación de entrada/salida (E/S)
- running a ready: interrupción
- waiting a ready: termina una operación E/S
- Un proceso termina

Las desiciones 1 y 4 se dice que son cooperativas o no expropiativas, la segunda es expropiativa.\
La no expropiativa no puede ser interrumpido por el sistema operativo hasta que termine o el proceso sesa el control voluntariamente. La expropiativa el SO si puede interrumpir el proceso para asignar la CPU a otro proceso.

##### Algoritmos de Planificacion

- First-Come, First-Served: Es el más simple, se ejecuta de principio a fin ininterrumpidamente, la CPU es asignada a los procesos por orden de llegada, es mediante una cola FIFO (first in first out), la única desventaja es que la cola de esprea es muy alta
- Round Robin: Cada proceso es asignado a la CPU por un tiempo fijo (time slice), una vez finalizado el proceso la CPU le es expropiada al proceso y el SO lo pone al final de la cola, requiere interrupciones de timer. Al usar este metodo se puede ver perdida de tiempo por los cambio de contexto, el tiempo de cambio no puede ser muy chico y el tiempo si es muy garnde tiende a comportarse como un FCFS.
- Shortest job first: Se requiere información del futuro, cuantas rafagas de CPU requieren los procesos, este algoritmo asigna primero el proceso que requiere menos catidad de computo, su finalidad es terminar los procesos cortos primero.
- SRTF Adaptivo: Cambiala política basándose en el comportamiento pasado, funciona cuando el proceso tiene comportamientos predecibles.
- Planificación por prioridad: Se le asigna una proriedad a cada proceso y la CPU toma la cual tiene proriedad más alta, por ejemplo podemos ver que SJF la proriedad esta en la catidad de computos, la más corta va primero.


Planificación Multinivel Se consideran categorias de procesos, los batch, interactivos, entre otros. Hay distintos algoritmos para cada categoria.

![multinivel](multinivel.png)

##### Multilevel Feedback Queue

Diseñado por Corbato, busca optimizar el tiempo e completar los procesos (turnaround time) y al mismo tiempo busca minimizar el tiempo de respuesta. Para esto este sistema tiene unas reglas que sigue para optimizar de mejor manera el tiempo.
- Regla 1: si prioridad(A) > prioridad(B), se ejecuta A.
- Regla 2: si prioridad(A) = prioridad(B), se ejecutan ambas en Round Robin.
- Regla 3: Cuando un trabajo llega, es puesto en la maxima prioridad.
- Regla 4: Una vez que el proceso usa toda su asignación de tiempo en un nivel dado, su prioridad baja y se mueve a una cola de menor prioridad.
- Regla 5: Despues de un periodo S, se mueven todos los procesos de prioridad a una más alta.

Es interesante la conclusión de esto ya que observa la ejecución de los procesos suponiendo que en un comienza son cortos y luego es capaz de ajustar la priorización dependiendo de lo largo y la carga al CPU

##### Inter-Process Communocation
![IPC](IPC.png)

Mecanismo para que puedan comunicarse entre si, se pueden comunicar sin la necesida de tener variables globales en el sistema, puede ser directa entre procesos o indirecta mediante casillas de mensajería.\
Mensajería directa:
- send(P,message), envia un mensaje al proceso P.
- receive(message, Q), recibe un mensaje del proceso Q.

Mensajería indirecta: 
- send(A, message), envia mensaje al casilla A.
- receive(message,A), recibe mensaje de la casilla A.

Una casilla puede ser de un proceso en particular o del mismo SO, para que dos procesos se comunique deben establecer un canal de comunicación, es necesario un canal físico y lógico.

##### Pipes ordinarios

son un canal de comunicación unidireccional, un proceso escribe al pipe y el otro pide leer lo del pipe.

Aca va un ejemplo de pipe ordinario en codigo.

![pipe](pipe_ordinario.png)

##### Named Pipes
Hacen uso del sistema de archivos pero no son archivos. Se crean usando el comando mkfifo

````sh
mkfifo named_pipe
echo "Hi" > named_pipe &
cat named_pipe
````
La primeria linea de codigo crea el named pipe, luego se escribe dentro del pipe y la ultima lee adentro del pipe.

##### Unix domain Sockets

Son canales de comunicación bidimensionales, podemos conectar procesos servidores y procesos clientes, varios clientes pueden comunicarse con el servidor mediante este metodo, postgresql es un ejemplo de utulización de de UDS.

Resumen OSTEP
---

Cuando hacemos un proseso en modo de usuario esta limitado a lo que puede hacer, por ejemplo no puede hacer E/S request, ya que el SO es muy probable que mate el proceso. La otra manera de correr el proceso es atravez del modo kernel, este lo corre directamente el SO y si puede hacer peticiones de E/S.

##### Cooperative Approach: wait for system calls

Se puede decir que el SO confía en que el proceso se va a comportar bien, procesos que corren por demaciado tiempo el SO asume que va a dar CPU para que corra otro proceso, esto se logra mediante un System Call llamado yield, que transfiere el control al SO y ver si decide correr otro proceso.

##### Non-Cooperative Approach: The OS takes control

Cuando una aproximación cooperativa se queda pegado en un loop la única solución es reiniciar el sistema. Para solucionar esto se utiliza un timer interrupt, el cual interrumpe cada cierto tiempo el proceso y toma control el SO y decide que proceso correr en ese minuto, luego que parte otro proceso, el timer vuelve a activarse.

##### Saving or Restoring Context 

Ya sabemos que pasa con el SO al interrumpir un proceso puede eligir uno nuevo o quedar con el que se esta ejecutando en el minuto, para saber eso existen los *sheduler*, si decide cambiar hablamos de un cambio de contexto, cuando pasa esto el SO tiene que ver si guardar datos en un registro y pedir despues los datos para otro proceso.

##### Sheduling Rules

 1. Cada proceso corre por la misma cantidad de tiempo.
 2. Todos los procesos llegan al mismo tiempo.
 3. Una vez que empieza termina
 2. Solo usan CPU
 2. El run-time es conocido

 ##### Propocitional Share

 Los tickets son medidas para ver cuanto vale el *peso* del proceso, un ejemplo de esto es un proceso A y B, el A tiene 75 tickets y el B 25 tickets, podemos decir que A va a recibir el 75% del CPU y B el sobrante.

 ##### Lottery Sheduling

 Lo que hace este tipo de planificador es que elije de manera probabilistica el ticket ganador, osea no necesariamente el de mayor tamaño corre primero, se puede decir que tiene su cierto grado de alazar pero igual el proceso que tiene más tickets sale más veces seguidas.

Resumen ppt 4
---

##### Productor-Consumidor

Supongamos que tenemos dos procesos, uno consumidor y otro productor.
````c
// productor
while(true){
    while(counter == BUFFER_SIZE)
        //no hace nada
    buffer[in]=item;
    in = (in+1) % BUFFER_SIZE;
    counter++;
    
}
````

````c
// consumidor
while(true){
    while(counter == 0)
        // no hace nada
    item = buffer[out]
    out = (out+1) % BUFFER_SIZE;
    counter--;

    
}
````
### explicacion de un buffer 
Un buffer es una región de memoria utilizada para almacenar datos temporalmente mientras se transfieren de un lugar a otro. Los buffers son ampliamente utilizados en la programación y en los sistemas operativos para gestionar el flujo de datos entre procesos, dispositivos de entrada/salida, y redes.

Características de un Buffer
Almacenamiento Temporal: Los buffers almacenan datos temporalmente para equilibrar las diferencias en la velocidad de procesamiento entre dos sistemas o componentes.
Tamaño Fijo o Dinámico: Los buffers pueden tener un tamaño fijo o dinámico, dependiendo de la implementación y el uso específico.
FIFO (First In, First Out): En muchos casos, los buffers funcionan como colas FIFO, donde el primer dato en entrar es el primero en salir.
Sincronización: En entornos de múltiples hilos o procesos, el acceso al buffer debe estar sincronizado para evitar condiciones de carrera y asegurar la coherencia de los datos.


##### Condición de carrera

Cuando varios procesos acceden y manipulan un mismo dato en forma concurrente, y el resultado depende de el orden en que se hayan ejecutado, estamos ante una condición de carrera. Para evitar esto debemos hacer que solo uno pueda manipular el dato que se esta procesando.

##### Problema de la sección critica 

Permite que los procesos puedan cooperar sin estorbarse, cada proceso debe pedir permiso para ejecutar su sección crítica.\
![critica](critica.png)\
Acá podemos ver que pidepermiso mediante la *entry section*, despues de ejecuta la sección critica, y termina con un protocolo de salida que es el *exit section* y ejecuta el resto del programa.

Requisitos:
- Exclusión mutua: Solo un proceso puede entrar a su sección crítica.
- Progreso: Ningún proceso fuera de una sección critica puede impedir que un proceso entre en su sección crítica.
- Espera acotada: El tiempo de espera debe ser limitado, tiene que existir un numero de veces que un proceso puede acceder a su sección cuando otro pidó acceso a la sección critica.

##### Semáforos 

Permite sincronizar procesos asincronicos, un semáforo S lo podemos modelar como una variable entera que accede a travéz de operaciones atómicas; wait() y Signal()

````c
wait(S){
    while S <= 0; 
        S--;
}

signal(S){
    S++;
}
````

##### Semáforo Binario (Mutex)

Son aquellos que sus valores pueden ser 1 o 0, podemos usar semáforos para solucionar problemas de sección crítica de varios procesos, los n procesos pueden compartir un semáforo de la siguente manera.

````c
do{
    wait(mutex);
        // sección crítica
    signal(mutex);
    //código
    
}while(true)
````

Es posible implementar semáforos se pueden implementar para evitar la espera ocupada.
````c
typedef struct{
    int value;
    struct process *list;
} semaphore;

wait(semaphore *S){
    S-> value--;
    if (S->value < 0){
        // agregar proceso
        block();
    }
}
````


