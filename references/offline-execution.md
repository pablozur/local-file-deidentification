# Ejecución offline y frontera con el modelo

## 1. Qué significa la garantía

El procesamiento del corpus ocurre íntegramente en el equipo/servidor autorizado.
No se envía su contenido a proveedores de IA, OCR, almacenamiento, telemetría o
conversión. El chat puede redactar código genérico y recibir estados mínimos, no
leer el corpus ni sus resultados intermedios. La entrega posterior de un derivado
es una operación distinta y requiere autorización explícita.

No confundir `localhost` con un procesador local: puede ser un proxy hacia una
API remota. No llamar a APIs de IA aunque su URL sea local. Tampoco usar un modelo
local en este procedimiento: preferir parsers, OCR clásico y reglas deterministas.

## 2. Preparar sin datos

1. Comprobar sistema y herramientas instaladas sin listar originales sensibles.
2. Elegir un directorio de trabajo privado no sincronizado y fuera del origen.
3. Preparar código genérico sin identificadores reales en argumentos, comandos,
   excepciones, nombres de archivos de código o cadenas escritas por el modelo.
4. Confirmar dependencias y datos OCR instalados. Si faltan, bloquear. Una eventual
   instalación se acuerda y realiza aparte, antes de montar el corpus; no permitir
   que librerías descarguen modelos, consulten licencias o suban errores al abrirlo.
5. No arrancar otra instancia de Hermes/Codex/Claude ni usar agentes de evaluación
   con archivos reales. No fijar esta tarea a un proveedor diferente como atajo.

## 3. Aislamiento verificable

Usar un mecanismo de SO que niegue red al proceso y todos sus descendientes:
un contenedor/VM ya disponible con red deshabilitada y aislamiento de IPC, o en
Linux namespaces y un sandbox como Bubblewrap. No descargar imágenes de contenedor
para este paso. No cambiar la red global del host ni bloquear el gateway del usuario.

Propiedades requeridas:

- Sin salida IP/DNS y sin red del host; no interfaces enrutable ni namespace compartido.
- Sin sockets Unix del host: Docker/Podman, D-Bus, SSH agent, proxies y servicios de IA.
  Un namespace de red por sí solo no impide reenviar datos por estos canales.
- Entorno por lista blanca; sin claves API, tokens, proxies, SSH_AUTH_SOCK o variables
  de telemetría. No imprimir ni inspeccionar sus valores para informar al modelo.
- Entrada montada sólo lectura; salida privada nueva en lectura/escritura.
- No montar el HOME completo, credenciales, carpetas sincronizadas o sockets del
  anfitrión. Montar sólo runtime/dependencias y archivos necesarios por lista blanca.
- Procesos e IPC aislados, sin privilegios, sin capacidades administrativas ni
  posibilidad de crear namespaces o montajes privilegiados para escapar del sandbox.
- Descriptores heredados cerrados, salvo canales explícitamente controlados;
  stdout/stderr del trabajador dirigidos al registro privado, stdin sin corpus.
- Límites de CPU, memoria, tiempo, cantidad de archivos, profundidad y descompresión.
- Sin core dumps; temporales en el espacio privado. Considerar swap y snapshots;
  preferir almacenamiento cifrado, porque borrar archivos no garantiza borrado físico.

`unshare --user --map-root-user --net` puede ayudar en Linux, pero **no es por sí
solo un sandbox suficiente**: el kernel puede prohibir namespaces y los sockets
Unix/rutas del host pueden seguir accesibles. No prometer aislamiento porque el
binario exista. No usar variables offline o quitar API keys como única barrera.

Antes de montar el corpus, hacer una sonda inocua y guardar localmente evidencia
sanitizada de la configuración efectiva: namespace/montajes, interfaces, IPC y
reintentos de conexión denegados. Probar también el aislamiento de los procesos
hijos. Una única conexión fallida o un servidor que no responde no demuestra una
política de egreso denegado. Confirmar la barrera de SO, no sólo conectividad.

### Detalles prácticos de Bubblewrap

- Al combinar `--unshare-all` con `--disable-userns`, especificar también
  `--unshare-user`: la variante que intenta crear el namespace no siempre cumple
  los requisitos de `--disable-userns`.
- Establecer un cwd interno y un `PWD` no sensible. Bubblewrap puede introducir
  `PWD` aun usando `--clearenv`; comprobar las claves efectivas, sin imprimir valores.
- Un Python que importa librerías en el host puede fallar dentro del sandbox si
  están en user-site. Montar sólo directorios de dependencias autorizados en lectura
  y fijar `PYTHONPATH` a sus rutas internas; no montar el HOME para resolverlo.
- Verificar capacidades efectivas nulas, ausencia de sockets del host, namespaces
  nuevos de red/IPC, sólo loopback sin rutas, herencia de aislamiento en hijos y
  montajes de código/entrada en lectura. Hacer una prueba OCR/PDF sintética dentro
  del sandbox real antes de abrir originales, no únicamente fuera de él.
- Guardar fallos de configuración en un registro de la sonda sin corpus. Un fallo
  de importación es distinto de una barrera de red fallida; resolverlo sin debilitar
  montajes, permisos ni salida controlada y repetir la sonda.

Si no existe una forma verificable de aislarlo, emitir `NETWORK_ISOLATION_UNAVAILABLE`
y detenerse antes de abrir los datos. No elevar privilegios ni debilitar la garantía
silenciosamente. En macOS/Windows utilizar aislamiento equivalente ya autorizado;
no presentar una receta Linux como universal.

## Runtimes multimedia y navegador local sin Playwright

- No nombrar scripts auxiliares `inspect.py`, `json.py`, `socket.py` ni como módulos estándar: `/app` puede estar en `sys.path`; un import transitivo de PyMuPDF puede ejecutar accidentalmente ese archivo. Usar nombres específicos como `archive_inventory.py`.
- FFmpeg/ffprobe pueden requerir enlaces de `/etc/alternatives` (por ejemplo `libblas.so.3`) aunque `/usr` y `/lib` estén montados. Probar `ffprobe -version` dentro del namespace y, si corresponde, montar `/etc/alternatives` sólo lectura; no ampliar a todo `/etc`.
- En Debian, ejecutar `/usr/lib/chromium/chromium` directamente evita que el wrapper `/usr/bin/chromium` intente cargar `/etc/chromium.d/*`. Usar un perfil temporal dentro del namespace y no un perfil personal. Node 22 dispone de `fetch` y `WebSocket` para pruebas por CDP (Chrome DevTools Protocol) sin instalar Playwright.
- Servidor de prueba y navegador pueden compartir el loopback del namespace sin acceso a red externa. Guardar capturas localmente y emitir sólo resultados escalares; no enviar imágenes médicas a herramientas de visión o proveedores externos.
- En CDP, una solicitud con `canceled: true` y `errorText: net::ERR_ABORTED` puede ser una cancelación esperada al cambiar rápidamente de corte o restablecer el visor. Conservar el evento y distinguirlo de errores reales; no ignorar otros fallos de red.
- Si se autoriza publicar una copia ya aprobada, su verificación remota es una fase separada: montar sólo la entrega aprobada y un directorio vacío de resultados, nunca los originales, el ZIP de entrada, diccionarios privados ni credenciales. Mantener entorno limpio y perfil temporal. Si el wrapper usa `--unshare-all`, retirar `--unshare-net` no restablece conectividad: en esta fase pública exclusivamente se necesita `--share-net`; conservar aislamiento de red estricto en todo procesamiento de originales. Montar sólo `resolv.conf` y certificados públicos si el navegador requiere DNS/TLS.
- Cloudflare puede inyectar su beacon de analítica sólo a agentes de usuario de navegador, aunque el GET de `requests` conserve el HTML byte a byte. Si la política CSP existente lo bloquea, no relajar la política para eliminar el aviso. Registrar ese bloqueo exacto (host y ruta del beacon, tipo Script, `blockedReason: csp`) aparte de errores de aplicación; no ignorar otros bloqueos.

