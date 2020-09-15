# .NET Core - DevOps Lab
En este laboratorio sobre DevOps se prueban los procesos de Integración Contínua (CI) y Despliegue Continuo (CD) haciendo uso de las funcionalidades ofrecidas por la herramienta Azure DevOps.

Se utilizará como base el código fuente de la aplicación de ejemplo creada por Ben Coleman y disponible en el [este repositorio GutHub](https://github.com/benc-uk/dotnet-demoapp).  
Esta aplicación permite monitorizar en tiempo real los recursos de CPU y memoria utilizados por la aplicación, así como forzar el uso de recursos y lanzar excepciones que serán recogidas por aplicaciones de monitorización, como App Insights.


## Requisitos
1. Una suscripción de Azure, de pago por uso, MSDN o Trial.
   - Para completar este laboratorio, asegúrate de que tu cuenta tenga los siguientes roles:
     - El rol [Owner](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles#owner) para la suscripción que vayas a utilizar.
     - Eres un usuario [Miembro](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/users-default-permissions#member-and-guest-users) del directorio Azure AD que vayas a utilizar. (Los usuarios invitados no tendrán los permisos necesarios).
     - No tienes restringida la creación de Aplicaciones Azure AD. Es posible que el administrador del Directorio Azure AD lo haya limitado.

     > **Nota** Si no cumples estos requisitos, es posible que tengas que pedir a otro usuario miembro con derechos de propietario de suscripción que inicie sesión en el portal y ejecute con antelación el paso de creacion de la aplicación Azure AD.
  
2. Máquina local o una máquina virtual (Windows o Linux) configurada con:
   -  Un navegador, preferiblemente Chrome o Microsoft Edge.
   - [dotnet core sdk](https://dotnet.microsoft.com/download) instalado, para las pruebas de compilación y ejecucion locales
   - [Visual Studio Code](https://code.visualstudio.com/download) o cualquier otro IDE o editor.

# Ejercicio 1: Ejecutar localmente
Como primer ejercicio probaremos a ejecutar la aplicación de forma local, para comprobar su correcto funcionamiento y familiarizarnos con la la funcionalidad que nos ofrece.

Para ello nos descargaremos el codigo fuente desde el repositorio de referencia de este laboratorio, ejecutando el siguiente comando.


Eliminaremos las referencias al repositorio Git desde el que hemos descargado el contenido.


> **Nota** En entornos linux el comando a ejecutar sería
```

```


```
cd src
dotnet restore
dotnet run
```

La aplicación web escuchará en los puertos 5000 (http) y 5001 (https), habituales de Kestrel, pero esto puede cambiarse configurando la variable de entorno `ASPNETCORE_URLS` o con el parámetro `--urls` ([ver docs](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-3.1)).  

Probaremos la aplicacion web en el navegador, accediendo a alguna de las siguientes URLs:
```
http://localhost:5000
https://localhost:5001
```

# Ejercicio 2: Integración Continua con Azure DevOps

## Crear organizacion en Azure DevOps 

## Subir código a repositorio Git

## Editor clásico de pipelines

## Fichero de definición de pipeline

## Probar integración continua

# Ejercicio 3: Desplegar infraestructura en Azure

# Ejercicio 4: Despliegue Continuo desde Azure DevOps

# Ejercicio 5: Habilitar Operación

# Ejercicio 6: Habilitar Seguridad