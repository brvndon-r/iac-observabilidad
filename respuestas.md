# Respuestas — Laboratorio de Observabilidad

## Pregunta 1
¿Por qué necesitamos Loki además de Prometheus si ya tenemos `/metrics`?

Porque Prometheus está diseñado exclusivamente para métricas numéricas en el tiempo (por ejemplo CPU, memoria, requests por segundo) y no puede almacenar ni consultar texto libre, en cambio Loki está optimizado para logs: líneas de texto que describen eventos, errores y trazas de las aplicaciones, quiere decir que son complementarias ya que con Prometheus detectamos qué está mal (una métrica supera un umbral) y con Loki investigamos por qué ocurrió (revisando los logs del momento exacto del incidente).

## Pregunta 2
¿Qué ventaja aporta que las fuentes de datos de Grafana estén aprovisionadas como código y no creadas a mano?

Que al definir las fuentes de datos en archivos de configuración versionables, cualquier persona puede reproducir el entorno exacto ejecutando el comando docker compose up, sin depender de que alguien recuerde qué configuraciones hacer manualmente y esto elimina errores humanos, facilita el trabajo en equipo y garantiza que todos los entornos (desarrollo, producción) sean idénticos.

## Pregunta 3
El panel "CPU contenedor" y el panel "CPU host" pueden mostrar valores muy distintos. ¿Por qué? ¿Cuál usarías para alertar sobre una aplicación concreta?

El panel de CPU del host mide el uso total de todos los procesos del sistema operativo, mientras que el panel de CPU del contenedor mide únicamente los recursos consumidos por ese contenedor específico y los valores son distintos porque el host puede tener decenas de procesos corriendo simultáneamente. Para alertar se debe usar la métrica del contenedor ya que permite aislar el comportamiento de ese servicio sin el ruido del resto del sistema.

## Pregunta 4
¿Qué diferencia hay entre el evaluation interval y el pending period de una alarma?

El evaluation interval define cada cuánto tiempo Grafana evalúa la condición de la alarma (en este caso cada 10 segundos) y el pending period define cuánto tiempo debe mantenerse la condición verdadera de forma continua antes de que la alarma pase a estado Firing (en este caso 30 segundos). Esta diferencia es importante porque evita falsas alarmas por picos momentáneos porque si la métrica supera el umbral por un instante pero baja de inmediato, la alarma no se dispara.

---

## Instrucciones para validar el trabajo

### Requisitos previos
- Docker Desktop instalado y corriendo
- Git instalado

### Pasos para reproducir el entorno

1. Clonar el repositorio:# Respuestas — Laboratorio de Observabilidad

## Pregunta 1
¿Por qué necesitamos Loki además de Prometheus si ya tenemos `/metrics`?

Porque Prometheus está diseñado exclusivamente para métricas numéricas en el tiempo (por ejemplo CPU, memoria, requests por segundo) y no puede almacenar ni consultar texto libre, en cambio Loki está optimizado para logs: líneas de texto que describen eventos, errores y trazas de las aplicaciones, quiere decir que son complementarias ya que con Prometheus detectamos qué está mal (una métrica supera un umbral) y con Loki investigamos por qué ocurrió (revisando los logs del momento exacto del incidente).

## Pregunta 2
¿Qué ventaja aporta que las fuentes de datos de Grafana estén aprovisionadas como código y no creadas a mano?

Que al definir las fuentes de datos en archivos de configuración versionables, cualquier persona puede reproducir el entorno exacto ejecutando el comando docker compose up, sin depender de que alguien recuerde qué configuraciones hacer manualmente y esto elimina errores humanos, facilita el trabajo en equipo y garantiza que todos los entornos (desarrollo, producción) sean idénticos.

## Pregunta 3
El panel "CPU contenedor" y el panel "CPU host" pueden mostrar valores muy distintos. ¿Por qué? ¿Cuál usarías para alertar sobre una aplicación concreta?

El panel de CPU del host mide el uso total de todos los procesos del sistema operativo, mientras que el panel de CPU del contenedor mide únicamente los recursos consumidos por ese contenedor específico y los valores son distintos porque el host puede tener decenas de procesos corriendo simultáneamente. Para alertar se debe usar la métrica del contenedor ya que permite aislar el comportamiento de ese servicio sin el ruido del resto del sistema.

## Pregunta 4
¿Qué diferencia hay entre el evaluation interval y el pending period de una alarma?

El evaluation interval define cada cuánto tiempo Grafana evalúa la condición de la alarma (en este caso cada 10 segundos) y el pending period define cuánto tiempo debe mantenerse la condición verdadera de forma continua antes de que la alarma pase a estado Firing (en este caso 30 segundos). Esta diferencia es importante porque evita falsas alarmas por picos momentáneos porque si la métrica supera el umbral por un instante pero baja de inmediato, la alarma no se dispara.

---

## Instrucciones para validar el trabajo

### Requisitos previos
- Docker Desktop instalado y corriendo
- Git instalado

### Pasos para reproducir el entorno

1. Clonar el repositorio:

git clone https://github.com/brvndon-r/iac-observabilidad.git
cd iac-observabilidad

2. Levantar el stack:

docker compose up -d --build

3. Verificar que los contenedores están corriendo:

docker compose ps

Deben aparecer 6 contenedores en estado Up: lab-backend, lab-frontend, lab-prometheus, lab-loki, lab-alloy, lab-grafana.

4. Verificar los servicios en el navegador:

| Servicio         | URL                           | Credenciales  |
|------------------|-------------------------------|---------------|
| Frontend         | http://localhost:8080         | —             |
| Backend métricas | http://localhost:3001/metrics | —             |
| Prometheus       | http://localhost:9090         | —             |
| Grafana          | http://localhost:3000         | admin / admin |

5. En Grafana verificar:
   - Connections → Data sources: Prometheus y Loki en estado OK
   - Dashboards → Observabilidad: 4 paneles con datos
   - Alerting → Alert rules: regla "CPU backend > 50%" configurada

6. Para probar la alarma, generar carga abriendo en el navegador:

http://localhost:3001/load?seconds=60

La alarma debe pasar de Normal → Pending → Firing en menos de 40 segundos.

### Nota sobre compatibilidad con Windows
Los servicios node-exporter y cAdvisor fueron deshabilitados por incompatibilidad con Docker Desktop en Windows (requieren montaje directo del sistema de archivos del host). En un servidor Linux funcionarían correctamente y proveerían métricas de CPU del host y por contenedor.

### Reset total

docker compose down -v

