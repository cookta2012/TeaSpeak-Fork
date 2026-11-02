BUILD GUIDE - TeaSpeak Voice Server v1.6.0
==========================================

1. VOICE SERVER LOCATION AND PROJECT LAYOUT
-------------------------------------------
Voice server source: Server/Server/server/

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

2. OPERATING SYSTEM AND ARCHITECTURES
-------------------------------------
Original documented OS: Ubuntu 14.04 LTS (Trusty Tahr)
Documentation: Root/setup_amd64.md

The original guide also lists Ubuntu 16.04, 18.04, 20.04, 22.04,
Debian 8 through 12, and WSL2 Ubuntu. Those compatibility claims have
not been verified; older releases and packages may require adjustments.

Architecture values used by the build scripts:
  amd64  = x86_64 (Intel/AMD 64-bit)
  arm64  = ARM 64-bit
  armv7  = ARM 32-bit

The commands below target amd64 Ubuntu/Debian.

STEP 1: INSTALL DEPENDENCIES
----------------------------
1.1 Update package lists and installed packages

    sudo apt-get update
    sudo apt-get upgrade -y

1.2 Basic build tools and development libraries

    sudo apt-get install -y \
        autoconf automake build-essential cmake curl gettext git \
        libcurl4-openssl-dev libssl-dev libpcre3-dev libncurses-dev \
        libsqlite3-dev libjemalloc-dev libtool realpath wget nano \
        software-properties-common unzip gcc g++ golang-go \
        default-libmysqlclient-dev

Note: Package names and availability differ by distribution and release.
The obsolete python-software-properties package and redundant
libmysqlclient-dev entry have been removed from the combined command.

1.3 GCC 9 or newer

    gcc --version
    g++ --version

If the installed compiler is too old, install an available supported
compiler for your distribution. For systems that provide gcc-9 and g++-9:

    sudo apt-get install -y gcc-9 g++-9

Optional: select GCC 9 explicitly, without changing the system-wide default:

    export CC=gcc-9
    export CXX=g++-9

The old guide used the ubuntu-toolchain-r/test PPA and changed gcc, g++,
cc, and c++ system-wide. Those commands are intentionally omitted here
because they are not generally applicable to all listed OS versions.

1.4 CMake 3.16 or newer

    cmake --version

On distributions offering a sufficiently recent version:

    sudo apt-get install -y cmake

Optional source installation of the version named by the original guide:

    cd /tmp
    wget https://github.com/Kitware/CMake/releases/download/v3.16.3/cmake-3.16.3.tar.gz
    tar xvf cmake-3.16.3.tar.gz
    cd cmake-3.16.3
    ./bootstrap --parallel=4
    make -j4
    sudo make install
    cmake --version

1.5 Git

    git --version

The normal package installation above includes Git. The old instructions
to compile Git 2.25.0 are unnecessary unless a particular version is
required by the project.

1.6 Go

    go version

The normal package installation above includes golang-go. If the project
requires a different Go version, install that specific version separately.
The original alternative was Go 1.19 for linux-amd64.

1.7 Patchelf (if required by the build)

    sudo apt-get install -y patchelf

1.8 Protocol Buffers (Protobuf)

First try distribution packages:

    sudo apt-get install -y protobuf-compiler libprotobuf-dev
    protoc --version

The original guide built Protobuf from the current, unpinned Git default
branch. Doing so could select an incompatible version. Use the project's
specified version if a source build is required.

1.9 SQLite3 (if needed)

The development library is included in step 1.2. If command-line SQLite
is needed:

    sudo apt-get install -y sqlite3

1.10 MySQL client libraries

The default-libmysqlclient-dev package is included in step 1.2.
The original alternative attempted to compile the entire MySQL server
just to obtain client libraries, and included inconsistent CMake paths;
it has been omitted.

1.11 Jemalloc

The libjemalloc-dev package is included in step 1.2.

STEP 2: CLONE THE PROJECT
-------------------------
    mkdir -p ~/teaspeak-build
    cd ~/teaspeak-build
    git clone https://git.did.science/WolverinDEV/TeaSpeak-Parent.git server
    cd server
    git submodule update --init --recursive

If you cannot access the repository, use the source code you already have.
The original command used --remote, which advances submodules to their
remote branches rather than checking out the revisions recorded by the
parent project. This guide uses the recorded revisions instead.

STEP 3: CONFIGURE VERSION 1.6.0
--------------------------------
File: ~/teaspeak-build/server/Server/server/CMakeLists.txt

Original version fields:

    SET(CPACK_PACKAGE_VERSION_MAJOR "1")
    SET(CPACK_PACKAGE_VERSION_MINOR "5")
    SET(CPACK_PACKAGE_VERSION_PATCH "6")

Target version fields:

    SET(CPACK_PACKAGE_VERSION_MAJOR "1")
    SET(CPACK_PACKAGE_VERSION_MINOR "6")
    SET(CPACK_PACKAGE_VERSION_PATCH "0")

