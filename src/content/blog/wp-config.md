---
title: "El poder oculto del wp-config.php"
description: "Mejora el rendimiento, la seguridad y el consumo de recursos de tu WordPress mediante configuración avanzada del wp-config.php."
pubDate: "2026-01-16"
lang: "es"
---
Instalar plugins para cada ajuste de rendimiento es el primer paso para tener una web lenta e insegura. El archivo `wp-config.php` es el corazón de tu instalación y permite ejecutar cambios de arquitectura, límites de memoria y políticas de seguridad antes incluso de que WordPress cargue su primera línea de código. 

EL objetivo es mantener el core lo más limpio posible. En este post, vamos a filtrar la documentación oficial para quedarnos con los "hacks" reales que impactan en la estabilidad de un servidor en producción.

## Seguridad: "Si no se puede editar, no se puede hackear"
Uno de los vectores de ataque más comunes es el editor de archivos interno de WordPress. Si un atacante compromete una cuenta de administrador, puede inyectar código directamente en EL temas o plugins.

```php
// Desactiva el editor interno de temas y plugins
define( 'DISALLOW_FILE_EDIT', true );

// Prohíbe instalaciones y actualizaciones desde el panel (Ideal para entornos controlados por Git)
define( 'DISALLOW_FILE_MODS', true );
```

## Rendimiento: Control de revisiones
Por defecto, WordPress guarda una copia de cada cambio que haces. En webs con mucho contenido, la tabla wp_posts se infla innecesariamente, ralentizando las consultas SQL.
```php
// Limita las revisiones a 3 (suficiente para emergencias)
define( 'WP_POST_REVISIONS', 3 );

// O desactívalas por completo si el cliente no las usa
define( 'WP_POST_REVISIONS', false );
```

## Optimización de Base de Datos y Autosave
WordPress hace un guardado automático cada 60 segundos. En servidores con recursos ajustados, esto genera una carga de IOPS (lectura/escritura) que podemos evitar.
```php
// Aumenta el intervalo de autosave a 3 minutos
define( 'AUTOSAVE_INTERVAL', 180 );

// Vacía la papelera cada 7 días en lugar de los 30 por defecto
define( 'EMPTY_TRASH_DAYS', 7 );
```

## Gestión de Memoria (Adiós al "Memory Exhausted")
Si trabajas con Page Builders pesados o WooCommerce, los 40MB base de WordPress son insuficientes y provocan errores 500 aleatorios.
```php
// Límite para el frontend
define( 'WP_MEMORY_LIMIT', '256M' );

// Límite para tareas administrativas (importaciones, backups, etc.)
define( 'WP_MAX_MEMORY_LIMIT', '512M' );
```

## Cron de WordPress: El hack para el rendimiento real
El sistema wp-cron se dispara con cada visita. Si tienes mucho tráfico, el servidor trabaja de más; si no lo tienes, las tareas programadas no se ejecutan.

```php
// Desactivar el cron virtual de WordPress
define( 'DISABLE_WP_CRON', true );
```

**Nota técnica**: Desactívalo aquí y configúralo como un cron job real en tu panel (Plesk, cPanel o CLI).

## Debugging Profesional
Nunca dejes los errores a la vista en producción. Es poco profesional y da pistas a atacantes sobre tu estructura de archivos.

```php
// Debug activo, pero silencioso para el usuario
define( 'WP_DEBUG', true );
define( 'WP_DEBUG_LOG', true ); // Log guardado en /wp-content/debug.log
define( 'WP_DEBUG_DISPLAY', false ); // No mostrar errores en el renderizado
@ini_set( 'display_errors', 0 );
```

## Forzar SSL y Seguridad en Cookies
Si usas Cloudflare y tienes un certificado SSL, evita que WordPress genere contenido mixto o cookies inseguras.

```php
define( 'FORCE_SSL_ADMIN', true );

// Evita fugas de cookies en configuraciones multi-dominio
define( 'COOKIE_DOMAIN', 'xample.com' );
```

### Tip de emergencia: Reparación de base de datos
Si te encuentras con un "Error establishing a database connection" persistente, añade esto temporalmente:

```php
define( 'WP_ALLOW_REPAIR', true );
```

Accede a xample.com/wp-admin/maint/repair.php, ejecuta la reparación y borra la línea inmediatamente. **No requiere login, por lo que es un riesgo de seguridad dejarlo activo.**

**Nota técnica:** Todos estos define deben ir siempre antes de la línea: /* That's all, stop editing! Happy publishing. */

### Bibliografía
Documentación oficial de WordPress: https://developer.wordpress.org/advanced-administration/wordpress/wp-config/