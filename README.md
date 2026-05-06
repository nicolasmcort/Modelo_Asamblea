# Modelo de Simulación de Asamblea Estudiantil

Este documento brinda una descripción detallada de la arquitectura, lógica de eventos y estructura de datos implementada en el modelo de simulación de eventos discretos (SED) diseñado para representar la dinámica de una asamblea estudiantil.

---

## 1. Fundamentos del Modelo

La simulación se fundamenta en un enfoque de **Simulación de Eventos Discretos (SED)**, integrando dos procesos interdependientes:

1.  **Dinámica de Asistencia (Entidades):** Modela el flujo de arribos y la permanencia física de los asistentes en el recinto.
2.  **Gestión de Agenda (Servidor):** Modela la secuencia de intervenciones (uso del micrófono) y los procesos de votación de propuestas.

### Sistema de Relojes Duales
Para una representación cercana a la realidad, el modelo opera bajo dos ejes temporales:
-   **Tiempo de Arribo ($T_{real}$):** Cronometra la entrada física de los asistentes.
-   **Tiempo de Agenda ($T_{agenda}$):** Cronometra el progreso de la asamblea y las discusiones.

La interacción entre ambos relojes es primordial para la validación del **Quórum**, la cual se verifica relizando una comparación entre la cantidad de asistentes presentes en el instante exacto de cada votación y el quórum mínimo establecido.

---

## 2. Definición del Diccionario de Datos (Hoja: Simulación)

Cada registro en la hoja Excel de simulación corresponde a un asistente individual. A continuación, se describe la lógica funcional de las variables principales:

| ID | Variable | Definición y Lógica Estocástica |
| :--- | :--- | :--- |
| **A** | **N° Persona** | Identificador único incremental de la entidad. |
| **B** | **Estado Activo** | Variable binaria condicionada por: $T_{arribo} \leq 240$, $T_{agenda}$ activo y disponibilidad de aforo ($N \leq 100$). |
| **C** | **$\Delta t$ Inter-llegada** | Tiempo entre arribos sucesivos. Sigue una **Distribución Exponencial Negativa**. |
| **D** | **Llegada** | Instante acumulado de entrada al sistema. |
| **E** | **Servicio (Intervención)** | Duración de la ponencia ante el micrófono. Sigue una **Distribución Normal**. |
| **F** | **Inicio Intervención** | Instante de inicio en el reloj de agenda. Sigue una lógica de cola FIFO (First-In, First-Out). |
| **G** | **Fin Intervención** | Tiempo de culminación de la palabra: $F + E$. |
| **H** | **Tiempo de Voto** | Duración del proceso de escrutinio (Uniforme entre 2 y 5 minutos). |
| **I** | **Acumulado Agenda** | Progreso total del tiempo de asamblea: $G + H$. |
| **J** | **Permanencia** | Tiempo de estancia decidido por el asistente. Sigue una **Distribución Exponencial**. |
| **K** | **Salida** | Instante de abandono del recinto: $D + J$. |
| **L** | **Aforo Dinámico** | Conteo de entidades activas donde $Llegada \leq T_{actual}$ y $Salida \geq T_{actual}$. |
| **M** | **Validación Quórum** | Verificación lógica: $Ocupación \geq Quórum_{min}$. |
| **N** | **Resultado** | Decisión basada en probabilidad 0.5 si existe Quórum; de lo contrario, se anula. |

---

## 3. Estructura de Eventos y Estados

La simulación transita por los siguientes estados discretos:

1.  **Evento de Arribo:**
    -   Generación de marca de tiempo y verificación de restricciones existentes de aforo.
2.  **Acceso al Recurso / Servidor (Micrófono):**
    -   Asignación del turno de palabra según la disponibilidad del servidor de agenda.
    -   Cálculo de tiempos de espera si $T_{arribo} < T_{agenda\_actual}$.
3.  **Proceso de Votación:**
    -   Ejecución después de una intervención con consumo del reloj de agenda.
4.  **Evento de Salida:**
    -   Liberación de espacio en el aforo dinámico, permitiendo el ingreso de nuevas entidades bloqueadas.

---

## 4. Restricciones Operativas y Criterios de Parada

El modelo está sujeto a límites físicos y temporales estrictos:

-   **Capacidad de Aforo:** Límite máximo de 100 asistentes simultáneos. Las entradas se suspenden si el conteo dinámico llega a alcanzar este valor.
-   **Ventana Temporal (240 min):**
    -   **Cierre de Puertas:** No se admiten nuevos arribos superados los 240 minutos de tiempo real.
    -   **Clausura de Sesión:** Se cancelan intervenciones programadas si el reloj de agenda excede el límite establecido.

---

## 5. Métrica de Desempeño: Índice de Eficiencia

La efectividad de la asamblea se cuantifica mediante una función ponderada (escala 0-1):

$$I_e = 0.50(V) + 0.30(Q) + 0.20(A)$$

Donde:
*   **V (Velocidad):** Relación entre el ritmo de decisión real y el objetivo.
*   **Q (Quórum):** Proporción de decisiones tomadas con validez legal.
*   **A (Aprobación):** Tasa de éxito en la generación de consensos.

---

## Guía de Operación

1.  **Ejecución de Ciclos:** Utilizar la tecla **F9** para realizar un recálculo manual y generar una nueva iteración de la asamblea.
2.  **Ajuste de Parámetros:** Las variables de entrada (tasas de llegada, medias de servicio, quórum requerido) se gestionan desde la pestaña `Parámetros`.

