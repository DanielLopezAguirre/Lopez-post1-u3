# Lopez-post1-u3
Repositorio lab 3 post 1
POST-CONTENIDO — MANEJO DEL DEBUG: EXPLORACIÓN, ENSAMBLADO Y EJECUCIÓN PASO A PASO EN DOSBOX

1. DESCRIPCIÓN DEL LABORATORIO

PARTE 1: EXPLORACIÓN CON DEBUG EN DOSBOX

Se explora la memoria y los registros con DEBUG: se escriben bytes con E, se comprueba la escritura con D sin modificar los datos y se revisan los registros con R. También se compara el direccionamiento inmediato con el directo, verificando las codificaciones con U.

PARTE 2: ENSAMBLADO Y EJECUCIÓN PASO A PASO

Se ensamblan programas con A, se ejecutan instrucción por instrucción con T y se registran los valores de AX, BX, CX, IP y las banderas. Se programan dos bucles que suman 2 cuatro veces (AX = 0008): uno con LOOP y otro con DEC CX + JNZ. Finalmente se usa G para llegar a una dirección sin trazar cada instrucción.


2. COMANDOS DEBUG UTILIZADOS

D [dir]       Muestra el contenido de memoria sin modificarlo
E dir [bytes] Escribe o modifica bytes en memoria
F rango bytes  Rellena un rango de memoria con un patrón
R [reg]        Muestra o modifica registros
A [dir]        Ensambla instrucciones directamente en memoria
U [rango]      Desensambla código sin ejecutarlo
T              Ejecuta una sola instrucción
G [=ini] dir   Ejecuta a velocidad completa hasta un breakpoint
Q              Sale de DEBUG


3. TABLAS DE TRAZA

TABLA DE TRAZA 1: ADD

Instrucción  | AX   | BX   | CX   | IP siguiente | ZF | CF | SF
MOV AX,000A  | 000A | 0000 | 0000 | 0103         | 0  | 0  | 0
MOV BX,0005  | 000A | 0005 | 0000 | 0106         | 0  | 0  | 0
MOV CX,0003  | 000A | 0005 | 0003 | 0109         | 0  | 0  | 0
ADD AX,BX    | 000F | 0005 | 0003 | 010B         | 0  | 0  | 0
ADD AX,CX    | 0012 | 0005 | 0003 | 010D         | 0  | 0  | 0

Resultado final: AX = 0012


TABLA DE TRAZA 2: LOOP

Iteración | Instrucción | AX después | CX después | IP siguiente | ¿Salta LOOP?
1         | MOV AX      | 0000       | 0000       | 0103         | NO
1         | MOV CX      | 0000       | 0004       | 0106         | NO
1         | ADD AX      | 0002       | 0004       | 0109         | NO
1         | LOOP 0106   | 0002       | 0003       | 0106         | SI
2         | ADD AX      | 0004       | 0003       | 0109         | NO
2         | LOOP 0106   | 0004       | 0002       | 0106         | SI
3         | ADD AX      | 0006       | 0002       | 0109         | NO
3         | LOOP 0106   | 0006       | 0001       | 0106         | SI
4         | ADD AX      | 0008       | 0001       | 0109         | NO
4         | LOOP 0106   | 0008       | 0000       | 010B         | NO
5         | INT 20      | 0008       | 0000       | 010D         | NO

Resultado final: AX = 0008


TABLA DE TRAZA 3: DEC/JNZ

Iteración | Instrucción | AX después | CX después | IP siguiente | ¿JNZ salta?
Inicio    | MOV AX,0000 | 0000       | 0000       | 0203         | N/A
Inicio    | MOV CX,0004 | 0000       | 0004       | 0206         | N/A
1         | ADD AX,0002 | 0002       | 0004       | 0209         | N/A
1         | DEC CX      | 0002       | 0003       | 020A         | N/A
1         | JNZ 0206    | 0002       | 0003       | 0206         | Sí (ZF=0)
2         | ADD AX,0002 | 0004       | 0003       | 0209         | N/A
2         | DEC CX      | 0004       | 0002       | 020A         | N/A
2         | JNZ 0206    | 0004       | 0002       | 0206         | Sí (ZF=0)
3         | ADD AX,0002 | 0006       | 0002       | 0209         | N/A
3         | DEC CX      | 0006       | 0001       | 020A         | N/A
3         | JNZ 0206    | 0006       | 0001       | 0206         | Sí (ZF=0)
4         | ADD AX,0002 | 0008       | 0001       | 0209         | N/A
4         | DEC CX      | 0008       | 0000       | 020A         | N/A
4         | JNZ 0206    | 0008       | 0000       | 020C         | No (ZF=1)
Fin       | INT 20      | 0008       | 0000       | (termina)    | N/A

Resultado final: AX = 0008


4. COMPARACIÓN DE INSTRUCCIONES EJECUTADAS: LOOP VS. DEC/JNZ

