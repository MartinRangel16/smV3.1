# Semilla Dorada · Control local

PWA sin servidor para ventas a crédito y en efectivo, tiendas, precios, entradas de mercancía y comisiones. La aplicación funciona sin conexión y guarda sus datos en IndexedDB en el dispositivo actual.

## Primer inicio

1. Abre la app con conexión una vez para instalar sus archivos sin conexión.
2. Inicia sesión con una cuenta de `users-config.js`.
3. Inicia sesión con el usuario y contraseña de tu cuenta. No se pide contraseña maestra para el uso diario.
4. Si el dispositivo ya tiene una base cifrada de una versión anterior, se solicitará la contraseña maestra antigua **una sola vez** para migrarla sin borrar los registros.

Los datos locales siguen cifrados, pero la clave aleatoria se guarda en el navegador para evitar una segunda contraseña. Esto protege menos frente a alguien que tenga acceso al perfil del navegador o a los archivos del dispositivo. No borres los datos del sitio ni la clave local; haz respaldos antes de actualizar. El bloqueo por inactividad ocurre después de 15 minutos.

## Cifrado y límites

- Los registros de IndexedDB se cifran con AES-GCM de 256 bits. La clave se deriva de un secreto aleatorio almacenado en este navegador con PBKDF2-SHA-256 (310,000 iteraciones) y una sal aleatoria por instalación.
- Los nuevos respaldos JSON también se cifran con AES-GCM y una sal propia del archivo. Se pueden restaurar en otro dispositivo usando la contraseña elegida al exportar el respaldo.
- Los respaldos anteriores sin cifrar todavía se pueden importar; la app advierte antes de leerlos y luego guarda los datos importados cifrados.
- El cifrado protege los datos almacenados cuando la app está cerrada. Cuando está desbloqueada, el navegador necesita los datos y la clave en memoria; alguien que controle el dispositivo o las herramientas de desarrollo puede acceder a ellos. El acceso por usuario es una barrera local, no autorización resistente a manipulación.
- Perder la clave local del navegador puede impedir el acceso a los datos cifrados. Exporta respaldos cifrados regularmente y conserva las contraseñas de los respaldos por separado.
- IndexedDB no se sincroniza entre dispositivos. Utiliza un respaldo cifrado para mover datos.

## Respaldos

En Inicio, exporta respaldos cifrados con regularidad. La restauración limita el archivo a 25 MB, valida el formato y los bloques de datos, pide confirmación y guarda una copia interna anterior antes de reemplazar la información. El checksum detecta daños accidentales; el cifrado autentica el contenido. El CSV de notas es para Excel y no sirve para restaurar.

Los respaldos contienen información del negocio. No los publiques ni los agregues al repositorio. `users-config.js` también contiene material de autenticación: mantenlo privado y no publiques el ZIP completo.

## Seguridad incluida

- Content Security Policy sin scripts inline ni estilos inline.
- Sincronización entre pestañas para avisar de cambios simultáneos.
- Exportación CSV protegida contra inyección de fórmulas.
- UUID para registros, validaciones de entrada y confirmaciones propias para acciones destructivas.
- El service worker guarda únicamente recursos de la app; los datos operativos permanecen en IndexedDB.

## Publicar cambios

Desde la carpeta del proyecto, ejecuta:

```sh
node herramientas/actualizar-sw.js
```

Esto regenera la lista de archivos de la caché sin conexión y su versión. Cada cambio futuro debe volver a generar `sw.js` antes de publicar.

## Estructura

- `index.html`, `screens/`: interfaz por pantallas.
- `js/loader.js`, `js/navigation.js`, `js/pwa.js`: carga, navegación y funcionamiento sin conexión.
- `js/storage.js`: almacenamiento cifrado, migraciones y respaldos.
- `js/backup.js`: exportación y restauración cifradas.
- `js/auth.js`: acceso local y bloqueo por inactividad.
- `js/app-core.js`, `js/app-sales.js`, `js/app-stores.js`, `js/app-history.js`, `js/app-commission.js`: funciones operativas separadas por área.
- `js/app-scanner.js`, `js/app-export.js`, `js/app-settings.js`: lector QR, exportación, cierre semanal y configuración.
- `herramientas/`: utilidades de desarrollo, excluidas de la caché pública.

## Notas operativas

- La comisión se calcula sobre el importe de las notas; confirma la regla antes de usarla para pagos.
- El saldo teórico de inventario no contempla devoluciones, mermas ni traspasos.
- «Cerrar semana» archiva ventas y entradas para consultarlas en sus historiales.
- Los importes nuevos se redondean a dos decimales. Los registros históricos conservan sus valores originales.
