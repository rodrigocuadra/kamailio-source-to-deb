# 📦 Step-by-Step Guide: Compiling Kamailio on Debian 13 (with .deb packages)

This document provides a detailed, step-by-step guide to compile Kamailio from source on Debian 13 (Trixie) and generate modular `.deb` packages. It is designed to keep all packages organized under `/usr/src/kamailio/deb` to prevent conflicts or confusion with other builds (like FreeSWITCH or Kamailio admin packages).

> [!IMPORTANT]
> **Soporte Multi-Arquitectura (AMD64 / ARM64):**
> Este documento describe la compilación nativa de Kamailio en Debian 13.
> Si se compila en un host Intel/AMD (ej. VM Proxmox), los paquetes `.deb` resultantes tendrán el sufijo `_amd64.deb`.
> Si se compila en un host ARM64 (ej. VM en Macbook M5 o Raspberry Pi 5), los comandos de empaquetado generarán archivos con el sufijo `_arm64.deb`. Asegúrate de usar los nombres de archivos correctos según la arquitectura de tu máquina.

---

## 🛠️ Phase 1: Environment & Base Prerequisites

### 1.1 Prepare the Environment and Install Build Tools

First, install the general build tools and packaging utilities required for building Debian packages:

```bash
apt-get update
apt-get install -y \
  sudo git build-essential devscripts dpkg-dev equivs fakeroot \
  autoconf automake libtool libtool-bin cmake pkg-config \
  wget curl unzip python-is-python3 python3-all-dev \
  python3-setuptools docbook-xsl xsltproc \
  libglib2.0-dev graphviz dh-make
```

---

### 1.2 Install Kamailio Build Dependencies

Kamailio requires development libraries for its modules (MySQL, PostgreSQL, PCRE2, Lua, Python, etc.) as well as specific helper scripts for building CMake-based packages:

```bash
apt-get install -y \
  debhelper dh-cmake dh-sequence-cmake bison flex cmake \
  default-libmysqlclient-dev docbook-xml docbook-xsl \
  erlang-dev libcurl4-openssl-dev libev-dev libevent-dev \
  libexpat1-dev libhiredis-dev libjansson-dev libjson-c-dev \
  libjwt-dev libldap2-dev liblua5.4-dev libmaxminddb-dev \
  libmemcached-dev libmicrohttpd-dev libmnl-dev libmongoc-dev \
  libmosquitto-dev libnats-dev libncurses-dev libnghttp2-dev \
  libpcre2-dev libperl-dev libphonenumber-dev libpq-dev \
  librabbitmq-dev libradcli-dev librdkafka-dev libreadline-dev \
  libsasl2-dev libsctp-dev libsecp256k1-dev libsecsipid-dev \
  libsnmp-dev libsqlite3-dev libssl-dev libsystemd-dev \
  libunistring-dev libwebsockets-dev libwolfssl-dev libxml2-dev \
  lynx openssl python3 python3-dev ruby-dev \
  unixodbc-dev uuid-dev zlib1g-dev
```

---

## ⚙️ Phase 2: Compiling & Packaging Kamailio

### 2.1 Clone Kamailio and Prepare Packaging

To keep the output directory clean and separated from `/usr/src`, we will clone Kamailio into `/usr/src/kamailio/src`. This causes `dpkg-buildpackage` to output all built files (`.deb`, `.changes`, `.buildinfo`) in the parent directory `/usr/src/kamailio/`, where we can safely isolate them.

```bash
# Create target directory structure
mkdir -p /usr/src/kamailio/deb

# Clone Kamailio source code (stable branch like 6.1)
git clone -b 6.1 https://github.com/kamailio/kamailio.git /usr/src/kamailio/src
cd /usr/src/kamailio/src

# Link Debian Trixie packaging configuration to the debian/ directory
ln -s pkg/kamailio/deb/trixie debian
```

---

### 2.2 Generate Build Configuration

Prepare the source code configuration files:

```bash
make cfg
```

---

### 2.3 Build Kamailio .deb Packages

Run the compilation and packaging process, then isolate all built packages:

```bash
# Clean the packaging workspace
fakeroot debian/rules clean

# Build all packages (-b builds binary-only, -us/-uc skips GPG signing)
dpkg-buildpackage -us -uc -b -j$(nproc)

# Move all generated packages and build files to /usr/src/kamailio/deb/
mv /usr/src/kamailio/*.deb /usr/src/kamailio/deb/
mv /usr/src/kamailio/*.changes /usr/src/kamailio/deb/ 2>/dev/null || true
mv /usr/src/kamailio/*.buildinfo /usr/src/kamailio/deb/ 2>/dev/null || true
```

This will populate `/usr/src/kamailio/deb/` with all modular Kamailio `.deb` packages (e.g. `kamailio_*.deb`, `kamailio-mysql-modules_*.deb`, `kamailio-postgres-modules_*.deb`, `kamailio-ims-modules_*.deb`, etc.).

---

## 🚀 Phase 3: Local Installation

To install Kamailio directly from the compiled `.deb` packages on the local machine:

```bash
cd /usr/src/kamailio/deb

# Install Kamailio and the required PostgreSQL modules locally
apt-get install -y \
  ./kamailio_*.deb \
  ./kamailio-postgres-modules_*.deb \
  ./kamailio-presence-modules_*.deb \
  ./kamailio-websocket-modules_*.deb

# Fix any missing system dependencies
apt-get install -f -y

# Verify the installation and version
kamailio -v
```

---

## 🌐 Phase 4: Central Distribution (Optional)

If you want to distribute these compiled `.deb` packages to multiple target servers via a secure, central APT repository server using Nginx, Let's Encrypt SSL, and custom index builders, see the dedicated [Debian APT Repository Setup Guide](file:///Users/rodrigocuadra/Documents/Ring2All/docs/distribution/apt_repository_guide.md).

---

## 📚 Phase 5: Reference

### 5.1 Workflow Summary

1. Prepare the environment and install build dependencies.
2. Clone Kamailio `6.1` stable branch into `/usr/src/kamailio/src/` and link trixie package settings.
3. Configure the build parameters using `make cfg`.
4. Compile and package modular Kamailio `.deb` files using `dpkg-buildpackage`.
5. Install packages locally using `apt install -y ./kamailio*.deb`.
6. Test using: `kamailio -v`.

### 5.2 Expected Result
- ✔ Kamailio compiles successfully on Debian 13.
- ✔ Modular packages (`kamailio`, `kamailio-postgres-modules`, etc.) are built.
- ✔ Kamailio is installed and ready to be configured.
