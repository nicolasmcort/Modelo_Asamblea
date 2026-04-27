# Modelo Asamblea

Este documento detalla la mecánica, lógica de eventos y estructura de datos utilizada en el modelo de simulación de eventos discretos para la asamblea estudiantil.

---

## ⚙️ 1. Mecánica General del Modelo

El modelo utiliza un enfoque de **Simulación de Eventos Discretos (DES)** que integra dos procesos paralelos pero interconectados:

1.  **Proceso de Arribos (Entidades):** Modela físicamente cuándo llegan las personas al recinto y cuánto tiempo se quedan.
2.  **Proceso de Agenda (Servidor):** Modela el uso del micrófono y el proceso de votación de las propuestas.

### El Reloj Dual
Una característica clave es que el modelo maneja dos relojes:
-   **Reloj de Tiempo Real (Columna D):** Cuándo entra la persona físicamente al recinto.
-   **Reloj de Agenda (Columna I):** En qué minuto de la discusión se encuentra la asamblea.
*La interacción entre ambos ocurre cuando se verifica cuántas personas han llegado antes del tiempo de votación actual pero aún no han salido (Quórum).*

---

## 📂 2. Análisis Detallado de Columnas (Hoja: Simulación)

Cada fila representa a un asistente y su participación. Aquí está la mecánica de cada columna:

| Col | Nombre | Lógica Funcional |
| :--- | :--- | :--- |
| **A** | **N° Persona** | Contador de personas que han intentado entrar al sistema. |
| **B** | **Activo** | **Condición lógitica:** Solo es `1` si la persona llegó antes de los 240 min, si la agenda no ha terminado y si el aforo (100) permite su entrada. Si es `0`, el resto de la fila queda vacía. |
| **C** | **Δt (Inter-llegada)** | Tiempo que pasó desde que llegó la persona anterior. Sigue una **distribución Exponencial Negativa**. |
| **D** | **Llegada** | Tiempo acumulado de llegada. Si una persona llega en el minuto 241, el sistema la bloquea (col B). |
| **E** | **Servicio (Intervención)** | Tiempo que la persona hablará ante el micrófono. Sigue una **distribución Normal**. |
| **F** | **Inicio Intervención** | Momento en el reloj de agenda donde la persona toma el micrófono. Es igual al `T.Acum` de la persona anterior (Cola FIFO). |
| **G** | **Fin Intervención** | `Inicio + Servicio`. Momento en que deja de hablar la persona. |
| **H** | **T. Voto** | Tiempo aleatorio entre 2 y 5 minutos dedicado a votar físicamente lo propuesto. |
| **I** | **T. Acum (Reloj Agenda)** | `Fin Intervención + T. Voto`. Este es el reloj que define si la asamblea se está extendiendo demasiado. |
| **J** | **Permanencia** | Tiempo que la persona decide quedarse en la asamblea. Sigue una **distribución Exponencial**. |
| **K** | **Salida** | `Llegada + Permanencia`. Crucial para calcular el aforo dinámico. |
| **L** | **Ocupación** | **Mecánica:** Cuenta cuántas personas tienen un `Tiempo de Llegada <= D` y un `Tiempo de Salida >= D`. Esto simula el aforo real al momento de la votación. |
| **M** | **¿Quórum?** | Verifica si la **Ocupación >= Quórum Mínimo** definido en parámetros. |
| **N** | **Resultado** | Lógica de decisión: Si hay quórum, lanza un aleatorio 50/50. Si no, anula la votación. |

---

## 🗓️ 3. Los Eventos y sus Estados

El sistema transita por los siguientes estados:

1.  **Evento de Llegada:** 
    - Se genera el tiempo de llegada.
    - Se verifica el aforo: `(Cuentas de llegada anteriores < 100)`.
2.  **Evento de Ocupación del Servidor (Micrófono):**
    - Se le entrega el micrófono a la persona en cuanto se termina la votación.
    - Si la persona llega al recinto en el min 10, pero la agenda va en el min 50, la persona debe esperar 40 minutos para hablar.
3.  **Evento de Votación:**
    - Ocurre inmediatamente después de la intervención.
    - Consume tiempo del reloj de agenda (recurso fijo).
4.  **Evento de Salida:**
    - La persona sale del sistema según su tiempo de permanencia, permitiendo que nuevas personas entren si el aforo estaba lleno.

---

## 📏 4. Restricciones y Paradas

-   **Aforo:** Capacidad máxima de 100 personas. El modelo bloquea la entrada si el conteo dinámico (Llegadas vs Salidas) llega a 100.
-   **Tiempo Límite (240 min):** 
    - Si el `Reloj de Llegada > 240`, no entran más personas.
    - Si el `Reloj de Agenda > 240`, se cancelan las intervenciones pendientes.

---

## 🏆 5. Índice de Eficiencia (Cálculo)

Para medir qué tan "buena" fue la asamblea, se calcula un índice de **0 a 1**:
-   **Velocidad (50%):** Compara el tiempo real por decisión frente a un ritmo esperado.
-   **Quórum (30%):** Resalta las asambleas donde la mayoría de decisiones se tomaron con quórum válido.
-   **Aprobación (20%):** Refleja la capacidad de la asamblea para llegar a consensos (decisiones aprobadas).

---

## 🚀 Instrucciones de Simulación
1.  **Recálculo:** Presiona **F9** para ejecutar una asamblea completa.
2.  **Configuración:** Cambia la media de llegadas o el quórum en la hoja de `Parámetros` para ver cómo cambia la probabilidad de éxito.
