# Manejadores por formato

Aplicar estas operaciones dentro del entorno privado sin red. Este documento es
una guía de implementación por formato, no un programa que prometa soportarlos
todos. Detectar versiones/capacidades localmente; bloquear si no se dispone de un
manejador y una revisión suficientes. Conservar originales byte a byte.

## PDF

**Revisar:** texto visible y OCR, imágenes, gráficos/vectoriales de firmas,
campos y valores de formularios, anotaciones, capas opcionales, links, acciones,
JavaScript, archivos adjuntos, miniaturas, metadatos Info/XMP, índices, etiquetas
de accesibilidad, revisiones y firmas digitales.

Opciones locales:

1. Redacción estructural con una librería instalada como PyMuPDF. Localizar texto
   por diccionario/variantes dentro del proceso privado, ampliar regiones para
   cubrir antialiasing y aplicar redacciones reales a texto, imágenes y gráficos
   según la versión de la librería. Eliminar objetos auxiliares sensibles. Guardar
   un archivo nuevo con reescritura completa, compactación y sin actualización
   incremental. Confirmar que los identificadores no sobreviven en objetos, OCR
   o imágenes compartidas/reutilizadas. No basta dibujar rectángulos encima.
2. Reconstrucción visual: renderizar cada página localmente, ejecutar OCR offline,
   aplicar máscaras sólidas a píxeles identificadores y reconstruir un PDF nuevo
   únicamente con esos rasters saneados. No adjuntar el original ni copiar su
   OCR, árbol de etiquetas, comentarios, miniaturas o campos. Si se agrega una
   capa OCR nueva, generarla sólo desde los rasters saneados y revisarla otra vez.

Reextraer texto y OCR de todas las páginas de salida; comprobar redacciones y
legibilidad localmente. La rasterización pierde búsqueda, accesibilidad, vectores
y firmas verificables: declararlo, no afirmar que preserva todo. Una firma digital
original deja de ser válida al modificar el documento; si hay que conservar su
validez, detenerse y acordar la utilidad de una copia claramente derivada.

## JPEG, PNG, TIFF, WebP y otros rasters

**Revisar:** píxeles y bordes de imagen, texto impreso, firmas, rostros, QR/barras,
placas/etiquetas; EXIF/GPS, XMP, IPTC, comentarios JPEG COM, miniaturas y previews,
texto PNG tEXt/iTXt/zTXt, páginas TIFF, frames animados y blobs de aplicación.

Usar Pillow/OpenCV y OCR clásico como Tesseract ya instalado, sin descargar datos.
Respetar orientación EXIF antes de quitarla y procesar todas las páginas/frames.
Eliminar identificadores visibles con máscaras opacas o recorte autorizado, no
blur/pixelado reversible o ambiguo. No ocultar con transparencia ni capas.

Crear un nuevo buffer de píxeles y codificar desde él con metadatos por lista
blanca; no preservar `image.info`/EXIF ni copiar contenedores completos por defecto.
Eliminar miniaturas y bloques auxiliares; conservar sólo parámetros técnicos
justificados como un perfil de color genérico. No declarar todo comentario
personal: puede ser sólo un marcador técnico. Tampoco inferir anonimato de EXIF vacío.

Volver a decodificar la salida, inspeccionar metadatos con un parser independiente
cuando esté disponible y ejecutar OCR offline. Regiones ilegibles o firmas sin un
método suficiente de detección requieren revisión humana local. El OCR no detecta
por sí solo rostros ni garantiza lectura de toda escritura manuscrita.

Para imágenes médicas preservar la anatomía y parámetros necesarios; preferir
salidas sin pérdida cuando corresponda. Un JPEG del visor no es el DICOM original
ni una representación necesariamente apta para diagnóstico. No modificar anatomía
para ocultar identidad sin una política clínica y validación específicas.

## TXT, CSV, JSON, XML y HTML

Usar parsers adecuados; sustituir **valores y claves**, nombres de campos dinámicos,
rutas y referencias cuando sean identificadores. Recorrer estructuras anidadas y
normalizar variantes, codificaciones HTML/Unicode, RUT/IDs concatenados a nombres,
texto libre y cadenas que contienen otros JSON/XML serializados.

En JSON/XML preferir lista blanca del esquema necesario frente a eliminar sólo
claves llamadas `PatientName`. Un nombre geométrico como `ImagePositionPatient`
no contiene por sí mismo la identidad. Un UID de estudio es diferente de una
constante técnica de formato y puede vincularse a registros originales.

