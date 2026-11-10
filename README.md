# Desidentificación local de archivos

Skill para agentes que necesitan preparar copias desidentificadas de archivos mediante herramientas locales, con aislamiento de red y sin enviar el contenido a proveedores de IA, OCR o conversión externos.

**Este repositorio contiene instrucciones, referencias y una plantilla de política. No incluye un anonimizador ejecutable, un sandbox preconfigurado ni una garantía universal de anonimato.**

## Qué propone

- Conservar los originales y trabajar sobre copias nuevas.
- Aislar el procesamiento del acceso a red, sockets y credenciales del anfitrión.
- Revisar contenido visible, píxeles, OCR, metadatos, nombres, adjuntos y contenedores.
- Utilizar parsers y OCR clásico locales, sin modelos de IA para analizar el contenido protegido.
- Mantener diccionarios, mapas y registros dentro del entorno privado; el agente sólo recibe estados y contadores depurados.
- Verificar que la desidentificación no dañe el contenido útil, incluida la geometría de imágenes médicas.
- Bloquear la entrega si quedan identificadores, formatos no cubiertos o revisiones pendientes.
- Separar el procesamiento local de cualquier publicación o entrega posterior expresamente autorizada.

## Archivos

- [`SKILL.md`](SKILL.md): procedimiento principal y límites de confianza.
- [`references/offline-execution.md`](references/offline-execution.md): aislamiento, supervisión y salida controlada.
- [`references/format-handlers.md`](references/format-handlers.md): consideraciones para PDF, imágenes, documentos Office, JSON/XML/HTML, DICOM y archivos comprimidos.
- [`references/artifact-lifecycle-and-delivery.md`](references/artifact-lifecycle-and-delivery.md): versiones, validación independiente y entregas autorizadas.
- [`templates/policy.json`](templates/policy.json): política de ejemplo para adaptar dentro del entorno privado.

## Uso

Carga `SKILL.md` y sus archivos de referencia en un agente compatible con este formato de skills. Conserva las carpetas `references/` y `templates/` junto al archivo principal.

Antes de procesar datos reales, sigue la referencia de ejecución offline y verifica el aislamiento con datos sintéticos. Las herramientas y manejadores deben estar instalados y disponibles localmente; si falta alguno o no se puede comprobar el aislamiento, el procedimiento indica detenerse.

La plantilla no contiene identificadores reales. Cualquier configuración privada debe permanecer fuera de este repositorio y de los mensajes del agente.

## Límites

Una copia desidentificada no equivale necesariamente a anonimato irreversible. La combinación de datos, las fechas, la anatomía y otros atributos pueden permitir reidentificación. Los controles automáticos no sustituyen la revisión local cuando ésta sea necesaria.

Esta skill no certifica cumplimiento legal o normativo ni valida el uso diagnóstico de imágenes derivadas. La cobertura de cada formato depende de la implementación y de las herramientas locales disponibles.

Este repositorio publica únicamente el procedimiento genérico: no incluye documentos de pacientes, imágenes médicas, datos personales, resultados de una ejecución ni credenciales.
