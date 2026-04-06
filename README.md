# ChatIA Space – Documentación Oficial

Bienvenido a la documentación del sistema **ChatIA Space**, un SaaS multi-tenant de agentes IA para WhatsApp con integración a Google Calendar y Excel.

## 📘 Contenido

- [Configuración de perfiles de cliente](docs/configuracion-perfiles.md) – Estructura de JSON, ejemplos, y guía para generarlos con IA.
- [Ejemplos de JSON](docs/ejemplos-json/) – Perfiles listos para usar (colegio, restaurante, clínica, etc.)
- [Guía del prompt builder](docs/prompt-builder-guia.md) – Cómo funciona el sistema de prompts neutro.

## 🚀 Uso rápido

Para crear un nuevo cliente, copia la carpeta `docs/ejemplos-json/colegio/`, renómbrala con el ID del cliente, edita los JSON según la [guía](docs/configuracion-perfiles.md) y súbelos a `clients/{client_id}/` en el servidor.

## 🤖 Generación automática

Usa el documento [configuracion-perfiles.md](docs/configuracion-perfiles.md) como instrucción para una IA (ChatGPT, Claude, etc.). Pídele que genere los JSON para cualquier negocio.

---

© 2026 ChatIA Space – Fabio Vega