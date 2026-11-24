# adb-parear

Una utilidad Bash para emparejar un teléfono Android con ADB inalámbrico desde Ubuntu. Incluye un menú interactivo, emparejamiento por QR, modo de código manual y listado de dispositivos ADB.

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
adb-parear --qr            Mostrar el QR y esperar el servicio de emparejamiento
adb-parear --code          Escribir IP:puerto y luego el PIN que pide adb
adb-parear --list          Listar dispositivos ADB
adb-parear --timeout 180   Esperar hasta 180 segundos en modo QR
adb-parear --help          Mostrar ayuda
```

En el menú, usa las flechas ↑/↓ y Enter, o elige una opción con las teclas `1`–`5`. `q` cancela/sale. `Ctrl+C` interrumpe una espera.

## Emparejar por QR

1. Ejecuta `adb-parear --qr` o elige **Emparejar por QR** en el menú.
2. En el teléfono, abre **Opciones de desarrollador → Depuración inalámbrica → Emparejar dispositivo con código QR**.
3. Escanea con el teléfono el QR que aparece en el terminal de Ubuntu.
4. El script detecta los servicios de emparejamiento por mDNS. Si encuentra varios, pregunta cuál teléfono escaneó el QR.
5. ADB completa el emparejamiento y muestra el estado de dispositivos.

El QR contiene un identificador temporal de servicio y un código de emparejamiento de corta duración. La dirección IP y el puerto se descubren mediante mDNS en la red local; el script no consulta ni almacena la contraseña Wi-Fi.

## Emparejar con PIN

Elige **Emparejar con código** o ejecuta `adb-parear --code`. Cuando el teléfono muestre **Emparejar con código de emparejamiento**, introduce la dirección `IP:puerto` y escribe el PIN cuando ADB lo solicite en el terminal.

## Seguridad y alcance

El script envía la solicitud de emparejamiento a la dirección local detectada o introducida. La autorización se completa con el QR o PIN y la acción en el teléfono. No cambia el firewall, no inicia servicios del sistema, no instala paquetes, no eleva privilegios y no guarda el código temporal. El menú y los argumentos no sustituyen la autorización que Android exige para ADB.

Emparejar autoriza el equipo Ubuntu para depuración ADB según las reglas de confianza de Android. Para retirar esa autorización, abre **Depuración inalámbrica → Dispositivos emparejados** en el teléfono y olvida esta estación de trabajo, o revoca las autorizaciones de depuración.

## Solución de problemas

- **No aparece `adb`:** instala Android SDK Platform-Tools y verifica `adb version`.
- **No aparece `qrencode`:** instala el paquete `qrencode` o utiliza el modo `--code`.
- **El teléfono no detecta el QR:** aumenta el tamaño de la terminal y comprueba que los caracteres UTF-8 se vean correctamente.
- **Se agota el tiempo de espera:** confirma que el QR fue escaneado, que la depuración inalámbrica sigue activa y que ambos equipos están en la misma red. Las redes de invitados pueden bloquear mDNS mediante aislamiento de clientes; prueba una red privada compartida o usa el modo de código.
- **Emparejado pero no conectado:** el emparejamiento y la conexión ADB son estados separados. Consulta `adb devices -l`; si tu versión/flujo de Android no conecta automáticamente, usa la dirección de conexión que muestra **Depuración inalámbrica** con `adb connect IP:puerto`.

## Licencia

Este repositorio no incluye un archivo de licencia; la publicación no especifica permisos de reutilización, modificación o redistribución.

## Referencias

- [Android Debug Bridge: conexión por Wi-Fi](https://developer.android.com/tools/adb)
- [Ejecutar apps en un dispositivo de hardware: emparejar por Wi-Fi](https://developer.android.com/studio/run/device)