Edit and check the file:

    cd ~/teaspeak-build/server/Server/server
    cp CMakeLists.txt CMakeLists.txt.backup
    sed -i 's/SET(CPACK_PACKAGE_VERSION_MINOR "5")/SET(CPACK_PACKAGE_VERSION_MINOR "6")/' CMakeLists.txt
    sed -i 's/SET(CPACK_PACKAGE_VERSION_PATCH "6")/SET(CPACK_PACKAGE_VERSION_PATCH "0")/' CMakeLists.txt
    grep 'CPACK_PACKAGE_VERSION' CMakeLists.txt

NOTE: Changing packaging version variables does not necessarily change
runtime version strings or implement changes introduced in TeaSpeak 1.6.0.

STEP 4: CONFIGURE BUILD ENVIRONMENT
------------------------------------
    cd ~/teaspeak-build/server/Root
    export build_os_type=linux
    export build_os_arch=amd64
    export CMAKE_MAKE_OPTIONS="-j$(nproc)"
    export MAKE_OPTIONS="$CMAKE_MAKE_OPTIONS"

Optional optimization flags (only if consumed by the build scripts):

    export CXX_FLAGS="-O3 -march=native"

STEP 5: BUILD REQUIRED LIBRARIES
---------------------------------
    cd ~/teaspeak-build/server/Root
    ./libraries/build.sh

Libraries identified by the original guide:
  libevent, ed25519, jsoncpp, opus, protobuf, spdlog,
  StringVariable, ThreadPool, tomcrypt/tommath, yaml-cpp,
  zstd, breakpad, DataPipes, CXXTerminal.

STEP 6: BUILD TEASPEAK SERVER
------------------------------
Release (production):

    cd ~/teaspeak-build/server/Root
    ./build_teaspeak.sh stable

Debug (development):

    cd ~/teaspeak-build/server/Root
    export i_really_wanna_debug=1
    ./build_teaspeak.sh debug

Other build modes listed by the original guide:

    ./build_teaspeak.sh nightly
    ./build_teaspeak.sh optimized

STEP 7: VERIFY THE BUILD
-------------------------
7.1 Locate generated files

    cd ~/teaspeak-build/server/Server/server/environment
    ls -lh

Files listed by the original guide (depending on build configuration):
  TeaSpeakServer       Main server executable
  libTeaFile.so        File library
  ProviderFFMpeg.so   FFmpeg provider
  ProviderYT.so       YouTube provider

7.2 Inspect the version

    ./TeaSpeakServer --version

The original guide expected a line similar to:

    TeaSpeak-Server v1.6.0 [Build: 1699999999]

The build ID above is only the original guide's example, not a verified value.

7.3 Quick launch test

    mkdir -p ~/.config/TeaSpeak
    cd ~/.config/TeaSpeak
    ~/teaspeak-build/server/Server/server/environment/TeaSpeakServer

The original guide notes that startup might stop if a required license
is missing. Actual behavior depends on the built version and configuration.

TROUBLESHOOTING
---------------
Cannot find MySQL:

    sudo apt-get install -y default-libmysqlclient-dev

CMake version too old:

    cmake --version

Install CMake 3.16 or newer using the package manager, or use the
source-build instructions in step 1.4.

Protobuf not found:

    protoc --version
    sudo ldconfig

Check whether the build requires a specific Protobuf version or library path.

Cannot find libevent:

    cd ~/teaspeak-build/server/Root/libraries
    ./build.sh

Build is too slow or consumes too much memory:

    export CMAKE_MAKE_OPTIONS="-j2"
    export MAKE_OPTIONS="-j2"

Source directory does not contain CMakeLists.txt:

The original guide suggests these symlinks, but verify the actual
repository layout before creating them:

    cd ~/teaspeak-build/server/Root
    ln -s ../Server TeaSpeak
    ln -s ../../rtc TeaSpeak/rtclib

Checks:

    ls -la TeaSpeak/CMakeLists.txt
    ls -la TeaSpeak/rtclib/generate_shared_library.sh

The symlink locations in the original guide were inconsistent. These
commands assume Root/TeaSpeak links to ../Server and the rtc directory
is a sibling of Server/ in the repository root. Do not use the symlinks
if those source paths already exist or the project expects another layout.

OPTIONAL: ALL-IN-ONE BUILD SCRIPT
---------------------------------
Save as compile_teaspeak_1.6.0.sh after obtaining the source repository.
This follows steps 1, 3, 4, 5, and 6. It uses the available system GCC,
CMake and Protobuf versions; requirements must be checked beforehand.

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

Run the saved script with:

    chmod +x compile_teaspeak_1.6.0.sh
    ./compile_teaspeak_1.6.0.sh

ADDITIONAL INFORMATION
----------------------
Original guide's approximate figures (not verified):
  Source code:           ~500 MB
  Built dependencies:    ~2 GB
  Final binary:          ~50 MB
  4-core CPU:            ~45-60 minutes
  8-core CPU:            ~25-35 minutes
  16-core CPU:           ~15-20 minutes

Server components described by the original guide:

TeaSpeakServer
├── VoiceServer
├── QueryServer
├── WebServer
├── LicenseService
└── FileServer

END OF GUIDE

