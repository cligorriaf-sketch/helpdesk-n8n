# 🎫 HelpDesk Pro — Sistema de Gestión de Tickets

**Universidad Mariano Gálvez de Guatemala**  
**Curso:** Análisis de Sistemas I  
**Estudiante:** Carlos Fernando Ligorria Flores  
**Proyecto Final — Automatización de Procesos con n8n en Entorno Local**

---

## 📋 Descripción

HelpDesk Pro es un sistema de gestión de tickets de soporte técnico implementado completamente en entorno local. Permite crear, listar y eliminar tickets de soporte, clasificándolos automáticamente por tipo y prioridad mediante workflows de automatización en n8n, con persistencia en base de datos PostgreSQL.

---

## 🏗️ Arquitectura

```
┌─────────────────────────────────────────────────────────┐
│                    ENTORNO LOCAL                         │
│                                                         │
│  ┌──────────────┐    ┌──────────────┐    ┌───────────┐  │
│  │   Frontend   │───▶│     n8n      │───▶│PostgreSQL │  │
│  │  index.html  │    │  :5678       │    │  :5432    │  │
│  │  :5500       │◀───│  Workflows   │◀───│  tickets  │  │
│  └──────────────┘    └──────────────┘    │  audit_log│  │
│                                          │  error_log│  │
│  ┌─────────────────────────────────────┐ └───────────┘  │
│  │           Docker Compose            │                │
│  │  helpdesk-n8n / n8n_app / postgres  │                │
│  └─────────────────────────────────────┘                │
└─────────────────────────────────────────────────────────┘
```

---

## 🛠️ Tecnologías utilizadas

| Componente | Tecnología | Versión |
|---|---|---|
| Frontend | HTML, CSS, JavaScript | Vanilla |
| Backend / Automatización | n8n | Latest |
| Base de datos | PostgreSQL | 15 |
| Contenedores | Docker / Docker Compose | Latest |
| Servidor local | Python HTTP Server | 3.x |

---

## 📁 Estructura del proyecto

```
helpdesk-n8n/
├── index.html              # Frontend del sistema
├── docker-compose.yml      # Configuración de contenedores
├── .env                    # Variables de entorno (credenciales)
├── data/                   # Datos persistentes de n8n
├── workflows/              # Exportaciones JSON de workflows
│   ├── crear_ticket.json
│   ├── listar_tickets.json
│   ├── eliminar_ticket.json
│   └── error_handler.json
├── docs/                   # Documentación técnica
│   └── documentacion_tecnica.pdf
└── README.md               # Este archivo
```

---

