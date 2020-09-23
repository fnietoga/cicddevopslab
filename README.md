# CI/CD con Azure DevOps Lab
La finalidad de este laboratorio sobre DevOps es la practica sobre los procesos de Integración Contínua (CI) y Despliegue Continuo (CD) utilizando las funcionalidades ofrecidas por la herramienta Azure DevOps.

Se utilizará como base para las prácticas el código fuente de la aplicación de ejemplo creada por Ben Coleman y disponible en el [este repositorio GutHub](https://github.com/benc-uk/dotnet-demoapp). 

Esta aplicación web de ejemplo permite monitorizar en tiempo real los recursos de CPU y memoria utilizados por la aplicación, así como forzar el uso de recursos y lanzar excepciones que serán recogidas por aplicaciones de monitorización, como App Insights.

## Requisitos
1. Una suscripción de Azure, de pago por uso, MSDN o Trial.
   - Para completar este laboratorio, asegúrate de que tu cuenta tenga los siguientes roles y permisos:
     - El rol [Owner](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#owner) para la suscripción que vayas a utilizar.
     - Ser un usuario [Miembro](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/users-default-permissions#member-and-guest-users) del directorio Azure AD que vayas a utilizar. (Los usuarios invitados no tendrán los permisos necesarios).
     - No tener restringida la creación de Aplicaciones Azure AD. Es posible que el administrador del Directorio Azure AD lo haya limitado.  

     > **Nota** Si no cumples estos requisitos, es posible que tengas que pedir a otro usuario miembro con derechos de propietario de suscripción que inicie sesión en el portal y ejecute con antelación el paso de creacion de la aplicación Azure AD.
  
2. Máquina local o una máquina virtual (Windows o Linux) configurada con:
   - Un navegador, preferiblemente Chrome o Microsoft Edge.
   - [dotnet core sdk](https://dotnet.microsoft.com/download) instalado, para las pruebas de compilación y ejecucion locales.
   - [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest) para la ejecución de algunos comandos en Azure. 
   - [Visual Studio Code](https://code.visualstudio.com/download) o cualquier otro IDE o editor.

# Ejercicio 1: Ejecutar localmente

**Duracion**: 10 minutes

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

**Duracion**: 20 minutes

El propósito de este ejercicio es familiarizarnos con las principales opciones de Azure DevOps y preparar el entorno para qie se realice un proceso de compilacion de forma automática cada vez que se sube un cambio en el codigo fuente por algun desarrollador.

### Tarea 1: Crear proyecto y subir código fuente

Para comenzar abre una nueva pestaña del navegador para visitar [Azure DevOps](https://dev.azure.com), y a continuación inicia sesión en tu cuenta.  

Si es la primera vez que inicias sesion con esa cuenta, Azure DevOps te guiará a través de un asistente:
- Confirma tu informacion de contacto y selecciona "Siguiente"
- Selecciona "Continuar" para crear una nueva cuenta en Azure DevOps
- Introduce el nombre que deseas para tu nueva Organización y selecciona "Continuar"

También puedes crear una nueva organizacion utilizando el enlace "Nueva Organizacion", en la parte superior izquierda, o trabajar sobre una de tus organizaciones ya existentes.  
   <kbd>![screen](https://user-images.githubusercontent.com/4158659/93192213-b6ce5100-f745-11ea-9604-bea1f89d7cbd.png)</kbd>

Si hemos creado una nueva organización nos mostrará automáticamente las opciones para la creación del primero proyecto.  
   <kbd>![screen](https://user-images.githubusercontent.com/4158659/93192382-eaa97680-f745-11ea-92f1-20efffb2b07a.png)</kbd>

Si hemos seleccionado una organización ya existente, tendremos que hacer uso del botón "Nuevo Proyecto" que se nos muestra en la parte superior derecha.  

Especificaremos el nombre, descripción (opcional) y seleccionaremos la metodología que utilizaremos en nuestro nuevo proyecto para la gestión de tareas. Podemos dejar los valores por defecto para el ámbito de este laboratorio.  
   <kbd>![screen](https://user-images.githubusercontent.com/4158659/93192436-f9902900-f745-11ea-9e57-47fbfa68801b.png)</kbd>

En nuestro nuevo proyecto encontraremos varios opciones, en el menú situado en el lado izquierdo, y una de ellas es la relacionada con los repositorios de código (Repos).  
   <kbd>![DevOps_Menu](https://user-images.githubusercontent.com/4158659/93218031-09216900-f76a-11ea-8be1-bb5e0dbd6cec.png)</kbd>

Comprobaremos que se ha creado de forma automática un repositorio de código con el mismo nombre del proyecto, y desplegando en la parte superior tendremos la posibilidad de crear otros repositorios. Para este laboratorio haremos uso del creado por defecto.  
   <kbd>![screen](https://user-images.githubusercontent.com/4158659/93192491-06148180-f746-11ea-8e17-c4a668169919.png)</kbd>

La primera acción que tenemos que realizar sobre nuestro proyecto de DevOps en subir el código fuente de la aplicación que actualmente tenemos en nuestra maquina de trabajo. Para ello realizaremos la siguientes acciones:  

1. Nos posicionaremos en la carpeta raiz de nuestro proyecto (netcoredevops), ejecutando 
   ```
   cd ..
   ```
   <kbd>![screen](https://user-images.githubusercontent.com/4158659/93192580-1c224200-f746-11ea-9313-49a6dff6bf6c.png)</kbd>

2. Configuraremos nuestro entorno local de git, especificando los datos del usuario que se incluiran en los cambios realizados.
   ```
   git config --global user.email "you@example.com"
   git config --global user.name "Your Name"
   git config --global credential.helper cache
   ```
   <kbd>![screen](https://user-images.githubusercontent.com/4158659/93192638-2c3a2180-f746-11ea-82f6-600b998f4068.png)</kbd>

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
   - Antes de continuar, y para simplificar la autenticación con el repositorio Git, pulsaremos sobre el botón `Generate Git Credential`, y copiaremos el Personal Access Token generado, almacenándolo para su uso posterior.
      <kbd>![Repository_GenerateCredentials](https://user-images.githubusercontent.com/4158659/93901904-4a2ff500-fcf7-11ea-9874-eae4e06c1423.png)</kbd>  
   - A continuación copiaremos la URL de nuestro repositorio
      <kbd>![screen](https://user-images.githubusercontent.com/4158659/93192713-4116b500-f746-11ea-9b6a-9cbf44146dbd.png)</kbd>  

   - Y ejecutaremos el siguiente comando para configurarlo como repositorio remoto (sustituyendo la url del repositorio por la copiada en el punto anterior).
      ```
      git branch -M master
      git remote add origin https://[UserName]@dev.azure.com/[OrganizationName]/netcore_devops/_git/netcore_devops
      ```

6. Subiremos todos los cambios al repositorio remoto
   ```
   git push -u origin master
   ```
   > **Nota** La primera vez que subamos cambios al repositorio remoto nos solicitará credenciales, y podremos utilizar como password el Personal Access Token generado anteriormente.

7. Por ultimo, comprobaremos a través del navegador que los ficheros se han subido correctamente a nuestro repositorio en DevOps.
   <kbd>![screen](https://user-images.githubusercontent.com/4158659/93192756-4d9b0d80-f746-11ea-87da-70fe654c691d.png)</kbd>  

### Tarea 2: Crear compilación con editor clásico
Procederemos a crear una definicion de pipeline que nos permitirá compilar nuestra aplicacion y publicar un artefacto con el contenido de la aplicación listo para ser desplegado. Para ello seguiremos los siguientes pasos

1. En nuestro proyecto de Azure DevOps accederemos al apartado de Pipelines, y pulsaremos sobre el botón "Crear Pipeline"
   <kbd>![screen](https://user-images.githubusercontent.com/4158659/93215009-2d7b4680-f766-11ea-8a98-cf3e9c025c44.png)</kbd>

2. Seleccionaremos utilizar el editor clásico, en el enlace de la parte inferior.
   <kbd>![screen](https://user-images.githubusercontent.com/4158659/93215202-5ef41200-f766-11ea-8be4-fada77ca73f4.png)</kbd>

3. En la configuración del repositorio de codigo a utilizar, dejaremos la configuracion por defecto, y pulsaremos en el botón "Continuar"
   <kbd>![screen](https://user-images.githubusercontent.com/4158659/93215584-e17cd180-f766-11ea-8aab-2671bae3e3ac.png)</kbd>

4. Buscaremos el template con nombre "ASP.NET Core", ya que es el que mas se asemeja a nuestra aplicacion web .Net Core.
   <kbd>![screen](https://user-images.githubusercontent.com/4158659/93215763-29035d80-f767-11ea-8b11-2053b6674639.png)</kbd>

5. La pipeline creada sería operativa, pero haremos algunos pequeños ajustes:
   - Modificaremos el nombre del Agent Job por uno mas descriptivo, como "Build and Publish artifact"
   - Modificaremos el nombre del artefacto publicado, en la tarea "Publish Artifact", campo "Artifact name", sustituiremos el valor "drop" por el valor "demoapp". 
   <kbd>![screen](https://user-images.githubusercontent.com/4158659/93216631-3ff67f80-f768-11ea-99fc-8ce846d94d9c.png)</kbd>

6. Pulsaremos sobre el boton "Save & queue" en la parte superior, que grabará y pondrá en ejecución una instancia de nuestra definición de pipeline.
   <kbd>![screen](https://user-images.githubusercontent.com/4158659/93216684-50a6f580-f768-11ea-8585-215ad37d5ec0.png)</kbd>

7. Pulsando sobre el job en ejecucion de la pipeline podremos revisar el avance y visualizar los logs generados por cada una de las tareas.
   <kbd>![screen](https://user-images.githubusercontent.com/4158659/93216688-53094f80-f768-11ea-8fa0-1aad6153836f.png)</kbd>

8. Esperaremos a que finalice la ejecucion de la pipeline y comprobaremos que se ha generado un artefacto, que podremos descargarnos y visualizar su contenido.
   <kbd>![screen](https://user-images.githubusercontent.com/4158659/93217212-040fea00-f769-11ea-8f33-92ba6f2d4cfa.png)</kbd>

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
<kbd>![git commit and push](https://user-images.githubusercontent.com/4158659/93790197-0d9ac580-fc33-11ea-8e96-2c53892bd76e.png)</kbd>

Desde la interfaz web, al visualizar nuestro repositorio, visualizaremos el nuevo fichero subido, y tendremos habilitado el botón "Set up build",
<kbd>![Set up build](https://user-images.githubusercontent.com/4158659/93790206-1095b600-fc33-11ea-9289-452aac30e2f0.png)</kbd>


Al pulsarlo se nos creará automáticamente una nueva pipeline, lista para ser ejecutada
<kbd>![Run pipeline](https://user-images.githubusercontent.com/4158659/93790542-74b87a00-fc33-11ea-9a37-ee86bfffeb62.png)</kbd>

Podremos visualizar el proceso de ejecución del proceso de compilación, con los mismos pasos y resultado que en el caso anterior.


### Tarea 4: Probar integración continua
El proceso de integración continua consiste en que las tareas de compilación y pruebas unitarias se ejecuten tras cada nuevo commit de cambios sobre la rama de código, para permitir comprobar que los nuevos cambios realizados se integran con el resto del código existente sin provocar errores, y que se mantiene la retro-compatibilidad con las versiones anteriores, al ejecutarse las tests sin errores.

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
<kbd>![pipeline trigger](https://user-images.githubusercontent.com/4158659/93791695-d75e4580-fc34-11ea-9540-0b2f3efbb2cb.png)</kbd>

Tras guardar y subir los cambios al repositorio, podremos comprobar que la pipeline se lanza de forma automática tras el cambio realizado.
<kbd>![pipeline run auto](https://user-images.githubusercontent.com/4158659/93792129-666b5d80-fc35-11ea-8a0f-3c2b211cd75b.png)</kbd>

# Ejercicio 3: Despliegue Continuo

**Duracion**: 30 minutes

El propósito de este ejercicio es ampliar la definición del proceso de compilación para que de forma automática se despliegue en un entorno de pruebas, de forma automática, la versión de la aplicación que se genera tras el proceso de compilación haciendo uso de infraestructura cloud para su ejecución.

### Tarea 1: Desplegar infraestructura en Azure
El primer paso será crear un fichero de definición para la infraestructura necesaria. Para poder ejecutar nuestra aplicación de ejemplo haremos uso del servicio PaaS `Azure App Service`, que permite la ejecución de aplicaciones de distintos tipos y en varios lenguajes de programación sin requerir desplegar ninguna infraestructura de servidores, es la arquitectura conocida como `serverless`.

Utilizaremos un fichero de definición ARM Template de Azure, y en el mismo incluiremos el despliegue de los siguientes servicios:
- [Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/). Alojará y ejecutará nuestra aplicación web.
- [Application Insigths](https://docs.microsoft.com/en-us/azure/azure-monitor/azure-monitor-app-hub). Lo utilizaremos mas adelante para monitorizar la actividad de nuestra aplicacion web.

En nuestro proyecto local crearemos una carpeta `iac` para alojar los ficheros necesarios para el despliegue de infraestructura, y en su interior crearemos un fichero con el nombre `azuredeploy.json`. El contenido del fichero para desplegar la infraestructura mencionada es el siguiente

```
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
    },
    "variables": {
        "hostingPlanName": "netcoredevopsplan",
        "appName": "netcoredevopsapp",
        "insightsName": "netcoredevopsinsights",
        "location": "[resourceGroup().location]"
    },
    "resources": [
        {
            "apiVersion": "2020-06-01",
            "name": "[variables('appName')]",
            "type": "Microsoft.Web/sites",
            "location": "[variables('location')]",
            "tags": {},
            "dependsOn": [
                "[concat('microsoft.insights/components/',variables('insightsName'))]",
                "[concat('Microsoft.Web/serverfarms/', variables('hostingPlanName'))]"
            ],
            "properties": {
                "name": "[variables('appName')]",
                "siteConfig": {
                    "appSettings": [                                            
                    ],
                    "metadata": [
                        {
                            "name": "CURRENT_STACK",
                            "value": "dotnetcore"
                        }
                    ],
                    "phpVersion": "OFF",
                    "alwaysOn": false
                },
                "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]",
                "clientAffinityEnabled": true
            }
        },
        {
            "apiVersion": "2020-06-01",
            "name": "[variables('hostingPlanName')]",
            "type": "Microsoft.Web/serverfarms",
            "location": "[variables('location')]",
            "properties": {
                "targetWorkerCount": "1"
            },
            "sku": {
                "Tier": "Shared",
                "Name": "D1"
            }
        },
        {
            "apiVersion": "2020-02-02-preview",
            "name": "[variables('insightsName')]",
            "type": "microsoft.insights/components",
            "location": "[variables('location')]",
            "tags": {},
            "properties": {
                "Application_Type": "web",
                "Request_Source": "IbizaWebAppExtensionCreate"
            }
        }
    ]
}
```

El template ha sido creado sin parámetros de entrada, pero si se precisara que el usuario pudiera especificar algun valor de configuración sobre los recursos desplegados, como el nombre de los mismos, simplmemente tendriamos que incorporarlos en el apartado `parameters` y posteriormente, durante el despliegue, se podrá especificar un valor. [Mas información sobre parámetros](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-parameters)


### Tarea 2: Despliegue Continuo desde Azure DevOps
En esta tarea incluiremos el despliegue de la infraestructura Azure y la publicación del contenido de nuestro sitio web en la misma pipeline de despliegue que creamos anteriormente.

Necesitamos crear una conexión en nuestro proyecto de Azure DevOps que nos permita interactuar con nuestra suscripción de Azure.  
Para simplificar el proceso haremos uso de un asistente que incluye el editor clásico que nos permite seleccionar una de nuestras suscripciones y crear de forma automática todos los recursos necesarios.  
Editaremos la pipeline que creamos en primera instancia, utilizando el editor clásico, y que quedó guardada con el nombre `netcore_devops-ASP.NET Core-CI` (o similar).  
Añadiremos una nueva tarea, buscando la tarea con nombre `ARM template deployment` y la añadiremos a la pipeline.
<kbd>![ARM Template_Task](https://user-images.githubusercontent.com/4158659/93871470-fc9f9200-fcce-11ea-9862-ad7892893d4b.png)</kbd>

En la nueva tarea, en el parámetro `Azure Resource Manager connection`, seleccionaremos en la lista la suscripción que utilizaremos para realizar el despliegue, y pulsaremos sobre el botón "Authorize".  
<kbd>![ARM Connection authorize](https://user-images.githubusercontent.com/4158659/93871865-9c5d2000-fccf-11ea-9774-2f713c92b31c.png)</kbd>

El asistente nos pedirá las credenciales que se utilizarán para el proceso de creación y configuración de los elementos Azure y Azure AD necesarios.
Tras finalizar el asistente de autorización, podremos descartar los cambios sobre la pipeline.

Este proceso generará de forma automática los siguientes elementos:
- [Azure Service Principal](https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals). En el Azure Active Directory asociado a la suscripción seleccionada se creará un Service Principal al que se le asignarán el rol [Contributor](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#contributor) sobre la suscripción para permitir el despliegue de recursos. 
- [Azure DevOps Service Connection](https://docs.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints?view=azure-devops&tabs=yaml). En el proyecto actual de Azure DevOps se creará una conexión y quedará configurada con los valores de la suscripción, y utilizando el Service Principal creado para la autenticación.

Haremos unos pequeños ajustes a la conexión creada, para utilizar un nombre mas facil de recordar, y para permitir su uso por las distintas pipelines que creemos sin necesidad de autorizar expresamente.  
En el apartado `Project Settings` de nuestro proyecto, accederemos al elemento `Service connections` y editaremos la conexión.
<kbd>![Service Connection Edit](https://user-images.githubusercontent.com/4158659/93872261-2e652880-fcd0-11ea-83ed-b0c1c51e9e1c.png)</kbd>

Le cambiaremos el nombre por `azurerm` y marcaremos el check `Grant access permission to all pipelines`, guardando los cambios.  
<kbd>![Service Connection Grant](https://user-images.githubusercontent.com/4158659/93872460-7f751c80-fcd0-11ea-9c63-7c5ef428fe98.png)</kbd>

Editaremos el fichero `azure-pipelines.yml` para incluir al final un nuevo elemento `stage` con un elemento `deployment` que incluye varias tareas adicionales.
```
# Deploy and Publish
  - stage: DeployPublish
    displayName: Deploy Infra and Publish app    
    jobs:
    - deployment: DeployInfraAndPublishApp
      displayName: 'Deploy infra to Azure and Publish App'
      workspace:
        clean: all
      environment: demoapp-dev
      variables:
        resourceGroupName: netcoredevops
        webAppName: netcoredevopsapp
        location: 'West Europe'
        connectionName: 'azurerm'
      strategy:
        runOnce:
          deploy:
            steps:
            - checkout: self
            - task: AzureResourceManagerTemplateDeployment@3
              displayName: 'Deploy Azure infra using ARM Template deployment'
              inputs:
                azureResourceManagerConnection: '${{variables.connectionName}}'
                action: 'Create Or Update Resource Group'
                resourceGroupName: $(resourceGroupName)
                location: $(location)
                templateLocation: 'Linked artifact'
                csmFile: iac/azuredeploy.json
                deploymentMode: 'Incremental'            
            - task: AzureWebApp@1
              displayName: 'Azure Web App Deploy: $(webAppName)'
              inputs:
                azureSubscription: '${{variables.connectionName}}'
                appType: webApp
                appName: $(webAppName)
                package: $(Pipeline.Workspace)/**/*.zip
                deploymentMethod: auto
```
   > **ATENCION** a la tabulación del nuevo contenido, el nuevo elemento `stage` debe de quedar alineado verticalmente con el existente, y el resto de elementos con la identación mostrada.
 
Prepararemos un commit & push de nuestros cambios, para subirlos al repositorio remoto, y observaremos como nuestra pipeline comienza a ejecutarse de forma automática. Esperaremos a que finalice su ejecución.  

Al finalizar observaremos en el [Portal de Azure](https://portal.azure.com/) que nuestros recursos han quedado perfectamente desplegados.
<kbd>![Portal Azure](https://user-images.githubusercontent.com/4158659/93872913-21950480-fcd1-11ea-965b-757313f8dc6c.png)</kbd>

Y si accedemos a la URL del App Service nuestra aplicación se mostrará correctamente.
<kbd>![App Service Url](https://user-images.githubusercontent.com/4158659/93873019-4f7a4900-fcd1-11ea-9e7d-6b531653e991.png)</kbd>

Con estos cambios, cada vez que se realice un cambio por parte del equipo de desarrollo sobre el codigo de la aplicación, se compilará y desplegará de forma automática la nueva version sobre el entorno de desarrollo, lo que se denomina "Despliegue Continuo"


Al haber hecho uso del elemento `deployment` en la definición de la pipeline, se habrá creado de forma automática un elemento en la seccion `Environments` de Azure DevOps, donde podremos visualizar el historia de despliegues sobre nuestra infraestructura de desarrollo, contando con la trazabilidad de las distintas versiones desplegadas.
<kbd>![Environments](https://user-images.githubusercontent.com/4158659/93873441-ff4fb680-fcd1-11ea-88c4-d1652dd65722.png)</kbd>

### Tarea 3: Habilitar Operación
En esta última tarea habilitaremos la integración entre el App Service y el [App Insights](https://docs.microsoft.com/en-us/azure/azure-monitor/azure-monitor-app-hub), para que todas las operaciones, indicadores y excepciones que se produzcan durante la ejecución de la aplicación sean recogidas y almacenados por el servicio de Monitorización de Azure.

Para ello el servicio App Service necesita conocer el Instrumentation Key asociado al servicio App Insights, y con ello de forma automática se trazarán todos los valores obtenidos desde los contadores de rendimiento estandard del servidos y del propio servicio web, así como la informacion de las excepciones que se produzcan.
También permite la gestion de eventos y métricas personalizados, si la aplicación está codificada para ello.

El valor del Instrumentation Key necesario se puede configurar en el servicio App Service a través de dos App Settings, que pueden especificarse en tiempo de despliegue de la insfraestructura. Los valores necesarios de configurar se pueden extraer del servicio Insigths desplegado en el mismo ARM template. Incluiremos el siguiente fragmento en el fichero `azuredeploy.json`.
El fragmento a añadir es el siguiente:  
```
   {
         "name": "APPINSIGHTS_INSTRUMENTATIONKEY",
         "value": "[reference(concat('microsoft.insights/components/',variables('insightsName')), '2020-02-02-preview').InstrumentationKey]"
   },
   {
         "name": "APPLICATIONINSIGHTS_CONNECTION_STRING",
         "value": "[reference(concat('microsoft.insights/components/',variables('insightsName')), '2020-02-02-preview').ConnectionString]"
   },
   {
         "name": "ApplicationInsightsAgent_EXTENSION_VERSION",
         "value": "~2"
   }         
```


Y en la siguiente imagen se muestra donde debe ser insertado.
<kbd>![Template_InstrumentationKey](https://user-images.githubusercontent.com/4158659/93896318-12be4a00-fcf1-11ea-878a-866191e3acf8.png)
</kbd>

Tras guardar y subir los cambios al repositorio, la pipeline de despliegue se lanzará de forma automática.  
Esperaremos a que termine y comprobaremos en el Portal de Azure que se han aplicado los cambios correctamente, visualizando las propiedades del App Service, en el apartado `Application Insights`, podremos visualizar asociado el recurso con nombre `netcoredevopsinsights`, como se muestra en la imagen.
<kbd>![Portal Insigths configured](https://user-images.githubusercontent.com/4158659/93893538-071d5400-fcee-11ea-8adf-0a4a4b7a66c4.png)</kbd>

Accederemos a la URL de la aplicación web y generaremos eventos haciendo uso de TODAS las herramientas que facilita la aplicación de ejemplo utilizada.  
A través del portal de Azure visualizaremos los valores mostrados en la App Insigths desplegada con nombre `netcoredevopsinsights` en el cuadro de mando por defecto.
<kbd>![Portal Insigths activity](https://user-images.githubusercontent.com/4158659/93894614-34b6cd00-fcef-11ea-9efe-25d639be27f3.png)
</kbd>

Algunas opciones de interes del servicio son:
- [Live Metrics](https://docs.microsoft.com/en-us/azure/azure-monitor/app/live-stream): Permite monitorizar en tiempo real las métricas y contadores de rendimiento de la aplicación.
- [Failures](https://docs.microsoft.com/en-us/azure/azure-monitor/app/proactive-failure-diagnostics): Permite analizar los errores y excepciones ocurridos en la aplicación, según su tipología y la página que los ha originado.
- [Search](https://docs.microsoft.com/en-us/azure/azure-monitor/app/diagnostic-search): Permite buscar y explorar elementos de telemetría individuales, como visitas de páginas, excepciones o solicitudes en la web.
- [Metrics](https://docs.microsoft.com/en-us/azure/azure-monitor/app/pre-aggregated-metrics-log-metrics): Permite la monitorización basada en métricas pre-agregadas y obtenidas desde logs.
- [Alerts](https://docs.microsoft.com/en-us/azure/azure-monitor/platform/alerts-metric): Permite especificar alertas basadas en umbrales sobre valores obtenidos en las métricas y búsquedas predefinidas sobre los logs

# Ultimo Ejercicio: Limpieza
Es recomendable que elimines los recursos Azure generados para evitar generar costes innecesarios.
Para ello será suficiente con borrar el grupo de recursos con nombre `netcoredevops`.

# Ayúdanos a mejorar
Con la única finalidad de conocer tu opinión sobre este laboratorio y poder generar mas contenidos que se adapten a vuestras necesidades de formación e inquietudes tecnológicas, te pedimos que rellenes esta [breve encuesta anónima](https://forms.office.com/Pages/ResponsePage.aspx?id=KwcNNUnY902Tv9VZNVaFHd8wREGDZNxHtUeYg-i5ZI1UNlgzU1FKSDU3RlNPNjJPSlpYOEU4QTc2My4u). 