--------------

ESPANOL:
GUÍA DE COMPILACIÓN - Servidor de Voz TeaSpeak
v1.6.0
RESPUESTA 1: Carpeta del Servidor de Voz
Ubicación Principal
Server/Server/server/     ← SERVIDOR DE VOZ (VoiceServer)
Server/Server/server/     ← SERVIDOR DE VOZ (VoiceServer)
Estructura Completa del Proyecto
Server/
Server/
├── Root/                  ← Scripts de compilación y setup
├── Root/                  ← Scripts de compilación y setup
│   ├── build_teaspeak.sh  ← Script principal de build
│   ├── build_teaspeak.sh  ← Script principal de build
│   ├── setup_amd64.md     ← Instrucciones Ubuntu/Debian
│   ├── setup_amd64.md     ← Instrucciones Ubuntu/Debian
│   ├── ubuntu_setup_env.sh
│   ├── ubuntu_setup_env.sh
│   └── libraries/         ← Librerías necesarias
│   └── libraries/         ← Librerías necesarias
├── Server/                ← Proyecto TeaSpeak
├── Server/                ← Proyecto TeaSpeak
│   ├── server/            ← 🎯 SERVIDOR DE VOZ (VoiceServer)
│   ├── server/            ← 🎯 SERVIDOR DE VOZ (VoiceServer)
│   ├── license/           ← Sistema de licencias
│   ├── license/           ← Sistema de licencias
│   ├── MusicBot/          ← Bot de música
│   ├── MusicBot/          ← Bot de música
│   ├── file/              ← Servidor de archivos
│   ├── file/              ← Servidor de archivos
│   └── shared/            ← Código compartido
│   └── shared/            ← Código compartido
├── rtc/                   ← WebRTC
├── rtc/                   ← WebRTC
└── teafile/               ← Sistema de archivos
└── teafile/               ← Sistema de archivos
RESPUESTA 2: Sistema Operativo de las Instrucciones
Sistema Operativo: Ubuntu 14.04 LTS (Trusty Tahr)
Archivo: Root/setup_amd64.md  - Línea 2
Compatibilidad
Las instrucciones funcionan en:
✅ Ubuntu 14.04 (base original)
✅ Ubuntu 16.04, 18.04, 20.04, 22.04 (con ajustes menores)
✅ Debian 8, 9, 10, 11, 12 (equivalente)
✅ WSL2 Ubuntu (Windows Subsystem for Linux)
Arquitecturas Soportadas
build_os_arch = amd64    ← x86_64 (Intel/AMD 64-bit)
build_os_arch = amd64    ← x86_64 (Intel/AMD 64-bit)
build_os_arch = arm64    ← ARM 64-bit
build_os_arch = arm64    ← ARM 64-bit
build_os_arch = armv7    ← ARM 32-bit
build_os_arch = armv7    ← ARM 32-bit
PASO 1: DEPENDENCIAS PARA UBUNTU/DEBIAN
1.1 Actualizar Sistema
bash
sudosudo  apt-get
apt-get update
 update
sudosudo  apt-get
apt-get upgrade -y
 upgrade -y
1.2 Dependencias Básicas
bash
sudosudo  apt-get
apt-get  install
install -y  -y \\
        sudosudo  \\
    autoconf 
    autoconf \\
        gitgit  \\
        wgetwget  \\
    gettext 
    gettext \\
    build-essential 
    build-essential \\
    realpath 
    realpath \\
        nanonano  \\
        curlcurl  \\
    software-properties-common 
    software-properties-common \\
    python-software-properties
    python-software-properties
1.3 Librerías de Desarrollo
bash
sudosudo  apt-get
apt-get  install
install -y  -y \\
    libcurl4-openssl-dev 
    libcurl4-openssl-dev \\
    libssl-dev 
    libssl-dev \\
    libmysqlclient-dev 
    libmysqlclient-dev \\
    libpcre3-dev 
    libpcre3-dev \\
    libncurses5-dev 
    libncurses5-dev \\
    libsqlite3-dev
    libsqlite3-dev
1.4 Compilador GCC 9 o Superior
Opción A: Desde PPA (Recomendado para Ubuntu)
bash
sudosudo add-apt-repository ppa:ubuntu-toolchain-r/test -y
 add-apt-repository ppa:ubuntu-toolchain-r/test -y
sudosudo  apt-get
apt-get update
 update
sudosudo  apt-get
apt-get  install
install -y gcc-9 g++-9
 -y gcc-9 g++-9
# Configurar como predeterminado
# Configurar como predeterminado
sudosudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 
 update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 6060  \\
    --slave /usr/bin/g++ g++ /usr/bin/g++-9
    --slave /usr/bin/g++ g++ /usr/bin/g++-9
sudosudo update-alternatives --install /usr/bin/cc cc /usr/bin/gcc-9 
 update-alternatives --install /usr/bin/cc cc /usr/bin/gcc-9 6060  \\
    --slave /usr/bin/c++ c++ /usr/bin/g++-9
    --slave /usr/bin/c++ c++ /usr/bin/g++-9
