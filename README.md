# .NET Core - DevOps Lab
La finalidad de este laboratorio sobre DevOps es la practica sobre los procesos de Integración Contínua (CI) y Despliegue Continuo (CD) utilizando las funcionalidades ofrecidas por la herramienta Azure DevOps.

Se utilizará como base para las prácticas el código fuente de la aplicación de ejemplo creada por Ben Coleman y disponible en el [este repositorio GutHub](https://github.com/benc-uk/dotnet-demoapp). 

Esta aplicación permite monitorizar en tiempo real los recursos de CPU y memoria utilizados por la aplicación, así como forzar el uso de recursos y lanzar excepciones que serán recogidas por aplicaciones de monitorización, como App Insights.

## Requisitos
1. Una suscripción de Azure, de pago por uso, MSDN o Trial.
   - Para completar este laboratorio, asegúrate de que tu cuenta tenga los siguientes roles:
     - El rol [Owner](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#owner) para la suscripción que vayas a utilizar.
     - Eres un usuario [Miembro](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/users-default-permissions#member-and-guest-users) del directorio Azure AD que vayas a utilizar. (Los usuarios invitados no tendrán los permisos necesarios).
     - No tienes restringida la creación de Aplicaciones Azure AD. Es posible que el administrador del Directorio Azure AD lo haya limitado.  

     > **Nota** Si no cumples estos requisitos, es posible que tengas que pedir a otro usuario miembro con derechos de propietario de suscripción que inicie sesión en el portal y ejecute con antelación el paso de creacion de la aplicación Azure AD.
  
2. Máquina local o una máquina virtual (Windows o Linux) configurada con:
   - Un navegador, preferiblemente Chrome o Microsoft Edge.
   - [dotnet core sdk](https://dotnet.microsoft.com/download) instalado, para las pruebas de compilación y ejecucion locales.
   - [Visual Studio Code](https://code.visualstudio.com/download) o cualquier otro IDE o editor.

# Ejercicio 1: Ejecutar localmente

**Duracion**: XX minutes

Como primer ejercicio probaremos a ejecutar la aplicación de forma local, para comprobar su correcto funcionamiento y familiarizarnos con la la funcionalidad que nos ofrece.

Para ello nos descargaremos el codigo fuente desde el repositorio de referencia de este laboratorio, ejecutando el siguiente comando.
```
git clone https://github.com/fnietoga/netcoredevops.git
```

Eliminaremos las referencias al repositorio Git desde el que hemos descargado el contenido.
```
Remove-Item -Recurse -Force ./netcoredevops/.git
```

> **Nota** En entornos linux el comando a ejecutar sería
```
rm -rf netcoredevops/.git
```

Realizaremos una compilacion de la aplicacion web, y la ejecutaremos de forma local, ejecutando
```
cd netcoredevops/src
dotnet restore
dotnet run
```

La aplicación web escuchará en los puertos 5000 (http) y 5001 (https), habituales de Kestrel, pero esto puede cambiarse configurando la variable de entorno `ASPNETCORE_URLS` o con el parámetro `--urls` ([ver docs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-3.1)).  

Probaremos la aplicacion web en el navegador, accediendo a alguna de las siguientes URLs:
```
http://localhost:5000
https://localhost:5001
```

Probaremos la opciones de la aplicación:
- **'Info'** - Mostrará información del sistema y del entorno de ejecución, incluyendo las variables de entorno.
- **'Tools'** - Algunas herramientas útiles en demos, como la carga forzada de CPU, y las páginas de error/excepción para su uso con App Insights u otra herramienta de monitorización.
- **'Monitoring'** - Muestra la carga de la CPU y memoria en tiempo real en formato gráfico, obtenidos utilizando una API REST (/api/monitoringdata)y mostrados utilizando chart.js. Recomendable abrirlo en una pestaña adicional del navegador para poder observar en tiempo real la carga generada utilizando las herramientas anteriores.

# Ejercicio 2: Integración Continua con Azure DevOps

**Duracion**: XX minutes

El propósito de este ejercicio es familiarizarnos con las principales opciones de Azure DevOps y preparar el entorno para qie se realice un proceso de compilacion de forma automática cada vez que se sube un cambio en el codigo fuente por algun desarrollador.

### Tarea 1: Crear proyecto y subir código fuente

Para comenzar abre una nueva pestaña del navegador para visitar [Azure DevOps](https://dev.azure.com), y a continuación inicia sesión en tu cuenta.  

Si es la primera vez que inicias sesion con esa cuenta, Azure DevOps te guiará a través de un asistente:
- Confirma tu informacion de contacto y selecciona "Siguiente"
- Selecciona "Continuar" para crear una nueva cuenta en Azure DevOps
- Introduce el nombre que deseas para tu nueva Organización y selecciona "Continuar"

También puedes crear una nueva organizacion utilizando el enlace "Nueva Organizacion", en la parte superior izquierda, o trabajar sobre una de tus organizaciones ya existentes.  
   ![screen](https://user-images.githubusercontent.com/4158659/93192213-b6ce5100-f745-11ea-9604-bea1f89d7cbd.png)

Si hemos creado una nueva organización nos mostrará automáticamente las opciones para la creación del primero proyecto.  
   ![screen](https://user-images.githubusercontent.com/4158659/93192382-eaa97680-f745-11ea-92f1-20efffb2b07a.png)

Si hemos seleccionado una organización ya existente, tendremos que hacer uso del botón "Nuevo Proyecto" que se nos muestra en la parte superior derecha.  

Especificaremos el nombre, descripción (opcional) y seleccionaremos la metodología que utilizaremos en nuestro nuevo proyecto para la gestión de tareas. Podemos dejar los valores por defecto para el ámbito de este laboratorio.  
   ![screen](https://user-images.githubusercontent.com/4158659/93192436-f9902900-f745-11ea-9e57-47fbfa68801b.png)

En nuestro nuevo proyecto encontraremos varios opciones, en el menú situado en el lado izquierdo, y una de ellas es la relacionada con los repositorios de código (Repos).  
   ![DevOps_Menu](https://user-images.githubusercontent.com/4158659/93218031-09216900-f76a-11ea-8be1-bb5e0dbd6cec.png)

Comprobaremos que se ha creado de forma automática un repositorio de código con el mismo nombre del proyecto, y desplegando en la parte superior tendremos la posibilidad de crear otros repositorios. Para este laboratorio haremos uso del creado por defecto.  
   ![screen](https://user-images.githubusercontent.com/4158659/93192491-06148180-f746-11ea-8e17-c4a668169919.png)

La primera acción que tenemos que realizar sobre nuestro proyecto de DevOps en subir el código fuente de la aplicación que actualmente tenemos en nuestra maquina de trabajo. Para ello realizaremos la siguientes acciones:  

1. Nos posicionaremos en la carpeta raiz de nuestro proyecto (netcoredevops), ejecutando 
   ```
   cd ..
   ```
   ![screen](https://user-images.githubusercontent.com/4158659/93192580-1c224200-f746-11ea-9313-49a6dff6bf6c.png)  

2. Configuraremos nuestro entorno local de git, especificando los datos del usuario que se incluiran en los cambios realizados.
   ```
   git config --global user.email "you@example.com"
   git config --global user.name "Your Name"
   git config --global credential.helper cache
   ```
   ![screen](https://user-images.githubusercontent.com/4158659/93192638-2c3a2180-f746-11ea-82f6-600b998f4068.png)

3. Inicializaremos un repositorio git local
   ```
   git init
   ```

4. Añadiremos a nuestro repositorio local todo el contenido de nuestra carpeta actual y subcarpetas
   ```
   git add .
   git commit -m "Initial Commit"
   ```

5. Configuraremos el repositorio remoto de git, apuntando al repositorio en nuestro proyecto de Azure DevOps
   - Primero copiaremos la URL de nuestro repositorio
      ![screen](https://user-images.githubusercontent.com/4158659/93192713-4116b500-f746-11ea-9b6a-9cbf44146dbd.png)  

   - Y ejecutaremos el siguiente comando para configurarlo como repositorio remoto (sustituyendo la url del repositorio por la copiada en el punto anterior).
      ```
      git branch -M master
      git remote add origin https://[UserName]@dev.azure.com/[OrganizationName]/netcore_devops/_git/netcore_devops
      ```

6. Subiremos todos los cambios al repositorio remoto
   ```
   git push -u origin master
   ```

7. Por ultimo, comprobaremos a través del navegador que los ficheros se han subido correctamente a nuestro repositorio en DevOps.
   ![screen](https://user-images.githubusercontent.com/4158659/93192756-4d9b0d80-f746-11ea-87da-70fe654c691d.png)  

### Tarea 2: Crear compilación con editor clásico
Procederemos a crear una definicion de pipeline que nos permitirá compilar nuestra aplicacion y publicar un artefacto con el contenido de la aplicación listo para ser desplegado. Para ello seguiremos los siguientes pasos

1. En nuestro proyecto de Azure DevOps accederemos al apartado de Pipelines, y pulsaremos sobre el botón "Crear Pipeline"
   ![screen](https://user-images.githubusercontent.com/4158659/93215009-2d7b4680-f766-11ea-8a98-cf3e9c025c44.png)

2. Seleccionaremos utilizar el editor clásico, en el enlace de la parte inferior.
   ![screen](https://user-images.githubusercontent.com/4158659/93215202-5ef41200-f766-11ea-8be4-fada77ca73f4.png)

3. En la configuración del repositorio de codigo a utilizar, dejaremos la configuracion por defecto, y pulsaremos en el botón "Continuar"
   ![screen](https://user-images.githubusercontent.com/4158659/93215584-e17cd180-f766-11ea-8aab-2671bae3e3ac.png)

4. Buscaremos el template con nombre "ASP.NET Core", ya que es el que mas se asemeja a nuestra aplicacion web .Net Core.
   ![screen](https://user-images.githubusercontent.com/4158659/93215763-29035d80-f767-11ea-8b11-2053b6674639.png)

5. La pipeline creada sería operativa, pero haremos algunos pequeños ajustes:
   - Modificaremos el nombre del Agent Job por uno mas descriptivo, como "Build and Publish artifact"
   - Modificaremos el nombre del artefacto publicado, en la tarea "Publish Artifact", campo "Artifact name", sustituiremos el valor "drop" por el valor "demoapp". 
   ![screen](https://user-images.githubusercontent.com/4158659/93216631-3ff67f80-f768-11ea-99fc-8ce846d94d9c.png)

6. Pulsaremos sobre el boton "Save & queue" en la parte superior, que grabará y pondrá en ejecución una instancia de nuestra definición de pipeline.
   ![screen](https://user-images.githubusercontent.com/4158659/93216684-50a6f580-f768-11ea-8585-215ad37d5ec0.png)

7. Pulsando sobre el job en ejecucion de la pipeline podremos revisar el avance y visualizar los logs generados por cada una de las tareas.
   ![screen](https://user-images.githubusercontent.com/4158659/93216688-53094f80-f768-11ea-8fa0-1aad6153836f.png)

8. Esperaremos a que finalice la ejecucion de la pipeline y comprobaremos que se ha generado un artefacto, que podremos descargarnos y visualizar su contenido.
   ![screen](https://user-images.githubusercontent.com/4158659/93217212-040fea00-f769-11ea-8f33-92ba6f2d4cfa.png)

### Tarea 3: Fichero de definición de pipeline
En esta tarea crearemos la definición de nuestra pipeline haciendo uso de un fichero en formato YAML, que podremos guardar y versionar junto con el resto del codigo fuente de nuestra aplicación, en el repositorio Git.

Cada una de las tareas y acciones que se pueden realizar a través del editor clásico tienen una codificación en YAML. En los siguientes enlaces se encuentra toda la documentación relativa a:
- Referencia del [esquema YAML](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema) para Azure pipelines.
- Referencia en YAML de las [tareas de compilacion y despliegue](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/?view=azure-devops) incluidas por defecto en Azure DevOps.

En esta parte del ejercicio crearemos una pipeline a partir de un fichero de definicion en YAML.
Para ello simplemente tenemos que añadir a nuestro repositorio de código un fichero denominado `azure-pipelines.yml`, y se nos habilitará un asistente que nos permitirá crear automáticamente una pipeline basada en ese archivo de definición.
La principal ventaja de crear las pipelines de compilación y despliegue a partir de un fichero de definición es que estas quedan versionadas junto con el resto del código de nuestra aplicación, y de esta forma permiten que estén alineadas con la versión de nuestra aplicación, ya que según vaya evolucionando es posible que requiera de cambios en la pipeline.

En nuestro editor de código local, crearemos el fichero `azure-pipelines.yml`, y como contenido incluiremos lo siguiente, que se corresponde con las mismas tareas que la pipeline creada en el paso anterior utilizando el asistente.
```
name: $(Date:yyyyMMdd)$(Rev:.r)

# Build and Test
stages:
  - stage: BuildPublish
    displayName: Build and Publish Artifact
    variables:
      BuildConfiguration: Release
      RestoreBuildProjects: "**/*.csproj"
      TestProjects: "**/*[Tt]ests/*.csproj"
    jobs:
    - job: Build
      displayName: Build and Publish artifact
      pool:
        vmImage: 'ubuntu-16.04'
      steps:
      - task: DotNetCoreCLI@2
        displayName: Restore
        inputs:
          command: restore
          projects: '$(RestoreBuildProjects)'
      - task: DotNetCoreCLI@2
        displayName: Build
        inputs:
          projects: '$(RestoreBuildProjects)'
          arguments: '--configuration $(BuildConfiguration)'
      - task: DotNetCoreCLI@2
        displayName: Test
        inputs:
          command: test
          projects: '$(TestProjects)'
          arguments: '--configuration $(BuildConfiguration)'
      - task: DotNetCoreCLI@2
        displayName: Publish
        inputs:
          command: publish
          publishWebProjects: True
          arguments: '--configuration $(BuildConfiguration) --output $(build.artifactstagingdirectory)'
          zipAfterPublish: True
      - task: PublishBuildArtifacts@1
        displayName: 'Publish Artifact'
        inputs:
          PathtoPublish: '$(build.artifactstagingdirectory)'
          ArtifactName: demoapp
        condition: succeededOrFailed()
```

Grabaremos nuestro fichero de definición y haremos commit y push al repositorio de código remoto.
![git commit and push](https://user-images.githubusercontent.com/4158659/93790197-0d9ac580-fc33-11ea-8e96-2c53892bd76e.png)

Desde la interfaz web, al visualizar nuestro repositorio, tendremos habilitado el botón "Set up build",
![Set up build](https://user-images.githubusercontent.com/4158659/93790206-1095b600-fc33-11ea-9289-452aac30e2f0.png)


Al pulsarlo se nos creará automáticamente una nueva pipeline, lista para ser ejecutada
![Run pipeline](https://user-images.githubusercontent.com/4158659/93790542-74b87a00-fc33-11ea-9a37-ee86bfffeb62.png)

Podremos visualizar el proceso de ejecución del proceso de compilación, con los mismos pasos y resultado que en el caso anterior.


### Tarea 4: Probar integración continua
El proceso de integracioó continua consiste en que los procesos de compilación y pruebas unitarias se ejecuten tras cada nuevo commit de cambios sobre la rama de código, para permitir comprobar que los nuevos cambios realizados se integran con el resto del código existente sin provocar errores, y que se mantiene la retro-compatibilidad con las versiones anteriores, al ejecutarse las tests sin errores.

Para activarla en nuestra pipeline unicamente tendremos que incluir un pequeño fragmento en su archivo de definición, para habilitar un `disparador` que esté atento ante cualquier cambio en el repositorio de código, en las ramas que indiquemos, pudiendo utilizar caracteres comodín. 

El fragmento a incluir es el siguiente:
```
trigger:
  branches:
    include:
    - master
    - develop/*   
  paths:
    exclude:
    - tests/*
    - README.md
```

Y nuestro archivo de definición debería quedar de la sigiente forma:
![pipeline trigger](https://user-images.githubusercontent.com/4158659/93791695-d75e4580-fc34-11ea-9540-0b2f3efbb2cb.png)

Tras guardar y subir los cambios al repositorio, podremos comprobar que la pipeline se lanza de forma automática tras el cambio realizado.
![pipeline run auto](https://user-images.githubusercontent.com/4158659/93792129-666b5d80-fc35-11ea-8a0f-3c2b211cd75b.png)

# Ejercicio 3: Despliegue Continuo

**Duracion**: XX minutes

El propósito de este ejercicio es ampliar la definición del proceso de compilación para que de forma automática se despliegue en un entorno de pruebas, de forma automática, la versión de la aplicación que se genera tras el proceso de compilación.

### Tarea 1: Desplegar infraestructura en Azure

### Tarea 2: Despliegue Continuo desde Azure DevOps

### Tarea 3: Habilitar Operación

### Tarea 4: Habilitar Seguridad