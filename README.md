# DMARC Dashboard

Dashboard web estático para visualizar reportes DMARC agregados desde Supabase.

El proyecto está implementado en un único archivo, [`index.html`](/Volumes/usb_disk_4TB/VIVIO/DMARCAutomationPlatform/fase4/deploy/index.html), y funciona sin build, sin backend propio y sin dependencias locales. Toda la lógica de UI, estilos y carga de datos vive en ese archivo.

## Qué hace

La interfaz muestra el estado de autenticación DMARC de uno o varios dominios a partir de tablas de Supabase:

- Resumen global de emails enviados, autenticados, bloqueados y organizaciones reporteras.
- Vista por dominio con métricas, alertas de alineación SPF, barras por día y tabla de registros recientes.
- Timeline diaria por dominio.
- Inventario de IPs y fuentes emisoras.
- Resumen de organizaciones que envían informes DMARC.

## Arquitectura

- Frontend estático en HTML/CSS/JavaScript vanilla.
- Consumo directo de la API REST de Supabase con `fetch`.
- Configuración de conexión guardada en `localStorage`.
- Refresco automático cada 5 minutos.
- Dependencias CDN:
  - Google Fonts (`DM Sans`, `DM Mono`)
  - Chart.js 4.4.1

Nota: actualmente `Chart.js` se carga pero no se usa en la implementación visible del dashboard.

## Configuración

La aplicación necesita dos valores de Supabase:

- `SUPABASE_URL`
- `SUPABASE_ANON_KEY`

Se pueden introducir desde el modal `Config` del propio dashboard. Esos valores se guardan en `localStorage` bajo la clave `dmarc_supabase_config`.

También existe fallback a valores hardcodeados en `index.html`, aunque ahora mismo están vacíos:

```js
return {
  url: '',
  key: '',
};
```

## Tablas esperadas en Supabase

Según las consultas definidas en `index.html`, el dashboard espera estas tablas:

### `dmarc_domains`

Consulta usada:

```text
select=*&active=eq.true&order=domain.asc
```

Campos usados por la UI:

- `id`
- `domain`
- `active`

### `dmarc_reports`

Consulta usada:

```text
select=*&order=date_begin.desc&limit=500
```

Campos usados por la UI:

- `id`
- `domain_id`
- `date_begin`
- `email_count`
- `spf_fail`
- `spf_pass`
- `dkim_fail`
- `blocked`
- `reporter_org`
- `policy_published_p`

### `dmarc_records`

Consulta usada:

```text
select=*&order=email_count.desc&limit=1000
```

Campos usados por la UI:

- `report_id`
- `source_ip`
- `email_count`
- `dkim_result`
- `dkim_selector`
- `spf_result`
- `envelope_from`
- `policy_disposition`
- `disposition`

### `dmarc_known_sources`

Consulta usada:

```text
select=*
```

Campos usados por la UI:

- `ip`
- `provider`
- `label`

## Cómo ejecutarlo

No requiere instalación. Basta con servir el archivo como sitio estático o abrirlo en navegador.

Opciones simples:

1. Abrir `index.html` directamente en el navegador.
2. Servir esta carpeta con cualquier servidor HTTP estático.

Ejemplo con Python:

```bash
python3 -m http.server 8000
```

Luego abrir:

```text
http://localhost:8000
```

## Flujo de uso

1. Abrir el dashboard.
2. Pulsar `Config`.
3. Introducir `Supabase URL` y `Anon Key`.
4. Guardar.
5. El dashboard carga dominios, reportes, registros y fuentes conocidas desde Supabase.

Si faltan credenciales, la app muestra un banner de configuración y abre el modal automáticamente la primera vez.

## Limitaciones actuales

- No hay autenticación de usuario en la propia app.
- Toda la lectura se hace con la `anon key`, así que las políticas RLS de Supabase tienen que estar bien definidas.
- No hay paginación ni filtros avanzados.
- No hay sistema de build, tests ni separación por módulos.
- Toda la lógica está concentrada en un único archivo HTML.

## Siguiente paso razonable

Si este dashboard va a crecer, lo más natural sería separar:

- configuración y cliente Supabase,
- funciones de agregación,
- renderizado por vistas,
- y documentación del esquema SQL real de las tablas.
