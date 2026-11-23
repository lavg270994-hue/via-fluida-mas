# Vía Fluida — Sistema Multiagente de Movilidad Urbana

Proyecto académico · Inteligencia Artificial · Universidad Anáhuac Puebla · Equipo 3
Integrantes: 
Bello León Cristian 00608910
Carrizosa Dávila Carlos Andrés 00616058
Córdova Flores Alejandro Oliver 00233775
Gutiérrez Avalos Nallely Gabriela 00629159
Vera Guevara Luis Alberto 00456386

Vía Fluida es un prototipo de sistema multiagente (MAS) que analiza una red vial sintética, propone ajustes de semáforos y distribuye vehículos entre rutas alternativas para reducir la congestión. Utiliza CrewAI para coordinar cuatro agentes especializados y herramientas deterministas de Python para realizar los cálculos. La propuesta se complementa con una interfaz de supervisión diseñada en Stitch y Figma Make.

> Estado del proyecto: prototipo académico funcional en Google Colab, con datos sintéticos. La interfaz publicada es navegable, pero todavía no se conecta directamente al motor CrewAI ni controla semáforos reales.

## 1. Problema y objetivo

Ante una salida masiva de vehículos —por ejemplo, después de un partido—, la lluvia y un accidente pueden provocar saturación de intersecciones. Además, recomendar una sola ruta alternativa a todos los conductores puede crear un nuevo cuello de botella, conocido aquí como *efecto manada*.

El objetivo de Vía Fluida es apoyar a un operador de movilidad con un diagnóstico de congestión, un plan semafórico coordinado, recomendaciones de reparto de rutas y una validación de las metas del proyecto antes de tomar decisiones.

## 2. Arquitectura del sistema

```text
Operador / escenario sintético
            |
            v
    Datos de sensores (Python)
            |
            v
      CrewAI: Process.sequential
            |
            +--> T1. Analista de Percepción Vial
            |         Diagnóstico y TTI
            |
            +--> T2. Optimizador Semafórico Adaptativo
            |         Webster, demoras y ola verde
            |
            +--> T3. Asesor de Rutas Alternas y Anti-Manada
            |         Reparto de demanda y tiempos BPR
            |
            +--> T4. Supervisor y Sintetizador de Movilidad
                      Validación PEAS y reporte
            |
            v
   Reporte Markdown + registro de ejecución
            |
            v
   Operador revisa la propuesta
```

Las tareas se ejecutan secuencialmente. T2 y T3 toman como contexto el diagnóstico de T1; T4 recibe los resultados de las tres tareas anteriores. El Supervisor tiene permiso de delegación para casos de incumplimiento o contradicción, aunque la corrida documentada no requirió delegar.

### Agentes y responsabilidades

| Agente | Responsabilidad | Herramientas principales | ¿Puede delegar? |
|---|---|---|---|
| Analista de Percepción Vial | Leer sensores, calcular TTI y clasificar la congestión | `leer_sensores_interseccion`, `calcular_indice_congestion` | No |
| Optimizador Semafórico Adaptativo | Proponer ciclos, verdes, offset de ola verde y estimar demoras | `calcular_plan_semaforico`, `calcular_offset_ola_verde`, `resumir_demora_ponderada` | No |
| Asesor de Rutas Alternas y Anti-Manada | Simular rutas y repartir la demanda sin saturar una sola alternativa | `simular_asignacion_rutas` | No |
| Supervisor y Sintetizador de Movilidad | Revisar consistencia, comprobar la meta PEAS y generar el informe | `validar_contra_peas` | Sí |

El prototipo incluye siete herramientas de cálculo y cuatro tareas de CrewAI (`t_diagnostico`, `t_semaforos`, `t_rutas` y `t_reporte`).

## 3. Marco PEAS

| Elemento | Aplicación en Vía Fluida |
|---|---|
| P — Desempeño | Reducir al menos 15 % el tiempo medio de espera y el tiempo de viaje; observar demoras, colas y consistencia de las recomendaciones. |
| E — Entorno | Red vial urbana sintética de Puebla, con intersecciones, lluvia, accidentes y demanda variable; entorno dinámico y parcialmente observable. |
| A — Actuadores | Propuestas de ciclos y tiempos de verde, desfases de ola verde, distribución de rutas, mensajes para paneles y alertas. En esta versión son recomendaciones simuladas. |
| S — Sensores | Conteos por acceso, colas, velocidades, tiempos de espera, flujos y reportes de incidentes. Los datos actuales se definen dentro del notebook. |

La herramienta `validar_contra_peas` calcula si se cumple la reducción mínima del 15 % en las dos métricas principales.

