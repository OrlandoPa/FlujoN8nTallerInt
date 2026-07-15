# Sistema de Automatización de Citas y Recordatorios (n8n)

Este repositorio contiene los flujos de trabajo (*workflows*) desarrollados en la plataforma **n8n** para la automatización e integración de la gestión de citas de una clínica odontológica.

Los flujos están diseñados para conectarse con **Chatwoot** (vía WhatsApp) para interactuar con los pacientes, **PostgreSQL** para la persistencia de datos (mensajes, estados de citas, datos de pacientes) y **Google Calendar** para la programación física de las citas.

---

## 📋 Flujos de Trabajo Incluidos

### 1. Gestión Inteligente de Citas por WhatsApp
* **Archivo:** `FlujoWhatsAppN8N.json`
* **Descripción:** Este flujo controla la interfaz conversacional de cara al paciente a través de WhatsApp (integrado con Chatwoot). Clasifica los mensajes entrantes y delega las respuestas a agentes especializados impulsados por Inteligencia Artificial (Google Gemini).
* **Funciones Clave:**
  * **Verificación de Seguridad:** Filtra y bloquea payloads maliciosos de entrada (inyecciones de prompt o textos excesivamente largos).
  * **Clasificador de Intenciones:** Determina si el paciente desea agendar una cita, tiene una consulta frecuente (FAQ), requiere soporte humano directo, o si no se trata de una cita.
  * **Agentes de IA:**
    * **Agente Maestro Citas:** Se encarga de buscar espacios libres, crear, actualizar o cancelar eventos interactuando con las herramientas de *Google Calendar*.
    * **Agente FAQ:** Resuelve dudas sobre tratamientos, especialidades y métodos de pago sin dar presupuestos cerrados, redirigiendo al usuario a agendar una cita de evaluación.
    * **Agente de Citas y Procesos:** Acompaña al paciente en el flujo conversacional estructurado para la recopilación de datos.
    * **Agente de Productos:** Responde consultas relacionadas con productos dentales de la clínica.
  * **Base de Datos:** Persiste los datos de pacientes y registros de citas.
  * **Notificaciones:** Envía correos de alerta por Gmail al administrador en caso de error crítico o solicitud expresa de comunicación humana.

### 2. Recordatorios Automáticos de Citas
* **Archivo:** `FlujoRecordatoriosAutomaticosN8N.json`
* **Descripción:** Automatiza el envío diario de recordatorios a los pacientes cuyas citas están programadas a futuro, garantizando la asistencia y reduciendo el ausentismo en la clínica.
* **Funciones Clave:**
  * **Disparador Programado (*Schedule Trigger*):** Ejecuta la verificación de forma periódica dentro de los horarios de atención de la clínica.
  * **Consulta SQL Dinámica:** Busca en la base de datos de PostgreSQL las citas programadas con una anticipación configurada (e.g. 25 horas) que se encuentren en estado `AGENDADA` o `REPROGRAMADA` y que aún no hayan recibido recordatorio.
  * **Filtro de Estado:** Verifica que los recordatorios se envíen solo a pacientes con correos y números telefónicos válidos.
  * **Envío y Actualización:** Envía el recordatorio correspondiente y actualiza la fila de la cita en PostgreSQL (`recordatorio_enviado = true`) para evitar duplicidad de envíos.

---

## 🐳 Despliegue Local con Docker

Para correr n8n localmente utilizando la imagen específica de desarrollo `nightly-038d2ca` y conectarlo a una base de datos PostgreSQL externa, utiliza el siguiente comando:

```bash
docker run -d \
  --name n8n \
  -p 5678:5678 \
  -e DB_TYPE=postgresdb \
  -e DB_POSTGRESDB_HOST=tu_host_postgres \
  -e DB_POSTGRESDB_PORT=5432 \
  -e DB_POSTGRESDB_DATABASE=n8ndb \
  -e DB_POSTGRESDB_USER=n8nuser \
  -e DB_POSTGRESDB_PASSWORD=tu_password_seguro \
  -e N8N_ENCRYPTION_KEY=clave_de_encriptacion_aqui \
  n8nio/n8n:nightly-038d2ca
```

---

## 🚀 Despliegue en Render (Render Blueprints)

Para desplegar la infraestructura de n8n en Render (el servicio web junto con la base de datos PostgreSQL 18 dedicada), **solo es necesario el archivo `render.yaml`** en tu repositorio de GitHub. 

### Servicios Creados por el Blueprint:
1. **`n8n-db`**: Base de datos de PostgreSQL 18 gestionada y hospedada en Render.
2. **`n8n-service`**: Servicio Web de n8n basado en la imagen Docker oficial (`nightly-038d2ca`), configurado para conectarse internamente a `n8n-db`.

### Instrucciones de Despliegue:

1. Asegúrate de tener este repositorio con el archivo `render.yaml` en tu cuenta de **GitHub**.
2. Ingresa a tu panel de **Render**.
3. Haz clic en **New** (Nuevo) y selecciona **Blueprint**.
4. Conecta tu repositorio de GitHub. Render detectará automáticamente el archivo `render.yaml`.
5. Haz clic en **Apply** (Aplicar) para iniciar el aprovisionamiento de la base de datos y de n8n.
6. Una vez completado el despliegue:
   * Abre la URL pública provista por Render para tu servicio de n8n.
   * Crea o ingresa con tu cuenta de propietario en la plataforma.
   * Importa manualmente los archivos JSON de los flujos (`FlujoWhatsAppN8N.json` y `FlujoRecordatoriosAutomaticosN8N.json`) desde la interfaz de n8n.
   * Configura las credenciales necesarias (Google Calendar, Gmail, PostgreSQL, Chatwoot) dentro de cada flujo importado para iniciar las pruebas y ajustes del sistema.
