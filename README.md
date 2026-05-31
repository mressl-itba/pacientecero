# Paciente Cero

En esta práctica vas a construir una herramienta de **identificación de cepas virales por alineamiento genómico**.

## Contexto: la muestra de las 6:47

![Logo](img/logo.jpg)

Son las 6:47 de la mañana. El *subject* del mail dice `URGENTE — Muestra genómica H5N1 — ANLIS Malbrán`.

Dos días antes, en un establecimiento avícola a las afueras de Viedma, murieron de manera repentina más de ochenta mil aves en el lapso de pocas horas. El SENASA activó el protocolo de emergencia sanitaria. Esa misma tarde, uno de los trabajadores del galpón ingresó al Hospital Zatti con fiebre alta, dificultad respiratoria severa y una radiografía que la médica de guardia describió en su informe como "compromiso bilateral difuso, patrón intersticial severo".

El test rápido confirmó Influenza A. El subtipo, H5N1.

Esa noche, el equipo de secuenciación del Malbrán trabajó sin dormir. El genoma del virus, extraído de la muestra del paciente, quedó secuenciado a las 5:13. El archivo FASTA tardó una hora y media más en llegar.

Se llama `muestra.fasta`. El tamaño: 1.704 pares de bases.

A las 9:00 hay una videollamada con la OPS, la OMS y el Ministerio de Salud. La primera pregunta que van a hacer es: ¿cuál es el **clado**?

La respuesta no es cosmética. Determina los antivirales que se priorizan, el nivel de alerta que activa la OMS, el protocolo de rastreo de contactos, y el grado de pánico que alguien en Ginebra va a tratar de no mostrar en cámara.

El mail está dirigido a ti.

## H5N1 y la pregunta del clado

El virus de influenza aviar **H5N1** es una de las cepas de gripe con mayor potencial pandémico monitoreado por la OMS desde hace décadas. No todos los H5N1 son iguales: el virus ha evolucionado en múltiples **linajes genéticos** llamados **clados**, definidos por la secuencia del gen que codifica la **hemaglutinina** (HA, segmento 4), la proteína de superficie que el virus usa para unirse a las células del huésped.

Actualmente circulan varios clados en paralelo, con perfiles epidemiológicos distintos:

| Clado | Zona principal | Notas |
| --- | --- | --- |
| **2.3.4.4b** | Global (dominante desde 2020) | Detectado en mamíferos marinos patagónicos desde 2023; brote en bovinos de EEUU (2024) |
| **2.3.4.4a** | Europa y Asia (2016–2020) | Desplazado progresivamente por 2.3.4.4b |
| **2.3.2.1c** | Bangladesh, India, China | Circulación endémica en aves de corral asiáticas |
| **2.2.1.2** | Egipto y Medio Oriente | Mutaciones de resistencia a oseltamivir documentadas |
| **2.1.3.2** | Indonesia | Alta mortalidad en casos humanos históricos |
| **1** | Sudeste asiático | Clado histórico; referencia filogenética fundamental |

Identificar el clado de una muestra desconocida es, en la práctica, lo que hacen los laboratorios de referencia epidemiológica cuando llega una nueva cepa. Se compara el genoma secuenciado contra una biblioteca de secuencias conocidas y se determina cuál linaje conocido es el más cercano.

Eso es exactamente lo que tienes que hacer.

## Los datos

Los archivos de secuencias usan formato **FASTA**. Cada entrada tiene:

- Una línea de cabecera que empieza con `>` (identificador y descripción).
- Una o más líneas con la secuencia (A, C, G, T; a veces letras ambiguas como N).

En estas secuencias, **A**, **C**, **G** y **T** representan las bases nitrogenadas adenina, citosina, guanina y timina, respectivamente.

Ejemplo:

```text
>OP590533.1 A/pintail/Egypt/RA19853OP/2021(H5N1)...
AGCAAAAGC...
AAAAGTGATC...
```

### El genoma incógnita

La cátedra provee el archivo `muestra.fasta`, que contiene la secuencia del segmento HA del virus aislado del paciente. No sabes de qué clado es. Ése es el problema.

### Los genomas de referencia: NCBI

Vas a descargar las secuencias de referencia directamente del **NCBI**, la base de datos de secuencias genómicas del National Center for Biotechnology Information. Así trabajan los bioinformáticos de vigilancia epidemiológica.