Opción B: Para Debian o versiones más recientes
bash
sudosudo  apt-get
apt-get  install
install -y gcc g++
 -y gcc g++
# Debian 10+ y Ubuntu 20.04+ ya tienen GCC 9+
# Debian 10+ y Ubuntu 20.04+ ya tienen GCC 9+
Verificar versión:
bash
gcc --version
gcc --version
# Debe mostrar: gcc (Ubuntu 9.x.x o superior)
# Debe mostrar: gcc (Ubuntu 9.x.x o superior)
1.5 CMake 3.16 o Superior
Opción A: Desde Repositorios (Ubuntu 20.04+)
bash
sudosudo  apt-get
apt-get  install
install -y cmake
 -y cmake
Opción B: Compilar desde Fuentes (Ubuntu 14.04-18.04)
bash
cdcd /tmp
 /tmp
wgetwget https://github.com/Kitware/CMake/releases/download/v3.16.3/cmake-3.16.3.tar.gz
 https://github.com/Kitware/CMake/releases/download/v3.16.3/cmake-3.16.3.tar.gz
tartar xvf cmake-3.16.3.tar.gz
 xvf cmake-3.16.3.tar.gz
cdcd cmake-3.16.3
 cmake-3.16.3
./configure --parallel
./configure --parallel==44
makemake -j4 -j4
sudosudo  makemake  install
install
# Verificar
# Verificar
cmake --version
cmake --version
1.6 Git (Versión Reciente)
bash
# Si tu versión de git es antigua (< 2.0), instalar desde source:
# Si tu versión de git es antigua (< 2.0), instalar desde source:
cdcd /tmp
 /tmp
wgetwget https://github.com/git/git/archive/v2.25.0.tar.gz
 https://github.com/git/git/archive/v2.25.0.tar.gz
tartar xvf v2.25.0.tar.gz
 xvf v2.25.0.tar.gz
cdcd git-2.25.0
 git-2.25.0
makemake configure
 configure
./configure
./configure
makemake -j4 -j4
sudosudo  makemake  install
install
1.7 Go (Golang)
Opción A: Desde PPA
bash
sudosudo add-apt-repository ppa:longsleep/golang-backports -y
 add-apt-repository ppa:longsleep/golang-backports -y
sudosudo  apt-get
apt-get update
 update
sudosudo  apt-get
apt-get  install
install -y golang-go
 -y golang-go
Opción B: Descarga Directa
bash
cdcd /tmp
 /tmp
wgetwget https://go.dev/dl/go1.19.linux-amd64.tar.gz
 https://go.dev/dl/go1.19.linux-amd64.tar.gz
sudosudo  tartar -C /usr/local -xzf go1.19.linux-amd64.tar.gz
 -C /usr/local -xzf go1.19.linux-amd64.tar.gz
echoecho  'export PATH=$PATH:/usr/local/go/bin'
'export PATH=$PATH:/usr/local/go/bin'  >>>> ~/.bashrc
 ~/.bashrc
source
source ~/.bashrc
 ~/.bashrc
# Verificar
# Verificar
go version
go version
1.8 Patchelf
bash
cdcd /tmp
 /tmp
wgetwget http://nixos.org/releases/patchelf/patchelf-0.10/patchelf-0.10.tar.bz2
 http://nixos.org/releases/patchelf/patchelf-0.10/patchelf-0.10.tar.bz2
tartar xf patchelf-0.10.tar.bz2
 xf patchelf-0.10.tar.bz2
cdcd patchelf-0.10
 patchelf-0.10
./configure
./configure
makemake
sudosudo  makemake  install
install
1.9 Protobuf (Protocol Buffers)
bash
sudosudo  apt-get
apt-get  install
install -y  -y gitgit autoconf automake libtool 
 autoconf automake libtool curlcurl  makemake g++ 
 g++ unzip
unzip
cdcd /tmp
 /tmp
gitgit clone https://github.com/google/protobuf.git
 clone https://github.com/google/protobuf.git
cdcd protobuf/
 protobuf/
./autogen.sh
./autogen.sh
./configure
./configure
makemake -j4 -j4
makemake check
 check
sudosudo  makemake  install
install
sudosudo ldconfig
 ldconfig
# Verificar
# Verificar
protoc --version
protoc --version
1.10 SQLite3 (si es necesario)
bash
cdcd /tmp
 /tmp
wgetwget https://sqlite.org/2020/sqlite-autoconf-3310100.tar.gz
 https://sqlite.org/2020/sqlite-autoconf-3310100.tar.gz
tartar xvf sqlite-autoconf-3310100.tar.gz
 xvf sqlite-autoconf-3310100.tar.gz
cdcd sqlite-autoconf-3310100
 sqlite-autoconf-3310100
./configure
./configure
makemake -j4 -j4
sudosudo  makemake  install
install
1.11 MySQL Client Libraries
Para Ubuntu/Debian moderno:
bash
sudosudo  apt-get
apt-get  install
install -y default-libmysqlclient-dev
 -y default-libmysqlclient-dev
Alternativamente (compilar desde fuentes):
bash
cdcd /tmp
 /tmp
