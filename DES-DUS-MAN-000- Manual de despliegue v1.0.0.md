# Manual de configuración y despliegue para **CICLOS**

## Preparar ambiente

### Prerequisitos

#### Servidor de aplicaciones

Windows Server 2012 R2 deberá ser instalado en el equipo, utilizará Internet Information Services.

### Servidor de base de datos

Windows Server trabajará con un servidor remoto de SQL Server que deberá ser configurado aparte.

### Instalar SQL Server

* Descargue el [instalador de SQL Server](https://www.microsoft.com/es-mx/download/details.aspx?id=29062)
* Seguir los pasos de instalación como instancia, recordar los datos de autenticación o seleccionar autenticación de Windows para ambientes de desarrollo locales.

#### Crear la base de datos.

* Usando el backup de la base de datos de producción, realice la importación con la herramienta de Managament Studio con datos de producción.

### Configurar el servidor de aplicaciones

#### Instalar Internet Information Services

* Clic derecho sobre el boton de Inicio de Widnows y dar clic sobre Programs and features
* Clic en 'Turn Windows features on or off'
* Seleccione 'Role-based or feature-based installation'
* Seleccione el servidor actual para instalar los roles
* De la lista 'Roles' desplegar 'Web Server (IIS)
* Marque las casillas:
    - Common HTTP Features
        - Default Document
        - Directory Browsing
        - HTTP Errors
        - Static Content
    - Health and Diagnostics
        - HTTP Logging
        - Custom Logging
        - Logging Tools
        - Request Monitor
    - Performance
        - Static Content Compression
    - Security
        - Reques Filtering
    - Application Development
        - .NET Extensibility 3.5
        - .NET Extensibility 4.5
        - Application Initialization
        - CGI
        - ISAPI Extensions
        - ISAPI Filters
    - FTP Server
        - FTP Service
        - FTP Extensibility
    - Management Tools
        - IIS Management Console
        - IIS 6 Management Compatibilty (seleccione todas debajo)
* Probablemente requiera un reinicio

#### Instalar PHP 5.6

* Descargue el [instalador de PHP 5.6](http://php.net/releases/)
* En el Instalador de plataformas hacer clic en 'Productos' e instalar:
    - Microsoft Drivers for PHP v5.6 for SQL Server
    - Windows Cache Extension for PHP 5.6
    - PHP 5.6.xx
    - PHP Manager para IIS
* Dar clic en instalar y seguir los pasos
* PHP y los drivers de SQL Server estarán instalados.
* Termine la configuración e instale los archivos vcredist para 32 y 64 bits.

#### Instalar freeSSHd

* Descargue el [instalador de freeSSHd](http://www.freesshd.com/)
* Instale el software
* Una vez instalado modifique la configuración en el apartado SSH de la ventana, introduce la IP local del servidor y usa C:\Windows\system32\cmd.exe como el 'Command shell'
* Genere nuevas keys usando el boton de 'New'
* Guarde los cambios

#### Instalar Git

* Descargue el [instalador de Git](https://git-scm.com/downloads)
* En la pantalla de seleccionar componentes marcamos las casillas:
    - Git Bash Here
    - Git GUI Here
    - Associate .git cibfiguration files with the default text editor
    - Associate .sh files to be run with Bash
* En la ventana de Default editor seleccione 'Use Vim' as Git's default editor'
* En la siguiente pantalla seleccione 'use Git from the Windows Command Prompt'
* Después seleccione 'Use the OpenSSL library'
* Seleccione 'Checkout Windows-style, commit Unix's stle line endings'
* Seleccione 'use MinTTY'
* Seleccione 'Enable file system caching' y 'Enable Git credential manager'
* Finalice la instalación

### Desplegar aplicación

Desde nuestra ubicación local, ingrese por SSH al servidor y navegue a la ruta del proyecto.

```
$ cd C:\inetpub\wwwroot\ciclos
```
Si la carpeta 'ciclos' no existe, creela

```

$ mkdir ciclos

```
Haga el despliegue desde BitBucket:

```
# inicialice un repositorio de git clonando desde el remoto
$ git clone --branch=master URL .

# verifique que la dirección del repositorio remoto haya quedado registrado
$ git remote -v

# reinicie IIS
$ iisreset
```

### Desplegar nuevos cambios al servidor

#### Flujo para el desarrollo de nuevas funciones o funciones iniciales de la aplicación y su despliegue en el servidor de aplicaciones como QA.

```
# inicialice un repositorio de git clonando desde el remoto
$ git clone URL .

# verifique que la dirección del repositorio remoto haya quedado registrado
$ git flow init

```

Crear la rama de 'feature'

```
$ git flow feature start [branchName]
```

Una vez que realice los cambios necesarios, realice su commit a su rama 'feature'
```
$ git add -A
$ git commit -m [ticket number and description]
```

Suba sus cambios en la rama 'feature' a BitBucket para realizar el dspliegue a QA
```
$ git push -u origin feature/[branchName]
```

Acceda al servidor de aplicaciones mediante SSH y navegue a la carpeta de ciclosqa y realice el despliegue de su rama 'feature' para QA
```
$ cd C:\inetpub\wwwroot\ciclosqa
$ git branch [feature]/[branchName]
$ git fetch
$ git branch --set-upstream-to=origin/[feature]/[branchName] [feature]/[branchName]
$ git checkout [feature]/[branchName]
$ git pull
```

Ahora QA podrá realizar las pruebas de su feature branch. Si QA acepta los cambios continué al paso 'Flujo para el despliegue de una rama 'feature' a test', si no, realice los cambios en su rama feature y repita los pasos de este apartado.

#### Flujo para el despliegue de una rama feature a test

Cierre su rama 'feature' en su entorno de desarrollo.
```
$ git flow feature finish [branchName]
```

Esto provocará que la rama 'feature' que usted creo ahora esté contenida en la rama 'develop'. Después de esto, inicialice una rama 'release', está será su rama para test.

```
$ git flow release start [versionTag]
$ git push -u origin release/[versionTag]

# suba la rama develop a bitbucket
$ git push -u origin develop

# limpie la rama feature en bitbucket
$ git push origin :feature/[branchName]
```

Procure incluir su número de version en lugar de '[versionTag]'

Acceda al servidor de aplicaciones mediante SSH y navegue a la carpeta de ciclostest y realice el despliegue de su rama 'release' para el testing por parte de Ciclos
```
$ cd C:\inetpub\wwwroot\ciclostest
$ git branch [release]/[versionTag]
$ git fetch
$ git branch --set-upstream-to=origin/[release]/[versionTag] [release]/[versionTag]
$ git checkout [release]/[versionTag]
$ git pull
```

Ahora ciclos podrá realizar las pruebas de su 'release' branch. Si Ciclos acepta los cambios continué al paso 'Flujo para el despliegue de una rama 'release' a 'producción', si no, realice los cambios en su rama 'release' y repita los pasos de este apartado.

#### Flujo para el despliegue de una rama release a producción

Cierre su rama 'release' en su entorno de desarrollo y suba sus cambios a producción
** Al cerrar su rama release, los cambios que haya realizado aquí se unirán a la rama 'develop' y a la rama 'master'

```
$ git flow release finish [versionTag]
$ git push -u origin master

# suba la rama develop a bitbucket
$ git push -u origin develop

# limpie la rama feature en bitbucket
$ git push origin :release/[versionTag]

```

Acceda al servidor de aplicaciones mediante SSH y navegue a la carpeta de ciclos y realice el despliegue de su rama 'master'.

```
$ git checkout master
$ git pull
```

#### Limpieza de repositorio de entorno de QA y Test

Finalmente acceda al servidor de aplicaciones y elimine los branches feature y release que hayan quedado de su iteración actual. CiclosQA y CiclosTest deberán trabajar con la rama develop.

```
# primero navuegue a ciclosqa
cd C:\inetpub\wwwroot\ciclosqa

# cambie a la rama develop y obtenga los ultimos cambios
$ git checkout develop
$ git pull

# elimine la antigua rama feature
$ git branch -d feature/[branchName]

```

```
# ahora navegue a ciclostest
cd C:\inetpub\wwwroot\ciclostest

# cambie a la rama develop y obtenga los ultimos cambios
$ git checkout develop
$ git pull

# elimine la antigua rama feature
$ git branch -d release/[versionTag]

```