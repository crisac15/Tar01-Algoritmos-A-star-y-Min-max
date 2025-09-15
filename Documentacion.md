# Algoritmos A\* y Min-max

## Introducción

Este proyecto implementa el juego **Peg Solitaire** con algoritmos de búsqueda informada como A\*, así como MinMax en **Lines and Boxes**:

- **A\*** para encontrar secuencias óptimas de movimientos en el clásico juego puzzle.
- **Min-Max** para modelar escenarios de decisión competitiva en un juego player vs player.

El objetivo es explorar y diseñar estos algoritmos para familiarizarnos con su uso y características funcionales y teóricas.

---

## Peg Solitaire

###  Representación del Tablero

- El tablero se modela como una **matriz** donde:

  - `-1` indica casillas no jugables, representada visualmente con "O".
  - `0` representa una casilla vacía, representada visualmente con "\_".
  - `1` representa una ficha, representada visualmente con "X".

- Se implementó la clase `Board` con métodos:
  - `legal_moves()` → genera los movimientos válidos.
  - `apply()` → aplicar movimientos.
  - `pegs_count` → cuenta las fichas restantes en el tablero.
  - `is_goal()` → verifica si se alcanzó el estado meta.
  - `key()` → devuelve un identificador hashable para registrar estados visitados.

Esta abstracción permite cambiar fácilmente entre distintas configuraciones iniciales (tablero inglés, europeo, o personalizados) através de 3 metodos estaticos que modifican la matriz de la instancia ya sea Inglesa(default), o según su tipo (0 y 1) en el constructor de la clase Board. Una instancia de Board con parámetro 0 dará resultado a una matriz de juego 7x5, mientras que una instancia creada con parámetro 1 generará una partida con tablero 9x9.

###  Algoritmo A\*

#### Descripción

- Cada nodo almacena:

  - Estado del tablero.
  - `g`: costo acumulado (saltos realizados).
  - `h`: heurística estimada.
  - Puntero al padre y al movimiento aplicado.

- La **lista abierta (open set)** se gestiona con un heap de prioridad (`heapq` en Python) para mayor eficiencia.
- Se mantiene un `g_score` para registrar el mejor costo conocido por estado, y un `closed_set` para evitar re-expansiones.
- Al alcanzar un estado objetivo, se reconstruye el camino usando los punteros `parent`.

#### Heurística

La heurística diseñada combina:

1. **Admisible:** número de fichas restantes − 1.
2. **Factor estructural:** penalización por fichas aisladas.
3. **Factor geométrico:** distancia Manhattan de las fichas al centro.

###  Decisiones de diseño

1. **Uso de matrices con `-1,0,1`:**  
   Más simple para imprimir y validar movimientos que una lista de posiciones, aunque de cara al usuarios tengan representaciones grafiacas más de acorde al juego.

2. **Clonación de estados en A\*:**  
   Se eligió por claridad sobre el uso de `apply` en un único objeto, es importante mantener el objeto sobre el que probamos movimientos y luego diferenciarlo sobre del que lo aplicamos.

3. **Heap de prioridad (`heapq`):**  
   Garantiza eficiencia en la selección de nodos con menor `f = g+h`, cosa que no pasaría con una lista o con una cola normal.

###  Discusion de Resultados

#### Pruebas realizadas en Peg Solitaire

- **Tablero inglés (7×7 default)** con hueco inicial en el centro.
- **Tablero asimetrico 7x5** como variante que no es simétrica.
- **Tablero europeo (9×9)** como variante más amplia.

#### Resultados y Discusión

Los experimentos realizados muestran diferencias claras en el desempeño del algoritmo A\* al resolver distintas configuraciones de **Peg Solitaire**:

##### 1. Tiempo de ejecución

- El tablero **inglés (7×7)** requirió aproximadamente **3 segundos**, mientras que la **variante 7×5** fue la más rápida (~1 segundo).
- El tablero **europeo (9×9)** presentó un incremento muy notable, alcanzando **~17 segundos**.
- Esto refleja cómo el **espacio de estados crece exponencialmente** con el tamaño del tablero: más casillas jugables implican más movimientos legales y, por tanto, un árbol de búsqueda mucho más grande.
- La variante 7×5, al tener menos casillas activas que el tablero inglés, reduce drásticamente el espacio de búsqueda y por ende el tiempo.