gitgit clone https://github.com/mysql/mysql-server.git
 clone https://github.com/mysql/mysql-server.git
cdcd mysql-server
 mysql-server
mkdir
mkdir build 
 build &&&&  cdcd build
 build
cmake 
cmake ....  \\
    -DBUILD_CONFIG
    -DBUILD_CONFIG==mysql_release 
mysql_release \\
    -DWITH_ZLIB
    -DWITH_ZLIB==bundled 
bundled \\
    -DDOWNLOAD_BOOST
    -DDOWNLOAD_BOOST==11  \\
    -DWITH_BOOST
    -DWITH_BOOST==``pwdpwd``/boost-library/ 
/boost-library/ \\
    -DCMAKE_BUILD_TYPE
    -DCMAKE_BUILD_TYPE==Release 
Release \\
    -DCMAKE_CXX_FLAGS
    -DCMAKE_CXX_FLAGS=="-lrt"
"-lrt"  \\
    -DCMAKE_INSTALL_PREFIX
    -DCMAKE_INSTALL_PREFIX==``pwdpwd``//..../out/
/out/
       
makemake -j4  
 -j4  # Puede fallar, pero el cliente se compila
# Puede fallar, pero el cliente se compila
cmake --install 
cmake --install ..
1.12 Jemalloc
bash
sudosudo  apt-get
apt-get  install
install -y libjemalloc-dev
 -y libjemalloc-dev
PASO 2: CLONAR EL PROYECTO
bash
# Crear directorio de trabajo
# Crear directorio de trabajo
mkdir
mkdir -p ~/teaspeak-build
 -p ~/teaspeak-build
cdcd ~/teaspeak-build
 ~/teaspeak-build
# Clonar el repositorio (usa tu URL)
# Clonar el repositorio (usa tu URL)
gitgit clone https://git.did.science/WolverinDEV/TeaSpeak-Parent.git server
 clone https://git.did.science/WolverinDEV/TeaSpeak-Parent.git server
cdcd server
 server
# Inicializar submódulos
# Inicializar submódulos
gitgit submodule update --init --recursive --remote
 submodule update --init --recursive --remote
NOTA: Si no tienes acceso al repositorio oficial, usa el código fuente que ya tienes.
PASO 3: CONFIGURAR VERSIÓN 1.6.0
3.1 Editar CMakeLists.txt
Archivo: Server/server/CMakeLists.txt
Líneas 223-225:
cmake
SETSET((CPACK_PACKAGE_VERSION_MAJOR
CPACK_PACKAGE_VERSION_MAJOR  "1""1"))
SETSET((CPACK_PACKAGE_VERSION_MINOR
CPACK_PACKAGE_VERSION_MINOR  "5""5"))    # ← Cambiar a "6"
# ← Cambiar a "6"
SETSET((CPACK_PACKAGE_VERSION_PATCH
CPACK_PACKAGE_VERSION_PATCH  "6""6"))    # ← Cambiar a "0"
# ← Cambiar a "0"
Cambiar a:
cmake
SETSET((CPACK_PACKAGE_VERSION_MAJOR
CPACK_PACKAGE_VERSION_MAJOR  "1""1"))
SETSET((CPACK_PACKAGE_VERSION_MINOR
CPACK_PACKAGE_VERSION_MINOR  "6""6"))
SETSET((CPACK_PACKAGE_VERSION_PATCH
CPACK_PACKAGE_VERSION_PATCH  "0""0"))
3.2 Aplicar el Cambio
bash
cdcd ~/teaspeak-build/server/Server/server
 ~/teaspeak-build/server/Server/server
# Hacer backup
# Hacer backup
cpcp CMakeLists.txt CMakeLists.txt.backup
 CMakeLists.txt CMakeLists.txt.backup
# Editar con sed
# Editar con sed
sedsed -i  -i 's/SET(CPACK_PACKAGE_VERSION_MINOR "5")/SET(CPACK_PACKAGE_VERSION_MINOR "6")/'
's/SET(CPACK_PACKAGE_VERSION_MINOR "5")/SET(CPACK_PACKAGE_VERSION_MINOR "6")/' CMakeLis
 CMakeLis
sedsed -i  -i 's/SET(CPACK_PACKAGE_VERSION_PATCH "6")/SET(CPACK_PACKAGE_VERSION_PATCH "0")/'
's/SET(CPACK_PACKAGE_VERSION_PATCH "6")/SET(CPACK_PACKAGE_VERSION_PATCH "0")/' CMakeList
 CMakeList
# Verificar
# Verificar
grepgrep  "CPACK_PACKAGE_VERSION"
"CPACK_PACKAGE_VERSION" CMakeLists.txt
 CMakeLists.txt


Salida esperada:
SET(CPACK_PACKAGE_VERSION_MAJOR "1")
SET(CPACK_PACKAGE_VERSION_MAJOR "1")
SET(CPACK_PACKAGE_VERSION_MINOR "6")
SET(CPACK_PACKAGE_VERSION_MINOR "6")
SET(CPACK_PACKAGE_VERSION_PATCH "0")
SET(CPACK_PACKAGE_VERSION_PATCH "0")
PASO 4: CONFIGURAR VARIABLES DE ENTORNO
bash
cdcd ~/teaspeak-build/server/Root
 ~/teaspeak-build/server/Root
