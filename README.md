# MusaAudio

**Plataforma de música de escritorio y web: biblioteca personal, playlists y búsqueda unificada en YouTube, Spotify y archivos locales, con backend propio desplegado en servidor.**

> 🔒 Código privado.
> Demo disponible bajo solicitud · Revisión de código en vivo disponible para procesos de entrevista.
> Contacto: javierkarimandradetalamante@gmail.com

*EN: Desktop (Tauri) + web music platform backed by a TypeScript/Express REST API (~36 endpoints) with JWT auth, Prisma, unified multi-source search with fuzzy ranking, DB-backed caching, rate limiting, per-route metrics and web push, containerized with Docker Compose behind Nginx (TLS) and deployed through a GitHub Actions CI/CD pipeline to a self-hosted Ubuntu server.*

<!-- TODO: captura del reproductor + GIF de búsqueda → agregar a playlist -->

---

## Funcionalidades

- **Cuentas de usuario** con registro, login y sesión por JWT.
- **Playlists y "Me gusta"**, biblioteca personal y ajustes por usuario.
- **Búsqueda unificada** en YouTube, Spotify y biblioteca local, con un *parser* propio que separa artista y canción y ordena los resultados por similitud (búsqueda difusa).
- **Discografía y metadatos** de artistas y álbumes, con verificación de pistas y puntaje de popularidad.
- **Letras, podcasts y géneros.**
- **Subida de audio local** (MP3, FLAC, WAV) con Multer.
- **Notificaciones push** (Web Push).
- **App nativa de escritorio** con Tauri 2 (Rust) para Windows, macOS y Linux, además de la versión web.

## Arquitectura

```text
  App de escritorio (Tauri)    Navegador
            │                     │
            └────────┬────────────┘
                     │ HTTPS
                   Nginx  ── TLS 1.2/1.3 · rate limit por IP · proxy inverso
                     │
        ┌────────────┴─────────────┐
        │  API REST  (Docker)      │  Node.js · Express 5 · TypeScript
        │  routes → controllers → services
        │  ├─ auth (JWT + bcrypt)
        │  ├─ playlists / tracks / settings
        │  ├─ search (YouTube · Spotify · local)
        │  ├─ metadata / discografía / popularidad
        │  ├─ cache persistente con TTL
        │  └─ middleware: auth · rate limit · métricas
        └────────────┬─────────────┘
                     │ Prisma ORM (migraciones versionadas)
                   SQLite  (volumen persistente)

  GitHub push → GitHub Actions (runner self-hosted) → deploy en servidor Ubuntu
```

## Decisiones técnicas

| Área | Implementación |
|---|---|
| Arquitectura | Capas **routes → controllers → services** en TypeScript, ~36 endpoints REST |
| Datos | Prisma ORM con 8 modelos y migraciones versionadas (usuarios, playlists, pistas, metadatos externos, caché…) |
| Autenticación | Contraseñas con bcrypt, JWT con expiración y middleware de rutas protegidas |
| Rendimiento | Caché en memoria y caché persistente en base de datos con TTL, para no saturar las APIs externas |
| Protección | Rate limiting en la aplicación (por ventana y clave) y en Nginx (`limit_req` por IP) |
| Observabilidad | Middleware de métricas por ruta: peticiones, errores, latencia media y máxima, *cache hits* |
| Despliegue | Docker Compose (backend y frontend), contenedor sin root, *healthchecks*, Nginx con TLS |
| CI/CD | GitHub Actions con runner self-hosted que despliega automáticamente al hacer push |

## Calidad

- Tests con **Jest** para el *parser* de búsqueda, búsqueda difusa, discografía, caché persistente, popularidad, verificación de pistas y sanitización de entradas de búsqueda.
- Pruebas de integración, de carga (usuarios concurrentes) y de estrés.
- Post-mortem documentado de un incidente en el servidor de producción, con auditoría de seguridad posterior.

## Stack

`Node.js` · `Express 5` · `TypeScript` · `Prisma` · `SQLite` · `JWT` · `bcrypt` · `Multer` · `Web Push` · `Jest` · `Docker Compose` · `Nginx` · `GitHub Actions` · `Ubuntu Server` · `Tauri 2 (Rust)` · `React 18` · `Zustand` · `Tailwind CSS`

## Lo que aprendí

- Integrar varias fuentes externas (YouTube, Spotify) detrás de una sola API, con caché para controlar latencia y límites de uso.
- Diseñar un ranking de búsqueda propio con tests que protegen contra regresiones.
- Operar un servidor real: CI/CD, TLS, contenedores, monitoreo y respuesta a incidentes.

---

**Autor:** Javier Karim Andrade Talamante — Backend Developer · Hermosillo, México