**Invariancia geométrica obligatoria:** proteger también los valores, no sólo las
claves. Una regex genérica de RUT/ID puede confundir una fracción decimal larga con
un identificador y convertir una coordenada en texto redactado, aunque la clave
siga presente y el JSON sea sintácticamente válido. No pasar valores técnicos
validados por sustituciones de texto libre ni desactivar controles de privacidad
globalmente. Definir esquema, tipo, aridad y dominio para cada campo geométrico;
una lista blanca de clave sola no autoriza texto arbitrario dentro del campo.

Antes de entregar, comparar origen y salida por registro: posición (3 componentes),
orientación (6), espaciado de píxeles (2), grosor, separación y ubicación de cortes,
filas/columnas y demás geometría presente. Exigir conservación de todos los valores
numéricos, precisión y asociaciones; validar aparte los UIDs remapeados y sus
referencias. Distinguir ausencias de origen, campos eliminados y valores alterados.
Incluir regresiones sintéticas con decimales largos, negativos, ceros y exponentes,
y las variantes de esquema reales. Emitir sólo contadores y nombres técnicos fijos.
Bloquear la aprobación ante pérdida geométrica; no inventar coordenadas. CRC/SHA-256,
JSON válido, OCR sin residuos y píxeles preservados no demuestran esta invariancia.

Para reparar geometría de un derivado ya saneado, generar otra versión sin alterar
la anterior. Restituir sólo hojas numéricas validadas desde el original de lectura:
no copiar objetos enteros, nombres, atributos, UIDs ni otros textos por estar junto
a un valor técnico. En wrappers `vr`/`Value`, reparar únicamente `Value`. Probar el
fallo antes de corregir y cubrir precisión, negativos, exponentes, ceros y texto
personal mal ubicado bajo una clave geométrica. Verificar correspondencia por
registro/ruta y relaciones; no usar sólo igualdad de conjuntos de coordenadas.
Comprobar que el JSON fuera de esas hojas es idéntico al derivado previo y que los
demás archivos mantienen sus bytes verificados. Esto permite heredar controles
previos de archivos intactos, sin afirmar que se volvió a ejecutar OCR. Reconstruir
manifiesto, archivos de integridad y ZIP; un verificador independiente debe abrir
el ZIP cerrado y cotejar valores/rutas con los originales y bytes con el manifiesto.

En HTML retirar comentarios, `data-*`, atributos ocultos, scripts, formularios,
metadatos, links y URLs identificadoras. Procesar imágenes y recursos locales
embebidos, incluidos data URIs; no cargar URLs externas. Regenerar HTML estático
mínimo y limpio, sin recursos remotos, en lugar de mantener una aplicación completa.
No ejecutar JavaScript para extraer datos sin un sandbox específico ya aprobado.

Revalidar sintaxis y coherencia entre referencias de todos los derivados. Si una
sustitución cambia longitudes/offsets que usa el formato, reconstruirlo correctamente.
Un reemplazo binario indiscriminado puede corromper archivos o dejar variantes.

## DOCX, XLSX, PPTX y ODF

Son contenedores, no sólo el texto visible. Inspeccionar localmente su estructura
completa: propiedades core/custom, autores, revisiones, cambios controlados,
comentarios, texto alternativo, notas, diapositivas/hojas/filas ocultas, nombres
definidos, celdas, fórmulas y cachés, vínculos externos, miniaturas, documentos
incrustados, imágenes y firmas. No ejecutar macros ni actualizar datos externos.

Reconstruir sólo partes necesarias y verificadas con librerías locales, o generar
un derivado PDF/imagen si el usuario acepta perder editabilidad y después sanearlo
como tal. Aceptar cambios o quitar comentarios no elimina todos los historiales o
metadatos. Un XML limpio no basta si quedan recursos o binarios incrustados.

Archivos heredados, macros o componentes no cubiertos: bloquear o acordar una
conversión aislada. No reenviar el archivo intacto como fallback. LibreOffice
headless requiere perfil temporal aislado, sin macros ni conexiones externas,
y sigue necesitando saneamiento y revisión del derivado.

## DICOM e información médica relacionada

Usar un parser DICOM local, por ejemplo pydicom instalado, y una política explícita
basada en el perfil aplicable de DICOM PS3.15, Annex E. No afirmar cumplimiento
normativo completo por haber borrado unas etiquetas.

- Recorrer secuencias, tags estándar, privados, textos libres, identificadores,
  PatientName/ID, nacimiento, accession, centros, estaciones, médicos y fechas.
- Eliminar tags privados salvo lista blanca técnicamente justificada. Revisar
  overlays, icon images, previews, SR, KO, encapsulated PDF y documentos asociados.