# Variables de build
# Variables de build
export
export  build_os_type
build_os_type==linux
linux
export
export  build_os_arch
build_os_arch==amd64
amd64
export
export  CMAKE_MAKE_OPTIONS
CMAKE_MAKE_OPTIONS=="-j"-j$($(nproc
nproc))""
export
export  MAKE_OPTIONS
MAKE_OPTIONS==""$CMAKE_MAKE_OPTIONS
$CMAKE_MAKE_OPTIONS""
# Flags de compilación (opcional, para optimizar)
# Flags de compilación (opcional, para optimizar)
export
export  CXX_FLAGS
CXX_FLAGS=="-O3 -march=native"
"-O3 -march=native"
PASO 5: COMPILAR LIBRERÍAS
bash
cdcd ~/teaspeak-build/server/Root
 ~/teaspeak-build/server/Root
# Este script compila todas las librerías necesarias
# Este script compila todas las librerías necesarias
./libraries/build.sh
./libraries/build.sh
Este proceso puede tardar 30-60 minutos dependiendo de tu CPU.
Librerías que se Compilan:
event (libevent)
ed25519
jsoncpp
opus
protobuf
spdlog
StringVariable
ThreadPool
tomcrypt/tommath
yaml-cpp
zstd
breakpad
DataPipes
CXXTerminal
PASO 6: COMPILAR TEASPEAK SERVER
6.1 Compilación Modo Release (Producción)
bash
cdcd ~/teaspeak-build/server/Root
 ~/teaspeak-build/server/Root
# Compilar versión estable
# Compilar versión estable
./build_teaspeak.sh stable
./build_teaspeak.sh stable
6.2 Compilación Modo Debug (Desarrollo)
bash
# Establecer flag de debug
# Establecer flag de debug
export
export  i_really_wanna_debug
i_really_wanna_debug==11
# Compilar versión debug
# Compilar versión debug
./build_teaspeak.sh debug
./build_teaspeak.sh debug
6.3 Otras Opciones de Build
bash
# Nightly (con debug info pero optimizado)
# Nightly (con debug info pero optimizado)
./build_teaspeak.sh nightly
./build_teaspeak.sh nightly
# Optimized (beta)
# Optimized (beta)
./build_teaspeak.sh optimized
./build_teaspeak.sh optimized
PASO 7: VERIFICAR COMPILACIÓN
7.1 Localizar el Binario
bash
cdcd ~/teaspeak-build/server/Server/server/environment
 ~/teaspeak-build/server/Server/server/environment
# Listar archivos
# Listar archivos
lsls -lh -lh
Deberías ver:
TeaSpeakServer          ← Binario principal del servidor
TeaSpeakServer          ← Binario principal del servidor
libTeaFile.so           ← Librería de archivos
libTeaFile.so           ← Librería de archivos
ProviderFFMpeg.so       ← Proveedor FFmpeg
ProviderFFMpeg.so       ← Proveedor FFmpeg
ProviderYT.so           ← Proveedor YouTube
ProviderYT.so           ← Proveedor YouTube
7.2 Verificar Versión
bash
cdcd ~/teaspeak-build/server/Server/server/environment
 ~/teaspeak-build/server/Server/server/environment
# Ejecutar con --version
# Ejecutar con --version
./TeaSpeakServer --version
./TeaSpeakServer --version
Salida esperada:
TeaSpeak-Server v1.6.0 [Build: 1699999999]
TeaSpeak-Server v1.6.0 [Build: 1699999999]
7.3 Test Rápido
bash
# Crear directorio de configuración
# Crear directorio de configuración
mkdir
mkdir -p ~/.config/TeaSpeak
 -p ~/.config/TeaSpeak
cdcd ~/.config/TeaSpeak
 ~/.config/TeaSpeak
# Ejecutar servidor (se detendrá por falta de licencia)
# Ejecutar servidor (se detendrá por falta de licencia)
~/teaspeak-build/server/Server/server/environment/./TeaSpeakServer
~/teaspeak-build/server/Server/server/environment/./TeaSpeakServer
RESUMEN DE DEPENDENCIAS UBUNTU/DEBIAN
Lista Completa (Copiar y Pegar)
bash
# Actualizar sistema
# Actualizar sistema
sudosudo  apt-get
apt-get update 
 update &&&&  sudosudo  apt-get
apt-get upgrade -y
 upgrade -y
# Instalar TODAS las dependencias
# Instalar TODAS las dependencias
sudosudo  apt-get
apt-get  install
install -y  -y \\
        sudosudo autoconf 
 autoconf gitgit  wgetwget gettext build-essential 
 gettext build-essential \\
    realpath 
    realpath nanonano  curlcurl software-properties-common 
 software-properties-common \\
    libcurl4-openssl-dev libssl-dev libmysqlclient-dev 
    libcurl4-openssl-dev libssl-dev libmysqlclient-dev \\
    libpcre3-dev libncurses5-dev libsqlite3-dev 
    libpcre3-dev libncurses5-dev libsqlite3-dev \\
    libjemalloc-dev cmake gcc g++ golang-go 
    libjemalloc-dev cmake gcc g++ golang-go \\
    libtool automake 
    libtool automake unzip