##### 2. Nodos explorados

- En el tablero **inglés (7×7)** se expandieron alrededor de **31 nodos**.
- La **variante 7×5** requirió aún menos (**~25 nodos**), confirmando que al haber menos piezas el algoritmo tiene menos caminos que evaluar.
- En contraste, el tablero **europeo (9×9)** disparó la exploración a más de **55 nodos**, duplicando prácticamente la cantidad del tablero inglés.
- Esto muestra que A\* es sensible al **número de piezas iniciales**: más fichas generan más combinaciones posibles de saltos, lo que incrementa tanto la frontera como la profundidad del árbol.

##### 3. Interpretación global

- Los resultados confirman que la **heurística base** (fichas restantes − 1, con penalizaciones adicionales) ayuda a guiar la búsqueda, pero no elimina la explosión combinatoria en tableros más grandes.
- La relación casi proporcional entre **nodos explorados y tiempo de ejecución** indica que el cuello de botella está en la **expansión de estados** y la **clonación de tableros**.
- La variante 7×5 se resuelve con relativa facilidad porque tiene menos fichas y menos posibilidades de movimiento, mientras que la variante europea evidencia la dificultad de escalar el algoritmo a tableros mayores.

##### 4. Implicaciones

- Para **tableros pequeños y medianos** (7×7, 7×5), A\* es eficiente y encuentra soluciones en tiempos razonables.
- En **tableros grandes** (9×9 o más), se requieren heurísticas más fuertes u optimizaciones adicionales como **poda** o **paralelización**.
- Los resultados sugieren que la heurística actual es adecuada como punto de partida, pero no suficiente para tableros de mayor complejidad.

---

##  Algoritmo Min-Max

### Descripción

- Modela el juego como un escenario de decisión donde el **jugador** intenta completar la mayor cantidad de boxes llegando a ser el último que coloque una línea para que complete el box, y un **oponente** puede escoger movimientos que compliquen la resolución o así mismo que busque el gane completando los boxes.
- Implementación clásica de **Min-Max**:
  - Nodo MAX → selecciona el movimiento más prometedor para el jugador.
  - Nodo MIN → selecciona el movimiento más prometedor para la computado.

- Se crea una clase con los diferentes estados del tablero, por lo que en cada momento que se pase un tablero con un movimiento realizado por el algoritmo se creará una clase y se le pasará como parámetro al algoritmo para que lo siga desarrolando, lo que permite que el juego principal nunca se llegue a modificar, esto proque los argumentos en python se pasan por referencia y no por valor.

### Heurística de evaluación

Como heurística se implementó el score de cada jugador de la partida, es decir, la cantidad de boxes completados, en caso de que la computadora obtenga 4 boxes, se le colocará como score -4, esto para que pueda obtener un valor mínimo con base en el algoritmo. En caso de que el algoritmo note que el jugador obtenga 4 boxes se le dará como score un 4 positivo.

##  Desafíos encontrados

- **Generalización del objetivo:** inicialmente estaba fijo en `(2,2)`; se adaptó para que funcione con distintas dimensiones.
- **Heurísticas informativas:** diseñar heurísticas más fuertes sin perder la eficiencia fue un reto.
- **Escalabilidad en Min-Max:** el árbol crece exponencialmente; se consideró limitar profundidad o aplicar poda alfa-beta.
- **Equilibrio entre rendimiento y claridad:** clonar tableros es más limpio, pero cuesta en memoria, así como realizar copias profundas, sin embargo se utilizó para que no hubiera problemas de sobreescribir en los datos de las clases anteriores.

---

##  Conclusiones

- Se implementaron con éxito **A\*** y **Min-Max** aplicados a Peg Solitaire y Lines and Boxes.
- El proyecto permitió estudiar tanto un enfoque de búsqueda informada como de juego adversarial.
- Se logró modelar e implementar el juego como un árbol de decisión, evaluando un score para cada jugador, según corresponda
- La implementación del algoritmo Min-Max permitió que se llegara a la conclusión de que entre mas profundidad de el árbol las decisiones son mas precisas, sin embargo, el poder computacional que requiere es inmenso, además, que se logró notar la diferencia entre el algoritmo con poda y sin poda alfa-beta.
- El principal desafío fue la explosión exponencial del árbol de juego, lo que hace necesaria la poda alfa-beta

---
