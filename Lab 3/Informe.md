Informe del Laboratorio 3 - Sistemas Operativos - 2023

Grupo 3: Boolean Club
    * Valentín Fantini
    * Gabriel Guimpelevich
    * Nahuel Fernandez
    * Luciano Rojo

parte 1: Estudiando el planificador de XV6 original.

    Actividades:
        1.1 ¿Qué política utiliza XV6 para elegir el próximo proceso a correr? (linea 445 proc.c)
                
                 El planificador empleado por XV6 utiliza la política ROUND ROBIN, que ejecuta procesos de la tabla
                 de procesos en orden y de manera uniforme, cada uno durante una cantidad de tiempo determinada llamada QUANTUM.

        1.2 ¿Cuánto dura un quantum en XV6?
            
            El quantum dura aproximadamente una décima de segundo, como se ve en la siguiente línea extraída del archivo start.c:
                int interval = 1000000; // cycles; about 1/10th second in qemu.

        1.3 ¿Cuánto dura un cambio de contexto en XV6?
            
            El cambio de contexto dura aproximadamente 800 ticks. Llegamos a esta conclusión luego de reducir el quantum hasta una cantidad de tiempo tal que los programas ya no se ejecuten, siendo 800 la mínima obtenida.

        1.4 ¿El cambio de contexto consume tiempo de un quantum?
            
            Si, el cambio de contexto consume tiempo del quantum. Esto se dá sobretodo cuando el proceso anterior no terminó de consumir su quantum antes de bloquearse, pero también de forma general.

        1.5(b) ¿Hay alguna forma de que a un proceso se le asigne menos tiempo?
            
            A los procesos se les asigna siempre la misma cantidad de tiempo (Quantum) y este es constante, pero estos programas pueden terminar antes de que este pase, o realizar una operacion I/O que lo translada al estado BLOQUEADO hasta que esta termine, dando lugar a la ejecución de otro programa.

        1.6 ¿Cuáles son los estados en los que un proceso puede permanecer en XV6 y qué los hace cambiar de estado?
            
            Los estados son: SLEEPING, RUNNABLE, RUNNING, USED, UNUSED y ZOMBIE. El cambio entre estos estados está determinado por el estado de las operaciones bloqueantes de cada proceso (ej. I/O), su estado de ejecución actual y el quantum, en caso de que esté siendo ejecutado.
            Un programa se considera Zombie cuando ya llamó a exit() pero su padre aún no llamó a wait().
            Un programa pasa de SLEEPING a RUNNABLE cuando sus operaciones bloqueantes terminaron y está listo para seguir ejecutando.
            Un programa pasa de UNUSED a USED si es encontrado por la función allocproc() en la tabla de procesos y el pedido de memoria para este proceso es exitoso.
            Un programa pasa de RUNNING a SLEEPING si debe realizar alguna operación ajena al uso de CPU, esperando la administracion de recursos, como solicitudes I/O.


