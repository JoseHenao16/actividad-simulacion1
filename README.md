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
      PID 0 ejecuta la instrucción de E/S en el tick 1, luego entra en estado bloqueado durante 5 ticks (del 2 al 6).

      Mientras PID 0 está bloqueado, PID 1 comienza a usar la CPU desde el tick 2 al 5.

      En el tick 6, PID 1 termina.

      En el tick 7, PID 0 se desbloquea (RUN:io_done) y finaliza.

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
      Sí, el orden en que se ejecutan los procesos sí influye.
      Si el proceso que hace E/S se ejecuta primero, mientras espera que termine su operación, el otro proceso (que solo usa CPU) puede aprovechar ese tiempo para trabajar. Así, ambos procesos avanzan al mismo tiempo y el sistema termina más rápido, en solo 7 ciclos en lugar de 11.

      Esto muestra cómo un buen manejo del orden de los procesos puede hacer que el sistema sea más eficiente, usando mejor el tiempo disponible en vez de dejar recursos esperando sin hacer nada.
</details> <br> ```

4. We'll now explore some of the other flags. One important flag is `-S`, which determines how the system reacts when a process issues an I/O. With the flag set to SWITCH ON END, the system will NOT switch to another process while one is doing I/O, instead waiting until the process is completely finished. What happens when you run the following two processes (`-l 1:0,4:100 -c -S SWITCH ON END`), one doing I/O and the other doing CPU work?

   <details>
   <summary>Answer</summary>
   Coloque aqui su respuerta
   </details>
   <br>

5. Now, run the same processes, but with the switching behavior set to switch to another process whenever one is WAITING for I/O (`-l 1:0,4:100 -c -S SWITCH ON IO`). What happens now? Use `-c` and `-p` to confirm that you are right.

   <details>
   <summary>Answer</summary>
   Coloque aqui su respuerta
   </details>
   <br>

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
