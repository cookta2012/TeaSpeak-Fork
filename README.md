# TeaSpeak Voice Server v1.6.0 | Guía de compilación / Build Guide

[English](#english) | [Español](#español)

> **Compatibility / Compatibilidad:** The operating system compatibility and build output described here are from the supplied guide and have not been independently verified. / La compatibilidad con sistemas operativos y los resultados de compilación aquí descritos proceden de la guía proporcionada y no se han verificado de forma independiente.

---

# English

## 1. Voice server location and project layout

**Voice server source:** `Server/Server/server/`

```text
Server/
├── Root/                    Build and setup scripts
│   ├── build_teaspeak.sh    Main build script
│   ├── setup_amd64.md       Ubuntu/Debian instructions
│   ├── ubuntu_setup_env.sh
│   └── libraries/           Required libraries
├── Server/                  TeaSpeak project
│   ├── server/              VoiceServer
│   ├── license/             Licensing system
│   ├── MusicBot/            Music bot
│   ├── file/                File server
│   └── shared/              Shared code
├── rtc/                     WebRTC
└── teafile/                 File system
```

## 2. Operating systems and architectures

**Originally documented OS:** Ubuntu 14.04 LTS (Trusty Tahr). **Documentation:** `Root/setup_amd64.md`.

The original guide also lists Ubuntu 16.04, 18.04, 20.04, 22.04, Debian 8 through 12, and WSL2 Ubuntu. These compatibility claims are **unverified**; older releases and packages may require adjustments.

| Build value | Architecture |
| --- | --- |
| `amd64` | x86_64 (Intel/AMD 64-bit) |
| `arm64` | ARM 64-bit |
| `armv7` | ARM 32-bit |

The following commands target **amd64 Ubuntu/Debian**.

## 3. Install dependencies

### 3.1 Update the system

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

### 3.2 Basic tools and development libraries

```bash
sudo apt-get install -y \
    autoconf automake build-essential cmake curl gettext git \
    libcurl4-openssl-dev libssl-dev libpcre3-dev libncurses-dev \
    libsqlite3-dev libjemalloc-dev libtool realpath wget nano \
    software-properties-common unzip gcc g++ golang-go \
    default-libmysqlclient-dev
```

Package names and availability vary by distribution and release. The obsolete `python-software-properties` and redundant `libmysqlclient-dev` were removed from this combined command.

### 3.3 GCC 9 or newer

```bash
gcc --version
g++ --version
```

If the installed compiler is too old, install a supported compiler for your distribution. Where GCC 9 packages are available:

```bash
sudo apt-get install -y gcc-9 g++-9
```

Optionally select this compiler for the current shell, without changing the system-wide default:

```bash
export CC=gcc-9
export CXX=g++-9
```

The original PPA and system-wide `update-alternatives` changes are omitted because they are not appropriate for every listed OS version.

### 3.4 CMake 3.16 or newer

```bash
cmake --version
sudo apt-get install -y cmake
```

If your repository does not provide a new enough version, the original guide's CMake 3.16.3 source installation is:

```bash
cd /tmp
wget https://github.com/Kitware/CMake/releases/download/v3.16.3/cmake-3.16.3.tar.gz
tar xvf cmake-3.16.3.tar.gz
cd cmake-3.16.3
./bootstrap --parallel=4
make -j4
sudo make install
cmake --version
```

### 3.5 Git and Go

Both are included in the main dependency command. Check versions:

```bash
git --version
go version
```

If the project needs another Go version, install that version separately. The original guide proposed Go 1.19 for `linux-amd64`. Compiling Git 2.25.0 from source is normally unnecessary unless specifically required.

### 3.6 Patchelf, if required

```bash
sudo apt-get install -y patchelf
```

### 3.7 Protocol Buffers (Protobuf)

```bash
sudo apt-get install -y protobuf-compiler libprotobuf-dev
protoc --version
```

The original instructions cloned the unpinned Protobuf default branch. If building from source is necessary, use the version specified by the project.

### 3.8 SQLite3, if needed

The development library is already installed above. Install the command-line client if needed:

```bash
sudo apt-get install -y sqlite3
```

### 3.9 MySQL client libraries and Jemalloc

`default-libmysqlclient-dev` and `libjemalloc-dev` are already in the main dependency command. The original alternative to compile an entire MySQL server for client libraries has been omitted because its CMake instructions were inconsistent.

## 4. Clone the project

```bash
mkdir -p ~/teaspeak-build
cd ~/teaspeak-build
git clone https://git.did.science/WolverinDEV/TeaSpeak-Parent.git server
cd server
git submodule update --init --recursive
```

If the repository is inaccessible, use the source code already available. Using submodules without `--remote` checks out the revisions recorded by the parent repository rather than advancing to their remote branch heads.

## 5. Configure version 1.6.0

**File:** `~/teaspeak-build/server/Server/server/CMakeLists.txt`

Original packaging version:

```cmake
SET(CPACK_PACKAGE_VERSION_MAJOR "1")
SET(CPACK_PACKAGE_VERSION_MINOR "5")
SET(CPACK_PACKAGE_VERSION_PATCH "6")
```

Target packaging version:

```cmake
SET(CPACK_PACKAGE_VERSION_MAJOR "1")
SET(CPACK_PACKAGE_VERSION_MINOR "6")
SET(CPACK_PACKAGE_VERSION_PATCH "0")
```

To back up the file, apply the changes, and inspect the values:

```bash
cd ~/teaspeak-build/server/Server/server
cp CMakeLists.txt CMakeLists.txt.backup
sed -i 's/SET(CPACK_PACKAGE_VERSION_MINOR "5")/SET(CPACK_PACKAGE_VERSION_MINOR "6")/' CMakeLists.txt
sed -i 's/SET(CPACK_PACKAGE_VERSION_PATCH "6")/SET(CPACK_PACKAGE_VERSION_PATCH "0")/' CMakeLists.txt
grep 'CPACK_PACKAGE_VERSION' CMakeLists.txt
```

> **Note:** Changing packaging version variables does not necessarily change the runtime version or implement features introduced in TeaSpeak 1.6.0.

## 6. Configure the build environment

```bash
cd ~/teaspeak-build/server/Root
export build_os_type=linux
export build_os_arch=amd64
export CMAKE_MAKE_OPTIONS="-j$(nproc)"
export MAKE_OPTIONS="$CMAKE_MAKE_OPTIONS"
```

Optional optimization flags, if consumed by the build scripts:

```bash
export CXX_FLAGS="-O3 -march=native"
```

## 7. Build required libraries

```bash
cd ~/teaspeak-build/server/Root
./libraries/build.sh
```

Libraries listed by the original guide: `libevent`, `ed25519`, `jsoncpp`, `opus`, `protobuf`, `spdlog`, `StringVariable`, `ThreadPool`, `tomcrypt/tommath`, `yaml-cpp`, `zstd`, `breakpad`, `DataPipes`, and `CXXTerminal`.

## 8. Build TeaSpeak Server

**Release (production):**

```bash
cd ~/teaspeak-build/server/Root
./build_teaspeak.sh stable
```

**Debug (development):**

```bash
cd ~/teaspeak-build/server/Root
export i_really_wanna_debug=1
./build_teaspeak.sh debug
```

**Other modes mentioned in the original guide:**

```bash
./build_teaspeak.sh nightly
./build_teaspeak.sh optimized
```

## 9. Verify the build

### 9.1 Locate the generated files

```bash
cd ~/teaspeak-build/server/Server/server/environment
ls -lh
```

Depending on configuration, the original guide lists these possible outputs:

| File | Purpose |
| --- | --- |
| `TeaSpeakServer` | Main server executable |
| `libTeaFile.so` | File library |
| `ProviderFFMpeg.so` | FFmpeg provider |
| `ProviderYT.so` | YouTube provider |

### 9.2 Inspect the version

```bash
./TeaSpeakServer --version
```

Example from the original guide (the build ID is not verified):

```text
TeaSpeak-Server v1.6.0 [Build: 1699999999]
```

### 9.3 Quick launch test

```bash
mkdir -p ~/.config/TeaSpeak
cd ~/.config/TeaSpeak
~/teaspeak-build/server/Server/server/environment/TeaSpeakServer
```

The original guide notes that startup might stop without a required license; behavior depends on the actual built version and configuration.

## 10. Troubleshooting

| Problem | Suggested action |
| --- | --- |
| Cannot find MySQL | `sudo apt-get install -y default-libmysqlclient-dev` |
| CMake version too old | Run `cmake --version` and install CMake 3.16 or newer (see §3.4) |
| Protobuf not found | Run `protoc --version` and `sudo ldconfig`; check version and library paths |
| Cannot find libevent | Rebuild the project libraries (below) |
| Build is slow or memory-limited | Limit jobs to 2 (below) |

**Rebuild libraries:**

```bash
cd ~/teaspeak-build/server/Root/libraries
./build.sh
```

**Limit parallelism:**

```bash
export CMAKE_MAKE_OPTIONS="-j2"
export MAKE_OPTIONS="-j2"
```

**Error: `The source directory does not appear to contain CMakeLists.txt`**

The original guide suggests symbolic links for an expected directory layout. Verify the real repository structure first:

```bash
cd ~/teaspeak-build/server/Root
ln -s ../Server TeaSpeak
ln -s ../../rtc TeaSpeak/rtclib
ls -la TeaSpeak/CMakeLists.txt
ls -la TeaSpeak/rtclib/generate_shared_library.sh
```

These commands assume `Root/TeaSpeak` points to `../Server` and `rtc` is a sibling of `Server/` at the repository root. Do not create duplicate links or use these commands if the source layout differs.

## 11. Optional all-in-one build script

Save the following as `compile_teaspeak_1.6.0.sh` **after obtaining the source**. It installs dependencies, updates packaging version variables, sets the environment, and builds libraries and the server. Confirm that your compiler, CMake, and Protobuf versions satisfy the project before running it.

```bash
#!/usr/bin/env bash
set -euo pipefail

REPO_DIR="$HOME/teaspeak-build/server"
echo '[1/6] Installing dependencies...'
sudo apt-get update
sudo apt-get install -y \
    build-essential cmake gcc g++ git wget \
    libcurl4-openssl-dev libssl-dev libpcre3-dev libncurses-dev \
    libsqlite3-dev libjemalloc-dev golang-go autoconf automake \
    libtool default-libmysqlclient-dev protobuf-compiler libprotobuf-dev

echo '[2/6] Checking source directory...'
cd "$REPO_DIR"

echo '[3/6] Setting packaging version to 1.6.0...'
cd Server/server
cp -n CMakeLists.txt CMakeLists.txt.backup || true
sed -i 's/SET(CPACK_PACKAGE_VERSION_MINOR "5")/SET(CPACK_PACKAGE_VERSION_MINOR "6")/' CMakeLists.txt
sed -i 's/SET(CPACK_PACKAGE_VERSION_PATCH "6")/SET(CPACK_PACKAGE_VERSION_PATCH "0")/' CMakeLists.txt

echo '[4/6] Setting build variables...'
cd "$REPO_DIR/Root"
export build_os_type=linux
export build_os_arch=amd64
export CMAKE_MAKE_OPTIONS="-j$(nproc)"
export MAKE_OPTIONS="$CMAKE_MAKE_OPTIONS"

echo '[5/6] Building libraries...'
./libraries/build.sh

echo '[6/6] Building TeaSpeak Server...'
./build_teaspeak.sh stable

BINARY="$REPO_DIR/Server/server/environment/TeaSpeakServer"
if [[ -f "$BINARY" ]]; then
    echo "Build completed: $BINARY"
    "$BINARY" --version
else
    echo 'Build failed: TeaSpeakServer executable was not found.' >&2
    exit 1
fi
```

To run:

```bash
chmod +x compile_teaspeak_1.6.0.sh
./compile_teaspeak_1.6.0.sh
```

## 12. Additional information

Figures from the original guide; **not verified**:

| Item | Estimate |
| --- | --- |
| Source code | ~500 MB |
| Built dependencies | ~2 GB |
| Final binary | ~50 MB |
| 4-core CPU | ~45–60 minutes |
| 8-core CPU | ~25–35 minutes |
| 16-core CPU | ~15–20 minutes |

Server components described by the original guide:

```text
TeaSpeakServer
├── VoiceServer
├── QueryServer
├── WebServer
├── LicenseService
└── FileServer
```

---

# Español

## 1. Ubicación del servidor de voz y estructura del proyecto

**Código fuente del servidor de voz:** `Server/Server/server/`

```text
Server/
├── Root/                    Scripts de compilación y configuración
│   ├── build_teaspeak.sh    Script principal de compilación
│   ├── setup_amd64.md       Instrucciones para Ubuntu/Debian
│   ├── ubuntu_setup_env.sh
│   └── libraries/           Bibliotecas necesarias
├── Server/                  Proyecto TeaSpeak
│   ├── server/              Servidor de voz (VoiceServer)
│   ├── license/             Sistema de licencias
│   ├── MusicBot/            Bot de música
│   ├── file/                Servidor de archivos
│   └── shared/              Código compartido
├── rtc/                     WebRTC
└── teafile/                 Sistema de archivos
```

## 2. Sistemas operativos y arquitecturas

**Sistema operativo documentado originalmente:** Ubuntu 14.04 LTS (Trusty Tahr). **Documentación:** `Root/setup_amd64.md`.

La guía original también menciona Ubuntu 16.04, 18.04, 20.04 y 22.04, Debian 8 a 12 y Ubuntu bajo WSL2. Estas afirmaciones de compatibilidad **no se han verificado**; las versiones antiguas y algunos paquetes pueden requerir ajustes.

| Valor de compilación | Arquitectura |
| --- | --- |
| `amd64` | x86_64 (Intel/AMD de 64 bits) |
| `arm64` | ARM de 64 bits |
| `armv7` | ARM de 32 bits |

Los siguientes comandos están dirigidos a **Ubuntu/Debian en amd64**.

## 3. Instalar dependencias

### 3.1 Actualizar el sistema

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

### 3.2 Herramientas básicas y bibliotecas de desarrollo

```bash
sudo apt-get install -y \
    autoconf automake build-essential cmake curl gettext git \
    libcurl4-openssl-dev libssl-dev libpcre3-dev libncurses-dev \
    libsqlite3-dev libjemalloc-dev libtool realpath wget nano \
    software-properties-common unzip gcc g++ golang-go \
    default-libmysqlclient-dev
```

Los nombres y la disponibilidad de los paquetes varían según la distribución y la versión. Se eliminaron `python-software-properties`, ya obsoleto, y la entrada redundante `libmysqlclient-dev` del comando combinado.

### 3.3 GCC 9 o superior

```bash
gcc --version
g++ --version
```

Si el compilador instalado es demasiado antiguo, instale uno compatible con su distribución. Si están disponibles los paquetes de GCC 9:

```bash
sudo apt-get install -y gcc-9 g++-9
```

Opcionalmente, seleccione este compilador en la sesión actual sin cambiar la configuración predeterminada del sistema:

```bash
export CC=gcc-9
export CXX=g++-9
```

Se omiten el PPA original y las modificaciones globales mediante `update-alternatives`, ya que no son adecuados para todas las versiones de sistemas operativos indicadas.

### 3.4 CMake 3.16 o superior

```bash
cmake --version
sudo apt-get install -y cmake
```

Si los repositorios no ofrecen una versión suficientemente reciente, la instalación desde el código fuente de CMake 3.16.3 indicada en la guía original es:

```bash
cd /tmp
wget https://github.com/Kitware/CMake/releases/download/v3.16.3/cmake-3.16.3.tar.gz
tar xvf cmake-3.16.3.tar.gz
cd cmake-3.16.3
./bootstrap --parallel=4
make -j4
sudo make install
cmake --version
```

### 3.5 Git y Go

Ambos están incluidos en el comando principal de dependencias. Compruebe sus versiones:

```bash
git --version
go version
```

Si el proyecto requiere otra versión de Go, instálela por separado. La guía original proponía Go 1.19 para `linux-amd64`. Normalmente no es necesario compilar Git 2.25.0 desde el código fuente, salvo que el proyecto lo requiera específicamente.

### 3.6 Patchelf, si es necesario

```bash
sudo apt-get install -y patchelf
```

### 3.7 Protocol Buffers (Protobuf)

```bash
sudo apt-get install -y protobuf-compiler libprotobuf-dev
protoc --version
```

Las instrucciones originales clonaban la rama predeterminada de Protobuf sin fijar una versión. Si necesita compilarlo desde el código fuente, utilice la versión especificada por el proyecto.

### 3.8 SQLite3, si es necesario

La biblioteca de desarrollo ya está incluida. Si necesita el cliente de línea de comandos:

```bash
sudo apt-get install -y sqlite3
```

### 3.9 Bibliotecas cliente de MySQL y Jemalloc

`default-libmysqlclient-dev` y `libjemalloc-dev` ya están incluidos en el comando principal de dependencias. Se omite la alternativa original de compilar todo el servidor MySQL para obtener las bibliotecas cliente, porque contenía instrucciones de CMake inconsistentes.

## 4. Clonar el proyecto

```bash
mkdir -p ~/teaspeak-build
cd ~/teaspeak-build
git clone https://git.did.science/WolverinDEV/TeaSpeak-Parent.git server
cd server
git submodule update --init --recursive
```

Si no puede acceder al repositorio, utilice el código fuente que ya tenga. Al omitir `--remote`, los submódulos se obtienen en las revisiones registradas por el repositorio principal en lugar de avanzar hasta las últimas revisiones de sus ramas remotas.

## 5. Configurar la versión 1.6.0

**Archivo:** `~/teaspeak-build/server/Server/server/CMakeLists.txt`

Versión original del paquete:

```cmake
SET(CPACK_PACKAGE_VERSION_MAJOR "1")
SET(CPACK_PACKAGE_VERSION_MINOR "5")
SET(CPACK_PACKAGE_VERSION_PATCH "6")
```

Versión de paquete deseada:

```cmake
SET(CPACK_PACKAGE_VERSION_MAJOR "1")
SET(CPACK_PACKAGE_VERSION_MINOR "6")
SET(CPACK_PACKAGE_VERSION_PATCH "0")
```

Para realizar una copia de seguridad, aplicar los cambios y comprobar los valores:

```bash
cd ~/teaspeak-build/server/Server/server
cp CMakeLists.txt CMakeLists.txt.backup
sed -i 's/SET(CPACK_PACKAGE_VERSION_MINOR "5")/SET(CPACK_PACKAGE_VERSION_MINOR "6")/' CMakeLists.txt
sed -i 's/SET(CPACK_PACKAGE_VERSION_PATCH "6")/SET(CPACK_PACKAGE_VERSION_PATCH "0")/' CMakeLists.txt
grep 'CPACK_PACKAGE_VERSION' CMakeLists.txt
```

> **Nota:** Cambiar las variables de versión del paquete no cambia necesariamente la versión mostrada en ejecución ni incorpora las funciones introducidas en TeaSpeak 1.6.0.

## 6. Configurar el entorno de compilación

```bash
cd ~/teaspeak-build/server/Root
export build_os_type=linux
export build_os_arch=amd64
export CMAKE_MAKE_OPTIONS="-j$(nproc)"
export MAKE_OPTIONS="$CMAKE_MAKE_OPTIONS"
```

Opcionalmente, indicadores de optimización si los scripts de compilación los utilizan:

```bash
export CXX_FLAGS="-O3 -march=native"
```

## 7. Compilar las bibliotecas necesarias

```bash
cd ~/teaspeak-build/server/Root
./libraries/build.sh
```

Bibliotecas mencionadas en la guía original: `libevent`, `ed25519`, `jsoncpp`, `opus`, `protobuf`, `spdlog`, `StringVariable`, `ThreadPool`, `tomcrypt/tommath`, `yaml-cpp`, `zstd`, `breakpad`, `DataPipes` y `CXXTerminal`.

## 8. Compilar TeaSpeak Server

**Release (producción):**

```bash
cd ~/teaspeak-build/server/Root
./build_teaspeak.sh stable
```

**Debug (desarrollo):**

```bash
cd ~/teaspeak-build/server/Root
export i_really_wanna_debug=1
./build_teaspeak.sh debug
```

**Otros modos mencionados en la guía original:**

```bash
./build_teaspeak.sh nightly
./build_teaspeak.sh optimized
```

## 9. Verificar la compilación

### 9.1 Localizar los archivos generados

```bash
cd ~/teaspeak-build/server/Server/server/environment
ls -lh
```

Según la configuración, la guía original menciona los siguientes archivos posibles:

| Archivo | Función |
| --- | --- |
| `TeaSpeakServer` | Ejecutable principal del servidor |
| `libTeaFile.so` | Biblioteca de archivos |
| `ProviderFFMpeg.so` | Proveedor FFmpeg |
| `ProviderYT.so` | Proveedor YouTube |

### 9.2 Comprobar la versión

```bash
./TeaSpeakServer --version
```

Ejemplo de la guía original (el identificador de compilación no está verificado):

```text
TeaSpeak-Server v1.6.0 [Build: 1699999999]
```

### 9.3 Prueba rápida de inicio

```bash
mkdir -p ~/.config/TeaSpeak
cd ~/.config/TeaSpeak
~/teaspeak-build/server/Server/server/environment/TeaSpeakServer
```

La guía original señala que el inicio podría detenerse si falta una licencia necesaria; el comportamiento real depende de la versión compilada y su configuración.

## 10. Solución de problemas

| Problema | Acción sugerida |
| --- | --- |
| No se encuentra MySQL | `sudo apt-get install -y default-libmysqlclient-dev` |
| Versión de CMake demasiado antigua | Ejecute `cmake --version` e instale CMake 3.16 o superior (véase §3.4) |
| No se encuentra Protobuf | Ejecute `protoc --version` y `sudo ldconfig`; revise la versión y las rutas de las bibliotecas |
| No se encuentra libevent | Vuelva a compilar las bibliotecas del proyecto (abajo) |
| Compilación lenta o memoria insuficiente | Limite el número de trabajos paralelos a 2 (abajo) |

**Volver a compilar las bibliotecas:**

```bash
cd ~/teaspeak-build/server/Root/libraries
./build.sh
```

**Limitar el paralelismo:**

```bash
export CMAKE_MAKE_OPTIONS="-j2"
export MAKE_OPTIONS="-j2"
```

**Error: `The source directory does not appear to contain CMakeLists.txt`**

La guía original propone crear enlaces simbólicos para reproducir una estructura de directorios esperada. Primero compruebe la estructura real del repositorio:

```bash
cd ~/teaspeak-build/server/Root
ln -s ../Server TeaSpeak
ln -s ../../rtc TeaSpeak/rtclib
ls -la TeaSpeak/CMakeLists.txt
ls -la TeaSpeak/rtclib/generate_shared_library.sh
```

Estos comandos suponen que `Root/TeaSpeak` apunta a `../Server` y que `rtc` está junto a `Server/` en la raíz del repositorio. No cree enlaces duplicados ni utilice estos comandos si la estructura del código fuente es diferente.

## 11. Script opcional de compilación integral

Guarde lo siguiente como `compile_teaspeak_1.6.0.sh` **después de obtener el código fuente**. Instala dependencias, actualiza las variables de versión del paquete, configura el entorno y compila las bibliotecas y el servidor. Antes de ejecutarlo, compruebe que las versiones de GCC, CMake y Protobuf cumplen los requisitos del proyecto.

```bash
#!/usr/bin/env bash
set -euo pipefail

REPO_DIR="$HOME/teaspeak-build/server"
echo '[1/6] Installing dependencies...'
sudo apt-get update
sudo apt-get install -y \
    build-essential cmake gcc g++ git wget \
    libcurl4-openssl-dev libssl-dev libpcre3-dev libncurses-dev \
    libsqlite3-dev libjemalloc-dev golang-go autoconf automake \
    libtool default-libmysqlclient-dev protobuf-compiler libprotobuf-dev

echo '[2/6] Checking source directory...'
cd "$REPO_DIR"

echo '[3/6] Setting packaging version to 1.6.0...'
cd Server/server
cp -n CMakeLists.txt CMakeLists.txt.backup || true
sed -i 's/SET(CPACK_PACKAGE_VERSION_MINOR "5")/SET(CPACK_PACKAGE_VERSION_MINOR "6")/' CMakeLists.txt
sed -i 's/SET(CPACK_PACKAGE_VERSION_PATCH "6")/SET(CPACK_PACKAGE_VERSION_PATCH "0")/' CMakeLists.txt

echo '[4/6] Setting build variables...'
cd "$REPO_DIR/Root"
export build_os_type=linux
export build_os_arch=amd64
export CMAKE_MAKE_OPTIONS="-j$(nproc)"
export MAKE_OPTIONS="$CMAKE_MAKE_OPTIONS"

echo '[5/6] Building libraries...'
./libraries/build.sh

echo '[6/6] Building TeaSpeak Server...'
./build_teaspeak.sh stable

BINARY="$REPO_DIR/Server/server/environment/TeaSpeakServer"
if [[ -f "$BINARY" ]]; then
    echo "Build completed: $BINARY"
    "$BINARY" --version
else
    echo 'Build failed: TeaSpeakServer executable was not found.' >&2
    exit 1
fi
```

Para ejecutarlo:

```bash
chmod +x compile_teaspeak_1.6.0.sh
./compile_teaspeak_1.6.0.sh
```

## 12. Información adicional

Cifras de la guía original, **sin verificar**:

| Elemento | Estimación |
| --- | --- |
| Código fuente | ~500 MB |
| Dependencias compiladas | ~2 GB |
| Binario final | ~50 MB |
| CPU de 4 núcleos | ~45–60 minutos |
| CPU de 8 núcleos | ~25–35 minutos |
| CPU de 16 núcleos | ~15–20 minutos |

Componentes del servidor descritos en la guía original:

```text
TeaSpeakServer
├── VoiceServer
├── QueryServer
├── WebServer
├── LicenseService
└── FileServer
```