unzip default-libmysqlclient-dev
 default-libmysqlclient-dev
# GCC 9+ (si es necesario)
# GCC 9+ (si es necesario)
sudosudo add-apt-repository ppa:ubuntu-toolchain-r/test -y
 add-apt-repository ppa:ubuntu-toolchain-r/test -y
sudosudo  apt-get
apt-get update
 update
sudosudo  apt-get
apt-get  install
install -y gcc-9 g++-9
 -y gcc-9 g++-9
sudosudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 
 update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 6060  \\
    --slave /usr/bin/g++ g++ /usr/bin/g++-9
    --slave /usr/bin/g++ g++ /usr/bin/g++-9
# Protobuf
# Protobuf
cdcd /tmp
 /tmp
gitgit clone https://github.com/google/protobuf.git
 clone https://github.com/google/protobuf.git
cdcd protobuf 
 protobuf &&&& ./autogen.sh 
 ./autogen.sh &&&& ./configure 
 ./configure &&&&  makemake -j4 -j4
sudosudo  makemake  install
install  &&&&  sudosudo ldconfig
 ldconfig
SOLUCIÓN DE PROBLEMAS
Error: "Cannot find MySQL"
bash
sudosudo  apt-get
apt-get  install
install -y default-libmysqlclient-dev libmysqlclient-dev
 -y default-libmysqlclient-dev libmysqlclient-dev
Error: "CMake version too old"
bash
# Instalar CMake 3.16+
# Instalar CMake 3.16+
sudosudo snap 
 snap install
install cmake --classic
 cmake --classic
# O compilar desde fuentes (ver paso 1.5)
# O compilar desde fuentes (ver paso 1.5)
Error: "Protobuf not found"
bash
sudosudo ldconfig
 ldconfig
protoc --version
protoc --version
# Si no aparece, reinstalar protobuf
# Si no aparece, reinstalar protobuf
Error: "Cannot find libevent"
bash
cdcd ~/teaspeak-build/server/Root/libraries
 ~/teaspeak-build/server/Root/libraries
./build.sh  
./build.sh  # Recompilar librerías
# Recompilar librerías
Error: Compilación muy lenta
bash
# Reducir hilos de compilación
# Reducir hilos de compilación
export
export  CMAKE_MAKE_OPTIONS
CMAKE_MAKE_OPTIONS=="-j2"
"-j2"
export
export  MAKE_OPTIONS
MAKE_OPTIONS=="-j2"
"-j2"
Error: "The source directory does not appear to contain CMakeLists.txt"
Este error ocurre cuando el script de build no encuentra la estructura de directorios esperada.

Solución - Crear enlaces simbólicos:
bash
cd /home/user/TeaSpeak/Server/Root
# Crear enlace al código fuente del servidor
ln -s ../Server TeaSpeak
# Crear enlace a la librería rtclib
cd TeaSpeak && ln -s ../rtc rtclib

Verificación:
bash
ls -la TeaSpeak/CMakeLists.txt         # Debe existir
ls -la TeaSpeak/rtclib/generate_shared_library.sh  # Debe existir

Nota: Estos enlaces simbólicos permiten que el script build_teaspeak.sh encuentre:
- Server/Root/TeaSpeak/ → apunta a Server/Server/ (código fuente)
- Server/Server/rtclib/ → apunta a Server/rtc/ (librería WebRTC)

COMPILACIÓN RÁPIDA (SCRIPT TODO-EN-UNO)
bash
#!/bin/bash
#!/bin/bash
# compile_teaspeak_1.6.0.sh
# compile_teaspeak_1.6.0.sh
setset -e -e
# Colores
# Colores
REDRED=='\033[0;31m'
'\033[0;31m'
GREEN
GREEN=='\033[0;32m'
'\033[0;32m'
YELLOW
YELLOW=='\033[1;33m'
'\033[1;33m'
NCNC=='\033[0m'
'\033[0m'
echoecho -e  -e ""${GREEN}
${GREEN}=== Compilación TeaSpeak Server v1.6.0 ===
=== Compilación TeaSpeak Server v1.6.0 ===${NC}
${NC}""
# 1. Instalar dependencias
# 1. Instalar dependencias
echoecho -e  -e ""${YELLOW}
${YELLOW}[1/6] Instalando dependencias...
[1/6] Instalando dependencias...${NC}
${NC}""
sudosudo  apt-get
apt-get update
 update
sudosudo  apt-get
apt-get  install
install -y  -y \\
    build-essential cmake gcc g++ 
    build-essential cmake gcc g++ gitgit  wgetwget  \\
    libcurl4-openssl-dev libssl-dev libmysqlclient-dev 
    libcurl4-openssl-dev libssl-dev libmysqlclient-dev \\
    libpcre3-dev libncurses5-dev libsqlite3-dev 
    libpcre3-dev libncurses5-dev libsqlite3-dev \\
    libjemalloc-dev golang-go autoconf libtool
    libjemalloc-dev golang-go autoconf libtool