Parte 2: Analizando el efecto del planificador en los procesos.

    Cada tabla se corresponde a un experimento.

                                         IObench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Original                        |
    -------------------------------------------------------------------------------
    | Scheduler                                 | Round Robin                     |
    -------------------------------------------------------------------------------
    | Caso                                      | IObench                         |
    -------------------------------------------------------------------------------
    | Promedio OPW/100T                         | 5066                            |
    -------------------------------------------------------------------------------
    | Promedio OPR/100T                         | 5066                            |
    -------------------------------------------------------------------------------
    | Cant. select. IO                          | 334694                          |
    -------------------------------------------------------------------------------
                       
                                         CPUbench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Original                        |
    -------------------------------------------------------------------------------
    | Scheduler                                 | Round Robin                     |
    -------------------------------------------------------------------------------
    | Caso                                      | CPUbench                        |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T                      | 586.3157                        |
    -------------------------------------------------------------------------------
    | Selecciones CPU                           | 2114                            |
    -------------------------------------------------------------------------------
    
                                   IObench &; CPUbench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Original                        |
    -------------------------------------------------------------------------------
    | Scheduler                                 | Round Robin                     |
    -------------------------------------------------------------------------------
    | Caso                                      | IObench &; CPUbench             |
    -------------------------------------------------------------------------------
    | Promedio OPW/100T IObench                 | 39,89                           |
    -------------------------------------------------------------------------------
    | Promedio OPR/100T IObench                 | 39,89                           |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones IO                | 2232                            |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T                      | 808.66                          |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones CPU               | 2127                            |
    -------------------------------------------------------------------------------
            
                                    CPUbench &; CPUbench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Original                        |
    -------------------------------------------------------------------------------
    | Scheduler                                 | Round Robin                     |
    -------------------------------------------------------------------------------
    | Caso                                      | CPUbench &; CPUbench            |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T 1er CPUbench         | 971,2                           |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T 2do CPUbench         | 973,7                           |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones 1er CPUbench      | 1054                            |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones 2do CPUbench      | 1062                            |
    -------------------------------------------------------------------------------
   
                             CPUbench &; CPUbench &; IObench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Original                        |
    -------------------------------------------------------------------------------
    | Scheduler                                 | Round Robin                     |
    -------------------------------------------------------------------------------
    | Caso                                      | CPUbench &; CPUbench &; IObench |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T 1er CPUbench         | 807.93                          |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T 2do CPUbench         | 841.12                          |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones 1er CPUbench      | 1059                            |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones 2do CPUbench      | 1061                            |
    -------------------------------------------------------------------------------
    | Promedio OPW/100T IObench                 | 19,25                           |
    -------------------------------------------------------------------------------
    | Promedio OPR/100T 1er IObench             | 19,25                           |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones IO                | 1234                            |
    -------------------------------------------------------------------------------

    En los resultados anteriores vemos como se contrastan procesos CPU-bound contra los IO-bound.
    En general, los IO-bound son seleccionados un mayor número de veces respecto a los CPU-bound, y que estos ultimos a su vez producen más operaciones en el mismo periodo de tiempo.
    
A continuación se detallan las mediciones de los mismos experimentos, realizados con un quantum 10 veces menor.

                                        IObench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Diez veces menor                |
    -------------------------------------------------------------------------------
    | Scheduler                                 | Round Robin                     |
    -------------------------------------------------------------------------------
    | Caso                                      | IObench                         |
    -------------------------------------------------------------------------------
    | Promedio OPW/100T                         | 5610.88                         |
    -------------------------------------------------------------------------------
    | Promedio OPR/100T                         | 5610.88                         |
    -------------------------------------------------------------------------------
    | Cant. select. IO                          | 362405                          |
    -------------------------------------------------------------------------------

                                        CPUbench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Diez veces menor                |
    -------------------------------------------------------------------------------
    | Scheduler                                 | Round Robin                     |
    -------------------------------------------------------------------------------
    | Caso                                      | CPUbench                        |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T                      | 577.3157                        |
    -------------------------------------------------------------------------------
    | Selecciones CPU                           | 21267                           |
    -------------------------------------------------------------------------------

                                  IObench &; CPUbench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Diez veces menor                |
    -------------------------------------------------------------------------------
    | Scheduler                                 | Round Robin                     |
    -------------------------------------------------------------------------------
    | Caso                                      | IObench &; CPUbench             |
    -------------------------------------------------------------------------------
    | Promedio OPW/100T IObench                 | 334,23                          |
    -------------------------------------------------------------------------------
    | Promedio OPR/100T IObench                 | 334,23                          |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones IO                | 21067                           |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T                      | 647,72                          |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones CPU               | 21143                           |
    -------------------------------------------------------------------------------
            
                                  CPUbench &; CPUbench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Diez veces menor                |
    -------------------------------------------------------------------------------
    | Scheduler                                 | Round Robin                     |
    -------------------------------------------------------------------------------
    | Caso                                      | CPUbench &; CPUbench            |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T 1er CPUbench         | 694,2                           |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T 2do CPUbench         | 831,7                           |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones 1er CPUbench      | 10517                           |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones 2do CPUbench      | 10525                           |
    -------------------------------------------------------------------------------
   
                              CPUbench &; CPUbench &; IObench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Diez veces menor                |
    -------------------------------------------------------------------------------
    | Scheduler                                 | Round Robin                     |
    -------------------------------------------------------------------------------
    | Caso                                      | CPUbench &; CPUbench &; IObench |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T 1er CPUbench         | 768.93                          |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T 2do CPUbench         | 835.12                          |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones 1er CPUbench      | 10488                           |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones 2do CPUbench      | 10617                           |
    -------------------------------------------------------------------------------
    | Promedio OPW/100T IObench                 | 169,13                          |
    -------------------------------------------------------------------------------
    | Promedio OPR/100T 1er IObench             | 169,13                          |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones IO                | 10663                           |
    -------------------------------------------------------------------------------

