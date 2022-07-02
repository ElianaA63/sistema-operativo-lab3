# SISTEMAS OPERATIVOS 2021 - LABORATORIO 3: PLANIFICACIÓN
 
## Grupo 5:
* Arias, Eliana - eliana.arias@mi.unc.edu.ar
* Leiva, Mauro - mauro.leiva@mi.unc.edu.ar
* Toyos, Milagros - milagros.toyos@mi.unc.edu.ar
 
## Modo de trabajo
Cada uno de los integrantes del grupo colaboró con el trabajo usando su computadora y programando en tiempo real conectados por Google meet/grupo en Whatsapp.
 
## Modo de uso
 
Para compilar: 
> $ make CPUS=1 qemu-nox

En donde con **qemu-nox** permitimos solo el modo texto y trabajar con solo 1 procesador.

Dentro de qemu, compilamos los casos que correspondan, como por ejemplo:
*Caso 1: 1 iobench 1 cpubench* 
> $ iobench &; cpubench

 
## Proceso de desarrollo
 
 
 
### Parte 1: Estudiando el planificador de XV6

1. Analizar el código del planificador y responda: *¿Qué política utiliza xv6 para elegir el próximo proceso a correr?* Pista: xv6 nunca sale de la función scheduler por medios "normales".
 
El planificador `*xv6*` utiliza el algoritmo más simple para distribuir el tiempo de procesador entre los procesos, `*Round Robin*`. Cada CPU recorre de forma independiente la lista de procesos y a cada proceso que se marca como listo para ejecutarse se le asigna un quantum de tiempo de procesador. Esta política es equitativa porque asigna el mismo tiempo máximo de procesador a todos los procesos elegibles.
 
2. Analizar el código que interrumpe a un proceso al final de su quantum y responda:
a. *¿Cuánto dura un quantum en xv6?*
 
En xv6, cada quantum dura por defecto 10ms.
 
 
 
### Parte 2: cómo el planificador afecta a los procesos
 
Queremos ver si un planificador MLFQ cambia el rendimiento de los procesos ligados a la CPU y a la Entrada/Salida, y si lo hace, compararlo con el rendimiento del Round Robin.
Para obtener resultados realizamos una serie de mediciones utilizando `qemu` con una sola CPU, para ver cómo trabaja el sistema al ejecutar diferentes combinaciones de procesos `iobench` y `cpubench`.
Iobench es un programa de usuario que, según el quantum actual, calcula cuántas operaciones de entrada/salida por tick (IOPT) se producen, también llamado el *tiempo de respuesta I/O*.
A su vez, el programa cpubench calcula la potencia mediante Kilo-Float Operations per Tick (KFLOPT).
 
Cuando se ejecuta en una máquina de una sola CPU las operaciones ligadas a la CPU son más lentas cada vez que un nuevo proceso entra en el planificador. Sin embargo, algunas de ellas tienen un mayor impacto en el proceso en ejecución.



### Parte 3: Rastreando la prioridad de los procesos
 
1. Agregue un campo en struct proc que guarde la prioridad del proceso (entre 0 y NPRIO-1 para #define NPRIO 3 niveles en total).
Se añadió un campo de prioridad a la estructura proc. Además de definir el número máximo de prioridades (3) en el archivo param.h. Definimos la prioridad más alta como el valor más bajo (0), del mismo modo, la prioridad más baja se definió como el valor más alto **NPRIO-1**

- MLFQ regla 3: Cuando un proceso se inicia, su prioridad será máxima.
Se implementó en el archivo proc.c, en la función allocproc().
 
- MLFQ regla 4: Descender de prioridad cada vez que el proceso pasa todo un quantum realizando cómputo.
Se implementaron cambios en proc.c. Cuando un proceso llama a `yield()`, la prioridad disminuye llamando a la función auxiliar `decr_priority` (disminuir es acercarse a NPRIO-1) ya que asumimos que el quantum ha terminado.
 
- Ascender de prioridad cada vez que el proceso bloquea antes de terminar su quantum.
Se implementaron cambios en proc.c. Cuando un proceso llama a `sleep()` la prioridad aumenta llamando a la función auxiliar `incr_priority` (aumentar es acercarse a 0) ya que cede lo que queda del quantum.
 
2. Modifique la función procdump (que se invoca con CTRL-P ) para que imprima la prioridad de los procesos. Así, al correr nuevamente iobench y cpubench , debería darse que cpubench tenga baja prioridad mientras que iobench tenga alta prioridad.
Se modificó la funcion procdump() en el archivo proc.c para mostrar la prioridad de cada proceso.



### Parte 4: Implementando MLFQ


1. En esta última etapa, se modificó el scheduler para incorporar las dos primeras reglas de MFLQ. Tomando dos procesos, se corre el que mayor prioridad tiene; si tienen igual prioridad, se corren ambos en formato round-robin.

3. Para análisis responda: ¿Se puede producir starvation en el nuevo planificador? Justifique su respuesta.
Al no incorporar la quinta regla del MFLQ (priority boost), se puede generar "starvation". Esto ocurriría si continuamente ingresan a la cola procesos de alta prioridad, no permitiendo la eventual llegada del scheduler a un proceso de muy baja prioridad.
