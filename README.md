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
Abre una nueva pestaña del navegador para visitar [Azure DevOps](https://dev.azure.com), y a continuación inicia sesión en tu cuenta.  

Si es la primera vez que inicias sesion con esa cuenta, Azure DevOps te guiará a través de un asistente:
- Confirma tu informacion de contacto y selecciona "Siguiente"
- Selecciona "Continuar" para crear una nueva cuenta en Azure DevOps
- Introduce el nombre que deseas para tu nueva Organización y selecciona "Continuar"

De lo contrario, puedes crear una nueva organizacion utilizando el enlace "Nueva Organizacion", o trabajar sobre una de tus organizaciones ya existentes.  
![screen]()

Si hemos creado una nueva organización nos mostrará automáticamente las opciones para la creación del primero proyecto.  
![screen]()

Si hemos seleccionado una organización ya existente, tendremos que hacer uso del botón "Nuevo Proyecto" que se nos muestra en la parte superior derecha.  

Especificaremos el nombre, descripción (opcional) y seleccionaremos la metodología que utilizaremos en nuestro nuevo proyecto para la gestión de tareas. Podemos dejar los valores por defecto para el ámbito de este laboratorio.  
![screen]()

En nuestro nuevo proyecto encontraremos varios opciones, en el menú situado en el lado izquierdo, y una de ellas es la relacionada con los repositorios de código (Repos).  
Comprobaremos que se ha creado de forma automática un repositorio de código con el mismo nombre del proyecto, y desplegando en la parte superior tendremos la posibilidad de crear otros repositorios. Para este laboratorio haremos uso del creado por defecto.  
![screen]()

La primera acción que tenemos que realizar sobre nuestro proyecto de DevOps en subir el código fuente de la aplicación que actualmente tenemos en nuestra maquina de trabajo. Para ello realizaremos la siguientes acciones:  

1. Nos posicionaremos en la carpeta raiz de nuestro proyecto (netcoredevops), ejecutando 
```
cd ..
```
![screen]()  

2. Configuraremos nuestro entorno local de git
```
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
git config --global credential.helper cache
```

3. Inicializaremos un repositorio git local
```
git init
```

4. Añadiremos a nuestro repositorio local todo el contenido de nuestra carpeta local
```
git add .
git commit -m "Initial Commit"
```

5. Configuraremos el repositorio remoto de git, apuntando al repositorio en nuestro proyecto de Azure DevOps
Primero copiaremos la URL de nuestro repositorio
![screen]()  

Y ejecutaremos el siguiente comando para configurarlo como repositorio remoto.
```
git branch -M master
git remote add origin https://[UserName]@dev.azure.com/[OrganizationName]/netcore_devops/_git/netcore_devops
```

6. Subiremos todos los cambios al repositorio remoto
```
git push -u origin master
```

7. Por ultimo, comprobaremos a través del navegador que los ficheros se han subido correctamente a nuestro repositorio en DevOps.
![screen]()  

## Editor clásico de pipelines

## Fichero de definición de pipeline

## Probar integración continua

# Ejercicio 3: Desplegar infraestructura en Azure

# Ejercicio 4: Despliegue Continuo desde Azure DevOps

# Ejercicio 5: Habilitar Operación

# Ejercicio 6: Habilitar Seguridad