Vemos que en todos los experimentos de esta última sección, los procesos se seleccionan 10 veces más,
un aumento directamente proporcial a la reducción del quantum.
Corroboramos además, que el planificador RR favorece a los benchmarks de IO al usar un quantum menor, ya que se
mejora el tiempo de respuesta. Como ahora tenemos más cambios de contexto en la misma cantidad de tiempo, vemos también que los procesos IO se ven beneficiados ya que no se ven muy afectados por el overhead que esto produce.
Se ve también que, al reducir el quantum, los escenarios de CPUbench otorgan resultados menores a los originales y producen valores con promedios similares entre si. 


Parte 3: Rastreando la prioridad de los procesos.
    
    Actividad 1: Agregar una noción de prioridad a los procesos.
        
        Se agregó un campo "priority" a la estructura proc. Este campo se inicializa en NPRIO-1, siendo esta la máxima prioridad (conforme a la regla 3 de RR). 
        Se observa en la función de inicialización de los procesos:
                
                void procinit(void) {
                    struct proc *p;
                    
                    initlock(&pid_lock, "nextpid");
                    initlock(&wait_lock, "wait_lock");
                    for(p = proc; p < &proc[NPROC]; p++) {
                        initlock(&p->lock, "proc");
                        p->state = UNUSED;
                        p->kstack = KSTACK((int) (p - proc));
                        p->selected = 0;
                        p->lastchosen = 0;
                        p->priority = NPRIO-1;
                    }
                }

        Este campo solo se ve modificado de forma decreciente cuando el proceso en cuestión no se bloquea antes de terminar su quantum, y de forma ascendente en caso contrario.
        Si un proceso llama a yield() para escapar a la interrupción (y asi evitar perder prioridad), su prioridad le será decrementada de todas formas en esta función para asegurar que programas maliciosos o mal diseñados no se mantengan siempre en máxima prioridad. 
        Por otra parte, se verá incrementado si el proceso se bloquea antes del quantum (cede el control del CPU sin una interrupción por tiempo) en la función sleep().
    
    Actividad 2: Modificar la función procdump()

        Se modificó la función procdump() para que esta muestre, además de los datos originales, la prioridad de los procesos.

            void procdump(void) {
                static char *states[] = {
                [UNUSED]    "unused",
                [USED]      "used",
                [SLEEPING]  "sleep ",
                [RUNNABLE]  "runble",
                [RUNNING]   "run   ",
                [ZOMBIE]    "zombie"
                };
                struct proc *p;
                char *state;

                printf("\n");
                for(p = proc; p < &proc[NPROC]; p++){
                    if(p->state == UNUSED)
                    continue;
                    if(p->state >= 0 && p->state < NELEM(states) && states[p->state])
                    state = states[p->state];
                    else
                    state = "???";
                    printf("%d %s %s\t%d %d", p->pid, state, p->name, p->selected, p->priority);
                    printf("\n");
                }
            }