Concepto                        | LOOP          | DEC/JNZ
Instrucciones de inicialización | 2 (MOV, MOV)  | 2 (MOV, MOV)
ADD ejecutados                  | 4             | 4
Control del bucle por iteración | 1 (LOOP)      | 2 (DEC + JNZ)
Control del bucle en total      | 4             | 8
INT 20                         | 1             | 1
TOTAL instrucciones ejecutadas | 11            | 15
Bytes del control del bucle     | 2 (E2 FB)     | 3 (49 75 FA)
Bytes totales del programa      | 13            | 14
Resultado final                 | AX = 0008     | AX = 0008

Conclusión: ambos producen el mismo resultado, pero DEC/JNZ ejecuta 4 instrucciones más (15 vs. 11) y ocupa 1 byte más.


5. DECISIONES TÉCNICAS JUSTIFICADAS

PARTE 1

Decisión 12: Comando de verificación de escritura en memoria

El comando correcto es D, porque permite visualizar el contenido de memoria sin modificarlo. E 300 sin una lista de bytes no es seguro para verificar, ya que entra en modo interactivo y permite sobrescribir accidentalmente los datos. F tampoco sirve porque rellena un rango de memoria y, por tanto, modificaría los datos que queremos comprobar. R muestra y permite modificar registros, pero no inspecciona directamente el contenido de memoria. Por ello, D es el único comando de los cuatro que permite comprobar la escritura de forma no destructiva y confirmar que 78 y 56 quedaron correctamente en las primeras posiciones.


Decisión 14: Direccionamiento inmediato vs. directo

MOV AX,0005 con direccionamiento inmediato no necesita un acceso adicional a memoria para obtener el operando, porque el valor 0005 está dentro de la propia instrucción. En cambio, MOV AX,[0300] usa direccionamiento directo a memoria y requiere leer el dato almacenado en la dirección 0300, por lo que necesita un acceso adicional. Preferiría el direccionamiento directo cuando el valor puede cambiar durante la ejecución, como un dato modificado previamente con E, mientras que el inmediato sirve para constantes conocidas. Finalmente, U permite comprobar las codificaciones sin ejecutar las instrucciones: muestra B8 05 00 para MOV AX,0005 y A1 00 03 para MOV AX,[0300].


PARTE 2

Paso 10: Decisión Técnica — Selección de Mecanismo de Control de Bucle (LOOP vs. DEC/JNZ)

Para este bucle contador simple, donde CX solo se usa como contador, recomiendo LOOP. LOOP 0106 se codifica en 2 bytes (E2 FB), mientras que DEC CX + JNZ 0206 ocupa 3 bytes (49 75 FA), por lo que LOOP ahorra 1 byte de código máquina. Además, el procesador extrae una sola instrucción por iteración con LOOP y dos con DEC/JNZ, lo que reduce los accesos a memoria. Preferiría DEC/JNZ si el cuerpo del bucle necesitara reutilizar CX en otra operación aritmética, o si la condición de salida no fuera solo CX distinto de cero sino una comparación con CMP. Con el comando U desensamblo ambos fragmentos sin ejecutar nada y comparo los bytes de la columna de código máquina.


Paso 12: Decisión Técnica — Comando de Verificación para Bucles de Muchas Iteraciones (T vs. G)

Con CX = 0x0064, G 20C es más práctico que repetir T unas 300 veces, porque ejecuta el bucle a velocidad completa y se detiene en IP=020C mostrando el resultado final (AX = 00C8). Lo que se pierde son los estados intermedios de AX, CX, IP y las banderas en cada iteración, así que un error dentro del bucle no sería visible. T fue necesario en el Paso 11, porque completar la tabla de traza exigía observar el efecto de cada instrucción, una granularidad que solo T ofrece. Tras ejecutar G 20C, usaría el comando R AX para confirmar únicamente el valor final de AX, sin haber observado ningún paso intermedio.


6. OBSERVACIONES DE CADA CHECKPOINT

Checkpoint Decisión 12: Se verificó con D que los bytes 78 y 56 quedaron en las primeras posiciones de 0300 sin alterar la memoria. Se justificó por qué E, F y R no sirven para verificar.

Checkpoint Decisión 14: Se comparó el direccionamiento inmediato (B8 05 00) con el directo (A1 00 03) usando U, sin ejecutar nada, y se explicó cuándo conviene cada uno.

Checkpoint Tabla ADD: Tras la última instrucción, AX = 0012 (000A + 0005 + 0003). Las banderas ZF, CF y SF permanecieron en 0 durante toda la traza.

Checkpoint Tabla LOOP: AX = 0008 y CX = 0000 al terminar. LOOP saltó a 0106 en las iteraciones 1 a 3 y no saltó en la 4 porque CX llegó a 0.

Checkpoint Tabla DEC/JNZ: Mismo resultado (AX = 0008). JNZ saltó con ZF=0 en las iteraciones 1 a 3 y no saltó en la 4 con ZF=1, pasando a INT 20 en 020C. Se observó que "T S" es un comando inválido en DEBUG (da "Error"); el correcto es solo "T".

Checkpoint Paso 10: La decisión tiene más de 100 palabras, recomienda LOOP justificando bytes e instrucciones por iteración, identifica un escenario para DEC/JNZ y explica cómo U compara tamaños.

Checkpoint Paso 12: La decisión tiene más de 100 palabras, compara G y T indicando lo que se pierde, identifica el Paso 11 como el que exigía T e indica R AX para confirmar AX.
