# Lopez-post1-u3
Repositorio lab 3 post 1
Post-contenido — Manejo del DEBUG: Exploración, Ensamblado y Ejecución Paso a Paso en DOSBox

Parte 1: Exploración con DEBUG en DOSBox

Decision 12 
El comando correcto es D, porque permite visualizar el contenido de memoria sin modificarlo. E 300 sin una lista de bytes no es seguro para verificar, ya que entra en modo interactivo y permite sobrescribir accidentalmente los datos. F tampoco sirve porque rellena un rango de memoria y, por tanto, modificaría los datos que queremos comprobar. R muestra y permite modificar registros, pero no inspecciona directamente el contenido de memoria. Por ello, D es el único comando de los cuatro que permite comprobar la escritura de forma no destructiva y confirmar que 78 y 56 quedaron correctamente en las primeras posiciones.
Decision 14

Parte 2: Ensamblado y Ejecución Paso a Paso

MOV AX,0005 con direccionamiento inmediato no necesita un acceso adicional a memoria para obtener el operando, porque el valor 0005 está dentro de la propia instrucción. En cambio, MOV AX,[0300] usa direccionamiento directo a memoria y requiere leer el dato almacenado en la dirección 0300, por lo que necesita un acceso adicional. Preferiría el direccionamiento directo cuando el valor puede cambiar durante la ejecución, como un dato modificado previamente con E, mientras que el inmediato sirve para constantes conocidas. Finalmente, U permite comprobar las codificaciones sin ejecutar las instrucciones: muestra B8 05 00 para MOV AX,0005 y A1 00 03 para MOV AX,[0300].
Tabla de trazas add
Instrucción | AX | BX | CX | IP siguiente | ZF | CF | SF
MOV AX,000A | 000A | 0000 | 0000 | 0103 | 0 | 0 | 0
MOV BX,0005 | 000A | 0005 | 0000 | 0106 | 0 | 0 | 0
MOV CX,0003 | 000A | 0005 | 0003 | 0109 | 0 | 0 | 0
ADD AX,BX | 000F | 0005 | 0003 | 010B | 0 | 0 | 0
ADD AX,CX | 0012 | 0005 | 0003 | 010D | 0 | 0 | 0

Resultado final: AX = 0012
tabla trazas loop
Iteración | Instrucción | AX después | CX después | IP siguiente | ¿Salta LOOP?
1 | MOV AX | 0000 | 0000 | 0103 | NO
1 | MOV CX | 0000 | 0004 | 0106 | NO
1 | ADD AX | 0002 | 0004 | 0109 | NO
1 | LOOP 0106 | 0002 | 0003 | 0106 | SI
2 | ADD AX | 0004 | 0003 | 0109 | NO
2 | LOOP 0106 | 0004 | 0002 | 0106 | SI
3 | ADD AX | 0006 | 0002 | 0109 | NO
3 | LOOP 0106 | 0006 | 00001 | 0106 | SI
4 | ADD AX | 0008 | 0001 | 0109 | NO
4 | LOOP 0106 | 0008 | 0000 | 0109 | NO
5 | INT 20 | 0008 | 0000 | 010B | NO
tabla trazas dec/jnz
Iteración | Instrucción  | AX después | CX después | IP siguiente | ¿JNZ salta?
Inicio    | MOV AX,0000  | 0000       | 0000       | 0203         | N/A
Inicio    | MOV CX,0004  | 0000       | 0004       | 0206         | N/A
1         | ADD AX,0002  | 0002       | 0004       | 0209         | N/A
1         | DEC CX       | 0002       | 0003       | 020A         | N/A
1         | JNZ 0206     | 0002       | 0003       | 0206         | Sí (ZF=0)
2         | ADD AX,0002  | 0004       | 0003       | 0209         | N/A
2         | DEC CX       | 0004       | 0002       | 020A         | N/A
2         | JNZ 0206     | 0004       | 0002       | 0206         | Sí (ZF=0)
3         | ADD AX,0002  | 0006       | 0002       | 0209         | N/A
3         | DEC CX       | 0006       | 0001       | 020A         | N/A
3         | JNZ 0206     | 0006       | 0001       | 0206         | Sí (ZF=0)
4         | ADD AX,0002  | 0008       | 0001       | 0209         | N/A
4         | DEC CX       | 0008       | 0000       | 020A         | N/A
4         | JNZ 0206     | 0008       | 0000       | 020C         | No (ZF=1)
Fin       | INT 20       | 0008       | 0000       | (termina)    | N/A

Paso 10: Decisión Técnica — Selección de Mecanismo de Control de Bucle (LOOP vs. DEC/JNZ)

Recomiendo LOOP para este bucle contador simple, porque CX solo sirve de contador. LOOP 0106 ocupa 2 bytes (E2 FB) frente a 3 bytes de DEC CX + JNZ 0206 (49 75 FA), y el procesador extrae una sola instrucción por iteración en lugar de dos. Preferiría DEC/JNZ si el cuerpo del bucle reutilizara CX en otra operación aritmética, o si la salida dependiera de una comparación con CMP y no solo de CX distinto de cero. Con el comando U desensamblo ambos fragmentos sin ejecutar nada y comparo los bytes mostrados en la columna de código máquina de cada versión.


Paso 12: Decisión Técnica — Comando de Verificación para Bucles de Muchas Iteraciones (T vs. G)

Con CX = 0x0064, G 20C es más práctico que repetir T unas 300 veces: ejecuta el bucle a velocidad completa y se detiene en IP=020C mostrando el resultado final. Lo que se pierde son los estados intermedios de AX, CX, IP y las banderas en cada iteración. T fue necesario en el Paso 11, porque completar la tabla de traza exigía observar el efecto de cada instrucción, granularidad que solo T ofrece. Tras ejecutar G 20C, usaría -R AX para confirmar únicamente el valor final de AX, sin observar ningún paso intermedio.