Parte 4: Implementando MLFQ

    Actividad 1: Modificando el planificador.
        Se modificó la función scheduler() para adoptar un esquema Multi-Level Feedback Queue en concordancia con las reglas 1 y 2 de dicho planificador.
            
            void scheduler(void) {
                struct proc *p;
                struct cpu *c = mycpu();
                struct proc *pmin = &proc[0];
                c->proc = 0;
                for(;;){
                    pmin = &proc[0];
                    // Avoid deadlock by ensuring that devices can interrupt.
                    intr_on();
                    
                    // Buscamos el próximo
                    for(p = proc; p < &proc[NPROC]; p++) {
                    acquire(&p->lock);
                    if(p->state == RUNNABLE){
                        if(pmin->priority < p->priority){
                        pmin=p;
                        }
                        else if(pmin->priority == p->priority){
                        if(pmin->selected > p->selected){
                            pmin=p;
                        }
                        }
                    }
                    release(&p->lock);
                    }
                    p = pmin;
                    acquire(&p->lock);
                    if(p->state == RUNNABLE) {
                    // Switch to chosen process.  It is the process's job
                    // to release its lock and then reacquire it
                    // before jumping back to us.
                    p->state = RUNNING;
                    c->proc = p;
                    p->selected++;
                    p->lastchosen = uptimemini();
                    swtch(&c->context, &p->context);

                    // Process is done running for now.
                    // It should have changed its p->state before coming back.
                    c->proc = 0;
                    }
                    release(&p->lock);
                }
            }

    Actividad 2: Repetir las mediciones con el nuevo scheduler.

    Cada tabla se corresponde a un experimento.

                                         IObench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Original                        |
    -------------------------------------------------------------------------------
    | Scheduler                                 | MLFQ                            |
    -------------------------------------------------------------------------------
    | Caso                                      | IObench                         |
    -------------------------------------------------------------------------------
    | Promedio OPW/100T                         | 5561,35                         |
    -------------------------------------------------------------------------------
    | Promedio OPR/100T                         | 5561.35                         |
    -------------------------------------------------------------------------------
    | Cant. select. IO                          | 345293                          |
    -------------------------------------------------------------------------------
                       
                                         CPUbench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Original                        |
    -------------------------------------------------------------------------------
    | Scheduler                                 | MLFQ                            |
    -------------------------------------------------------------------------------
    | Caso                                      | CPUbench                        |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T                      | 707.12                          |
    -------------------------------------------------------------------------------
    | Selecciones CPU                           | 2133                            |
    -------------------------------------------------------------------------------
    
                                   IObench &; CPUbench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Original                        |
    -------------------------------------------------------------------------------
    | Scheduler                                 | MLFQ                            |
    -------------------------------------------------------------------------------
    | Caso                                      | IObench &; CPUbench             |
    -------------------------------------------------------------------------------
    | Promedio OPW/100T IObench                 | 35,97                           |
    -------------------------------------------------------------------------------
    | Promedio OPR/100T IObench                 | 35,97                           |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones IO                | 2187                            |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T                      | 595.17                          |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones CPU               | 2113                            |
    -------------------------------------------------------------------------------
            
                                    CPUbench &; CPUbench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Original                        |
    -------------------------------------------------------------------------------
    | Scheduler                                 | MLFQ                            |
    -------------------------------------------------------------------------------
    | Caso                                      | CPUbench &; CPUbench            |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T 1er CPUbench         | 621,80                          |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T 2do CPUbench         | 856.22                          |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones 1er CPUbench      | 1051                            |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones 2do CPUbench      | 1063                            |
    -------------------------------------------------------------------------------
   
                             CPUbench &; CPUbench &; IObench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Original                        |
    -------------------------------------------------------------------------------
    | Scheduler                                 | MLFQ                            |
    -------------------------------------------------------------------------------
    | Caso                                      | CPUbench &; CPUbench &; IObench |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T 1er CPUbench         | 659,38                          |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T 2do CPUbench         | 674.11                          |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones 1er CPUbench      | 1064                            |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones 2do CPUbench      | 1054                            |
    -------------------------------------------------------------------------------
    | Promedio OPW/100T IObench                 | 46.27                           |
    -------------------------------------------------------------------------------
    | Promedio OPR/100T 1er IObench             | 46.27                           |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones IO                | 2227                            |
    -------------------------------------------------------------------------------

    En las mediciones observamos que los procesos IO-bound se ven beneficiados por el nuevo scheduler, presentando una mayor cantidad de operaciones y elecciones. Esto se dá debido a que los procesos IO-bound son más sensitivos al uso del tiempo ya que deben esperar al tráfico de datos, y debido a esto ocurre que generalmente usan el CPU durante una porción relativamente más pequeña que el quantum, ya que no suelen ser procesos demandantes de CPU y necesitan un tiempo de respuesta alto. Esto deriva en prioridades generalmente más altas en la queue, sobretodo cuando se los contrasta contra procesos CPU bound que suelen utilizar todo su quantum. 

    A continuación se detallan las mediciones de los mismos experimentos, realizados con un quantum 10 veces menor.

                                        IObench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Diez veces menor                |
    -------------------------------------------------------------------------------
    | Scheduler                                 | MLFQ                            |
    -------------------------------------------------------------------------------
    | Caso                                      | IObench                         |
    -------------------------------------------------------------------------------
    | Promedio OPW/100T                         | 6621.07                         |
    -------------------------------------------------------------------------------
    | Promedio OPR/100T                         | 6621.07                         |
    -------------------------------------------------------------------------------
    | Cant. select. IO                          | 425088                          |
    -------------------------------------------------------------------------------

                                        CPUbench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Diez veces menor                |
    -------------------------------------------------------------------------------
    | Scheduler                                 | MLFQ                            |
    -------------------------------------------------------------------------------
    | Caso                                      | CPUbench                        |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T                      | 791.67                          |
    -------------------------------------------------------------------------------
    | Selecciones CPU                           | 21041                           |
    -------------------------------------------------------------------------------

                                  IObench &; CPUbench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Diez veces menor                |
    -------------------------------------------------------------------------------
    | Scheduler                                 | MLFQ                            |
    -------------------------------------------------------------------------------
    | Caso                                      | IObench &; CPUbench             |
    -------------------------------------------------------------------------------
    | Promedio OPW/100T IObench                 | 335,65                          |
    -------------------------------------------------------------------------------
    | Promedio OPR/100T IObench                 | 335,65                          |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones IO                | 21044                           |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T                      | 751,12                          |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones CPU               | 21069                           |
    -------------------------------------------------------------------------------
            
                                  CPUbench &; CPUbench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Diez veces menor                |
    -------------------------------------------------------------------------------
    | Scheduler                                 | MLFQ                            |
    -------------------------------------------------------------------------------
    | Caso                                      | CPUbench &; CPUbench            |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T 1er CPUbench         | 755,19                           |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T 2do CPUbench         | 951,1                           |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones 1er CPUbench      | 10586                           |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones 2do CPUbench      | 10497                           |
    -------------------------------------------------------------------------------
   
                              CPUbench &; CPUbench &; IObench
    -------------------------------------------------------------------------------
    | HARDWARE                                  | AMD Ryzen 3 5300U               |
    -------------------------------------------------------------------------------
    | SOFTWARE                                  | Qemu 6.2.0                      |
    -------------------------------------------------------------------------------
    | Quantum                                   | Diez veces menor                |
    -------------------------------------------------------------------------------
    | Scheduler                                 | MLFQ                            |
    -------------------------------------------------------------------------------
    | Caso                                      | CPUbench &; CPUbench &; IObench |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T 1er CPUbench         | 705.19                          |
    -------------------------------------------------------------------------------
    | Promedio MFLOPS/100T 2do CPUbench         | 908.65                          |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones 1er CPUbench      | 10532                           |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones 2do CPUbench      | 10580                           |
    -------------------------------------------------------------------------------
    | Promedio OPW/100T IObench                 | 334.78                          |
    -------------------------------------------------------------------------------
    | Promedio OPR/100T 1er IObench             | 334.78                          |
    -------------------------------------------------------------------------------
    | Cantidad de selecciones IO                | 21046                           |
    -------------------------------------------------------------------------------

    Vemos que al reducir el quantum, obtenemos resultados proporcionalmente mayores en la cantidad de elecciones de cada proceso, particularmente en los que involucran IO, ya que los procesos CPU-bound se ven aún más afectados (negativamente, en términos de prioridad) por este quantum reducido. Como ya notamos en el analisis con el quantum original en este scheduler, los procesos CPU-bound ya tendían a utilizar el CPU durante todo su quantum, fenómeno que se vió intensificado cuando redujimos aún más el tiempo de ejecución que tenían disponible. 
    
    Actividad 3: ¿Se puede producir starvation en el nuevo planificador?
        
        Si, es posible que se produzca starvation ya que no cuenta con una función "priority boost" que le otorgue
        máxima prioridad a todos los procesos pasada una cantidad determinada de tiempo.
        En el estado actual del planificador, los procesos con tiempo de CPU largos podrían pasar mucho tiempo
        en prioridades bajas mientras que los procesos rápidos monopolizarían las posiciones altas indefinidamente.
    
Conclusión:

    En este proyecto aprendimos como se comporta el scheduler Round Robin con procesos IO-bound y CPU-bound mediante numerosos testeos. Además, implementamos el scheduler MLFQ y lo sometimos a las mismas pruebas para evaluar su comportamiento y elaborar así conclusiones sobre las ventajas y desventajas de cada uno respecto a los tipos de procesos mencionados.
    En el proceso aprendimos como obtener las propiedades de cada proceso desde la program table y a manipularlas para obtener información a cerca de ellos. También aprendimos a usar esta información tanto para observar el comportamiento de los schedulers como para implementar funciones que utilicen estos datos y así mejorar nuestro entendimiento a cerca de los procesos concurrentes en el sistema. Esto fue realizado utilizando como banco de pruebas el sistema RISC-V6 emulado en QEMU, forzando su ejecución en modo mono-core para asegurar así que los procesos debían compartir un solo núcleo y fuera más fácil observar las diferencias entre los tests. 