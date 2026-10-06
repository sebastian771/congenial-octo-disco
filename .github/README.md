# Integración continua

`workflows/ci.yml` se ejecuta en cada `push` y `pull_request`, con dos jobs independientes:

- **PHP validation**: PHP 8.3 con GMP/SOAP, validación estricta de Composer y su lockfile, instalación con dependencias de desarrollo, sintaxis de todos los archivos PHP versionados y `vendor/bin/pint --test`.
- **Vite build**: PHP/Composer para el alias de Ziggy en `vendor/`, Node.js 22, pnpm 10.11.0, instalación con `--frozen-lockfile` y `pnpm run build`.

Composer se instala con `--no-scripts`: no ejecuta Artisan ni el descubrimiento de paquetes. Los checks no arrancan la aplicación, no ejecutan migraciones y no necesitan secretos, worldserver, PostgreSQL ni Redis. Los errores de formato y build fallan el workflow; no se corrigen ni se ocultan automáticamente.

## Incorporar pruebas después

Pest y su integración con Laravel ya están declarados como dependencias de desarrollo, pero todavía no existe una suite. Añadir un job `tests` independiente cuando haya pruebas reales y configuración `phpunit.xml`; no añadir un check vacío que aparente haberlas ejecutado.

1. Reutilizar checkout, PHP 8.3 y la instalación Composer del job PHP.
2. Empezar con pruebas unitarias del dominio y del shared kernel, sin arrancar Laravel. Sustituir los puertos de SQL/SOAP por dobles de prueba.
3. Para pruebas de Laravel, preparar el arranque explícitamente en ese job y configurar `APP_ENV=testing`, SQLite en memoria (`pdo_sqlite`), caché/sesiones `array`, cola `sync` y correo `array`. Simular también los adaptadores de los reinos: SQLite por sí solo no aísla MySQL/SOAP.
4. Ejecutar `vendor/bin/pest` con la suite configurada. Las pruebas de integración contra servicios reales deben ir en otro workflow con entorno explícito.

Los comandos de validación, Pint y build también pueden ejecutarse localmente antes de enviar cambios.