## 4. Escenario de prueba reproducible

La ejecución incluida en el notebook utiliza los siguientes datos:

| Parámetro | Valor |
|---|---|
| Evento | Salida de partido en estadio + choque en el eje principal |
| Momento | Sábado, 19:30 |
| Clima | Lluvia moderada |
| Intersecciones | `INT-A`, `INT-B`, `INT-C`, `INT-D` |
| Origen y destino | `INT-A` → `PERIF` |
| Demanda | 2,400 vehículos por hora |
| Tramo con incidente | `INT-A`–`INT-B` |
| Pérdida de capacidad del tramo | 50 % |
| Meta PEAS | Reducción mínima del 15 % en espera y viaje |

El escenario está definido en las estructuras `SENSORES`, `TRAMOS` y `ENTRADAS` de `via_fluida_crew.ipynb`. No se necesitan archivos externos de sensores para la corrida predeterminada.

### Resultados del escenario sintético

| Indicador | Referencia | Con Vía Fluida | Resultado estimado |
|---|---:|---:|---:|
| Demora media ponderada | 94.1 s | 34.7 s | −63.1 % |
| Tiempo medio de viaje | 11.6 min | 6.1 min | −47.4 % |
| Efecto manada (todos por la alterna) | 16.0 min | — | Peor que no intervenir |

El reparto recomendado fue 60 % por la ruta principal (`INT-A → INT-B → PERIF`) y 40 % por la alternativa (`INT-A → INT-C → INT-D → PERIF`). La corrida guardada registra 11 invocaciones a herramientas y 0 delegaciones. Las cifras son resultados de un modelo simplificado, no mediciones reales ni ahorros comprobados en las calles de Puebla.

## 5. Archivos del proyecto

```text
via-fluida-mas/
├── README.md                 # Documentación del proyecto
├── via_fluida_crew.ipynb     # Código, datos, agentes, tareas y demostración
├── requirements.txt         # Dependencias para uso local (agregar al repositorio)
└── .env.example             # Variables de ejemplo, sin secretos (agregar al repositorio)
```

Al ejecutar el notebook se generan archivos de salida en el entorno de ejecución:

- `reporte_via_fluida.md`: informe final del Supervisor.
- `log_via_fluida.txt`: registro de tareas, herramientas y resultados de ejecución.

En Google Colab, estos archivos pueden desaparecer al reiniciar el entorno; descárgalos si deseas conservarlos.

## 6. Prerrequisitos

- Cuenta de Google para ejecutar el notebook en Google Colab (ruta recomendada).
- Python 3.11 o 3.12 y `pip` para ejecución local, de acuerdo con los prerrequisitos solicitados en la actividad final.
- Biblioteca `crewai` y un entorno de Jupyter si se ejecuta fuera de Colab.
- Conexión a internet para instalar dependencias. En el modo `simulado`, el prototipo no necesita una clave API de un proveedor de LLM.

El notebook incluye una primera celda que ejecuta `!pip -q install crewai`. Para una instalación local también puede utilizarse el archivo `requirements.txt` una vez añadido al repositorio.

## 7. Instalación y ejecución

### Opción A. Google Colab (recomendada para la demostración)

1. Abre el archivo [`via_fluida_crew.ipynb`](./via_fluida_crew.ipynb) en GitHub y selecciona `Open in Colab`.
2. En Colab, entra a **Entorno de ejecución → Ejecutar todas**.
3. Deja `os.environ["VIAFLUIDA_LLM"] = "simulado"` en la celda de configuración (valor predeterminado).
4. Revisa las salidas de las cuatro tareas y el reporte final.
5. Consulta el registro `log_via_fluida.txt` generado durante la ejecución.

El código ejecuta el Crew mediante `await via_fluida_crew.kickoff_async(inputs=ENTRADAS)` dentro del notebook, por lo que no requiere un servidor web para esta prueba.

### Opción B. Ejecución local con Jupyter

Clona el repositorio:

```bash
git clone https://github.com/lavg270994-hue/via-fluida-mas.git
cd via-fluida-mas
```

En macOS o Linux, crea y activa un entorno virtual con una versión compatible de Python:

```bash
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install crewai jupyterlab ipython
jupyter lab
```

Abre `via_fluida_crew.ipynb` desde JupyterLab y ejecuta las celdas en orden. Si ya agregaste `requirements.txt` al repositorio, puedes instalar con `python -m pip install -r requirements.txt` en lugar del comando de instalación de paquetes anterior.

En Windows, la activación del entorno cambia a `.venv\Scripts\activate`.

