# MCP LinkedIn — Plan del proyecto

## Contexto

Este proyecto es un fork adaptado de [adhikasp/mcp-linkedin](https://github.com/adhikasp/mcp-linkedin), un servidor MCP (Model Context Protocol) que permite a Claude interactuar con LinkedIn.

El repo original usa scraping de la API interna de LinkedIn a través de la librería `linkedin-api`. Lo usamos como base, pero lo adaptamos para nuestros propósitos con mejoras de seguridad, fiabilidad y funcionalidad.

---

## Por qué hacer un fork

| Problema en el original | Nuestra solución |
|---|---|
| Bug de sintaxis en Python < 3.12 | Compatibilidad desde Python 3.10 |
| Solo acepta email + password (provoca CAPTCHA challenge) | Autenticación via cookie `li_at` |
| Dependencias innecesarias (`uvicorn`, `requests` sin uso) | Solo las dependencias estrictamente necesarias |
| Poco mantenimiento, puede quedarse obsoleto | Control total del código |
| Funciones genéricas no adaptadas a nuestro caso de uso | Funciones específicas para búsqueda de empleo |

---

## Objetivo principal

Construir un **agente autónomo de búsqueda de empleo** que use este MCP como fuente de ofertas de LinkedIn, combinado con otras fuentes (APIs públicas), y que sea capaz de:

1. Buscar ofertas según criterios definidos (rol, ubicación, modalidad)
2. Filtrar y puntuar las ofertas según el perfil del usuario
3. Guardar un histórico de ofertas ya vistas para no repetir
4. Generar un resumen periódico con las mejores oportunidades

---

## Cambios técnicos planificados

### 1. Autenticación por cookie (`li_at`)
- Eliminar dependencia de email/password como método principal
- Aceptar la cookie de sesión `li_at` vía variable de entorno `LINKEDIN_LI_AT`
- Fallback a email/password si no hay cookie disponible
- **Motivo:** El login con email/password activa el challenge de LinkedIn (CAPTCHA). La cookie de sesión de un navegador ya autenticado evita este problema completamente.

### 2. Limpieza de dependencias
- Eliminar `uvicorn` y `requests` (no se usan en el código actual)
- Mantener solo: `linkedin-api`, `fastmcp`

### 3. Compatibilidad Python 3.10+
- Corregir f-strings con subscripts de diccionario que solo funcionan en Python 3.12+
- Usar sintaxis compatible: `f"texto {urn['clave']}"` con comillas simples dentro

### 4. Nuevas herramientas MCP
Además de las existentes (`search_jobs`, `get_feed_posts`), añadir:

- `get_job_details(job_id)` — Detalles completos de una oferta específica
- `search_people(keywords, company)` — Buscar personas (para networking)
- `get_company_info(company_name)` — Info de una empresa antes de aplicar

### 5. Manejo de errores robusto
- Detectar y reportar claramente el challenge de LinkedIn
- Reintentos con backoff en caso de rate limiting
- Mensajes de error útiles en vez de stack traces crudos

---

## Estructura de archivos objetivo

```
mcp-linkedin/
├── src/
│   └── mcp_linkedin/
│       ├── __init__.py
│       ├── client.py       # Servidor MCP principal
│       └── auth.py         # Lógica de autenticación (cookie / email+password)
├── pyproject.toml
├── README.md
└── PLAN.md                 # Este archivo
```

---

## Configuración en Claude Desktop

Una vez implementado, la config en `claude_desktop_config.json` será:

```json
"linkedin": {
  "command": "uvx",
  "args": ["--python", "3.12", "--from", "git+https://github.com/juanmartinsantos/mcp-linkedin", "mcp-linkedin"],
  "env": {
    "LINKEDIN_LI_AT": "<cookie de sesión de LinkedIn>"
  }
}
```

---

## Fuentes alternativas (para cuando LinkedIn bloquee)

Si LinkedIn presenta challenge o rate limiting, el agente caerá en estas fuentes:

- **Adzuna API** — Gratuita, amplia cobertura europea
- **Arbeitnow API** — Especializada en remoto + Europa
- **JSearch (RapidAPI)** — Agrega múltiples portales incluyendo LinkedIn e Indeed

---

## Estado actual

- [x] Repo creado y subido a GitHub
- [x] Identificados los problemas del repo original
- [ ] Implementar autenticación por cookie `li_at`
- [ ] Limpiar dependencias
- [ ] Corregir compatibilidad Python 3.10+
- [ ] Añadir nuevas herramientas MCP
- [ ] Añadir manejo de errores robusto
- [ ] Probar integración completa con Claude Desktop