- Remapear consistentemente UIDs identificadores de Study/Series/SOP Instance y
  Frame of Reference, y todas sus referencias en DICOM y JSON derivados. Usar UIDs
  nuevos válidos (p. ej. `2.25.<entero UUID>`), no hashes triviales del original.
- Conservar UIDs semánticos como SOP Class y Transfer Syntax, que describen el
  formato; no sustituir ciegamente todo valor con representación UID.
- Preservar integridad de píxeles, geometría, unidades, escala y relaciones entre
  series. No cambiar datos clínicos para que el archivo parezca consistente.
- No establecer `BurnedInAnnotation=NO` o `PatientIdentityRemoved=YES` sin efectuar
  y verificar la acción correspondiente. Documentar la técnica aplicada sin
  almacenar los valores originales en DeidentificationMethod u otros campos.
- Examinar texto quemado en píxeles y todos los frames. Datos volumétricos de
  cabeza/cara pueden permitir reconstrucción facial aunque no tengan etiquetas:
  bloquear o requerir un procedimiento de defacing específico validado localmente,
  con autorización sobre la eventual pérdida de utilidad clínica.

Dose Info y reportes de dosis pueden contener identificadores impresos aun si
las imágenes anatómicas no los contienen. Las vistas PNG, PDF/HTML/TXT y JSON del
mismo estudio deben sanearse también, no sólo los DICOM. No confundir respaldo
renderizado con un conjunto DICOM nativo.

JSON de visores médicos puede expresar atributos como claves hexadecimales DICOM,
objetos con `vr`/`Value` o pares `name`/`value`; no basta quitar claves `PatientName`.
Tratar esos esquemas localmente, retirar atributos identificadores/privados y
mantener geometría y referencias técnicas con sustitución consistente.

Al reconstruir PDF como páginas rasterizadas, comprobar que los píxeles de cada
imagen incrustada coinciden con el raster saneado esperado. Esto verifica la
conservación fuera de máscaras sin enviar páginas al modelo. Comprobar además
texto oculto, anotaciones, enlaces, adjuntos, XMP y metadatos originales; advertir
pérdida de búsqueda, accesibilidad estructurada y firmas.

Al buscar fechas, contemplar ISO 8601 contiguo a `T`/`Z`: límites de palabra pueden
no detectar la fecha o segundos. Separar límites numéricos de límites alfabéticos.
Usar normalización equivalente en detección y sustitución (acentos, Unicode y
mayúsculas). Si se detecta sobre texto normalizado, mantener un mapa de posiciones
para ocultar el rango original correcto y volver a verificar; no imprimir el término.
Los contadores de residuos deben bloquear el paquete hasta resolverse. Eliminar
OCR, diccionarios, mapas y registros temporales sensibles una vez verificado el
artefacto local final; conservar sólo auditoría depurada y originales autorizados.

## ZIP, TAR y contenedores anidados

Validar todas las entradas antes de extraer: rechazar rutas absolutas, `..`, rutas
Windows/UNC, NUL, colisiones tras normalización, duplicados ambiguos, symlinks,
hardlinks, dispositivos y tipos especiales. Aplicar límites de número de entradas,
bytes reales descomprimidos, ratio, profundidad y tiempo; no confiar sólo en tamaños
de cabecera. Rechazar un archivo cifrado sin clave disponible localmente sin pedirla
por chat. No mostrar contraseñas en argumentos ni registros.

Inventariar y sanear recursivamente. Crear un contenedor completamente nuevo con
nombres opacos, tiempos normalizados, sin comentario ni extra fields originales,
sin propietario, ACL o atributos extendidos del host. Incluir sólo derivados
aprobados y un manifiesto depurado; excluir diccionarios, OCR, mapas, logs,
originales, backups, archivos temporales y manifiestos de hashes antiguos.

Reabrir la copia final y comparar cada entrada con el manifiesto de derivados.
Comprobar bytes descomprimidos y rutas, no sólo que el ZIP pasa una prueba CRC.
El propio manifiesto es parte del corpus a revisar.

## Audio, video, CAD, bases de datos y otros formatos

No asumir cobertura. Además de metadatos, el audio puede decir identificadores y
contener voces reconocibles; el video, rostros/texto por frame y audio; una base
puede tener registros borrados, índices, WAL y páginas libres; CAD puede incluir
capas ocultas, autores, rutas y bloques. Requerir manejador local y revisión
específicos, o declarar el formato bloqueado. Quitar etiquetas con FFmpeg/ExifTool
no elimina el contenido identificador. No subir a una IA para resolverlo.
