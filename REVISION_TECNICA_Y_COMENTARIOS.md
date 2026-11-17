# Revisión técnica — Semilla Dorada

## Cambios incluidos

- Se añadió captura de incidencias desde cada nota del historial.
- Cada incidencia admite descripción de hasta 500 caracteres y una fotografía opcional.
- Las fotografías se aceptan en JPG, PNG o WebP, con un máximo de 8 MB de origen; se redimensionan a un máximo de 1280 px y se comprimen a JPEG de hasta 350 KB.
- Se limita cada nota a 10 incidencias para controlar el crecimiento de la base local.
- Las incidencias se guardan dentro del objeto de nota (`incidents`). Por ello usan la misma función `write(KEYS.sales, ...)` y el cifrado AES-GCM ya implementado en `storage.js`.
- La pantalla de impresión (`printSaleNote`) no se modificó: las incidencias no se imprimen.
- La edición de notas conserva `incidents` porque el editor crea `updatedNote` con `...note`. El archivado semanal conserva la nota completa porque archiva el array de notas.
- Se agregaron comentarios JSDoc a las declaraciones de funciones nombradas en los módulos JavaScript.
- Se incrementó la versión del caché del Service Worker para solicitar los recursos actualizados.

## Revisión de buenas prácticas: hallazgos

### Fortalezas observadas

1. Persistencia centralizada: los módulos de interfaz usan `read`/`write` y no escriben ventas directamente en IndexedDB.
2. Cifrado autenticado: `storage.js` utiliza AES-GCM y datos adicionales asociados a la clave del registro.
3. Escrituras asíncronas: la memoria se actualiza después de confirmar la transacción de IndexedDB.
4. Impresión separada de la captura: el HTML del ticket se construye en el módulo de impresión y la nueva interfaz de incidencias no interviene en él.
5. Varias salidas DOM usan `textContent` en vez de interpolar texto de usuario como HTML.
6. La aplicación define una Content Security Policy restrictiva y mantiene CSS en archivos externos.

### Riesgos y mejoras recomendadas

1. **Conflictos entre pestañas/dispositivos:** `sales()` seguido de `write()` es un patrón de lectura-modificación-escritura. Dos pestañas que guarden simultáneamente pueden sobrescribirse. Para una versión multiusuario se recomienda transacción atómica con control de versión o API central.
2. **Base local y cuota:** las fotos Base64 aumentan el tamaño de la colección completa de ventas y de los respaldos. La compresión y límites reducen el riesgo, pero conviene migrar fotos a un almacén IndexedDB separado con referencias por ID si el volumen crece.
3. **Acceso a fotografías:** la vista usa Data URLs almacenadas en el registro cifrado. La protección depende de que la clave de almacenamiento permanezca protegida y de que el dispositivo tenga bloqueo de sesión.
4. **`createImageBitmap`:** se requiere navegador moderno. Si se pretende soportar navegadores antiguos, añadir alternativa con `Image` y `URL.createObjectURL`.
5. **Pruebas de aceptación pendientes en dispositivo real:** cámara/QR, permisos, modo offline, almacenamiento lleno, restauración de respaldo y captura de fotos deben probarse en Android y escritorio. La revisión estática no sustituye esas pruebas.
6. **Validación de importación:** mantener las validaciones de respaldo alineadas con nuevos campos opcionales y probar restauraciones antiguas y nuevas. Los campos adicionales de las notas son compatibles con el esquema flexible observado, pero debe probarse el flujo real de respaldo/restauración.
7. **Autenticación local:** las credenciales configuradas en el cliente no equivalen a autenticación de servidor. No usar esta PWA como sistema multiusuario remoto sin API, control de acceso del lado servidor y sincronización segura.
8. **Service Worker:** la estrategia de caché debe revisarse cuando se publiquen cambios frecuentes; la actualización de versión aquí fuerza un nuevo caché, pero no sustituye un proceso de despliegue controlado.
9. **Mantenibilidad:** `app-history.js` concentra renderizado, edición y ahora incidencias. Conviene separar `app-incidents.js` y un módulo de repositorio cuando se haga una refactorización mayor, cuidando el orden del cargador.
10. **Privacidad:** definir retención, exportación y eliminación de fotografías/incidencias, así como quién puede verlas, antes de conectar vendedores y administración en un backend compartido.

## Límites de esta revisión

Esta entrega aplica la integración y una revisión estática del código del ZIP recibido. No se ejecutó una prueba funcional dentro de Chrome/Android ni se conectó a datos reales. No se garantiza ausencia total de defectos; se incluye una lista de pruebas manuales para completar la validación.

## Pruebas recomendadas antes de reemplazar la versión en uso

- [ ] Hacer una copia de seguridad desde la aplicación actual.
- [ ] Probar en una copia del proyecto, sin borrar IndexedDB real.
- [ ] Guardar una nota de crédito y otra de efectivo.
- [ ] Agregar incidencia solo con texto y comprobar que aparece al volver a abrir el historial.
- [ ] Agregar incidencia con fotografía y comprobar que se visualiza correctamente.
- [ ] Intentar archivo no permitido, imagen mayor de 8 MB y descripción vacía.
- [ ] Editar la nota y verificar que sus incidencias siguen presentes.
- [ ] Imprimir la nota y comprobar que no aparecen descripción ni fotografía.
- [ ] Cerrar la semana y comprobar que la nota archivada conserva el campo `incidents`.
- [ ] Exportar y restaurar un respaldo de prueba.
- [ ] Repetir pruebas sin conexión y en pantalla móvil.