# 2. Ir al directorio
# 2. Ir al directorio
echoecho -e  -e ""${YELLOW}
${YELLOW}[2/6] Configurando directorio...
[2/6] Configurando directorio...${NC}
${NC}""
cdcd ~/teaspeak-build/server
 ~/teaspeak-build/server
# 3. Cambiar versión a 1.6.0
# 3. Cambiar versión a 1.6.0
echoecho -e  -e ""${YELLOW}
${YELLOW}[3/6] Configurando versión 1.6.0...
[3/6] Configurando versión 1.6.0...${NC}
${NC}""
cdcd Server/server
 Server/server
sedsed -i  -i 's/SET(CPACK_PACKAGE_VERSION_MINOR "5")/SET(CPACK_PACKAGE_VERSION_MINOR "6")/'
's/SET(CPACK_PACKAGE_VERSION_MINOR "5")/SET(CPACK_PACKAGE_VERSION_MINOR "6")/' CMakeLis
 CMakeLis
sedsed -i  -i 's/SET(CPACK_PACKAGE_VERSION_PATCH "6")/SET(CPACK_PACKAGE_VERSION_PATCH "0")/'
's/SET(CPACK_PACKAGE_VERSION_PATCH "6")/SET(CPACK_PACKAGE_VERSION_PATCH "0")/' CMakeList
 CMakeList
# 4. Variables de entorno
# 4. Variables de entorno
echoecho -e  -e ""${YELLOW}
${YELLOW}[4/6] Configurando variables...
[4/6] Configurando variables...${NC}
${NC}""
cdcd  ....//..../Root
/Root
export
export  build_os_type
build_os_type==linux
linux
export
export  build_os_arch
build_os_arch==amd64
amd64
export
export  CMAKE_MAKE_OPTIONS
CMAKE_MAKE_OPTIONS=="-j"-j$($(nproc
nproc))""
export
export  MAKE_OPTIONS
MAKE_OPTIONS==""$CMAKE_MAKE_OPTIONS
$CMAKE_MAKE_OPTIONS""
# 5. Compilar librerías
# 5. Compilar librerías
echoecho -e  -e ""${YELLOW}
${YELLOW}[5/6] Compilando librerías (esto tardará)...
[5/6] Compilando librerías (esto tardará)...${NC}
${NC}""
./libraries/build.sh
./libraries/build.sh
# 6. Compilar TeaSpeak
# 6. Compilar TeaSpeak
echoecho -e  -e ""${YELLOW}
${YELLOW}[6/6] Compilando TeaSpeak Server...
[6/6] Compilando TeaSpeak Server...${NC}
${NC}""
./build_teaspeak.sh stable
./build_teaspeak.sh stable
# Verificar
# Verificar
cdcd  ..../Server/server/environment
/Server/server/environment
ifif  [[ -f  -f "./TeaSpeakServer"
"./TeaSpeakServer"  ]];;  thenthen
        echoecho -e  -e ""${GREEN}
${GREEN}✓ Compilación exitosa!
✓ Compilación exitosa!${NC}
${NC}""
        echoecho -e  -e ""${GREEN}
${GREEN}Binario: 
Binario: $($(pwdpwd))/TeaSpeakServer
/TeaSpeakServer${NC}
${NC}""
    ./TeaSpeakServer --version
    ./TeaSpeakServer --version
elseelse
        echoecho -e  -e ""${RED}
✗ Error en la compilación${NC}
${RED}✗ Error en la compilación
${NC}""
        exitexit  11
fifi
Guardar como: compile_teaspeak_1.6.0.sh
Ejecutar:
bash
chmod
chmod +x compile_teaspeak_1.6.0.sh
 +x compile_teaspeak_1.6.0.sh
./compile_teaspeak_1.6.0.sh
./compile_teaspeak_1.6.0.sh
INFORMACIÓN ADICIONAL
Tamaño Aproximado
Código fuente: ~500 MB
Dependencias compiladas: ~2 GB
Binario final: ~50 MB
Tiempo de Compilación
CPU 4 cores: ~45-60 minutos
CPU 8 cores: ~25-35 minutos
CPU 16 cores: ~15-20 minutos
Arquitectura del Servidor
TeaSpeakServer (binario principal)
TeaSpeakServer (binario principal)
├── Servidor de Voz (VoiceServer)
├── Servidor de Voz (VoiceServer)
├── Servidor de Consultas (QueryServer)
├── Servidor de Consultas (QueryServer)
├── Servidor Web (WebServer)
├── Servidor Web (WebServer)
├── Sistema de Licencias (LicenseService)
├── Sistema de Licencias (LicenseService)
└── Servidor de Archivos (FileServer)
└── Servidor de Archivos (FileServer)
¡Listo, Jorge! Con esta guía tienes todo lo necesario para compilar TeaSpeak Server v1.6.0 en Ubuntu/Debian.