## 8. Modelos de lenguaje y variables de entorno

El notebook admite estos valores en la variable `VIAFLUIDA_LLM`:

| Valor | Uso | Requisitos adicionales |
|---|---|---|
| `simulado` | Modo de demostración reproducible con `BaseLLM` simulado; predeterminado | No requiere clave API |
| `openai` | Modelo de OpenAI mediante CrewAI | `OPENAI_API_KEY` válida |
| `gemini` | Modelo de Gemini mediante CrewAI | `GEMINI_API_KEY` válida |
| `ollama` | Modelo local `llama3.2` mediante Ollama | Servidor Ollama y modelo instalados |

Para seleccionar el modo, modifica la línea de la celda de configuración:

```python
os.environ["VIAFLUIDA_LLM"] = "simulado"
```

El archivo `.env.example` es una plantilla de referencia; el notebook publicado no lo carga automáticamente. Si usas servicios externos, configura tus claves en un entorno seguro (por ejemplo, variables de entorno o secretos de Colab) y nunca las escribas en un notebook que subirás a GitHub.

Para utilizar Ollama localmente, se requiere tener el servicio activo y descargar el modelo correspondiente; la configuración del notebook utiliza `http://localhost:11434`. Ese `localhost` se refiere a la máquina donde corre Python, por lo que no basta con abrir Ollama en tu Mac si el notebook se ejecuta en un servidor de Colab.

## 9. Interfaz gráfica y control humano

Se diseñó un dashboard B2B para un operador de movilidad con cuatro vistas:

1. Escenario: configuración del evento, clima, demanda, incidente y red vial.
2. Ejecución del Crew: progreso de tareas, agentes y registro de herramientas.
3. Resultados y decisión: KPIs, planes semafóricos, rutas y controles para aprobar, ajustar o rechazar.
4. Bitácora: historial visual y trazabilidad de las ejecuciones y decisiones.

Prototipo interactivo: https://polish-rover-01259432.figma.site/

La maqueta se creó con Stitch y se desarrolló como beta navegable en Figma Make. Simula el flujo de interacción y aprobación humana (*Human-in-the-Loop*), pero no está conectada de forma operativa al notebook de CrewAI. Sus controles de aplicación y telemetría son demostrativos; no se transmiten órdenes a dispositivos viales reales.

## 10. Validaciones y seguridad

- Cálculos en herramientas de Python, separados de la generación de texto del LLM.
- Comparación explícita de demora y viaje contra la meta PEAS de 15 %.
- Revisión de coherencia entre el plan semafórico y las rutas recomendadas.
- Posibilidad de delegación del Supervisor ante problemas, aunque la corrida incluida no muestra delegación porque la validación fue satisfactoria.
- Registro de pasos y salidas en archivos para supervisar la ejecución.
- Aprobación humana diseñada en la GUI; aún no existe una pasarela conectada a semáforos o paneles reales.

## 11. Limitaciones y trabajo futuro

El prototipo utiliza una red pequeña y datos sintéticos. Los tiempos y reducciones dependen de fórmulas y supuestos del escenario, no de mediciones en campo. Tampoco se ha validado una delegación correctiva en la corrida guardada, ni una ejecución desde la GUI hacia CrewAI.

Las siguientes etapas propuestas son: integrar la interfaz con una API FastAPI, transmitir eventos de los agentes mediante SSE o WebSockets, almacenar decisiones en una base de datos y conectar la simulación con SUMO para estudiar el movimiento individual de vehículos antes de cualquier piloto real.

## 12. Equipo

Universidad Anáhuac Puebla — Inteligencia Artificial — Equipo 3.

- Cristian Bello León
- Carlos Andrés Carrizosa Dávila
- Alejandro Oliver Córdova Flores
- Nallely Gabriela Gutiérrez Avalos
- Luis Alberto Vera Guevara

Profesor: Paulo Daniel Vázquez Mora.

## 13. Referencias principales

- CrewAI. Documentación oficial: https://docs.crewai.com/
- FastAPI. Documentación oficial: https://fastapi.tiangolo.com/
- Google Stitch: https://stitch.withgoogle.com/
- Figma: https://www.figma.com/
- SUMO: https://eclipse.dev/sumo/
- Russell, S. J., & Norvig, P. (2021). *Artificial intelligence: A modern approach* (4.ª ed.). Pearson.
- Wooldridge, M. (2009). *An introduction to multiagent systems* (2.ª ed.). Wiley.

---

Proyecto académico desarrollado con fines de aprendizaje y demostración. No utilizar los resultados para controlar tráfico real sin validación independiente, calibración y autorizaciones correspondientes.