## ⚙️ Requisitos previos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado
- [VS Code](https://code.visualstudio.com/) (recomendado)
- Python 3.x (para servidor HTTP local)
- Navegador web moderno (Chrome recomendado)

---

## 🚀 Instalación y ejecución

### 1. Clonar o descargar el proyecto

```bash
git clone <url-del-repositorio>
cd helpdesk-n8n
```

### 2. Configurar variables de entorno

Crea un archivo `.env` en la raíz del proyecto con el siguiente contenido:

```env
POSTGRES_USER=n8n
POSTGRES_PASSWORD=n8n123
POSTGRES_DB=n8n

N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=admin123
```

### 3. Levantar los contenedores

```bash
docker-compose up -d
```

Verifica que los 3 contenedores estén corriendo:
- `helpdesk-n8n` (compose)
- `n8n_app` → puerto 5678
- `postgres_n8n` → puerto 5432

### 4. Crear las tablas en PostgreSQL

Accede al contenedor de PostgreSQL:

```bash
docker exec -it postgres_n8n psql -U n8n -d n8n
```

Ejecuta el siguiente SQL:

```sql
CREATE TABLE tickets (
  id SERIAL PRIMARY KEY,
  titulo TEXT,
  descripcion TEXT,
  tipo TEXT,
  prioridad TEXT,
  fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE audit_log (
  id SERIAL PRIMARY KEY,
  accion TEXT NOT NULL,
  ticket_id INT,
  detalle TEXT,
  estado TEXT DEFAULT 'exitoso',
  fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE error_log (
  id SERIAL PRIMARY KEY,
  workflow TEXT,
  error_mensaje TEXT,
  datos_entrada TEXT,
  fecha TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 5. Importar los workflows en n8n

1. Abre n8n en `http://localhost:5678`
2. Ve a cada workflow en la carpeta `workflows/`
3. Importa cada JSON usando el menú `...` → `Import`
4. Publica cada workflow

### 6. Iniciar el servidor del frontend

```bash
cd helpdesk-n8n
python -m http.server 5500
```

### 7. Abrir el sistema

Abre el navegador en:
```
http://localhost:5500/index.html
```

---

## 🔄 Workflows de n8n

### Workflow 1 — Crear Ticket (POST)
**Endpoint:** `POST http://localhost:5678/webhook/ticket`

**Flujo:**
```
Webhook → Codigo_tipo → Codigo_prioridad → Edit Fields → Execute SQL (INSERT) → BITACORA
```

**Nodos utilizados:** Webhook, Code (x2), Set/Edit Fields, PostgreSQL (x2)

---

### Workflow 2 — Listar Tickets (GET)
**Endpoint:** `GET http://localhost:5678/webhook/listar`

**Flujo:**
```
Webhook → Execute SQL (SELECT) → Respond to Webhook
```

**Nodos utilizados:** Webhook, PostgreSQL, Respond to Webhook

---

### Workflow 3 — Eliminar Ticket (DELETE)
**Endpoint:** `DELETE http://localhost:5678/webhook/eliminar`

**Flujo:**
```
Webhook → Execute SQL (DELETE) → Respond to Webhook
```

**Nodos utilizados:** Webhook, PostgreSQL, Respond to Webhook

---

### Workflow 4 — Error Handler
**Tipo:** Error Trigger (se activa automáticamente ante fallos)

**Flujo:**
```
Error Trigger → Execute SQL (INSERT en error_log)
```

**Nodos utilizados:** Error Trigger, PostgreSQL

---

## 🗄️ Modelo de datos

### Tabla: `tickets`
| Campo | Tipo | Descripción |
|---|---|---|
| id | SERIAL PK | Identificador único |
| titulo | TEXT | Título del ticket |
| descripcion | TEXT | Descripción del problema |
| tipo | TEXT | hardware / software / red |
| prioridad | TEXT | alta / media / baja |
| fecha | TIMESTAMP | Fecha de creación |

### Tabla: `audit_log`
| Campo | Tipo | Descripción |
|---|---|---|
| id | SERIAL PK | Identificador único |
| accion | TEXT | Acción realizada |
| ticket_id | INT | ID del ticket afectado |
| detalle | TEXT | Descripción del evento |
| estado | TEXT | exitoso / error |
| fecha | TIMESTAMP | Fecha del evento |

### Tabla: `error_log`
| Campo | Tipo | Descripción |
|---|---|---|
| id | SERIAL PK | Identificador único |
| workflow | TEXT | Nombre del workflow |
| error_mensaje | TEXT | Mensaje de error |
| datos_entrada | TEXT | Último nodo ejecutado |
| fecha | TIMESTAMP | Fecha del error |

---

## 🧪 Casos de prueba

### Caso 1 — Crear ticket válido ✅
```json
POST /webhook/ticket
{
  "titulo": "No funciona la impresora",
  "descripcion": "La impresora no imprime",
  "tipo": "hardware",
  "prioridad": "alta"
}
```
**Resultado esperado:** Ticket guardado en BD, registro en audit_log.

---

### Caso 2 — Crear ticket sin título ❌
```json
POST /webhook/ticket
{
  "titulo": "",
  "descripcion": "Sin título",
  "tipo": "software",
  "prioridad": "media"
}
```
**Resultado esperado:** Frontend valida y muestra toast de error, no envía petición.

---

### Caso 3 — Listar tickets ✅
```
GET /webhook/listar
```
**Resultado esperado:** JSON con todos los tickets ordenados por ID DESC.

---

### Caso 4 — Eliminar ticket existente ✅
```json
DELETE /webhook/eliminar
{ "id": 1 }
```
**Resultado esperado:** Ticket eliminado de BD, tabla actualizada en frontend.

---

### Caso 5 — Eliminar ticket inexistente ⚠️
```json
DELETE /webhook/eliminar
{ "id": 9999 }
```
**Resultado esperado:** n8n ejecuta DELETE sin filas afectadas, no genera error crítico.

---

## 🔒 Seguridad

- Credenciales de BD en variables de entorno (`.env`)
- Archivo `.env` excluido del repositorio (`.gitignore`)
- Autenticación básica activa en n8n
- Sin passwords hardcodeadas en el código

---

## 📊 Nodos de n8n utilizados (≥ 6 distintos)

1. **Webhook** — Entrada de datos HTTP
2. **Code** — Clasificación automática de tipo y prioridad
3. **Edit Fields (Set)** — Transformación de datos
4. **PostgreSQL** — Inserción, consulta y eliminación en BD
5. **Respond to Webhook** — Respuesta al cliente
6. **Error Trigger** — Captura de errores de workflows

---

## 👤 Autor

**Carlos Fernando Ligorria Flores**  
Universidad Mariano Gálvez de Guatemala  
Análisis de Sistemas I  
2026
