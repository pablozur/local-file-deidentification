# Ciclo de vida, validación independiente y entrega autorizada

Aplicar al reanudar trabajos, interpretar avisos de procesos en segundo plano,
entregar derivados individuales o guardar un paquete aprobado en almacenamiento externo.
No contiene rutas, cuentas, identificadores ni resultados de un corpus real.

## 1. Estados por generación, no por orden de llegada del mensaje

Mantener separados `processing_status`, `verification_status`,
`human_review_status` y `delivery_status`. Un exit code 0 sólo acredita que el
comando terminó normalmente: su JSON puede seguir diciendo `pending` o `blocked`.
Un número de archivos transformados tampoco demuestra aprobación.

- Asignar un ID opaco de generación y registrar privadamente la versión del código,
  configuración y huellas necesarias. El supervisor expone sólo estados depurados.
- No modificar código, diccionarios o resultados mientras un trabajador de esa
  generación los esté usando. Terminar o detener únicamente los procesos propios
  antes de cambiarla; reiniciar la fase afectada bajo una versión inequívoca.
- Si aparecen identificadores residuales, bloquear la liberación. Diagnosticar
  localmente por formato, regla y contadores, nunca devolviendo el valor detectado.
- Tras corregir, invalidar manifiestos, checksums y ZIP anteriores. Regenerar los
  derivados afectados, repetir la verificación y construir un paquete nuevo.
- Una notificación tardía describe esa ejecución, no necesariamente el estado
  actual. Contrastar generación, auditoría final y artefacto publicado antes de
  responder. No reiniciar ni invalidar una entrega correcta sólo porque llegó
  después un aviso de una etapa antigua.
- Un bloqueo explícito `LOCAL_REVIEW_REQUIRED` no se resuelve con una advertencia
  en la respuesta: necesita revisión local o una decisión de alcance autorizada.
  Distinguirlo de la limitación general «sin garantía de anonimato irreversible».

Las cachés OCR deben depender de los bytes exactos de la imagen o página y de la
configuración/versiones relevantes. No reutilizar una caché sólo por nombre o ID
cuando cambió el contenido o la política de detección.

## 2. Validación independiente del resultado

Ejecutarla dentro de la frontera local autorizada y sin devolver contenido al
modelo. Además de la verificación del transformador:

1. Reabrir todos los formatos y comparar cobertura con el inventario privado.
   Distinguir cantidad de documentos, páginas, imágenes y entradas del ZIP.
2. En PNG reconstruidos desde píxeles, inspeccionar los tipos de chunks. Usar una
   lista blanca adecuada al archivo generado y detectar EXIF, XMP y chunks de texto
   no previstos. No afirmar que todo PNG legítimo contiene sólo IHDR/IDAT/IEND.
3. En PDF reconstruidos como páginas rasterizadas, verificar con un extractor
   independiente —por ejemplo `pdftotext`— que no queda texto oculto. Reabrir las
   páginas y comprobar los píxeles de las imágenes incrustadas; no basta el texto vacío.
4. Comparar píxeles fuente/derivado fuera de las máscaras autorizadas cuando el
   método de transformación lo permita. Comprobar también que las máscaras sí
   destruyeron los datos identificadores. Una diferencia prevista no equivale a daño.
5. Validar CRC del ZIP, rutas seguras, conjunto exacto de miembros y bytes/hashes
   contra los derivados aprobados. Revisar comentarios, atributos extra y fechas
   del contenedor reconstruido. No filtrar nombres originales si falla la prueba.
6. Copiar sólo la liberación aprobada a un destino local nuevo y privado; verificar
   los bytes de la copia antes de comunicar su ubicación. El manifiesto describe
   derivados, nunca incluye hashes de originales ni mapas privados.

Guardar una auditoría depurada con alcance, contadores, estado de controles,
originales intactos, revisión humana efectuada o no y limitaciones. Limpiar OCR,
extracciones, diccionarios y mapas según la política acordada sólo después de
validar la copia persistente. Conservar originales autorizados. No prometer
borrado físico seguro por haber eliminado temporales.

## 3. Entregas posteriores por chat

