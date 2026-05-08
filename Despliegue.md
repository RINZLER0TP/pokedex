# 📦 Despliegue.md — Proceso de Despliegue PokeDex

**Plataforma:** Microsoft Azure Static Web Apps  
**Dominio:** [https://mipokedexbyrinzler.duckdns.org](https://mipokedexbyrinzler.duckdns.org)  
**Fecha:** Mayo 2025

---

## 📋 Índice

1. [Pre-requisitos](#pre-requisitos)
2. [Configuración del entorno local](#configuración-del-entorno-local)
3. [Compilación de la aplicación](#compilación-de-la-aplicación)
4. [Subida a GitHub](#subida-a-github)
5. [Despliegue en Azure Static Web Apps](#despliegue-en-azure-static-web-apps)
6. [Configuración de dominio personalizado](#configuración-de-dominio-personalizado)
7. [Configuración de headers de seguridad](#configuración-de-headers-de-seguridad)
8. [Errores encontrados y soluciones](#errores-encontrados-y-soluciones)

---

## 1. Pre-requisitos

- Cuenta en [GitHub](https://github.com)
- Cuenta en [Microsoft Azure](https://portal.azure.com) con créditos gratuitos
- Cuenta en [DuckDNS](https://www.duckdns.org) para dominio personalizado
- Windows 10/11
- Git instalado ([git-scm.com](https://git-scm.com))
- Node.js v16 (instalado via NVM)

---

## 2. Configuración del entorno local

### Instalación de Git

```bash
# Descargar desde git-scm.com e instalar con configuración por defecto
# Verificar instalación
git --version

# Configurar identidad global
git config --global user.name "RINZLER0TP"
git config --global user.email "correo@outlook.com"
```

### Instalación de NVM y Node.js

Se utilizó NVM (Node Version Manager) porque el proyecto requiere Node.js v16 específicamente. Node.js v25 era incompatible con Angular 14.

```bash
# Descargar nvm-setup.exe desde:
# https://github.com/coreybutler/nvm-windows/releases

# Instalar Node.js v16
nvm install 16
nvm use 16

# Verificar versiones
node -v   # debe mostrar v16.20.2
npm -v    # debe mostrar 8.x
```

---

## 3. Compilación de la aplicación

### Navegación a la carpeta del proyecto

```bash
cd /c/Users/sujes/Downloads/ac4dem1a-master/ac4dem1a-master/sistemas-distribuidos/poke-dex-lab/source/pokedex-angular
```

### Instalación de dependencias

```bash
# Cambiar registry a mirror más estable
npm config set registry https://registry.npmmirror.com
npm config set fetch-retry-mintimeout 20000
npm config set fetch-retry-maxtimeout 120000
npm config set fetch-retries 5

# Instalar dependencias
npm install --legacy-peer-deps
```

> **Nota:** Se usó `--legacy-peer-deps` para resolver conflictos entre dependencias antiguas de Angular 14.

### Compilación para producción

```bash
npm run build
```

**Resultado esperado:**
```
✔ Browser application bundle generation complete.
✔ Copying assets complete.
✔ Index html generation complete.
Build at: 2025-XX-XXTXX:XX:XX.XXXZ - Hash: XXXXXXXX - Time: 39890ms
```

Los archivos compilados quedan en `dist/pokedex-angular/`.

---

## 4. Subida a GitHub

### Crear repositorio

1. Ir a [github.com](https://github.com) → **New repository**
2. Nombre: `pokedex`
3. Visibilidad: **Public**
4. Inicializar con README: **Sí**
5. Clic en **Create repository**

### Subir código con Git

```bash
# Inicializar Git en la carpeta del proyecto
git init

# Agregar todos los archivos al staging
git add .

# Crear primer commit
git commit -m "feat: add pokedex angular app"

# Conectar con repositorio remoto
git remote add origin https://github.com/RINZLER0TP/pokedex.git

# Renombrar rama a main (estándar actual)
git branch -M main

# Fusionar con el README inicial de GitHub
git pull origin main --allow-unrelated-histories
git checkout --ours README.md
git add README.md
git commit -m "merge: resolve README conflict"

# Subir código
git push -u origin main
```

---

## 5. Despliegue en Azure Static Web Apps

### Crear el recurso

1. Ingresar a [portal.azure.com](https://portal.azure.com)
2. Buscar **"Static Web Apps"** en la barra de búsqueda
3. Clic en **+ Create**
4. Completar el formulario:

| Campo | Valor |
|---|---|
| Subscription | Free Trial |
| Resource Group | rg-pokedex (nuevo) |
| Name | Pokedex-with-RINZLER |
| Plan type | Free |
| Region | East US 2 |

5. En **Deployment details** seleccionar **GitHub**
6. Autorizar acceso de Azure a GitHub
7. Seleccionar repositorio `pokedex` y rama `main`
8. En **Build details:**
   - Framework: `Angular`
   - App location: `/`
   - Output location: `dist/pokedex-angular`
9. Clic en **Review + Create** → **Create**

### Resultado

Azure crea automáticamente un workflow en `.github/workflows/` que despliega la app en cada `git push`. La URL generada fue:

```
https://happy-ground-0911b4410.7.azurestaticapps.net
```

---

## 6. Configuración de dominio personalizado

### Crear dominio en DuckDNS

1. Ir a [duckdns.org](https://www.duckdns.org)
2. Iniciar sesión con cuenta Google/GitHub
3. En el campo **sub domain** escribir: `mipokedexbyrinzler`
4. Clic en **Add domain**
5. Copiar el **token** que aparece en la parte superior

### Vincular dominio con Azure

1. En Azure Portal → Static Web App → **Custom domains**
2. Clic en **+ Add**
3. Seleccionar **Custom domain on other DNS**
4. Ingresar: `mipokedexbyrinzler.duckdns.org`
5. Cambiar **Hostname record type** a `TXT`
6. Clic en **Generate code** — Azure genera un código de verificación
7. Registrar el código TXT en DuckDNS via API:

```
https://www.duckdns.org/update?domains=mipokedexbyrinzler&token=TU_TOKEN&txt=CODIGO_GENERADO
```

8. Volver a Azure y clic en **Validate + add**

### Apuntar IP de Azure a DuckDNS

DuckDNS no soporta registros CNAME, por lo que se obtiene la IP de Azure y se configura directamente:

```
# Obtener IP actual de Azure
https://dns.google/resolve?name=happy-ground-0911b4410.7.azurestaticapps.net&type=A

# Actualizar IP en DuckDNS
https://www.duckdns.org/update?domains=mipokedexbyrinzler&token=TU_TOKEN&ip=IP_OBTENIDA
```

**Resultado:** La app quedó accesible en `https://mipokedexbyrinzler.duckdns.org` con certificado HTTPS automático provisto por Azure.

---

## 7. Configuración de headers de seguridad

Se creó el archivo `staticwebapp.config.json` en la raíz del proyecto:

```json
{
  "globalHeaders": {
    "Content-Security-Policy": "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data: blob: https:; font-src 'self' data:; connect-src 'self' https://beta.pokeapi.co; frame-ancestors 'none';",
    "Strict-Transport-Security": "max-age=31536000; includeSubDomains; preload",
    "X-Content-Type-Options": "nosniff",
    "X-Frame-Options": "DENY",
    "Referrer-Policy": "no-referrer",
    "Permissions-Policy": "accelerometer=(), camera=(), geolocation=(), gyroscope=(), magnetometer=(), microphone=(), payment=(), usb=()"
  }
}
```

```bash
# Subir configuración de seguridad
git add staticwebapp.config.json
git commit -m "feat: add security headers for A+ rating"
git push
```

**Resultado del escaneo en securityheaders.com: Calificación A ✅**

---

## 8. Errores encontrados y soluciones

### Error 1 — Node.js incompatible

**Error:**
```
npm error Exit handler never called!
```

**Causa:** Node.js v25 es incompatible con Angular 14.

**Solución:** Instalar NVM y usar Node.js v16:
```bash
nvm install 16
nvm use 16
```

---

### Error 2 — Timeout de red en npm install

**Error:**
```
npm ERR! code ERR_SOCKET_TIMEOUT
npm ERR! network Socket timeout
```

**Causa:** Conexión inestable al descargar paquetes desde el registry oficial de npm.

**Solución:** Cambiar al mirror de Taobao y configurar reintentos:
```bash
npm config set registry https://registry.npmmirror.com
npm config set fetch-retries 5
npm install --legacy-peer-deps
```

---

### Error 3 — webpack-subresource-integrity incompatible

**Error:**
```
Cannot find module './util'
Require stack: webpack-subresource-integrity/plugin.js
```

**Causa:** Versión incompatible del paquete `webpack-subresource-integrity`.

**Solución:**
```bash
npm install webpack-subresource-integrity@1.5.1 --save-dev
```

---

### Error 4 — Imágenes con 404 en producción

**Error:** Las imágenes de Pokémon no cargaban en la URL de Azure.

**Causa:** `environment.prod.ts` tenía `imagesPath: '/pokedex-angular/assets/images'` (ruta absoluta con subruta).

**Solución:** Cambiar a ruta relativa:
```typescript
imagesPath: '/assets/images'
```

---

### Error 5 — Git push rechazado

**Error:**
```
! [rejected] main -> main (fetch first)
```

**Causa:** El repositorio remoto tenía cambios (README inicial) que no estaban localmente.

**Solución:**
```bash
git pull origin main --rebase
git push
```

---

### Error 6 — DuckDNS no soporta CNAME

**Causa:** DuckDNS solo permite configurar registros IP, no CNAME.

**Solución:** Obtener la IP de Azure con `dns.google` y configurarla directamente en DuckDNS via API.

---

*Documento generado como parte del proceso de despliegue — Sistemas Distribuidos 2025*
