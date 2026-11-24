# Práctica: Memoria RAM — Análisis, Auditoría y Rendimiento

**Equipo de referencia:** Acer TravelMate P216-51-TCO  
**Placa base:** RPL Xylophone_RTU (V1.20)  
**Procesador:** 13th Gen Intel Core i3-1315U (6 núcleos / 8 hilos)  
**Memoria instalada:** 8,00 GB DDR4 (7,69 GB utilizables)  
**Sistema Operativo:** Microsoft Windows 11 Education (64 bits, compilación 26200)  

---

## Ejercicio 1: Comparativa de Rendimiento Generacional (DDR4 vs. DDR5)

Valores obtenidos tomando como referencia los registros de **PassMark (MemoryBench)** para ambos kits de memoria:

* **Memoria A (DDR4):** G.Skill Ripjaws V 16GB (2x8GB) DDR4 3200MHz.
* **Memoria B (DDR5):** G.Skill Trident Z5 32GB (2x16GB) DDR5 6000MHz.

| Métrica | Memoria A (DDR4-3200) | Memoria B (DDR5-6000) | % de Mejora |
| :--- | :---: | :---: | :---: |
| **Memory Mark (Puntuación total)** | 2.650 | 3.710 | $+40,0\%$ |
| **Read Uncached (MB/s)** | 16.200 MB/s | 26.730 MB/s | $+65,0\%$ |
| **Write (MB/s)** | 12.400 MB/s | 23.560 MB/s | $+90,0\%$ |
| **Latency (Nanosegundos)** | 48 ns | 44 ns | $+8,3\%$ (reducción) |

### Pregunta de reflexión
> *A la vista de los resultados y aplicando la fórmula de mejora, ¿crees que la latencia ha mejorado al mismo ritmo que la velocidad de lectura? Justifica tu respuesta basándote en el concepto de CAS Latency (CL) visto en clase.*

**Respuesta:**  
No, la latencia mejora a un ritmo muy inferior ($8,3\%$) en comparación con la velocidad de lectura ($+65,0\%$) o escritura ($+90,0\%$). El *CAS Latency* (CL) indica el número de ciclos de reloj que transcurren desde que el controlador de memoria solicita un dato en una columna concreta hasta que ese dato está disponible en los pines de salida del módulo. Aunque la memoria DDR5 casi duplica la frecuencia de reloj respecto a DDR4 (lo que hace que cada ciclo de reloj dure prácticamente la mitad en nanosegundos), su latencia medida en ciclos también es mayor (pasando de CL16 en DDR4-3200 a CL30–CL36 en DDR5-6000). Como consecuencia, el tiempo físico inicial de respuesta apenas se reduce, mientras que el ancho de banda y la tasa de transferencia de datos aumentan de forma muy notable.

---

## Ejercicio 2: El impacto del "Dual Channel" y la Frecuencia

Comparativa de rendimiento tomando como referencia un procesador **Intel Core i5-12400** en la base de datos de **UserBenchmark**:

* **Configuración 1:** 1 módulo de 16GB (Single Channel) a 2400MHz.
* **Configuración 2:** 2 módulos de 8GB (Dual Channel) a 3600MHz.

| Métrica | Config. 1 (Single / Lenta) | Config. 2 (Dual / Rápida) | % de Mejora |
| :--- | :---: | :---: | :---: |
| **RAM Score** | $55,0\%$ | $104,5\%$ | $+90,0\%$ |
| **Memory Bandwidth (GB/s)** | 17,2 GB/s | 48,5 GB/s | $+182,0\%$ |
| **Precio estimado (actual)** | 34 € | 42 € | $+23,5\%$ (+8 €) |

### Pregunta de reflexión
> *Según la teoría analizada, si instalamos dos módulos de distintas velocidades (ej. uno de 2400MHz y otro de 3600MHz), ¿qué ocurriría con el rendimiento global del sistema según el fabricante?*

