# Actividad de seguimiento - Simulación 1

| Integrante                | correo                  | usuario github          |
| ------------------------- | ----------------------- | ----------------------- |
| José David Henao Gallego  | jose.henao1@udea.edu.co | JoseHenao16             |
| Nombre comp. integrante 2 | correo integrante 2     | gihub user integrante 2 |

## Instrucciones

Antes de empezar a realizar esta actividad haga un **fork** de este repositorio y sobre este trabaje en la solución de las preguntas planteadas en la actividad de simulación. Las respuestas deben ser respondidas en español o si lo prefiere en ingles en el lugar señalado para ello (La palabra **answer** muestra donde).

**Importante**:

- Como la actividad es en las parejas del laboratorio, solo uno de los integrantes tiene que hacer el fork; y sobre repositorio bifurcado que se genera, la modificación se realiza en equipo.
- Como la entrega se debe hacer modificando el archivo READNE, se recomienda que consulte mas sobre el lenguaje **Markdown**. En el repo adjuntan dos cheatsheet ([cheat sheet 1](Markdown_Cheat_Sheet.pdf), [cheatsheet 2](markdown-cheatsheet.pdf)) para consulta rapida.
- Entre mas creativo mejor.

## Homework (Simulation)

This program, [`process-run.py`](process-run.py), allows you to see how process states change as programs run and either use the CPU (e.g., perform an add instruction) or do I/O (e.g., send a request to a disk and wait for it to complete). See the [README](https://github.com/remzi-arpacidusseau/ostep-homework/blob/master/cpu-intro/README.md) for details.

### Questions

1. Run `process-run.py` with the following flags: `-l 5:100,5:100`. What should the CPU utilization be (e.g., the percent of time the CPU is in use?) Why do you know this? Use the `-c` and `-p` flags to see if you were right.

   <details>
   <summary>Respuesta</summary>
   
   El comando ejecutado fue:

   ```bash
   python process-run.py -l 5:100,5:100 -c -p   

   Lo que se esperaba para cada proceso está configurado para ejecutar 5 instrucciones, y todas son del tipo que usa solo la CPU (es decir, no hacen operaciones de entrada/salida como leer un archivo o esperar datos).

   Como los dos procesos solo usan la CPU y no se detienen esperando nada, siempre hay al menos uno listo para trabajar. Por eso, se espera que la CPU esté ocupada todo el tiempo mientras se ejecutan.

   Calculos:
      Instrucciones totales: 5 (Proceso 0) + 5 (Proceso 1) = 10
      No hay E/S → No hay tiempo de inactividad
      Tiempo total = 10 ciclos
      Tiempo de CPU ocupada = 10 ciclos

   Utilización CPU = (Tiempo ocupado) / (Tiempo total) = 10 / 10 = 100%

   Resultado:

      Time        PID: 0        PID: 1           CPU           IOs
      1        RUN:cpu         READY             1
      2        RUN:cpu         READY             1
      3        RUN:cpu         READY             1
      4        RUN:cpu         READY             1
      5        RUN:cpu         READY             1
      6           DONE       RUN:cpu             1
      7           DONE       RUN:cpu             1
      8           DONE       RUN:cpu             1
      9           DONE       RUN:cpu             1
      10           DONE       RUN:cpu             1

   Resultado de la simulación:
      Stats: Total Time 10
      Stats: CPU Busy 10 (100.00%)
      Stats: IO Busy  0 (0.00%)

   Conclusión:
      La utilización de la CPU es del 100%, como se esperaba. La CPU estuvo completamente ocupada ya que no hubo operaciones de entrada/salida que provocaran esperas o cambios de contexto.

</details> <br> ```

2. Now run with these flags: `./process-run.py -l 4:100,1:0`. These flags specify one process with 4 instructions (all to use the CPU), and one that simply issues an I/O and waits for it to be done. How long does it take to complete both processes? Use `-c` and `-p` to find out if you were right.

   <details>
   <summary>Respuesta</summary>
   
   El comando ejecutado fue:

   ```bash
   python process-run.py -l 4:100,1:0 -c -p

   Se crean dos procesos:
      PID 0 ejecuta 4 instrucciones de CPU (100% CPU)
      PID 1 tiene una única instrucción de E/S (0% CPU)

   Cálculo o análisis:
      PID 0 corre primero, ocupando la CPU del tick 1 al 4 con instrucciones de CPU

      En el tick 5, PID 0 termina (DONE) y el planificador permite que PID 1 ejecute su instrucción de E/S (RUN:io)

      PID 1 queda bloqueado por 5 ticks (de 6 a 10)

      Y por ultimo en el tick 11, PID 1 se desbloquea (RUN:io_done) y termina.

   Resultado:
      Time        PID: 0        PID: 1           CPU           IOs
      1        RUN:cpu         READY             1
      2        RUN:cpu         READY             1
      3        RUN:cpu         READY             1
      4        RUN:cpu         READY             1
      5           DONE        RUN:io             1
      6           DONE       BLOCKED                           1
      7           DONE       BLOCKED                           1
      8           DONE       BLOCKED                           1
      9           DONE       BLOCKED                           1
      10           DONE       BLOCKED                           1
      11*          DONE   RUN:io_done             1

   Resultado de la simulación:
      Stats: Total Time 11
      Stats: CPU Busy 6 (54.55%)
      Stats: IO Busy  5 (45.45%)

   Conclusión:
      Los dos procesos terminan su ejecución en 11 ciclos.
      Durante ese tiempo, la CPU estuvo trabajando en 6 ciclos y el dispositivo de E/S en los otros 5.

      Esto es lógico, porque uno de los procesos hizo una operación de E/S que tardó 5 ciclos en completarse. Mientras tanto, no había más procesos listos para usar la CPU, así que el sistema tuvo que esperar a que la E/S terminara antes de poder finalizar la simulación.

</details> <br> ```

3. Switch the order of the processes: `-l 1:0,4:100`. What happens now? Does switching the order matter? Why? (As always, use `-c` and `-p` to see if you were right)

   <details>
   <summary>Respuesta</summary>
   
   El comando ejecutado fue:

   ```bash
   python process-run.py -l 1:0,4:100 -c -p

   Se crean dos procesos, pero ahora en un orden inverso:
      PID 0: una instrucción de E/S (0% CPU)
      PID 1: 4 instrucciones de CPU (100% CPU)
   
   Cálculo o análisis:
      El proceso PID 0 inicia y rápidamente se bloquea por E/S

      A diferencia de cuando PID 1 era el primero (ver punto 2), esta vez el sistema aprovecha el tiempo mientras PID 0 está bloqueado, ejecutando el proceso que realiza CPU (PID 1)

      Cuando finaliza la E/S, el proceso PID 0 se completa inmediatamente.

   Resultado:
      Time        PID: 0        PID: 1           CPU           IOs
      1         RUN:io         READY             1
      2        BLOCKED       RUN:cpu             1             1
      3        BLOCKED       RUN:cpu             1             1
      4        BLOCKED       RUN:cpu             1             1
      5        BLOCKED       RUN:cpu             1             1
      6        BLOCKED          DONE                           1
      7*   RUN:io_done          DONE             1
   
   Resultado de la simulación:
      Stats: Total Time 7
      Stats: CPU Busy 6 (85.71%)
      Stats: IO Busy  5 (71.43%)

   Conclusión:
      Sí, el orden en que se ejecutan los procesos tiene un impacto importante.
      Al poner primero el proceso que realiza E/S, el sistema puede aprovechar el tiempo en que ese proceso está bloqueado para ejecutar el proceso que usa CPU. Esto permite que ambos procesos avancen en paralelo, usando mejor los recursos disponibles
      A diferencia del caso anterior (-l 4:100,1:0), donde hubo varios ciclos en los que la CPU no hizo nada, en este escenario se logró una utilización mucho más eficiente tanto de la CPU como del dispositivo de E/S.
</details> <br> ```

4. We'll now explore some of the other flags. One important flag is `-S`, which determines how the system reacts when a process issues an I/O. With the flag set to SWITCH ON END, the system will NOT switch to another process while one is doing I/O, instead waiting until the process is completely finished. What happens when you run the following two processes (`-l 1:0,4:100 -c -S SWITCH ON END`), one doing I/O and the other doing CPU work?

   <details>
   <summary>Respuesta</summary>
   
    El comando ejecutado fue:

   ```bash
   python process-run.py -l 1:0,4:100 -c -p -S SWITCH_ON_END

   Se ejecutan dos procesos:
      PID 0: 1 instrucción de E/S
      PID 1: 4 instrucciones de CPU

   La opción -S SWITCH ON END le dice al sistema que no cambie de proceso cuando uno inicia una operación de E/S. En lugar de hacer un cambio inmediato, el sistema espera a que ese proceso termine por completo, incluso si queda bloqueado por la E/S.

   Cálculo o análisis:
      El proceso PID 0 comienza con una instrucción de E/S y entra en estado bloqueado desde el tiempo 2 al 6

      Aunque PID 1 está listo para ejecutarse, el sistema no lo selecciona debido a la política SWITCH ON END

      La CPU permanece inactiva durante 5 unidades de tiempo, reduciendo la eficiencia general del sistema

      Una vez que termina la E/S (tick 7), se ejecuta PID 1 de forma continua.

   Resultado:
      Time        PID: 0        PID: 1           CPU           IOs
      1         RUN:io         READY             1
      2        BLOCKED         READY                           1
      3        BLOCKED         READY                           1
      4        BLOCKED         READY                           1
      5        BLOCKED         READY                           1
      6        BLOCKED         READY                           1
      7*   RUN:io_done         READY             1
      8           DONE       RUN:cpu             1
      9           DONE       RUN:cpu             1
      10           DONE       RUN:cpu             1
      11           DONE       RUN:cpu             1

   Resultado de la simulación:
      Stats: Total Time 11
      Stats: CPU Busy 6 (54.55%)
      Stats: IO Busy  5 (45.45%)

   Conclusión:
      Sí importa la política de cambio de procesos durante E/S
      En este caso, SWITCH ON END provoca tiempo ocioso innecesario de CPU, lo cual reduce la utilización total (54.55%)
      Una política más eficiente permitiría cambiar al proceso PID 1 mientras PID 0 está bloqueado, maximizando el uso del procesador.
</details> <br> ```

5. Now, run the same processes, but with the switching behavior set to switch to another process whenever one is WAITING for I/O (`-l 1:0,4:100 -c -S SWITCH ON IO`). What happens now? Use `-c` and `-p` to confirm that you are right.

   <details>
   <summary>Respuesta</summary>
   
   El comando ejecutado fue:

   ```bash
   python process-run.py -l 1:0,4:100 -c -p -S SWITCH_ON_IO

   Cálculo o análisis:
      El proceso PID 0 inicia con una operación de E/S y es bloqueado en el segundo ciclo

      Automáticamente, el sistema asigna el CPU al proceso PID 1, que puede ejecutar sus 4 instrucciones sin interrupciones

      Cuando el proceso PID 1 termina, la E/S del PID 0 finaliza y este también concluye.

   Resultado:
      Time        PID: 0        PID: 1           CPU           IOs
      1         RUN:io         READY             1
      2        BLOCKED       RUN:cpu             1             1
      3        BLOCKED       RUN:cpu             1             1
      4        BLOCKED       RUN:cpu             1             1
      5        BLOCKED       RUN:cpu             1             1
      6        BLOCKED          DONE                           1
      7*   RUN:io_done          DONE             1

   Resultado de la simulación:
      Stats: Total Time 7
      Stats: CPU Busy 6 (85.71%)
      Stats: IO Busy  5 (71.43%)

   Conclusión:
      Cambiar de inmediato al proceso que está listo permite aprovechar mejor la CPU
      El tiempo total de ejecución es de 7 unidades, igual que en el caso anterior con el orden -l 1:0,4:100, lo que demuestra que este tipo de conmutación evita dejar la CPU inactiva
      Este enfoque resulta más eficiente que usar SWITCH_ON_END, ya que no se pierde tiempo esperando a que un proceso termine si hay otro disponible para continuar.

</details> <br> ```

6. One other important behavior is what to do when an I/O completes. With `-I IO RUN LATER`, when an I/O completes, the process that issued it is not necessarily run right away; rather, whatever was running at the time keeps running. What happens when you run this combination of processes? (`./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH ON IO -c -p -I IO RUN LATER`) Are system resources being effectively utilized?

   <details>
   <summary>Answer</summary>
   Coloque aqui su respuerta
   </details>
   <br>

7. Now run the same processes, but with `-I IO RUN IMMEDIATE` set, which immediately runs the process that issued the I/O. How does this behavior differ? Why might running a process that just completed an I/O again be a good idea?

   <details>
   <summary>Answer</summary>
   Coloque aqui su respuerta
   </details>
   <br>

### Criterios de evaluación

- [x] Despligue de los resultados y analisis claro de los resultados respecto a lo visto en la teoria.
- [x] Creatividad y orden.
