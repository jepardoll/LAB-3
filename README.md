# Contador de 60 Segundos con Displays de 7 Segmentos

Este repositorio contiene la documentación técnica, el diseño y la solución del laboratorio **Implementación de Periféricos**. El proyecto consiste en el diseño de un periférico de hardware que actúa como un contador de segundos síncrono que se reinicia automáticamente cada 60 segundos (de `00` a `59`), utilizando una FPGA y visualizando el resultado en dos displays de 7 segmentos independientes.

El proyecto está estructurado bajo tres dominios fundamentales del diseño electrónico: **Comportamental**, **Estructural** y **Físico**.

---

## 1. Dominio Comportamental

Este dominio define *qué* hace el sistema, sus interfaces y su lógica de control algorítmica, sin entrar en detalles de la implementación del hardware interno.

### Diagrama de Caja Negra

Establece las fronteras del sistema, definiendo claramente las señales de entrada (estímulos y sensores) y las señales de salida (actuadores e indicadores visuales).

* **Entradas:** Reloj maestro (`clk`) de la tarjeta FPGA.
* **Salidas:** Buses de control combinacionales de 7 segmentos independientes para las unidades (`seg_uni[6:0]`) y para las decenas (`seg_dec[6:0]`).

![Diagrama de Caja Negra](Imagenes/DiagramaCajaNegra.png)

### Diagrama de Flujo

Describe el algoritmo de toma de decisiones y las prioridades del sistema:

1.  **Prioridad 1 (División de Reloj):** El acumulador incrementa con cada flanco ascendente de la señal maestra. Al alcanzar el límite establecido, genera un pulso síncrono de habilitación y limpia el registro.
2.  **Prioridad 2 (Contador de Unidades):** Incrementa bajo la señal del divisor. Al alcanzar el valor binario máximo de `9`, se reinicia inmediatamente a `0` y propaga un acarreo síncrono.
3.  **Prioridad 3 (Contador de Decenas):** Incrementa exclusivamente con el desborde del módulo de unidades. Al alcanzar el valor límite de `5` de manera simultánea con un `9` en las unidades (segundo 59), limpia ambos contadores.

![Diagrama de Flujo](Imagenes/DiagramaFlujo.jpeg)

### Tabla de Verdad y Ecuaciones Booleanas

Define la lógica combinacional exacta que rige el sistema. A partir de mapas de simplificación, se obtuvieron las siguientes ecuaciones booleanas simplificadas que dictan el comportamiento de los segmentos de salida ($a, b, c, d, e, f, g$) en función de los bits binarios BCD de los contadores internos ($D_3$, $D_2$, $D_1$, $D_0$):

* $a = D_3 \lor D_1 \lor (D_2 \land D_0) \lor (\\overline{D_2} \land \\overline{D_0})$
* $b = \\overline{D_2} \lor (\\overline{D_1} \land \\overline{D_0}) \lor (D_1 \land D_0)$
* $c = D_2 \lor \\overline{D_1} \lor D_0$
* $d = D_3 \lor (\\overline{D_2} \land \\overline{D_0}) \lor (D_1 \land \\overline{D_0}) \lor (D_2 \land \\overline{D_1} \land D_0) \lor (\\overline{D_2} \land D_1)$
* $e = (\\overline{D_2} \land \\overline{D_0}) \lor (D_1 \land \\overline{D_0})$
* $f = D_3 \lor (\\overline{D_1} \land \\overline{D_0}) \lor (D_2 \land \\overline{D_1}) \lor (D_2 \land \\overline{D_0})$
* $g = D_3 \lor (D_2 \land \\overline{D_1}) \lor (\\overline{D_2} \land D_1) \lor (D_1 \land \\overline{D_0})$

![Tabla de Verdad](Imagenes/TablaVerdad.png)

## Diccionario de Señales (Entradas / Salidas)

| Tipo | Variable Lógica | Etiqueta Física | Descripción |
| :--- | :---: | :---: | :--- |
| **Entrada** | `clk` | `clk` | Señal de reloj maestro de la placa FPGA (50 MHz) |
| **Salida** | `seg_uni` | `seg_uni[6:0]` | Bus de salida hacia los segmentos del display de unidades |
| **Salida** | `seg_dec` | `seg_dec[6:0]` | Bus de salida hacia los segmentos del display de decenas |
| **Interna** | `divisor` | `divisor[25:0]` | Registro contador encargado de la división de frecuencia |
| **Interna** | `unidades` | `unidades[3:0]` | Registro contador BCD para el dígito de las unidades (0-9) |
| **Interna** | `decenas` | `decenas[3:0]` | Registro contador BCD para el dígito de las decenas (0-5) |

### Descripción en Lenguaje de Hardware (Verilog)
A partir de las ecuaciones obtenidas, el comportamiento del sistema se describe utilizando el lenguaje de descripción de hardware Verilog. Este código es el que define la lógica que posteriormente será sintetizada en la FPGA:
