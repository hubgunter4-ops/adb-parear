# adb-parear

Una utilidad Bash para emparejar y conectar un teléfono Android por ADB inalámbrico desde Ubuntu. Incluye un menú interactivo, emparejamiento por QR o PIN, conexión automática y verificación en `adb devices`.

## Requisitos

- Ubuntu con Bash y Android SDK Platform-Tools (`adb`).
- Para el modo QR: `qrencode`.
- Teléfono Android 11 o posterior, con **Opciones de desarrollador → Depuración inalámbrica** activada.
- Teléfono y Ubuntu en la misma red Wi-Fi. La red debe permitir descubrimiento mDNS entre dispositivos.

El script no instala paquetes automáticamente ni requiere `sudo` para ejecutarse. Si falta una dependencia, muestra el comando sugerido para instalarla.

## Instalación

Clona el repositorio y entra al directorio:

```bash
git clone https://github.com/hubgunter4-ops/adb-parear.git
cd adb-parear
```

Instala las dependencias manualmente en Ubuntu:

```bash
sudo apt update
sudo apt install adb qrencode
```

`qrencode` solo hace falta para el modo QR; los modos de código y listado requieren `adb`.

Para instalar el comando en tu directorio personal:

```bash
mkdir -p "$HOME/.local/bin"
install -m 755 ./adb-parear "$HOME/.local/bin/adb-parear"
```

Comprueba que `~/.local/bin` esté en tu `PATH`. También puedes ejecutar el script directamente desde la carpeta del repositorio:

```bash
./adb-parear
```

## Uso

```text
adb-parear                 Menú de selección
adb-parear --qr            QR → emparejar → conectar → verificar
adb-parear --code          IP:puerto/PIN → emparejar → conectar → verificar
adb-parear --list          Listar dispositivos ADB
adb-parear --timeout 180   Esperar hasta 180 segundos por el servicio QR
adb-parear --help          Mostrar ayuda
```

En el menú, usa las flechas ↑/↓ y Enter, o elige una opción con las teclas `1`–`5`. `q` cancela/sale. `Ctrl+C` interrumpe una espera.

## Flujo completo por QR

1. Ejecuta `adb-parear --qr` o elige **Emparejar por QR** en el menú.
2. En el teléfono, abre **Opciones de desarrollador → Depuración inalámbrica → Emparejar dispositivo con código QR**.
3. Escanea con el teléfono el QR que aparece en el terminal de Ubuntu.
4. El script detecta el servicio de emparejamiento por mDNS y ejecuta `adb pair`.
5. Después busca el servicio mDNS de conexión `_adb-tls-connect._tcp`, elige el dispositivo si encuentra varios y ejecuta `adb connect`.
6. Por último, consulta `adb devices -l`. Solo informa que ADB está listo si la dirección conectada aparece con el estado `device`.

El QR contiene un identificador temporal de servicio y un código de emparejamiento de corta duración. La dirección de conexión se descubre mediante mDNS en la red local; el script no consulta ni almacena la contraseña Wi-Fi.

## Flujo completo con PIN

Elige **Emparejar con código** o ejecuta `adb-parear --code`. Introduce el `IP:puerto` que el teléfono muestra en **Emparejar con código de emparejamiento** y escribe el PIN cuando ADB lo solicite. Tras el emparejamiento, el script también busca el servicio de conexión, ejecuta `adb connect` y verifica el estado `device`.

El puerto de emparejamiento y el puerto de conexión son distintos. Si mDNS no anuncia el servicio de conexión en 15 segundos, el programa solicitará el `IP:puerto` de conexión que aparece en la pantalla principal de **Depuración inalámbrica**. No introduzcas ahí el puerto de emparejamiento.

Para inspeccionar el servicio y conectarte manualmente, usa el endpoint publicado para `_adb-tls-connect._tcp` (no el de `_adb-tls-pairing._tcp`):

```bash
adb mdns services
adb connect IP:PUERTO_DE_CONEXION
adb devices -l
```

La conexión está lista cuando el dispositivo aparece en `adb devices -l` con estado `device`.

## Seguridad y alcance

El script envía solicitudes ADB únicamente a la dirección local descubierta o introducida. La autorización se completa con el QR o PIN y la acción en el teléfono. No cambia el firewall, no inicia servicios del sistema, no instala paquetes, no eleva privilegios y no guarda el código temporal. El menú y los argumentos no sustituyen la autorización que Android exige para ADB.

Emparejar autoriza el equipo Ubuntu para depuración ADB según las reglas de confianza de Android. Para retirar esa autorización, abre **Depuración inalámbrica → Dispositivos emparejados** en el teléfono y olvida esta estación de trabajo, o revoca las autorizaciones de depuración.

## Solución de problemas

- **No aparece `adb`:** instala Android SDK Platform-Tools y verifica `adb version`.
- **No aparece `qrencode`:** instala el paquete `qrencode` o utiliza el modo `--code`.
- **El teléfono no detecta el QR:** aumenta el tamaño de la terminal y comprueba que los caracteres UTF-8 se vean correctamente.
- **No aparece mDNS:** confirma que ambos equipos estén en la misma red y que la red no aísle los clientes. Si no aparece el servicio de conexión, escribe el puerto de conexión que muestra **Depuración inalámbrica** cuando el programa lo solicite.
- **`adb connect` devuelve `Connection refused`:** comprueba que no estás reutilizando el puerto del pairing. Emparejar puede devolver éxito y aun así requerir un puerto de conexión distinto.
- **No aparece estado `device`:** el script no considera la conexión lista; revisa la pantalla del móvil y vuelve a probar con `adb-parear`.

## Licencia

Este repositorio no incluye un archivo de licencia; la publicación no especifica permisos de reutilización, modificación o redistribución.

## Referencias

- [Android Debug Bridge: conexión por Wi-Fi](https://developer.android.com/tools/adb)
- [Ejecutar apps en un dispositivo de hardware: emparejar por Wi-Fi](https://developer.android.com/studio/run/device)