**Pasos:**

1. Ingresa al [NCBI](https://www.ncbi.nlm.nih.gov/).
2. En el buscador, en All Databases, selecciona el filtro **Nucleotide**.
3. Busca cada secuencia por su número de accession:

    | Clado    | Cepa representativa                     | Accession |
    | -------- | --------------------------------------- | --------- |
    | 2.3.4.4b | A/pintail/Egypt/RA19853OP/2021(H5N1)    | OP590533  |
    | 2.3.4.4a | A/wild duck/Hunan/211/2005(H5N1)        | EU329186  |
    | 2.3.2.1c | A/duck/Sukoharjo/BBVW-1428-9/2012(H5N1) | KC417271  |
    | 2.2.1.2  | A/chicken/Egypt/RG-173CAL/2017(H5N1)    | MG192005  |
    | 2.1.3.2  | A/Chicken/West Java/PWT-WIJ/2006(H5N1)  | EU124148  |
    | 1        | A/Viet Nam/1203/2004(H5N1)              | AY818135  |

4. Para cada secuencia, selecciona la vista **FASTA**, y pega los datos en sendos archivos `.fasta`.

## El problema computacional: alineamiento global

Para cuantificar la similitud entre el genoma incógnita y cada referencia, vas a usar **alineamiento global de secuencias**, formalizado por Needleman y Wunsch en 1970.

La idea central es simple: queremos comparar dos secuencias A y B **de punta a punta** y permitir inserciones de gaps (`-`) para representar inserciones/deleciones biológicas. Entre todas las formas posibles de alinearlas, buscamos la que da el **mayor puntaje total**.

Para eso se usa programación dinámica con una matriz donde:

- Filas: los primeros $i$ caracteres de la secuencia A (para $i=0,1,\dots,n$). Es decir, cada fila representa un prefijo distinto de A, incluyendo el prefijo vacío cuando $i=0$.
- Columnas: los primeros $j$ caracteres de la secuencia B (para $j=0,1,\dots,m$). Del mismo modo, la columna $0$ representa el prefijo vacío de B.
- Celda $F(i,j)$: mejor puntaje posible al alinear los primeros $i$ caracteres de A con los primeros $j$ de B.

Cada celda se calcula mirando tres movimientos posibles (diagonal, arriba, izquierda):

$$
F(i, j) = \max \begin{cases}
F(i - 1, j - 1) + s(a_i, b_j) \\
F(i - 1, j) + g \\
F(i, j - 1) + g
\end{cases}
$$

donde:

- Diagonal: alineas $a_i$ con $b_j$ (match o mismatch).
- Arriba: alineas $a_i$ con un gap en B.
- Izquierda: alineas un gap en A con $b_j$.
- $s(a, b)$ da el puntaje de match/mismatch.
- $g$ es la penalización por gap (constante por posición).

Las condiciones de borde son:

- $F(0, 0) = 0$.
- $F(i, 0) = i \cdot g$: para alinear $i$ letras contra secuencia vacía solo puedes usar gaps.
- $F(0, j) = j \cdot g$: análogo para la otra secuencia.

Con esto, completas la matriz de izquierda a derecha y de arriba hacia abajo hasta llegar a $F(n, m)$, que es el mejor score global.

Luego haces **traceback**: desde $(n, m)$ vuelves a $(0, 0)$ siguiendo el movimiento que originó cada celda (diagonal/arriba/izquierda). Ese recorrido reconstruye el alineamiento óptimo caracter por caracter.

El **puntaje del alineamiento** $F(n, m)$ sirve como medida de similitud. La referencia con mayor puntaje identifica el clado más probable.

A partir del alineamiento también se computa la **identidad porcentual**:

$$
\text{identity} = \frac{\text{posiciones con match}}{\text{longitud del alineamiento}} \times 100
$$

### Ejemplo

Alineemos las secuencias $\text{AGT}$ y $\text{AT}$, con scoring:

- match $=+2$
- mismatch $=-1$
- gap $=-2$

La matriz $F$ queda:

| $F(i,j)$ | '' | A | T |
| --- | ---: | ---: | ---: |
| **''** | 0 | -2 | -4 |
| **A** | -2 | 2 | 0 |
| **G** | -4 | 0 | 1 |
| **T** | -6 | -2 | 2 |

Cómo se obtiene (ejemplos de celdas):

- $F(1, 1) = \max((0 + 2), ({-}2 + {-}2), ({-}2 + {-}2)) = 2$
- $F(2, 1) = \max(({-}2 + {-}1), (2 + {-}2), ({-}4 + {-}2)) = 0$
- $F(3, 2) = \max((0 + 2), (1 + {-}2), ({-}2 + {-}2)) = 2$

El score óptimo final es $F(3, 2) = 2$.

Traceback desde $(3, 2)$:

1. Diagonal: $T$ con $T$.
2. Arriba: $G$ con gap.
3. Izquierda: $A$ con $A$.

Alineamiento resultante:

```text
A G T
A - T
```

Score total: $2 + {-}2 + 2 = 2$.

Identidad porcentual: $2 / 3 * 100 = 66.7$.

## La herramienta

No hay código base. Implementa desde cero una herramienta de línea de comando llamada `pacientecero`.

### Interfaz

```text
pacientecero --query <muestra.fasta>
             --references <carpeta_con_archivos.fasta>
             --match <int>
             --mismatch <int>
             --gap <int>
```

| Parámetro | Descripción | Ejemplo |
| --- | --- | --- |
| `--query` | Archivo FASTA con el genoma incógnita | `muestra.fasta` |
| `--references` | Directorio con FASTAs de referencia | `references/` |
| `--match` | Puntaje por match | `2` |
| `--mismatch` | Penalización por mismatch | `-1` |
| `--gap` | Penalización por gap | `-2` |

### Formato de salida

El output debe ser una tabla en formato CSV ordenada de mayor a menor identidad porcentual:

```text
rank,accession,description,score,identity
1,PX823734,"A/American green-winged Teal/MN/25G02834-001-original/2025(H5N1)",2941,98.7
2,PV072077,"A/cattle/CA/24-038145-002-original/2024(H5N1)",2918,97.9
...
```

## Entrega

Entrega:

- El código completo de la herramienta.
- Un archivo `ENTREGA.md` con:
  - Resultado completo de tu herramienta sobre `muestra.fasta`: ranking entero y diagnóstico final.
  - La **complejidad temporal y espacial** de tu implementación en función del número de referencias *R* y las longitudes de las secuencias *n* (query) y *m* (referencia).
  - Dificultades encontradas y cómo las resolviste.
  - Reflexión: ¿qué limitaciones tiene el alineamiento global para este problema? ¿En qué escenario podría dar un diagnóstico incorrecto?

## Bonus points 🚀

- **Visualización del alineamiento**: con la opción `--show-alignment`, imprime el alineamiento completo entre el genoma incógnita y su referencia top en formato estándar:

```text
Query: ATCG--ATCGATCG
       ||||XX||||||||
Ref  : ATCGAAATCGATCG
```

## Recomendaciones

- Entiende el algoritmo antes de implementarlo. La programación dinámica y el traceback son el núcleo; el parseo de FASTA y la CLI son infraestructura.
- Para parsear la línea de comando puedes usar la biblioteca `argparse` disponible en `vcpkg`.
- El formato FASTA es simple: las líneas que empiezan con `>` son encabezados y el resto es secuencia. Impleméntalo tú mismo; no hace falta ninguna biblioteca externa.
- Verifica tu implementación con secuencias cortas para las cuales puedas calcular el resultado a mano.
- Prueba tu implementación con secuencias reales para confirmar que clasifica bien. Para descargarlas:

  1. Visita [NCBI Virus](https://www.ncbi.nlm.nih.gov/labs/virus/vssi/#/).
  2. En Begin typing a virus name, ingresa **Influenza A Virus** (taxid: 11320).
  3. En el filtro, selecciona:
     - **Identifiers and Classification > Genotype**: H5N1
     - **Genome Organization > Segment**: 4

## Referencias

- Needleman, S. B., & Wunsch, C. D. (1970). A general method applicable to the search for similarities in the amino acid sequence of two proteins. *Journal of Molecular Biology*, 48(3), 443–453.
- Gotoh, O. (1982). An improved algorithm for matching biological sequences. *Journal of Molecular Biology*, 162(3), 705–708. *(para el bonus de gap penalty afín)*
- NCBI: [https://www.ncbi.nlm.nih.gov/](https://www.ncbi.nlm.nih.gov/)