**Respuesta:**  
Si se instalan dos módulos de memoria con velocidades nominales diferentes (por ejemplo, uno de $2400\text{ MHz}$ y otro de $3600\text{ MHz}$), el controlador de memoria de la placa base ajustará automáticamente ambos módulos para que funcionen a la velocidad del módulo más lento con el fin de garantizar la estabilidad del sistema en *Dual Channel*. En este caso, todo el subsistema de memoria operaría a **$2400\text{ MHz}$**, desaprovechando el potencial superior del módulo de $3600\text{ MHz}$ y limitando el rendimiento global.

---

## Ejercicio 3: Auditoría de Memoria con CPU-Z

### 1. Tabla de Datos (Tu PC: Acer TravelMate P216-51-TCO)

| Parámetro en CPU-Z | Valor Detectado | Referencia Teórica (PDF) |
| :--- | :---: | :--- |
| **Type (Tipo de memoria)** | **DDR4** (SO-DIMM 260 pines) | DDR3 o DDR4 |
| **Channel # (Canales activos)** | **1 x 64-bit (Single Channel)** | Single o Dual Channel |
| **Uncore / DRAM Frequency** | **1596,1 MHz** (DRAM) / **2494,0 MHz** (Uncore) | Frecuencia de trabajo actual |
| **CAS# Latency (CL)** | **22.0 clocks** (`22-22-22-52`) | Latencia detectada |
| **Rank (En pestaña SPD)** | **Single (1Rx16)** | Organización de los chips |
| **Voltage (En pestaña SPD)** | **1.20 V** | Voltaje de operación (ej. 1.2V) |

### 2. Cálculo de diferencia de frecuencia
El módulo instalado de fábrica es **DDR4-3200**, cuya frecuencia real de reloj es de $1600\text{ MHz}$. CPU-Z marca **$1596,1\text{ MHz}$** debido a la ligera variación habitual del reloj base del sistema ($99,76\text{ MHz}$):

$$\text{Diferencia (\%)} = \frac{1600 - 1596,1}{1600} \times 100 = 0,24\%$$

La diferencia es mínima ($0,24\%$), lo que confirma que el módulo está trabajando correctamente a su frecuencia máxima estándar JEDEC sin limitaciones.

### 3. Análisis de Rendimiento y Configuración

* **A. El misterio de la frecuencia (DDR):**  
  Si una memoria es DDR4-2133MHz, CPU-Z muestra una cifra cercana a $1066\text{ MHz}$ (y en este equipo DDR4-3200 muestra unos $1600\text{ MHz}$) porque **DDR** significa *Double Data Rate* (doble tasa de transferencia). Esta tecnología transfiere datos dos veces por cada ciclo de reloj (tanto en el flanco de subida como en el de bajada de la señal). CPU-Z muestra la frecuencia real del reloj físico ($1066,6\text{ MHz}$), mientras que los fabricantes expresan la velocidad efectiva multiplicada por dos ($2133\text{ MT/s}$).

* **B. Verificación de "Matched Pairs":**  
  En la pestaña SPD de este equipo, el **Slot #1** muestra un módulo único de 8 GB DDR4-3200 CL22 (1.20 V) y el **Slot #2** aparece libre para una futura ampliación. Si se instalaran dos módulos diferentes (por ejemplo, uno de $1600\text{ MHz}$ y otro de $2133\text{ MHz}$), el sistema haría funcionar ambos módulos a **$1600\text{ MHz}$**, igualándose siempre a la frecuencia del módulo más lento.

* **C. Optimización Física:**  
  En una placa base de sobremesa con 4 ranuras como la mostrada en el manual (dos ranuras negras y dos rojas alternas), los 2 módulos deben instalarse en **ranuras alternas del mismo color** (habitualmente las ranuras A2 y B2) para activar el modo *Dual Channel*. El código de colores facilita identificar visualmente qué pares de ranuras pertenecen a canales emparejados (*matched pairs*) para duplicar el ancho de banda disponible.

