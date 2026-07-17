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

---

## 💬 Despliegue de Chatwoot y Traefik (VPS / Local)

Para conectar de manera gratuita y sin límites la API de WhatsApp Cloud (Meta) con n8n, se utiliza **Chatwoot** como plataforma omnicanal de soporte y pasarela. A continuación se detalla cómo desplegar Chatwoot junto con **Traefik** como proxy reverso con SSL automático en un servidor VPS.

### Archivos de Configuración en el Repositorio:
* **Traefik proxy:** [chatwoot-vps/traefik/docker-compose.yml](file:///c:/Users/lalop/Desktop/Nueva%20carpeta%20%282%29/FlujoN8nTallerInt/chatwoot-vps/traefik/docker-compose.yml)
* **Chatwoot compose:** [chatwoot-vps/chatwoot/docker-compose.yml](file:///c:/Users/lalop/Desktop/Nueva%20carpeta%20%282%29/FlujoN8nTallerInt/chatwoot-vps/chatwoot/docker-compose.yml)
* **Plantilla de variables (.env):** [chatwoot-vps/chatwoot/.env.example](file:///c:/Users/lalop/Desktop/Nueva%20carpeta%20%282%29/FlujoN8nTallerInt/chatwoot-vps/chatwoot/.env.example)

### Requisitos de Hardware mínimos:
* **CPU:** 2 núcleos o más.
* **RAM:** 4 GB de RAM mínimos (se recomiendan 8 GB para mejor desempeño en producción).
* **Almacenamiento:** 20 GB de disco SSD disponibles.

---

### Paso 1: Preparación del Servidor VPS

Actualiza tu servidor (Ubuntu/Debian) e instala la última versión estable de Docker y Docker Compose:

```bash
# Actualizar el sistema
sudo apt update && sudo apt upgrade -y

# Instalar dependencias requeridas
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y

# Agregar llave GPG oficial de Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Agregar el repositorio estable de Docker
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" -y

# Instalar Docker CE
sudo apt update && sudo apt install docker-ce docker-ce-cli containerd.io -y

# Habilitar Docker para iniciar con el sistema
sudo systemctl enable docker && sudo systemctl start docker

# Instalar Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

---

### Paso 2: Despliegue de Traefik (Proxy Reverso)

Traefik gestionará los certificados de seguridad SSL de Let's Encrypt de forma automática.

1. **Crear la red externa de Docker para Traefik:**
   ```bash
   docker network create traefik
   ```
2. **Crear carpetas y estructurar Traefik:**
   ```bash
   mkdir -p ~/traefik/letsencrypt
   cd ~/traefik
   ```
3. Copia el archivo [docker-compose.yml](file:///c:/Users/lalop/Desktop/Nueva%20carpeta%20%282%29/FlujoN8nTallerInt/chatwoot-vps/traefik/docker-compose.yml) de Traefik a este directorio y levanta el servicio reemplazando `${EMAIL}` con tu correo para Let's Encrypt:
   ```bash
   # Iniciar Traefik en segundo plano
   EMAIL="tu-correo@ejemplo.com" docker-compose up -d
   ```

---

### Paso 3: Configuración y Despliegue de Chatwoot

1. **Crear carpetas de Chatwoot:**
   ```bash
   mkdir -p ~/chatwoot
   cd ~/chatwoot
   ```
2. Copia los archivos del repositorio [docker-compose.yml](file:///c:/Users/lalop/Desktop/Nueva%20carpeta%20%282%29/FlujoN8nTallerInt/chatwoot-vps/chatwoot/docker-compose.yml) y [.env.example](file:///c:/Users/lalop/Desktop/Nueva%20carpeta%20%282%29/FlujoN8nTallerInt/chatwoot-vps/chatwoot/.env.example) a tu directorio.
3. Genera una cadena secreta segura para Chatwoot:
   ```bash
   openssl rand -hex 64
   ```
4. Renombra `.env.example` a `.env` (`mv .env.example .env`) y edita las siguientes variables obligatorias:
   * **`SECRET_KEY_BASE`**: Pega el resultado del comando anterior.
   * **`FRONTEND_URL`**: Tu dominio de acceso (e.g., `https://chatwoot.tudominio.com`).
   * **`ASSET_CDN_HOST`**: Coloca el mismo valor de tu `FRONTEND_URL`.
   * **`REDIS_PASSWORD`** y **`POSTGRES_PASSWORD`**: Define contraseñas seguras.
5. Edita el archivo `docker-compose.yml` de Chatwoot:
   * En las etiquetas de Traefik del servicio `rails`, cambia `chatwoot.tudominio.com` por tu dominio DNS real.
   * Verifica que las credenciales de base de datos coincidan con tu `.env`.
6. Levanta los contenedores descargando las imágenes primero:
   ```bash
   docker-compose pull
   docker-compose up -d
   ```

---

### Paso 4: Inicialización de la Base de Datos y Creación de Administrador

Una vez levantados los contenedores, ejecuta los siguientes comandos para configurar el esquema de la base de datos de Chatwoot:

1. **Correr las migraciones e inicialización:**
   ```bash
   docker-compose exec rails bundle exec rails db:create
   docker-compose exec rails bundle exec rails db:migrate
   docker-compose exec rails bundle exec rails db:seed
   ```
2. **Crear el primer usuario administrador de la plataforma:**
   Ingresa a la consola interactiva de Rails:
   ```bash
   docker-compose exec rails bundle exec rails console
   ```
   Dentro de la consola interactiva, ejecuta los siguientes comandos en Ruby:
   ```ruby
   # Instanciar el nuevo usuario
   u = User.new
   u.name = "Tu Nombre"
   u.email = "tu-email@ejemplo.com"
   u.password = "tu-contraseña-segura"
   u.password_confirmation = "tu-contraseña-segura"
   u.save!

   # Crear la cuenta principal
   account = Account.create!(name: "Mi Clinica")

   # Asociar el usuario creado a la cuenta con privilegios de administrador
   AccountUser.create!(
     account_id: account.id,
     user_id: u.id,
     role: "administrator"
   )

   # Salir de la consola
   exit
   ```

---

### Paso 5: Acceso y Mantenimiento

1. Abre tu navegador y accede a tu dominio configurado (e.g., `https://chatwoot.tudominio.com`). Inicia sesión con el correo y la contraseña creados en el paso anterior.
2. **Hacer Backup de la Base de Datos:**
   ```bash
   docker-compose exec postgres pg_dump -U postgres chatwoot > backup_$(date +%Y%m%d_%H%M%S).sql
   ```
3. **Restaurar Backup:**
   ```bash
   docker-compose exec -T postgres psql -U postgres chatwoot < backup_archivo.sql
   ```
4. **Actualizar Chatwoot:**
   ```bash
   docker-compose down
   docker-compose pull
   docker-compose up -d
   docker-compose exec rails bundle exec rails db:migrate
   ```

---

## 🧪 Documentación de Pruebas

Para garantizar y certificar el correcto funcionamiento de este flujo en n8n y del frontend CRM, se han elaborado tres documentos detallados de control de calidad:

1.  **Pruebas de Caja Negra:** [CASOS_DE_PRUEBA.md](file:///c:/Users/lalop/Desktop/FrontTallerInt/CASOS_DE_PRUEBA.md)
    *   Casos de prueba funcionales de cara al usuario final (7 casos para el CRM Web y 12 casos para el bot conversacional y recordatorios automáticos).
2.  **Pruebas Unitarias:** [PRUEBAS_UNITARIAS.md](file:///c:/Users/lalop/Desktop/FrontTallerInt/PRUEBAS_UNITARIAS.md)
    *   Verificación en aislamiento de utilidades de fecha (`dateHelpers.js`), renderizado de componentes y filtros de seguridad en nodos de código de n8n.
3.  **Pruebas de Integración:** [PRUEBAS_INTEGRACION.md](file:///c:/Users/lalop/Desktop/FrontTallerInt/PRUEBAS_INTEGRACION.md)
    *   Validación de la comunicación bidireccional entre el CRM Web, Supabase, Google Calendar, n8n, Chatwoot y Gmail.