Una petición como «envíame los PDF» o «envía los JSON», referida a la copia ya
aprobada, autoriza entregar esos derivados por el canal actual; no autoriza
originales, mapas, registros privados ni otros canales.

- Localizar los archivos en la liberación persistente, no en staging ni en el árbol
  original. No repetir todo el procesamiento sólo para adjuntar archivos existentes.
- Comprobar existencia y, si hay dudas sobre cambios, cotejar su hash con el
  manifiesto aprobado sin extraer su contenido al modelo.
- Adjuntar los archivos reales mediante el mecanismo de la plataforma; en Telegram,
  `MEDIA:/ruta/absoluta/al/derivado`. Una ruta escrita sin adjunto no equivale a enviarlo.
- Usar rótulos breves basados en un índice depurado o roles ya acreditados. No
  inferir el contenido de un `documento-01.pdf` únicamente por su nombre opaco.
- Si solicitan el `SKILL.md` utilizado, entregar ese archivo existente. Aclarar,
  sólo si importa para reutilizarlo, que sus referencias son archivos separados;
  no afirmar que el Markdown principal incluye un motor universal ejecutable.

## 4. Subida privada de un ZIP aprobado a Drive

La orden de guardar un archivo concreto en Drive autoriza esa subida, no hacerlo
público ni subir automáticamente el resto de la carpeta. Mantener el procesamiento
sin red separado de esta entrega externa expresamente solicitada.

Con un conector `gog` existente:

1. Comprobar autenticación sin imprimir secretos. Elegir explícitamente la cuenta
   personal prevista, no un recolector compartido. Si el contexto no resuelve cuál,
   preguntar antes de cargar datos sensibles.
2. Cotejar el SHA-256 del ZIP local con su comprobante y confirmar que corresponde
   a la generación aprobada. No volver a leer originales ni enviar el corpus a IA.
3. Verificar propietario y permisos de carpeta destino y ancestros. Para una copia
   privada, no aceptar permisos públicos, de dominio o de terceros inesperados.
4. Buscar un archivo del mismo nombre en el padre exacto antes de crear. Si ya existe,
   comparar tamaño y checksum; reutilizar sólo si coincide. No sobrescribir contenido
   distinto ni crear duplicados silenciosos. Tras timeout de una mutación, consultar
   Drive antes de reintentar porque la subida pudo haber terminado.
5. Subir sólo el archivo solicitado, sin conversión ni cambios de permisos.
6. Leer de nuevo sus metadatos desde Drive. Verificar ID, nombre, padre, tamaño,
   propietario, permisos y checksums calculados por el servicio frente a los locales.
   Si hay SHA-256 remoto, compararlo; MD5 permite un control adicional. Si no hay
   checksum adecuado o se necesita comprobación de extremo a extremo, descargar
   privadamente el ZIP, comparar SHA-256 y validar CRC; eliminar esa copia temporal.
7. Informar el enlace privado real devuelto por Drive, la cuenta de destino y la
   verificación realizada. No usar `share --type anyone` para facilitar la descarga.

Ejemplos genéricos; sustituir variables por cuenta y localizadores ya verificados:

```bash
gog drive get "$PARENT_ID" --account="$ACCOUNT" \
  --fields='id,parents,owners(emailAddress),permissions(type,role,emailAddress)' \
  --readonly --json --no-input

gog drive ls --parent="$PARENT_ID" --account="$ACCOUNT" \
  --query="trashed = false and name = 'paquete-desidentificado.zip'" \
  --no-all-drives --json --no-input

gog drive upload "$ARCHIVE" --parent="$PARENT_ID" --account="$ACCOUNT" \
  --mime-type=application/zip --json --no-input

gog drive get "$FILE_ID" --account="$ACCOUNT" \
  --fields='id,name,size,md5Checksum,sha256Checksum,parents,owners(emailAddress),permissions(type,role,emailAddress),webViewLink' \
  --readonly --json --no-input
```

Confirmar las opciones con la ayuda de la versión instalada cuando difieran.
Autenticación y transporte pertenecen a la skill de Workspace; esta referencia
cubre el límite de privacidad, la elección del artefacto y su liberación.