---

## Ejercicio 4: Prueba de Benchmark (CPU-Z)

Resultados obtenidos en la pestaña **Bench** de CPU-Z para el **13th Gen Intel Core i3-1315U** frente a un procesador de referencia en la web de validación de CPU-Z:

* **CPU Single Thread (Tu PC):** `662 puntos`
* **CPU Multi Thread (Tu PC):** `2.315 puntos`
* **Procesador de referencia (Intel Core i7-11370H):** `655 puntos` (Single Thread) / `2.310 puntos` (Multi Thread)

### Comentario de resultados
En rendimiento **monohilo (Single Thread)**, el Intel Core i3-1315U logra una puntuación muy competitiva gracias a la arquitectura Raptor Lake de sus 2 núcleos de rendimiento (*P-cores*), capaces de alcanzar frecuencias turbo de hasta $4,5\text{ GHz}$, superando ligeramente a un procesador de gama alta de 11.ª generación como el i7-11370H. En **multihilo (Multi Thread)**, la combinación híbrida de 2 *P-cores* y 4 *E-cores* (8 hilos en total) le permite igualar el rendimiento multinúcleo de procesadores anteriores de 4 núcleos y 8 hilos con un consumo energético mucho más reducido ($15\text{ W}$). Sin embargo, al disponer únicamente de **8 GB de RAM en Single Channel** (con apenas $1,04\text{ GB}$ de memoria física disponible en reposo), el equipo puede sufrir cuellos de botella en tareas exigentes de multitarea si no se amplía con un segundo módulo de 8 GB.

---

## Ejercicio 5: Latencias — El dilema del técnico

**Datos de partida:**
* **Kit A:** 32 GB DDR4 a $3200\text{ MHz}$ con latencia **CL16**.
* **Kit B:** 32 GB DDR4 a $4000\text{ MHz}$ con latencia **CL20**.

### 1. Cálculo del tiempo real de acceso (Latencia real en nanosegundos)

Aplicando la fórmula de latencia real:

$$\text{Latencia real (ns)} = \frac{\text{Ciclos de CL} \times 2000}{\text{Frecuencia (MHz)}}$$

* **Para el Kit A (3200 MHz CL16):**
  $$\text{Latencia real}_A = \frac{16 \times 2000}{3200} = \frac{32000}{3200} = 10\text{ ns}$$

* **Para el Kit B (4000 MHz CL20):**
  $$\text{Latencia real}_B = \frac{20 \times 2000}{4000} = \frac{40000}{4000} = 10\text{ ns}$$

**Conclusión:** **Ambos kits tardan exactamente el mismo tiempo físico ($10\text{ ns}$)** en responder a la primera petición. Aunque el Kit B necesita 4 ciclos más de espera (20 frente a 16), su frecuencia es un $25\%$ superior, por lo que cada ciclo dura menos tiempo y compensa la diferencia.

### 2. Razonamiento técnico
Elegiría el **Kit B (4000 MHz CL20)**. Dado que ambos kits ofrecen exactamente el mismo tiempo de respuesta inicial ($10\text{ ns}$), el Kit B no penaliza las peticiones aleatorias de datos, pero aporta un **$25\%$ más de ancho de banda** ($32\text{ GB/s}$ frente a $25,6\text{ GB/s}$ por canal) para mover grandes volúmenes de información de manera continua. Al tener el mismo precio, el Kit B es técnicamente superior.

### 3. El factor procesador
Si se instalan ambos kits en un procesador cuyo controlador de memoria está limitado de forma nativa a **$2933\text{ MHz}$** y no se activan los perfiles XMP/EXPO en la BIOS, ambos kits arrancarán utilizando los perfiles estándar JEDEC de fábrica y funcionarán exactamente a **$2933\text{ MHz}$**. En la práctica, ambos ofrecerán el mismo rendimiento y se estará desaprovechando la frecuencia extra del Kit B.