## 4. Separar trabajador y supervisor

**Trabajador privado:** abre archivos, extrae texto/metadata, mantiene diccionario,
transforma, revisa y guarda diagnóstico completo sólo en el espacio privado. No
construye comandos de shell con texto extraído. No ejecuta macros ni enlaces.

**Supervisor:** inicia el sandbox, captura toda salida del trabajador a archivos
privados y produce un JSON mínimo mediante un serializador de lista blanca.
No retransmitir stdout, stderr, tracebacks, nombres originales, fragmentos OCR,
mensajes de parser ni reportes arbitrarios del trabajador al chat. Si el trabajador
falla, devolver sólo un código fijo; el diagnóstico queda para inspección local.

Para fallos previos a ejecutar el trabajador, controlar también la salida del
sandbox: sus errores pueden mencionar rutas sensibles. No ejecutar `shell=True`,
`set -x`, listados, `head`/`cat` de registros ni paginadores para ver qué falló.

Esquema permitido, con valores construidos por el supervisor:

```json
{
  "status": "blocked",
  "files_total": 0,
  "files_processed": 0,
  "files_pending": 0,
  "network_isolation_verified": false,
  "originals_unchanged": null,
  "blocker_codes": ["NETWORK_ISOLATION_UNAVAILABLE"]
}
```

Aceptar únicamente estados de un enum fijo, contadores enteros no negativos,
booleanos y códigos de una lista cerrada; prohibir strings libres. No usar IDs
que deriven de datos personales o hashes de originales. La ruta de entrega, si
se necesita comunicar, la asigna el supervisor con un nombre opaco no sensible.
Los valores del ejemplo son un esquema, no resultados reales de una ejecución.

Otros códigos posibles: `DEPENDENCY_MISSING`, `FORMAT_UNSUPPORTED`,
`ENCRYPTED_INPUT`, `LOCAL_REVIEW_REQUIRED`, `RESIDUAL_IDENTIFIER`,
`OUTPUT_INVALID`, `INPUT_CHANGED`, `ARCHIVE_UNSAFE`, `WORKER_FAILED`.

## 5. Control de datos privados y entrega

- Construir el diccionario desde archivos dentro del sandbox; no solicitar valores
  personales por Telegram para poder insertarlos en un comando del modelo.
- Mantener el mapa y las evidencias fuera de `release/`, con permisos privados.
- Si se necesita revisión humana, generar un informe y visor **locales y sin red**.
  El usuario los abre en el equipo autorizado, no en un visor público/noVNC remoto
  no acordado. No pedir que adjunte una captura sin desidentificar al chat.
- No activar sincronización, backups remotos, crash reporting ni indexadores sobre
  el trabajo. No alterar servicios del host sin autorización; elegir otro espacio
  privado si no se puede excluirlos.
- No borrar originales. Limpiar temporales propios sólo tras aprobación/política
  acordada, sin prometer eliminación física segura en SSD, snapshots o nube.
- El pipeline no se publica, sube ni comparte automáticamente; sólo deriva archivos.
  Una entrega posterior incluye exclusivamente el paquete aprobado y exige el canal
  autorizado. Una carpeta privada de Drive sigue siendo almacenamiento externo.

## 6. Honestidad al informar

Distinguir: skill documentada, dependencias instaladas, aislamiento probado,
archivos transformados, verificación local completa y entrega realizada. Ninguno
implica automáticamente los demás. Si el usuario sólo pidió crear la skill sin
analizar ni evaluarla, no ejecutar sondas, procesamiento ni benchmarks del pipeline.
