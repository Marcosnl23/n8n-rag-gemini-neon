# 🤖 RAG Chatbot Local con n8n

> Chatbot conversacional con Generación Aumentada por Recuperación (RAG) usando n8n, PostgreSQL/pgvector en Neon Tech y Google Gemini como modelo de lenguaje.

---

## 📋 Descripción

Este proyecto implementa un sistema RAG (Retrieval-Augmented Generation) completamente funcional mediante n8n. Permite ingestar documentos en una base de datos vectorial y luego chatear con ellos usando inteligencia artificial, todo sin salir del entorno local de n8n.

El flujo se divide en dos pipelines principales:

- **Data Ingestion**: Carga y procesa documentos para almacenarlos como embeddings en PostgreSQL.
- **RAG Chatbot**: Recibe mensajes del usuario, recupera contexto relevante y genera respuestas con Gemini.

---

## 🏗️ Arquitectura

```
┌─────────────────────────────────┐     ┌──────────────────────────────────────────┐
│        DATA INGESTION           │     │              RAG CHATBOT                 │
│                                 │     │                                          │
│  On form submission             │     │  When chat message received              │
│         │                       │     │         │                                │
│         ▼                       │     │         ▼                                │
│  Default Data Loader            │     │  AI Agent                                │
│         │                       │     │    ├── Google Gemini Chat Model          │
│         ▼                       │     │    ├── Simple Memory                     │
│  Recursive Character            │     │    └── Postgres PGVector Store1          │
│  Text Splitter                  │     │              │                           │
│         │                       │     │              ▼                           │
│         ▼                       │     │    Embeddings Google Gemini              │
│  Postgres PGVector Store        │     │                                          │
│  (Embeddings Google Gemini)     │     │                                          │
└─────────────────────────────────┘     └──────────────────────────────────────────┘
```

---

## 🛠️ Tecnologías

| Componente | Tecnología |
|---|---|
| Orquestación | [n8n](https://n8n.io/) |
| Base de datos vectorial | [PostgreSQL + pgvector](https://github.com/pgvector/pgvector) en [Neon Tech](https://neon.tech/) |
| Modelo de lenguaje | [Google Gemini](https://deepmind.google/technologies/gemini/) |
| Embeddings | Google Gemini Embeddings |
| Memoria conversacional | Simple Memory (n8n) |

---

## ✅ Requisitos previos

- [n8n](https://docs.n8n.io/hosting/) instalado y en funcionamiento (self-hosted o cloud)
- Cuenta en [Neon Tech](https://neon.tech/) con una base de datos PostgreSQL creada
- Extensión `pgvector` habilitada en tu base de datos Neon
- API Key de [Google AI Studio](https://aistudio.google.com/) (para Gemini)

---

## 🚀 Instalación y configuración

### 1. Configurar Neon Tech (PostgreSQL + pgvector)

1. Crea una cuenta en [neon.tech](https://neon.tech/) y crea un nuevo proyecto.
2. En la consola SQL de Neon, habilita la extensión pgvector:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

3. Crea la tabla para almacenar los embeddings:

```sql
CREATE TABLE documents (
  id BIGSERIAL PRIMARY KEY,
  content TEXT,
  metadata JSONB,
  embedding VECTOR(768)
);
```

4. Copia la **connection string** de tu base de datos (formato: `postgresql://user:password@host/dbname`).

### 2. Configurar credenciales en n8n

#### Google Gemini
1. Ve a **Credentials** → **New** → busca `Google Gemini (PaLM) API`
2. Pega tu API Key de Google AI Studio.

#### PostgreSQL (Neon)
1. Ve a **Credentials** → **New** → busca `Postgres`
2. Completa los campos con los datos de tu conexión Neon:
   - **Host**: tu host de Neon (e.g., `ep-xxx.us-east-1.aws.neon.tech`)
   - **Database**: nombre de tu base de datos
   - **User / Password**: credenciales de Neon
   - **SSL**: activar (requerido por Neon)

### 3. Importar el workflow

1. En n8n, ve a **Workflows** → **Import from file** (o copia el JSON).
2. Asigna las credenciales configuradas a los nodos correspondientes:
   - Nodos `Postgres PGVector Store` y `Postgres PGVector Store1` → credencial PostgreSQL
   - Nodos `Embeddings Google Gemini` y `Google Gemini Chat Model` → credencial Gemini

---

## 📂 Uso

### Ingestar documentos (Data Ingestion)

1. Ejecuta el workflow o accede al formulario de ingesta.
2. Sube o proporciona el documento que deseas procesar.
3. El flujo automáticamente:
   - Carga el documento con **Default Data Loader**
   - Lo divide en fragmentos con **Recursive Character Text Splitter**
   - Genera embeddings con **Embeddings Google Gemini**
   - Almacena los vectores en **Postgres PGVector Store** (Neon)

### Chatear con los documentos (RAG Chatbot)

1. Haz clic en **Open chat** en el workflow de n8n.
2. Escribe tu pregunta — el agente:
   - Busca fragmentos relevantes en **Postgres PGVector Store1**
   - Usa **Simple Memory** para mantener el contexto de la conversación
   - Genera una respuesta con **Google Gemini Chat Model**

---

## ⚙️ Parámetros configurables

| Parámetro | Nodo | Descripción |
|---|---|---|
| Tamaño de chunk | Recursive Character Text Splitter | Tamaño en caracteres de cada fragmento (default: 1000) |
| Overlap | Recursive Character Text Splitter | Superposición entre fragmentos (default: 200) |
| Top K | Postgres PGVector Store1 | Número de fragmentos a recuperar por consulta |
| Modelo Gemini | Google Gemini Chat Model | Versión del modelo (e.g., `gemini-1.5-flash`) |
| Dimensión embeddings | Postgres PGVector Store | Debe coincidir con la columna `vector(768)` en Neon |

---

## 🗂️ Estructura del proyecto

```
n8n-rag-gemini-neon/
├── README.md
├── workflow.json          # Exportación del workflow de n8n
```

---

## 🔧 Solución de problemas

**Error de conexión SSL con Neon**
> Asegúrate de que la opción SSL esté activada en las credenciales de Postgres en n8n. Neon requiere SSL obligatoriamente.

**Error: `column "embedding" is of type vector but expression is of type text`**
> Verifica que la dimensión del vector en la tabla coincide con la dimensión de salida del modelo de embeddings de Gemini (768).

**El chatbot responde sin contexto del documento**
> Confirma que la ingesta de datos se ejecutó correctamente y que los embeddings están almacenados en Neon. Puedes verificarlo con: `SELECT COUNT(*) FROM documents;`