---

## Ejercicio 6: Prompt "Tutor de IA" (Simulación de Entrenamiento Técnico)

A continuación se incluye la sesión práctica realizada siguiendo las reglas del *prompt* del Ingeniero Jefe de Soporte Técnico:

> **Ingeniero Jefe (IA):** ¡Bienvenido al equipo! Soy el Ingeniero Jefe del área de integración y soporte. Aquí no venimos a memorizar teoría, sino a montar equipos fiables y rápidos. Primer caso real en el taller: un cliente nos pide una estación de trabajo para edición de vídeo con **32 GB de RAM**. Al revisar las licencias disponibles en almacén, ves copias de **Windows 10 Home (32 bits)** y **Windows 10 Pro (64 bits)**. ¿Qué sistema operativo debemos instalar y por qué?
> * **A)** Windows 10 Home (32 bits), porque consume menos recursos en segundo plano.
> * **B)** Windows 10 Pro (64 bits), porque los sistemas operativos de 32 bits están limitados a un máximo de 4 GB de RAM.
> * **C)** Cualquiera de los dos, porque la cantidad máxima de RAM depende únicamente de la placa base.

**Respuesta del Alumno:** **B**.

> **Ingeniero Jefe (IA):** ¡Exacto! Buena vista. Un sistema operativo de 32 bits solo puede direccionar hasta $4\text{ GB}$ de memoria ($2^{32}$ direcciones), dejando los otros $28\text{ GB}$ completamente inutilizados. Con Windows 10 Pro de 64 bits podemos gestionar hasta $2\text{ TB}$ de RAM sin problemas.  
> Subimos el nivel. Ahora tienes sobre la mesa una placa base con **4 ranuras DIMM** codificadas por colores (negro, rojo, negro, rojo) y un kit de **16 GB (2x8 GB) DDR4-3200** con disipadores de aluminio. ¿Cómo debes instalar los módulos para obtener el máximo ancho de banda?
> * **A)** En las dos primeras ranuras contiguas (negro y rojo) empezando desde el procesador.
> * **B)** En dos ranuras alternas del mismo color (por ejemplo, las dos rojas) para activar la arquitectura Dual Channel.
> * **C)** En cualquier par de ranuras, ya que los módulos actuales activan el Dual Channel por software.

**Respuesta del Alumno:** **B**.

> **Ingeniero Jefe (IA):** ¡Impecable! Al colocarlos en ranuras emparejadas del mismo color (*matched pairs*), activamos el **Dual Channel**, duplicando el ancho de banda disponible entre la RAM y el controlador de memoria.  
> Última prueba antes de dar por bueno el montaje: arrancas el equipo, entras en la BIOS o en CPU-Z y ves que las memorias DDR4-3200 están funcionando a una velocidad efectiva de **$2133\text{ MHz}$** ($1066\text{ MHz}$ reales). ¿Qué tienes que hacer para que trabajen a su velocidad anunciada?
> * **A)** Cambiar los módulos de ranura porque las ranuras rojas limitan la frecuencia.
> * **B)** Activar el perfil **XMP (Extreme Memory Profile)** en la BIOS para aplicar automáticamente la frecuencia, latencias y voltaje probados por el fabricante.
> * **C)** No hacer nada, es un error visual de la BIOS y ya están funcionando a 3200 MHz.

**Respuesta del Alumno:** **B**.

> **Ingeniero Jefe (IA):** ¡Trabajo cerrado! Sin activar el perfil XMP 2.0 en la BIOS, la placa base arranca por seguridad a la velocidad estándar base JEDEC ($2133\text{ MHz}$). Al habilitar XMP, la memoria ajusta su frecuencia y voltaje (por ejemplo, de $1,20\text{ V}$ a $1,35\text{ V}$) de forma segura. Prueba superada, técnico.